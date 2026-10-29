# @pipeworx/noaa-ndbc

Observed marine conditions from NOAA's National Data Buoy Center — ~1,350
active moored buoys, coastal C-MAN stations, DART tsunami stations and partner
platforms reporting wave height, wave period, sea-surface temperature, wind and
pressure.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ndbc_stations \
  -H 'Content-Type: application/json' \
  -d '{"near_latitude":37.77,"near_longitude":-122.42,"has_met":true,"limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ndbc_stations`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-noaa-ndbc` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/noaa-ndbc/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
