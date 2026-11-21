# @pipeworx/dk-elections

Danish national election and referendum results, final count, from the national total down to polling-station area: votes, percent and seats per party, eligible voters, turnout and invalid votes.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2024+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `dk_elections_list_elections(type?)` -- the elections covered (date, type, geography levels).
- `dk_elections_results(election?, type?, level?, area?, within?)` -- results of one election at country, landsdel (3), storkreds (10), opstillingskreds (92) or afstemningsomraade (polling-station area, about 1,300-1,600). Omit `area` at the two fine levels to list the units and their ids.
- `dk_elections_party_over_time(party, type?, level?, area?, within?, from_year?)` -- one party across elections, nationally or for a named area (matched across elections by name).
- `dk_elections_turnout(election?, type?, level?, area?, within?)` -- eligible voters, votes cast, turnout and invalid votes, for one election or across all elections of a type.

## Coverage (stated in every response)

National elections and referendums from 7 June 2009 to the latest held: Folketing elections (2011, 2015, 2019, 2022, 2026), European Parliament elections (2009, 2014, 2019, 2024) and referendums (2009, 2014, 2015, 2022).

Not covered: anything before 2009, municipal and regional elections, Greenland and the Faroe Islands. An election outside the window returns `covered: false` naming the window and the elections available.

## Auth

Keyless.

## Data sources

- Statistics Denmark (Danmarks Statistik), national election results, final count.

Live per request; nothing is stored. Results are final once published, so responses are cached for a day. Statistics Denmark also runs the Danish Election Database (1979 onward, municipal elections), which this pack does not read; the Statbank election tables are reachable through `dst-dk`.

## Notes

- Polling-area ids are assigned per election, so areas are matched across elections by name. Boundaries and polling stations change; a matched area is best effort, and the response says so.
- Party letters were reassigned over time (letter Q, for instance, belonged to different parties). A letter query returns the party that held it in each election, with the name.
- Referendums have yes/no choices in place of parties; seats are null.
- `data_as_of` is the publisher's last correction time for the returned unit.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "dk-elections": {
      "url": "https://gateway.pipeworx.io/dk-elections/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/dk-elections/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2024+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/dk_elections_list_elections \
  -H 'Content-Type: application/json' \
  -d '{"type":"folketingsvalg"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/dk_elections_list_elections`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-dk-elections` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/dk-elections/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "dk-elections": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-dk-elections"]
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
ask_pipeworx({ question: "your question about Dk Elections data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
