# redm-mcp

MCP server for RedM/RDR3 documentation. Gives AI agents (Claude Code, Cursor, Claude Desktop, etc.) exact native lookups (hash ↔ name), semantic search, framework docs (VORP, RSGCore, oxmysql), and community hashes/flags/enums from `rdr3_discoveries` (peds, weapons, animations, AI flags, …).

Runs as HTTP transport at `https://redm-mcp.fivem.no/mcp` — no local server install needed, just add it to your client.

> This repository contains only installation/usage docs and serves as the issue tracker. The server source code is proprietary and not distributed. The hosted endpoint is free to use.

## Why

When an agent reads RedM code it typically encounters:

```lua
Citizen.InvokeNative(0x09C28F828EE674FA, player, 1.5, 5000)
```

Without a lookup, the agent guesses at parameters and semantics. With this server:

```
lookup_native({ hash: "0x09C28F828EE674FA" })
→ BOOST_PLAYER_HORSE_SPEED_FOR_TIME(player Player, speedBoost float, duration int)
```

The server advertises its own usage via MCP `instructions` — no skill or extra client config needed.

## Tools

| Tool | Use |
|------|-----|
| `lookup_native` | Exact lookup by hash or name. O(1). Use for `Citizen.InvokeNative(0x...)` or `SCREAMING_SNAKE_CASE` names. |
| `semantic_search` | Search by behavior/concept ("teleport player", "spawn horse"). |
| `grep_docs` | Regex/literal grep across raw doc files. Required for the large `rdr3_discoveries` data tables (audio_banks, ingameanims, …) which are only preview-indexed in embeddings. |
| `list_namespaces` | List categories and namespaces. |
| `browse` | List document paths under a category/namespace. |
| `get_document` | Fetch full markdown for a doc path. |

## Access

The endpoint requires `Authorization: Bearer <token>`. Request a token by [opening an issue](../../issues/new) or contacting the owner directly.

## Client setup

All clients connect to `https://redm-mcp.fivem.no/mcp` with a Bearer token (`ACCESS_TOKEN`).

### Claude Code

```bash
claude mcp add --transport http redm-mcp https://redm-mcp.fivem.no/mcp \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

Or edit `~/.claude.json` manually:

```json
{
  "mcpServers": {
    "redm-mcp": {
      "type": "http",
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

### Cursor

Edit `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project root):

```json
{
  "mcpServers": {
    "redm-mcp": {
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

Restart Cursor. Check `Settings → MCP` for green status.

### Claude Desktop

Claude Desktop doesn't support HTTP MCP directly — use `mcp-remote` as a proxy. Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "redm-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://redm-mcp.fivem.no/mcp",
        "--header",
        "Authorization: Bearer YOUR_TOKEN"
      ]
    }
  }
}
```

Restart Claude Desktop.

### VS Code (Copilot / Continue)

`.vscode/mcp.json` in a project root:

```json
{
  "servers": {
    "redm-mcp": {
      "type": "http",
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

### Zed

`~/.config/zed/settings.json`:

```json
{
  "context_servers": {
    "redm-mcp": {
      "command": {
        "path": "npx",
        "args": [
          "-y",
          "mcp-remote",
          "https://redm-mcp.fivem.no/mcp",
          "--header",
          "Authorization: Bearer YOUR_TOKEN"
        ]
      }
    }
  }
}
```

### Other clients (generic)

- HTTP transport clients: point to `https://redm-mcp.fivem.no/mcp` with `Authorization: Bearer <token>`
- stdio-only clients: use [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) as a proxy

## Verify it works

1. `curl https://redm-mcp.fivem.no/health` → `{"ok":true}`
2. `curl https://redm-mcp.fivem.no/ingest-status` → shows version and last ingested timestamp
3. In your client: ask the agent to look up `0x09C28F828EE674FA` — should return `BOOST_PLAYER_HORSE_SPEED_FOR_TIME`

## Endpoints

| Path | Auth | Purpose |
|------|------|---------|
| `POST /mcp` | Bearer | MCP protocol |
| `GET /health` | no | Liveness |
| `GET /ingest-status` | no | `{ current, last, ingestedAt, upToDate }` |
| `GET /stats` | no | Usage stats (tool counters, durations, breakdowns). Metadata only — no queries or secrets stored. |
| `GET /dashboard` | no | HTML dashboard over `/stats`. |

## Issues / feedback

Bug, missing docs, incorrect native lookups, requests for new tools? [Open an issue](../../issues/new).

## Doc sources

Indexed upstreams:

- [iamvillain/sj-redm-mcp](https://github.com/iamvillain/sj-redm-mcp) — natives, VORP, RSGCore, oxmysql
- [femga/rdr3_discoveries](https://github.com/femga/rdr3_discoveries) — community ped/weapon/anim/prop hashes, AI flags, enums
