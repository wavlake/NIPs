# NIP-XX: Public Streaming Receipts

`withdrawn`

> **Withdrawn.** This draft is superseded. Do not use kind 9736. Wavlake's receipt is kind 9444, and its `a` tag names a 36787 track or a 34139 release. Current kinds: https://wavlake.com/nostr-kinds.json.

This NIP defines an event kind for publicly attesting to content streams, enabling transparent royalty tracking and social proof of listening activity.

## Abstract

When a user streams music through a paywall service, the service can publish a public "receipt" event that proves the stream occurred without revealing the listener's identity. This creates an auditable record of content consumption for royalty distribution and analytics.

## Motivation

1. **Royalty Transparency**: Artists can verify their play counts independently
2. **Social Proof**: Popular content becomes discoverable through stream counts
3. **Analytics**: Aggregated listening data without user tracking
4. **Auditing**: Third parties can verify paywall service claims

## Event Kind

| Kind | Name | Description |
|------|------|-------------|
| 9736 | Stream Receipt | Proof of content stream |

This is a **regular event** (not replaceable) - each stream creates a new event.

## Event Structure

```json
{
  "kind": 9736,
  "pubkey": "<paywall-service-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["a", "30440:<artist-pubkey>:<track-identifier>"],
    ["p", "<artist-pubkey>"],
    ["stream_type", "<free|paid|honor|free_tier>"],
    ["amount", "<credits-paid>"],
    ["unit", "<credits|sat|usd>"]
  ],
  "content": ""
}
```

## Tags

### Required Tags

| Tag | Description |
|-----|-------------|
| `a` | Reference to the track that was streamed |
| `p` | Artist pubkey (for notifications) |
| `stream_type` | How access was granted |

### Stream Types

| Type | Description |
|------|-------------|
| `free` | No payment required (free content) |
| `paid` | Ecash payment provided |
| `honor` | Honor-system track, payment optional |
| `free_tier` | Listener reached spending cap (free access) |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `amount` | Payment amount (if paid) |
| `unit` | Payment unit: "credits", "sat", "usd" |
| `mint` | Cashu mint URL used for payment |
| `duration` | Actual stream duration in seconds |
| `completed` | "true" if track was completed |
| `quality` | Stream quality (e.g., "high", "low", "128kbps") |

## Privacy Considerations

### What IS Published

- Track reference (which content was streamed)
- Stream type (free, paid, etc.)
- Payment amount (if applicable)
- Timestamp
- Service pubkey

### What is NOT Published

- Listener identity (pubkey)
- Listener IP address
- Geographic location
- Listening history linkage
- Payment proofs/tokens

### Anonymity Properties

- Individual receipts cannot be linked to users
- Multiple streams from same user appear independent
- Only the paywall service knows listener identity
- Service pubkey allows verification of legitimate receipts

## Use Cases

### Artist Dashboard

Artists can query receipts for their tracks:

```json
{
  "kinds": [9736],
  "#p": ["<artist-pubkey>"]
}
```

### Track Statistics

Get all streams for a specific track:

```json
{
  "kinds": [9736],
  "#a": ["30440:<artist-pubkey>:<track-id>"]
}
```

### Revenue Verification

Filter by paid streams:

```json
{
  "kinds": [9736],
  "#p": ["<artist-pubkey>"],
  "#stream_type": ["paid"]
}
```

## Service Implementation

### Publishing Receipts

```javascript
async function publishStreamReceipt(stream) {
  const event = {
    kind: 9736,
    pubkey: serviceKeys.pubkey,
    created_at: Math.floor(Date.now() / 1000),
    tags: [
      ["a", `30440:${stream.artistPubkey}:${stream.trackId}`],
      ["p", stream.artistPubkey],
      ["stream_type", stream.type],
    ],
    content: "",
  };
  
  if (stream.amount > 0) {
    event.tags.push(["amount", String(stream.amount)]);
    event.tags.push(["unit", "credits"]);
  }
  
  const signed = signEvent(event, serviceKeys.privateKey);
  await publishToRelays(signed);
}
```

### Rate Limiting

Services SHOULD:
- Batch receipts (e.g., publish every minute)
- Avoid duplicate receipts for the same stream
- Implement reasonable delays to prevent timing attacks

## Verification

### Service Identity

Clients verifying receipts should:
1. Check the service pubkey against known paywall services
2. Verify event signatures
3. Optionally verify NIP-05 for service identity

### Cross-Referencing

Receipts can be cross-referenced with:
- Artist earnings dashboards
- Payment proofs (for auditing)
- External analytics services

## Aggregation

### Daily Summaries

Services MAY publish aggregated summaries:

```json
{
  "kind": 9736,
  "tags": [
    ["a", "30440:abc:track-1"],
    ["summary", "daily"],
    ["date", "2026-02-01"],
    ["total_streams", "1234"],
    ["paid_streams", "567"],
    ["total_credits", "1134"]
  ]
}
```

### Privacy of Aggregates

Aggregated data should use differential privacy techniques when counts are small to prevent inference attacks.

## Security Considerations

1. **Sybil Resistance**: Only trusted paywall services should publish receipts
2. **Spam Prevention**: Relays may require NIP-05 verification for receipt publishers
3. **Timing Attacks**: Introduce random delays between stream and receipt
4. **Enumeration**: Don't include sequential IDs that could link streams

## Reference Implementation

- **Wavlake API**: Publishes receipts for all paywall streams
- **Artist Dashboard**: Displays receipt-based analytics

## Changelog

- 2026-02-01: Initial draft
