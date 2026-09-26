# @pipeworx/noaa-ndbc

Observed marine conditions from NOAA's National Data Buoy Center — ~1,350
active moored buoys, coastal C-MAN stations, DART tsunami stations and partner
platforms reporting wave height, wave period, sea-surface temperature, wind and
pressure.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `ndbc_stations(bbox?, near_latitude?, near_longitude?, name_contains?, owner?, type?, has_met?, has_waterquality?, has_currents?, limit?)` — the active-station inventory, distance-ranked when a point is given.
- `ndbc_latest_obs(station_id)` — the newest standard meteorological observation at one station.
- `ndbc_station_history(station_id, hours?, limit?)` — up to ~45 days of rows, newest first.

## Auth

Keyless. NDBC serves nothing without a `User-Agent`, which this pack always sends.

## Data sources

- `https://www.ndbc.noaa.gov/activestations.xml` — active-station inventory.
- `https://www.ndbc.noaa.gov/data/realtime2/<id>.txt` — last ~45 days of observations.

Things the next person would otherwise rediscover:

- **Missing values are the literal string `MM`**, not an empty field. Every
  numeric here is `null` when the station did not report it; a buoy reporting
  wind but no wave height is normal.
- **realtime2 rows are NEWEST FIRST.** Row 0 is the latest.
- Two `#` header lines, the first names columns and the second units. Columns
  are parsed positionally because some partner stations abbreviate the header.
- **Not every active station has a realtime2 file.** DART and water-quality-only
  platforms 404 there. That is a coverage fact about the station, and
  `ndbc_latest_obs` says so rather than reporting an outage.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "noaa-ndbc": {
      "url": "https://gateway.pipeworx.io/noaa-ndbc/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/noaa-ndbc/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ndbc_stations \
  -H 'Content-Type: application/json' \
  -d '{"near_latitude":37.77,"near_longitude":-122.42,"has_met":true,"limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ndbc_stations`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "noaa-ndbc": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-noaa-ndbc"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-noaa-ndbc
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Noaa Ndbc data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
