# @pipeworx/us-national-archives

The US National Archives Catalog (NARA) — descriptions of the federal
government's permanent records from every agency and presidential library, with
direct links to the digitised page images where they exist.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1684+ live data sources.

## Tools

- `nara_search(query, limit?, page?, available_online?, level_of_description?, record_group_number?, include_digital_objects?)`
  — keyword search across the catalog.
- `nara_record(naid)` — one record in full, with its archival hierarchy.
- `nara_digital_objects(naid, limit?)` — the scans attached to a record, with
  public-domain download URLs.

## Auth

Keyless. **No `x-api-key` is needed or used** — see below.

## Data sources

- <https://catalog.archives.gov/proxy/v3/records/search> — search and record fetch.

Things worth knowing:

- **The documented `/api/v2/` path with an `x-api-key` is not routed.** Every
  `catalog.archives.gov/api/v2/...` URL returns the catalog's single-page app as
  `text/html` with a **200** and the same ETag as a deliberately nonsensical
  path, with or without a key. It is not an outage and a key would not fix it;
  the path simply is not served. The endpoint the catalog's own front end calls
  is `/proxy/v3/records/search`, and it is unauthenticated.
- **`/proxy/records/search` (no `v3`) rejects most page sizes.** `limit` 1, 10,
  20, 50, 100 and 1000 return JSON; 2, 3, 4, 5, 15 and 25 return the SPA HTML
  with a 200. Deterministic over three runs each. `/proxy/v3/records/search`
  accepts any limit, which is why this pack only calls v3 — and it still checks
  the content type before parsing, because a 200 that is secretly HTML is
  exactly the shape of a silent failure.
- **Paging is `page` (1-based).** `offset` and `from` are ignored.
- **A single record is `naId_is=`**, not `naId=`; the latter is the comma-list
  multi-fetch form.
- **NAIDs are JSON numbers in the body and strings in arguments.** Reading only
  the string form blanks every id in a search result without erroring.
- The payload is raw Elasticsearch: `body.hits.total.value` and
  `body.hits.hits[]._source.record`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "us-national-archives": {
      "url": "https://gateway.pipeworx.io/us-national-archives/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/us-national-archives/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1684+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/nara_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"Apollo 11 moon","available_online":true,"limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/nara_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "us-national-archives": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-us-national-archives"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-us-national-archives
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Us National Archives data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
