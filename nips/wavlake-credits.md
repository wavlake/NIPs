# NIP-XX: Wavlake Streaming Credits (WLC)

`draft` `optional`

This document specifies how Wavlake Streaming Credits integrate with NIP-60 (Cashu Wallets) and NIP-61 (Nutzaps) for content payments.

## Abstract

Wavlake Streaming Credits (WLC) are non-redeemable Cashu tokens used for streaming content, tipping artists, and purchasing content access. This NIP extends NIP-60/61 with a platform-specific credit unit.

## Motivation

1. **Micropayments**: Enable sub-cent payments for individual streams
2. **Instant Settlement**: No Lightning channel management
3. **Offline Capability**: Tokens work without internet
4. **Nostr-Native**: Wallet state stored on relays via NIP-60

## Credit Properties

| Property | Value |
|----------|-------|
| Unit | `wlc` (Wavlake Credits) |
| Redeemable for sats | **No** |
| Transferable | Yes (via Nostr) |
| Mint | Platform-specific |
| Value | 1 WLC = 1 USD cent (fixed) |

## Protocol Overview

WLC uses standard NIP-60/61 event kinds with a custom unit:

| Kind | Name | Purpose |
|------|------|---------|
| 10019 | Nutzap Info | Publish receiving preferences |
| 17375 | Wallet Info | Store encrypted wallet keys |
| 7375 | Token Event | Store unspent proofs |
| 7376 | Spending History | Track spent tokens |
| 9321 | Nutzap | Send tokens to users |

## Kind 10019: Nutzap Info

Tells other users how to send WLC:

```json
{
  "kind": 10019,
  "pubkey": "<user-nostr-pubkey>",
  "content": "",
  "tags": [
    ["relay", "wss://relay.wavlake.com"],
    ["mint", "https://mint.wavlake.com", "wlc"],
    ["pubkey", "<p2pk-pubkey-hex>"]
  ]
}
```

### Tags

| Tag | Description |
|-----|-------------|
| `relay` | Relays for nutzap delivery |
| `mint` | Accepted mint with unit(s) |
| `pubkey` | P2PK pubkey for locking proofs |

### Multi-Unit Example (Future)

```json
["mint", "https://mint.wavlake.com", "wlc"],
["mint", "https://mint.minibits.cash", "sat"]
```

## Kind 17375: Wallet Info

Stores encrypted P2PK private key:

```json
{
  "kind": 17375,
  "pubkey": "<user-nostr-pubkey>",
  "content": "<nip44-encrypted-content>",
  "tags": [
    ["d", "wavlake-credits"]
  ]
}
```

### Encrypted Content (NIP-44)

```json
[
  ["privkey", "<p2pk-privkey-hex>"],
  ["mint", "https://mint.wavlake.com"]
]
```

### Namespace Separation

The `["d", "wavlake-credits"]` tag creates a separate addressable event, allowing users to have both WLC and standard ecash wallets.

## Kind 7375: Token Events

Stores unspent proofs:

```json
{
  "kind": 7375,
  "pubkey": "<user-nostr-pubkey>",
  "content": "<nip44-encrypted-proofs>",
  "tags": [
    ["a", "17375:<pubkey>:wavlake-credits"]
  ]
}
```

### Encrypted Content

```json
{
  "mint": "https://mint.wavlake.com",
  "proofs": [
    {"amount": 4, "id": "...", "secret": "...", "C": "..."},
    {"amount": 2, "id": "...", "secret": "...", "C": "..."}
  ],
  "unit": "wlc"
}
```

**Critical**: Always include `"unit": "wlc"` - NIP-60 defaults to "sat" if missing.

## Kind 9321: Nutzaps

Send credits to other users:

```json
{
  "kind": 9321,
  "pubkey": "<sender-pubkey>",
  "content": "Thanks for the great music! 🎵",
  "tags": [
    ["p", "<recipient-pubkey>"],
    ["e", "<event-being-tipped>"],
    ["proofs", "<p2pk-locked-proofs>"],
    ["u", "https://mint.wavlake.com"],
    ["amount", "10"]
  ]
}
```

