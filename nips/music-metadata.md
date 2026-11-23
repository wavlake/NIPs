# NIP-XX: Music Metadata Events

`withdrawn`

> **Withdrawn.** This draft is superseded. Kinds 30440–30443 are retired. Wavlake music is kind 36787 (track) and kind 34139 (a release with `["t","album"]`, a playlist without it). Current kinds: https://wavlake.com/nostr-kinds.json.

This NIP defines a set of addressable events for publishing music metadata on Nostr, enabling decentralized music streaming platforms and clients.

## Abstract

Music creators need a standard way to publish track, album, and artist information to Nostr relays. This NIP defines four parameterized replaceable event kinds (30440-30443) that form a complete music metadata schema.

## Motivation

Existing music platforms are centralized, giving platforms control over artist content and revenue. By standardizing music metadata on Nostr:

1. Artists own their content and metadata
2. Multiple clients can display the same catalog
3. Payments can flow directly to creators via Lightning/Cashu
4. Discoverability improves through relay federation

## Event Kinds

| Kind | Name | Description |
|------|------|-------------|
| 30440 | Track | Individual song or audio piece |
| 30441 | Album | Collection of tracks |
| 30442 | Artist Profile | Artist or band information |
| 30443 | Playlist | User-created track collections |

All kinds are **addressable** (NIP-01), using a `d` tag as the unique identifier within each pubkey's namespace.

---

## Kind 30440: Track Metadata

Represents a single audio track published by an artist.

### Event Structure

```json
{
  "kind": 30440,
  "pubkey": "<artist-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "<track-identifier>"],
    ["title", "<track-title>"],
    ["a", "30442:<artist-pubkey>:<artist-identifier>"],
    ["a", "30441:<artist-pubkey>:<album-identifier>"],
    ["duration", "<seconds>"],
    ["image", "<artwork-url>"],
    ["stream_url", "<audio-url>"],
    ["t", "<genre-tag>"],
    ["license", "<license-type>"]
  ],
  "content": "<optional-json-metadata>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique track identifier (e.g., UUID, slug) |
| `title` | Track title |

### Recommended Tags

| Tag | Description |
|-----|-------------|
| `a` | Reference to artist profile (30442:pubkey:identifier) |
| `a` | Reference to album (30441:pubkey:identifier) |
| `duration` | Track duration in seconds |
| `image` | Album art or track artwork URL |
| `stream_url` | Direct URL to audio file |
| `t` | Genre/mood tags (multiple allowed) |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `isrc` | International Standard Recording Code |
| `license` | License type (e.g., "cc-by-4.0", "all-rights-reserved") |
| `explicit` | "true" if explicit content |
| `bpm` | Beats per minute |
| `key` | Musical key (e.g., "C major", "Am") |
| `year` | Release year |
| `lyrics` | Song lyrics or URL to lyrics |

### Content Field

The `content` field MAY contain extended metadata as JSON:

```json
{
  "title": "Track Title",
  "artist": "Artist Name",
  "album": "Album Name",
  "genre": "Electronic",
  "duration": 234,
  "artwork_url": "https://...",
  "audio_url": "https://...",
  "description": "About this track...",
  "access_mode": "paywall",
  "price_credits": 2
}
```

### Pricing Tags (Paywall Support)

For paid content:

| Tag | Description |
|-----|-------------|
| `access_mode` | "free", "honor", or "paywall" |
| `price` | Price amount |
| `price_unit` | Unit: "credits", "sat", "usd" |
| `mint` | Cashu mint URL for payments |

Example:
```json
["access_mode", "paywall"],
["price", "2"],
["price_unit", "credits"],
["mint", "https://mint.wavlake.com"]
```

---

## Kind 30441: Album Metadata

Represents an album or EP containing multiple tracks.

### Event Structure

```json
{
  "kind": 30441,
  "pubkey": "<artist-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "<album-identifier>"],
    ["title", "<album-title>"],
    ["a", "30442:<artist-pubkey>:<artist-identifier>"],
    ["image", "<cover-art-url>"],
    ["year", "<release-year>"],
    ["track", "30440:<pubkey>:<track-identifier>", "<track-number>"]
  ],
  "content": "<optional-json-metadata>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique album identifier |
| `title` | Album title |
| `a` | Reference to artist profile |

### Recommended Tags

| Tag | Description |
|-----|-------------|
| `image` | Album cover art URL |
| `year` | Release year |
| `track` | Track references with position (multiple) |
| `description` | Album description |
| `genre` | Primary genre |

### Track Ordering

The `track` tag includes position for ordering:

```json
["track", "30440:abc123:track-1", "1"],
["track", "30440:abc123:track-2", "2"],
["track", "30440:abc123:track-3", "3"]
```

