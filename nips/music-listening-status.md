# NIP-XX: Music Listening Status

`withdrawn`

> **Withdrawn.** This draft is superseded. It points at retired kinds 30440, 30441 and 30442. Current kinds: https://wavlake.com/nostr-kinds.json.

This NIP extends NIP-38 (User Statuses) to support music listening status, enabling "now playing" features and social music discovery.

## Abstract

Users can publish their current listening activity as a status event, allowing friends and followers to discover music through their social graph.

## Motivation

1. **Social Discovery**: Find new music through what friends are listening to
2. **Real-time Activity**: Show "now playing" on profiles
3. **Engagement**: React to and discuss what others are listening to
4. **Interoperability**: Works with existing NIP-38 status readers

## Event Kind

| Kind | Name | Description |
|------|------|-------------|
| 30315 | User Status | Music listening status (NIP-38 extension) |

This is a **parameterized replaceable event** using `d` tag for status type.

## Event Structure

```json
{
  "kind": 30315,
  "pubkey": "<user-pubkey>",
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "music"],
    ["a", "30440:<artist-pubkey>:<track-identifier>"],
    ["expiration", "<unix-timestamp>"],
    ["r", "<client-url>"]
  ],
  "content": "Listening to Track Title by Artist Name"
}
```

## Tags

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Status type: "music" for listening status |

### Recommended Tags

| Tag | Description |
|-----|-------------|
| `a` | Reference to track being played (30440:pubkey:id) |
| `expiration` | When status should be considered stale |
| `r` | Client/app URL where user is listening |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `album` | Album reference (30441:pubkey:id) if applicable |
| `artist` | Artist reference (30442:pubkey:id) |
| `position` | Current playback position in seconds |
| `duration` | Track duration in seconds |
| `repeat` | "true" if on repeat |
| `shuffle` | "true" if playlist is shuffled |

## Content Field

The `content` field contains a human-readable status message:

```
"Listening to {track_title} by {artist_name}"
"Now playing: {track_title}"
"🎵 {track_title} - {artist_name}"
```

This allows status to be displayed even by clients that don't understand music-specific tags.

## Expiration

The `expiration` tag indicates when the status should be considered stale:

```json
["expiration", "1706745600"]  // 5 minutes from now
```

Clients SHOULD:
- Not display expired statuses
- Update status when track changes
- Clear status when playback stops

Recommended expiration: **track duration + 30 seconds**

## Client Implementation

### Publishing Status

```javascript
async function publishListeningStatus(track) {
  const event = {
    kind: 30315,
    pubkey: userPubkey,
    created_at: Math.floor(Date.now() / 1000),
    tags: [
      ["d", "music"],
      ["a", `30440:${track.artistPubkey}:${track.id}`],
      ["expiration", String(Math.floor(Date.now() / 1000) + track.duration + 30)],
      ["r", "https://wavlake.com"],
    ],
    content: `Listening to ${track.title} by ${track.artist}`,
  };
  
  const signed = signEvent(event, userPrivateKey);
  await publishToRelays(signed);
}
```

### Clearing Status

To clear listening status, publish with empty content:

```json
{
  "kind": 30315,
  "tags": [["d", "music"]],
  "content": ""
}
```

### Reading Statuses

```javascript
// Get listening status for followed users
const filter = {
  kinds: [30315],
  authors: followedPubkeys,
  "#d": ["music"],
};

const statuses = await relay.subscribe(filter);
```

## Social Features

### Now Playing Feed

Display a feed of what people in your network are listening to:

```json
{
  "kinds": [30315],
  "authors": ["<followed-pubkey-1>", "<followed-pubkey-2>"],
  "#d": ["music"],
  "since": <one-hour-ago>
}
```

### Reactions

Users can react to listening statuses with NIP-25 reactions:

```json
{
  "kind": 7,
  "content": "🔥",
  "tags": [
    ["e", "<status-event-id>"],
    ["p", "<status-author>"],
    ["k", "30315"]
  ]
}
```

### Comments

Users can comment on statuses using NIP-22 threading:

```json
{
  "kind": 1,
  "content": "Great taste! I love this album",
  "tags": [
    ["e", "<status-event-id>", "", "root"],
    ["p", "<status-author>"]
  ]
}
```

## Privacy Considerations

### Opt-in Publishing

Clients MUST:
- Make status publishing opt-in
- Clearly indicate when status is being shared
- Provide easy toggle to disable

### What IS Published

- Track currently playing
- Client/app being used
- Timestamp

### What is NOT Published

- Listening history
- Playlist contents
- Playback queue
- Precise location

## UI Examples

### Profile Status

```
🎵 Now Playing
"Midnight Drive" by Stellar Pulse
via Wavlake · 2 minutes ago
```

### Activity Feed

```
Alice is listening to "Summer Vibes" by Beach Collective
Bob is listening to "Code Flow" by Dev Beats
Carol is listening to "Jazz Standards" by Classic Trio
```

## Compatibility

This NIP is compatible with:
- **NIP-38**: Generic user statuses (d=music is a status type)
- **NIP-25**: Reactions to statuses
- **NIP-22**: Threaded comments on statuses

Clients implementing NIP-38 will see listening statuses as generic statuses with the human-readable content.

## Reference Implementation

- **Wavlake Web**: Publishes listening status when enabled
- **Wavlake Mobile**: Status sync across devices

## Changelog

- 2026-02-01: Initial draft
