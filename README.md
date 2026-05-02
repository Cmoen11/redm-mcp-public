# redm-mcp

MCP-server for RedM/RDR3-dokumentasjon. Gir AI-agenter (Claude Code, Cursor, Claude Desktop, osv.) eksakt oppslag mot natives (hash ↔ navn), semantisk søk, framework-docs (VORP, RSGCore, oxmysql), og community-hashes/flags/enums fra `rdr3_discoveries` (peds, weapons, animations, AI-flags, …).

Kjører som HTTP-transport på `https://redm-mcp.fivem.no/mcp` — ingen lokal installasjon av serveren trengs, bare legg den til i klienten din.

> Dette repoet inneholder kun installasjons-/bruksdokumentasjon og fungerer som issue tracker. Selve serverkoden er proprietær og distribueres ikke. Den hostede endepunktet er gratis å bruke.

## Hvorfor

Når en agent leser RedM-kode støter den typisk på:

```lua
Citizen.InvokeNative(0x09C28F828EE674FA, player, 1.5, 5000)
```

Uten oppslag gjetter agenten på parametere og semantikk. Med denne serveren:

```
lookup_native({ hash: "0x09C28F828EE674FA" })
→ BOOST_PLAYER_HORSE_SPEED_FOR_TIME(player Player, speedBoost float, duration int)
```

Serveren annonserer selv når den skal brukes via MCP `instructions` — ingen skill eller ekstra config trengs i klienten.

## Tools

| Tool | Bruk |
|------|------|
| `lookup_native` | Eksakt oppslag på hash eller navn. O(1). Bruk ved `Citizen.InvokeNative(0x...)` eller `SCREAMING_SNAKE_CASE`-navn. |
| `semantic_search` | Søk etter oppførsel/konsept ("teleport player", "spawn horse"). |
| `grep_docs` | Regex/literal-grep i rå doc-filer. Trengs for store `rdr3_discoveries`-datatabeller (audio_banks, ingameanims, …) som bare er preview-indeksert i embeddings. |
| `list_namespaces` | List kategorier og namespaces. |
| `browse` | List dokument-paths under en kategori/namespace. |
| `get_document` | Hent full markdown for en doc-path. |

## Tilgang

Endepunktet krever `Authorization: Bearer <token>`. Be om token ved å [opprette en issue](../../issues/new) eller kontakt eier direkte.

## Installasjon per klient

Alle klienter kobler til `https://redm-mcp.fivem.no/mcp` med en Bearer-token (`ACCESS_TOKEN`).

### Claude Code

```bash
claude mcp add --transport http redm-mcp https://redm-mcp.fivem.no/mcp \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

Eller rediger `~/.claude.json` manuelt:

```json
{
  "mcpServers": {
    "redm-mcp": {
      "type": "http",
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer DIN_TOKEN"
      }
    }
  }
}
```

### Cursor

Rediger `~/.cursor/mcp.json` (eller `.cursor/mcp.json` i prosjektrot):

```json
{
  "mcpServers": {
    "redm-mcp": {
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer DIN_TOKEN"
      }
    }
  }
}
```

Start Cursor på nytt. Sjekk `Settings → MCP` for grønn status.

### Claude Desktop

Claude Desktop støtter ikke HTTP-MCP direkte — bruk `mcp-remote` som proxy. Rediger `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) eller `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

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
        "Authorization: Bearer DIN_TOKEN"
      ]
    }
  }
}
```

Start Claude Desktop på nytt.

### VS Code (Copilot / Continue)

`.vscode/mcp.json` i prosjektrot:

```json
{
  "servers": {
    "redm-mcp": {
      "type": "http",
      "url": "https://redm-mcp.fivem.no/mcp",
      "headers": {
        "Authorization": "Bearer DIN_TOKEN"
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
          "Authorization: Bearer DIN_TOKEN"
        ]
      }
    }
  }
}
```

### Andre klienter (generisk)

- HTTP-transport klienter: pek mot `https://redm-mcp.fivem.no/mcp` med `Authorization: Bearer <token>`
- stdio-only klienter: bruk [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) som proxy

## Verifisere at det funker

1. `curl https://redm-mcp.fivem.no/health` → `{"ok":true}`
2. `curl https://redm-mcp.fivem.no/ingest-status` → viser versjon og sist ingested
3. I klient: spør agenten om å slå opp `0x09C28F828EE674FA` — skal returnere `BOOST_PLAYER_HORSE_SPEED_FOR_TIME`

## Endepunkter

| Path | Auth | Formål |
|------|------|--------|
| `POST /mcp` | Bearer | MCP-protokoll |
| `GET /health` | nei | Liveness |
| `GET /ingest-status` | nei | `{ current, last, ingestedAt, upToDate }` |
| `GET /stats` | nei | Bruksstats (tool-tellere, varighet, breakdowns). Kun metadata — ingen queries eller secrets lagres. |
| `GET /dashboard` | nei | HTML-dashboard over `/stats`. |

## Issues / feedback

Bug, manglende docs, feil i native-oppslag, ønsker om nye tools? [Opprett en issue](../../issues/new).

## Docs-kilder

Indekserte upstreams:

- [iamvillain/sj-redm-mcp](https://github.com/iamvillain/sj-redm-mcp) — natives, VORP, RSGCore, oxmysql
- [femga/rdr3_discoveries](https://github.com/femga/rdr3_discoveries) — community ped/weapon/anim/prop hashes, AI-flags, enums
