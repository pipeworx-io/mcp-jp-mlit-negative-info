# @pipeworx/jp-mlit-negative-info

Administrative dispositions against licensed businesses in Japan, as the Ministry of Land, Infrastructure, Transport and Tourism (国土交通省) lists them on its negative-information search site (ネガティブ情報等検索サイト): instructions (指示), business suspensions (営業停止), licence revocations (許可取消) and similar, against construction businesses and real-estate brokers.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_mlit_negative_info_search(sector?, query?, from?, to?, page?)` — `sector` is `construction` (建設業者, default) or `real_estate_broker` (宅地建物取引業者). `query` is the business name in Japanese (registered name; the 株式会社 prefix is optional). `from`/`to` are months as `2025-10` (2015 onward). Ten rows a page, newest first, with the total match count; `page` fetches the next ten. Each row: the firm, its corporate number (法人番号), head-office address, the disposition date, the authority that issued it (a regional bureau or a prefecture), the action and the record link. With neither a name nor a period the last three months are listed and the answer says so. Only firms with a corporate number are returned: a sole proprietor has none, so such rows are dropped and counted in `skipped_possible_individuals`. Zero matches states the list and period (`empty_reason: no_match`); a page past the end is `page_out_of_range`.

## Auth

Keyless.

## Data sources

- Ministry of Land, Infrastructure, Transport and Tourism (国土交通省), negative-information search site: read live per call, one request a page; nothing is stored. `data_as_of` is the newest disposition date on the page read.

Traps worth knowing: the list is of dispositions, so no match means no listed disposition, not a clean record. The site matches the registered name; a trading name may find nothing, so try the formal name. Other business fields on the site (condominium management, surveying, geological surveys, architects and so on) are not served by this tool. Rows name the disposing authority, which for a disposition by a prefecture is the prefecture.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-mlit-negative-info": {
      "url": "https://gateway.pipeworx.io/jp-mlit-negative-info/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-mlit-negative-info/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_mlit_negative_info_search \
  -H 'Content-Type: application/json' \
  -d '{"sector":"construction","query":"大林組"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_mlit_negative_info_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-mlit-negative-info` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-mlit-negative-info/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-mlit-negative-info": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-mlit-negative-info"]
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
ask_pipeworx({ question: "your question about Jp Mlit Negative Info data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
