# Tunetank MCP Server

**The Tunetank music & sound‑effects catalog as an [MCP](https://modelcontextprotocol.io) server** — so any AI assistant (Claude, ChatGPT, Cursor, …) can find the right royalty‑free track or SFX for a video, ad, podcast or stream.

Ask things like *“find an energetic rock track around 60 seconds for a YouTube intro”* or *“a whoosh SFX under 2 seconds”* and the assistant searches Tunetank's real catalog and hands back a preview link plus the track page on tunetank.com.

- 🎵 Search **music** by mood, genre, theme, artist, name and **target length (± tolerance)**
- 🔊 Search **sound effects (SFX)** by category and length
- 📚 Browse genres, moods, themes and curated playlists
- ▶️ Every result includes an **audio preview URL** and the **canonical track page** (`https://tunetank.com/track/<id>-<slug>/`)
- 🆓 **Completely free** — no API key, no auth, no rate‑limit sign‑up. Read‑only.

---

## Server

| | |
|---|---|
| **Endpoint** | `https://mcp.tunetank.com` |
| **Transport** | Streamable HTTP (JSON‑RPC 2.0) |
| **Auth** | None (public, read‑only) |
| **Cost** | Free |

---

## Setup

Point any MCP‑capable client at the endpoint above. Examples for common clients:

### Claude Desktop / Claude.ai (Connectors)
Add a **Custom Connector** with the URL:

```
https://mcp.tunetank.com
```

Or, via the config file (`claude_desktop_config.json`) using the `mcp-remote` bridge:

```json
{
  "mcpServers": {
    "tunetank": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.tunetank.com"]
    }
  }
}
```

### Cursor
`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "tunetank": {
      "url": "https://mcp.tunetank.com"
    }
  }
}
```

### VS Code / other clients
Any client that supports **remote (HTTP) MCP servers** — just use the URL `https://mcp.tunetank.com`. For clients that only speak stdio, bridge with `npx -y mcp-remote https://mcp.tunetank.com`.

---

## Tools

| Tool | Arguments | Returns |
|---|---|---|
| `search_music` | `query`, `name`, `artist`, `genre`, `mood`, `theme`, `duration`, `tolerance`, `limit` | Tracks: name, artist, duration, bpm, preview URL, track page URL, genres/moods/themes |
| `search_sfx` | `query`, `category`, `duration`, `tolerance`, `limit` | Sound effects: name, duration, preview URL, waveform |
| `list_genres` | — | All music genres (`id`, `name`, `alias`) |
| `list_moods` | — | All music moods (`id`, `name`, `alias`) |
| `list_themes` | — | All music themes (`id`, `name`, `alias`) |
| `list_playlists` | `limit` | Curated playlists (name, description, track count) |

### Argument notes

- **`query`** *(search_music)* — free text; matches the **track name or the artist name**.
- **`genre` / `mood` / `theme`** — accept either a **name/alias** (e.g. `"rock"`, `"happy"`) or a numeric **id** (from the `list_*` tools).
- **`duration`** — target length in **seconds**.
- **`tolerance`** — allowed **± seconds** around `duration` (default `5`). Example: `duration: 60, tolerance: 10` → tracks between 50s and 70s.
- **`category`** *(search_sfx)* — SFX category name or id.
- **`limit`** — max results (default `20`, max `50`; playlists default `50`, max `200`).

All string filters are partial, case‑insensitive matches. Only published tracks/SFX are returned.

---

## Examples

**“Energetic rock, ~60s, for an intro”**

```json
{
  "name": "search_music",
  "arguments": { "genre": "rock", "mood": "energetic", "duration": 60, "tolerance": 10, "limit": 5 }
}
```

**“A short whoosh sound effect under 2 seconds”**

```json
{
  "name": "search_sfx",
  "arguments": { "query": "whoosh", "duration": 1.5, "tolerance": 1.5 }
}
```

**Sample track result**

```json
{
  "id": 2270,
  "name": "Adventures",
  "artist": "A Himitsu",
  "duration": 138,
  "bpm": 120,
  "url": "https://tunetank.com/track/2270-adventures/",
  "preview": "https://…/preview.mp3",
  "genres": ["Pop"],
  "moods": ["Happy", "Inspiring"],
  "themes": ["Vlog"]
}
```

---

## Try it (raw JSON‑RPC)

The transport is spec‑strict, so include both `Content-Type` and `Accept` headers:

```bash
# List available tools
curl -s https://mcp.tunetank.com \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'

# Search music
curl -s https://mcp.tunetank.com \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"search_music","arguments":{"mood":"happy","duration":60,"tolerance":10,"limit":3}}}'
```

---

## Licensing

All music is **free to use for personal and non‑commercial purposes**.

For anything else (commercial projects, ads, monetized content, client work, etc.), head to [tunetank.com](https://tunetank.com) and review the licensing terms there.

---

## About

Music and sound effects come from [Tunetank](https://tunetank.com) — royalty‑free audio for creators. This MCP server is a **discovery** channel: it returns previews and track pages so you can listen and grab the track on tunetank.com. Downloading and licensing happen on the site.

Questions or issues? Reach out at **support@tunetank.com**.
