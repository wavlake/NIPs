# Wavlake NIPs

Wavlake's current Nostr kinds: https://wavlake.com/nostr-kinds.json (generated from Wavlake's server constants). The agent guide at https://wavlake.com/llms.txt describes what Wavlake reads and writes.

| Kind | What it is |
|---|---|
| 36787 | track |
| 34139 + `["t","album"]` | release |
| 34139 without that tag | playlist |
| 9444 | receipt |

Channels are plain [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) groups on Wavlake's relay.

Wavlake's NIP drafts for these kinds will be published here after the public launch.

## Related NIPs

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) - Basic protocol, including addressable events
- [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) - Relay-based groups

## License

Public domain (CC0)
