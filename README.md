# @pipeworx/open-media-search

One search across Openverse, Wikimedia Commons, Pixabay and Pexels for freely usable images, audio and video. Every result has the same shape and a paste-ready licence and attribution line.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2035+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_open_media(query, media_type?, licence_class?, min_width?, min_height?, sources?, limit?, pixabay_key?, pexels_key?)` -- searches the publishers that carry the media type (image, audio or video), interleaves their results, returns a file listed by two publishers once (`also_listed_by` names the other), and reports each publisher's status in `sources`. Each result carries title, creator, media url, thumbnail, width and height or duration, the publisher, the licence (id, name, version, url), `commercial_use`, `modification_allowed`, `attribution_required`, and an `attribution` line to paste. `licence_class` is `any`, `commercial`, `commercial_no_attribution` or `commercial_modifiable`.
- `list_open_media_sources()` -- what each publisher carries, the licence it publishes under, and whether a key is needed.

## Auth

Openverse and Wikimedia Commons are keyless. Pixabay is searched for every caller, anonymous included, using a platform key, so no key is needed. Pexels is searched without a key for registered callers; anonymous callers get Pexels only when they pass their own `pexels_key` (free from the publisher), because the shared Pexels key is not given to anonymous traffic. `pixabay_key` and `pexels_key` stay available to bring your own and take precedence over the platform key. When a publisher has no key for the call, or the platform key's hourly budget is spent, it appears in `sources` with status `needs_key`; it is never silently left out.

## Data sources

- Openverse (WordPress.org), image and audio search over Creative Commons and public domain works.
- Wikimedia Commons (Wikimedia Foundation), file search with per-file licence metadata.
- Pixabay, photo and video library under the Pixabay Content License.
- Pexels, photo and video library under the Pexels License.

Live per request, nothing is stored. Licence fields are as each publisher states them. A result whose licence could not be classified keeps its raw licence name, has null booleans, and is excluded (and counted in `excluded_unclassified`) when a `licence_class` is set.

Source status values: `ok`, `empty`, `skipped` (publisher does not carry that media type), `needs_key`, `rate_limited`, `error`. If every queried publisher fails the call fails rather than returning an empty list.

Notes: Openverse can take 10-15 seconds on a query it has not seen before; the call waits up to 22 seconds for it. Results are cached for an hour. Pixabay and Pexels attribution is not required by their licences; the line is supplied anyway as a courtesy credit.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "open-media-search": {
      "url": "https://gateway.pipeworx.io/open-media-search/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/open-media-search/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2035+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/search_open_media \
  -H 'Content-Type: application/json' \
  -d '{"query":"rain","media_type":"audio","licence_class":"commercial","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_open_media`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-open-media-search` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/open-media-search/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "open-media-search": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-open-media-search"]
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
ask_pipeworx({ question: "your question about Open Media Search data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
