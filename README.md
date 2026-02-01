# Wavlake NIPs

Nostr Implementation Possibilities (NIPs) proposed by Wavlake for music streaming and creator economy on Nostr.

## Overview

Wavlake is building a decentralized music streaming platform on Nostr. This repository contains our NIP proposals for:

- **Music metadata** - Standard formats for tracks, albums, and artist profiles
- **Streaming credits** - Cashu-based micropayments for content access
- **Social features** - Music-specific social interactions

## Proposed NIPs

| NIP | Title | Status | Event Kinds |
|-----|-------|--------|-------------|
| [NIP-XX](./nips/music-metadata.md) | Music Metadata Events | Draft | 30440, 30441, 30442, 30443 |
| [NIP-XX](./nips/streaming-receipts.md) | Public Streaming Receipts | Draft | 9736 |
| [NIP-XX](./nips/music-listening-status.md) | Music Listening Status | Draft | 30315 |
| [NIP-XX](./nips/wavlake-credits.md) | Wavlake Streaming Credits | Draft | (uses NIP-60/61) |

## Event Kind Registry

### Music Metadata (30440-30449 range)

| Kind | Name | Description |
|------|------|-------------|
| 30440 | Track Metadata | Addressable event for individual track information |
| 30441 | Album Metadata | Addressable event for album collections |
| 30442 | Artist Profile | Addressable event for artist/band profiles |
| 30443 | Music Playlist | User-created playlists referencing tracks |

### Streaming & Payments

| Kind | Name | Description |
|------|------|-------------|
| 9736 | Stream Receipt | Public proof of content stream (anonymous) |

### User Status

| Kind | Name | Description |
|------|------|-------------|
| 30315 | User Status | Music listening status (extends NIP-38) |

## Implementation Status

These NIPs are currently implemented in:

- **Wavlake Platform** - https://wavlake.com
- **Wavlake Relay** - wss://relay.wavlake.com

## Contributing

We welcome feedback and contributions! Please:

1. Open an issue for discussion
2. Submit PRs for proposed changes
3. Join our Discord for real-time discussion

## Related NIPs

These proposals build on existing NIPs:

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) - Basic protocol
- [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md) - Parameterized replaceable events
- [NIP-38](https://github.com/nostr-protocol/nips/blob/master/38.md) - User statuses
- [NIP-60](https://github.com/nostr-protocol/nips/blob/master/60.md) - Cashu wallets
- [NIP-61](https://github.com/nostr-protocol/nips/blob/master/61.md) - Nutzaps

## License

Public domain (CC0)
