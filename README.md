# redm-mcp — RedM / RDR3 docs MCP server

Model Context Protocol (MCP) server for **RedM** / **RDR3**: native lookups (hash ↔ name), semantic search, framework docs (**VORP**, **RSGCore**, **oxmysql**), and the `rdr3_discoveries` community data (peds, weapons, animations, AI flags, props, audio banks). Gives AI coding agents (Claude Code, Cursor, Claude Desktop, VS Code Copilot, Zed, …) exact answers instead of guesses.

Runs as a hosted HTTP-transport MCP at `https://redm-mcp.fivem.no/mcp` — no local install, no auth, just point your client at it.

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_redm--mcp-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](vscode:mcp/install?%7B%22name%22%3A%22redm-mcp%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fredm-mcp.fivem.no%2Fmcp%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_redm--mcp-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](vscode-insiders:mcp/install?%7B%22name%22%3A%22redm-mcp%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fredm-mcp.fivem.no%2Fmcp%22%7D)
[![Add to Cursor](https://img.shields.io/badge/Cursor-Add_redm--mcp-000000?style=flat-square&logo=cursor&logoColor=white)](cursor://anysphere.cursor-deeplink/mcp/install?name=redm-mcp&config=eyJ1cmwiOiJodHRwczovL3JlZG0tbWNwLmZpdmVtLm5vL21jcCIsInR5cGUiOiJodHRwIn0=)

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

## Client setup

All clients connect to `https://redm-mcp.fivem.no/mcp`. No authentication required.

VS Code and Cursor users: click an install badge above for one-click setup. For everything else, copy-paste the config below — also available as ready-made files in [`examples/`](examples/).

### Claude Code

```bash
claude mcp add --transport http redm-mcp https://redm-mcp.fivem.no/mcp
```

Or edit `~/.claude.json` manually:

```json
{
  "mcpServers": {
    "redm-mcp": {
      "type": "http",
      "url": "https://redm-mcp.fivem.no/mcp"
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
      "url": "https://redm-mcp.fivem.no/mcp"
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
        "https://redm-mcp.fivem.no/mcp"
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
      "url": "https://redm-mcp.fivem.no/mcp"
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
          "https://redm-mcp.fivem.no/mcp"
        ]
      }
    }
  }
}
```

### Other clients (generic)

- HTTP transport clients: point to `https://redm-mcp.fivem.no/mcp`
- stdio-only clients: use [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) as a proxy

## Verify it works

1. `curl https://redm-mcp.fivem.no/health` → `{"ok":true}`
2. `curl https://redm-mcp.fivem.no/ingest-status` → shows version and last ingested timestamp
3. In your client: ask the agent to look up `0x09C28F828EE674FA` — should return `BOOST_PLAYER_HORSE_SPEED_FOR_TIME`

## Endpoints

| Path | Auth | Purpose |
|------|------|---------|
| `POST /mcp` | no | MCP protocol |
| `GET /health` | no | Liveness |
| `GET /ingest-status` | no | `{ current, last, ingestedAt, upToDate }` |
| `GET /stats` | no | Usage stats (tool counters, durations, breakdowns). Metadata only — no queries or secrets stored. |
| `GET /dashboard` | no | HTML dashboard over `/stats`. |

## Example prompts

See [`examples/prompts.md`](examples/prompts.md) for things to try once installed (native lookups, behavior searches, framework / inventory questions, debugging).

## Issues / feedback

Bug, missing docs, incorrect native lookups, requests for new tools? [Open an issue](../../issues/new).

## Doc sources

Indexed upstreams:
- [femga/rdr3_discoveries](https://github.com/femga/rdr3_discoveries) — community ped/weapon/anim/prop hashes, AI flags, enums
