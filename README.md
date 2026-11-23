# Wavlake NIPs

> **These drafts are superseded. Current kinds: https://wavlake.com/nostr-kinds.json.**
> All four drafts below are **withdrawn**. Do not build against kinds 30440–30443 or 9736. The agent guide at https://wavlake.com/llms.txt describes what Wavlake reads and writes today.

Nostr Implementation Possibilities (NIPs) that Wavlake once proposed for music streaming on Nostr. They are kept for the record only.

## Current kinds

| Kind | What it is |
|---|---|
| 36787 | track |
| 34139 + `["t","album"]` | release |
| 34139 without that tag | playlist |
| 9444 | receipt |

Channels are plain [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) groups on Wavlake's relay. Kind 30445 is retiring. Kinds 30440–30443 are retired.

The machine-readable list, generated from Wavlake's server constants, is https://wavlake.com/nostr-kinds.json.

## Withdrawn drafts

| Draft | Title | Status | Event kinds |
|-----|-------|--------|-------------|
| [music-metadata](./nips/music-metadata.md) | Music Metadata Events | Withdrawn | 30440, 30441, 30442, 30443 (retired) |
| [streaming-receipts](./nips/streaming-receipts.md) | Public Streaming Receipts | Withdrawn | 9736 (receipts are 9444) |
| [music-listening-status](./nips/music-listening-status.md) | Music Listening Status | Withdrawn | 30315 |
| [wavlake-credits](./nips/wavlake-credits.md) | Wavlake Streaming Credits | Withdrawn | (the Cashu credit model was dropped) |

## Related NIPs

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) - Basic protocol, including addressable events
- [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) - Relay-based groups
- [NIP-38](https://github.com/nostr-protocol/nips/blob/master/38.md) - User statuses

## License

Public domain (CC0)