### Nutzap Flow

1. Sender fetches recipient's Kind 10019 nutzap info
2. Sender locks proofs to recipient's P2PK pubkey
3. Sender publishes Kind 9321 to specified relays
4. Recipient's wallet detects incoming nutzap
5. Recipient claims proofs with their P2PK private key

## Payment Flows

### Content Purchase

```
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│   Client    │        │   Paywall   │        │    Mint     │
└──────┬──────┘        └──────┬──────┘        └──────┬──────┘
       │                      │                      │
       │ GET /content (token) │                      │
       │─────────────────────>│                      │
       │                      │ Verify proofs        │
       │                      │─────────────────────>│
       │                      │<─────────────────────│
       │                      │                      │
       │ 200 + content URL    │                      │
       │<─────────────────────│                      │
```

### Artist Tip (Nutzap)

```
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│   Tipper    │        │   Relays    │        │   Artist    │
└──────┬──────┘        └──────┬──────┘        └──────┬──────┘
       │                      │                      │
       │ Fetch 10019 for artist                      │
       │─────────────────────────────────────────────>│
       │<─────────────────────────────────────────────│
       │                      │                      │
       │ Publish 9321 nutzap  │                      │
       │─────────────────────>│                      │
       │                      │ Deliver to artist    │
       │                      │─────────────────────>│
       │                      │                      │
       │                      │   Claim proofs       │
       │                      │<─────────────────────│
```

## Wallet Recovery

### Cross-Device Sync

1. User logs in with nsec on new device
2. Client fetches Kind 17375 with `d=wavlake-credits`
3. Client decrypts with NIP-44 using user's nsec
4. Client recovers P2PK keypair
5. Client fetches all Kind 7375 token events
6. Wallet is fully restored

### Key Derivation (Recommended)

To avoid storing keys, derive P2PK from nsec:

```javascript
// Deterministic P2PK derivation
const p2pkPrivkey = sha256(concat(nsec, "wavlake-p2pk-v1"));
const p2pkPubkey = secp256k1.getPublicKey(p2pkPrivkey);
```

This enables wallet recovery from just the nsec.

## Client Compatibility

### Interop with Other Wallets

| Client | Behavior |
|--------|----------|
| Nutsack (iOS) | ⚠️ Ignores unit field, treats as sats |
| Nutband | ✅ Respects unit field |
| Custom clients | Depends on implementation |

WLC tokens sent to non-compatible wallets may display incorrect values but remain cryptographically valid.

## Security Considerations

1. **NIP-44 Encryption**: All sensitive data encrypted before relay storage
2. **P2PK Separation**: P2PK key is NOT the user's Nostr key
3. **Mint Trust**: Users must trust the WLC mint (non-redeemable)
4. **Relay Privacy**: Token events visible only to key holder

## Implementation Notes

### Token Encoding

Always encode WLC tokens with explicit unit:

```javascript
const token = getEncodedTokenV4({
  mint: "https://mint.wavlake.com",
  proofs: proofs,
  unit: "wlc"  // REQUIRED - never omit
});
```

### API Payment

When paying for content:

```javascript
const response = await fetch(`/api/v1/content/${trackId}`, {
  headers: {
    "X-Ecash-Token": wlcToken
  }
});
```

The paywall API validates proofs and returns content access.

## Reference Implementation

- **Wavlake Web**: https://github.com/wavlake/monorepo
- **Wallet Package**: `packages/shared/cashu/`

## Related NIPs

- [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) - Encrypted payloads
- [NIP-60](https://github.com/nostr-protocol/nips/blob/master/60.md) - Cashu wallets
- [NIP-61](https://github.com/nostr-protocol/nips/blob/master/61.md) - Nutzaps

## Changelog

- 2026-02-01: Initial draft based on Wavlake implementation