---

## Kind 30442: Artist Profile

Represents an artist, band, or music creator profile.

### Event Structure

```json
{
  "kind": 30442,
  "pubkey": "<artist-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "<artist-identifier>"],
    ["name", "<display-name>"],
    ["image", "<profile-image-url>"],
    ["banner", "<banner-image-url>"],
    ["website", "<website-url>"],
    ["lud16", "<lightning-address>"],
    ["nip05", "<nip05-identifier>"]
  ],
  "content": "<json-metadata>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique artist identifier |
| `name` | Artist display name |

### Recommended Tags

| Tag | Description |
|-----|-------------|
| `image` | Profile picture URL |
| `banner` | Banner/header image URL |
| `website` | Official website |
| `lud16` | Lightning address for tips |
| `nip05` | NIP-05 verification |
| `bio` | Artist biography |

### Content Field (Extended Metadata)

```json
{
  "name": "Artist Name",
  "bio": "Biography text...",
  "picture": "https://...",
  "banner": "https://...",
  "website": "https://...",
  "genres": ["Electronic", "Ambient"],
  "location": "Austin, TX",
  "formed_year": 2020,
  "members": ["Alice", "Bob"],
  "social_links": {
    "twitter": "artisthandle",
    "instagram": "artisthandle",
    "spotify": "spotify:artist:123"
  },
  "lud16": "artist@wavlake.com",
  "nip05": "artist@wavlake.com"
}
```

---

## Kind 30443: Music Playlist

User-created playlists referencing tracks from any artist.

### Event Structure

```json
{
  "kind": 30443,
  "pubkey": "<user-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "<playlist-identifier>"],
    ["title", "<playlist-title>"],
    ["description", "<playlist-description>"],
    ["image", "<playlist-cover-url>"],
    ["a", "30440:<pubkey>:<track-id>"],
    ["a", "30440:<pubkey>:<track-id>"]
  ],
  "content": ""
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique playlist identifier |
| `title` | Playlist title |

### Recommended Tags

| Tag | Description |
|-----|-------------|
| `a` | Track references (ordered by tag position) |
| `description` | Playlist description |
| `image` | Playlist cover image |
| `public` | "true" or "false" for visibility |

### Track Ordering

Tracks are ordered by their position in the tags array:

```json
["a", "30440:abc:track-1"],  // Position 1
["a", "30440:def:track-5"],  // Position 2
["a", "30440:abc:track-3"]   // Position 3
```

---

## Discovery & Querying

### Finding All Tracks by an Artist

```json
{
  "kinds": [30440],
  "authors": ["<artist-pubkey>"]
}
```

### Finding Tracks in an Album

```json
{
  "kinds": [30440],
  "#a": ["30441:<pubkey>:<album-identifier>"]
}
```

### Finding Artist Profile

```json
{
  "kinds": [30442],
  "authors": ["<artist-pubkey>"],
  "#d": ["<artist-identifier>"]
}
```

### NIP-19 Encoding (naddr)

All music events can be referenced using `naddr` (NIP-19):

```
naddr1...  // Encodes kind:pubkey:identifier:relays
```

Example for a track:
```javascript
nip19.naddrEncode({
  kind: 30440,
  pubkey: "abc123...",
  identifier: "my-track",
  relays: ["wss://relay.wavlake.com"]
})
```

---

## Client Behavior

### Publishing

1. Generate or use existing keypair for the artist identity
2. Create track/album/artist events with appropriate tags
3. Sign and publish to music-supporting relays
4. Update by publishing new events with same `d` tag

### Reading

1. Subscribe to kinds 30440-30443 for music content
2. Parse both tags and content JSON for metadata
3. Resolve `a` tag references for related content
4. Handle missing optional fields gracefully

### Caching

Clients SHOULD:
- Cache resolved metadata locally
- Respect `created_at` for updates
- Implement reasonable TTLs for relay queries

---

## Relay Considerations

Relays supporting music content SHOULD:

1. Index the 30440-30443 kind range
2. Support `#a` tag queries for relationships
3. Consider storage implications for audio URLs
4. Implement rate limiting for bulk publishing

---

## Security Considerations

1. **Audio URLs**: The `stream_url` may point to paywalled content requiring authentication
2. **Ownership**: Track pubkey represents the publisher, not necessarily copyright owner
3. **Verification**: Use NIP-05 and signed content for artist verification
4. **Licensing**: Clients should respect license tags and display appropriately

---

## Reference Implementation

- **Wavlake Platform**: https://github.com/wavlake/monorepo
- **Relay**: wss://relay.wavlake.com

---

## Changelog

- 2026-02-01: Initial draft
