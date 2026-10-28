---
name: filespin-developer
description: >-
  Use this skill whenever you write, review or debug code that uses FileSpin:
  calls to the FileSpin REST API (upload, search, asset data and metadata,
  collections, conversions, video transcodes, addons, webhooks), FileSpin
  CDN / on-demand image (ODI) URLs such as resize, crop, watermark or format
  parameters, the File Picker or Web Connector widgets, or FileSpin video
  playback and HLS. Also use it for any "how do I…", "what's the URL for…" or
  "can FileSpin…" question about FileSpin's API, CDN, widgets, video, webhooks,
  addons or integrations, even when no code is asked for, and when the user asks
  where something is documented or why a FileSpin request fails.
---

# Building with FileSpin

FileSpin is a digital asset management platform with a REST API, an on-demand
image CDN, video transcoding and streaming, and embeddable upload widgets. The
public docs are at https://developers.filespin.io/.

**Never write a FileSpin endpoint, parameter, header, URL option or response
field from memory.** FileSpin's syntax is specific (for example a face crop is
`crop=400,400,face`, and there are no `w`/`h` parameters), and guessed syntax
looks plausible but fails in production. Look it up with the FileSpin MCP tools
every time, then write the code.

## The lookup loop

The FileSpin MCP server (`filespin`, installed with this plugin) has four
read-only docs tools. Use them in this order:

1. **`search_api_operations`**: find the endpoint for a task ("upload a
   file", "update asset metadata", "create a collection").
2. **`get_api_operation`**: get that operation's exact method, path,
   parameters, request body, responses and auth. Call it before writing any
   request, even one you think you know.
3. **`search_developer_docs`**: for anything that isn't a single endpoint:
   CDN/ODI URL syntax, video playback, widgets, webhooks, limits and how-to
   guides. Results can include API operations too (`kind: "api_operation"`).
4. **`get_doc_page`**: read a whole page when a search snippet isn't enough.

Then:

- Use the exact path, parameter names and body fields the tools return.
- **Give the whole documented answer, not just the main syntax.** Before you
  reply, re-read the page or operation you used and carry over what it states
  about each parameter and field you mention: the allowed values, the default,
  limits and caps, required versus optional, the response fields the caller
  will read, and any warning or counter-intuitive behaviour. If the page names
  sibling endpoints or alternatives for the same task (a different variant for
  uploaded files, a bulk version, a recommended alternative to a slower or
  throttled endpoint), mention them too. Take these from the tool output;
  don't fill them in from memory.
- Cite the `doc_url` (or page `url`) you relied on, in a code comment or in
  your reply.
- If the docs don't cover something, say so and ask, or point the user to
  support@filespin.io. Don't fill the gap with a guess.

If the docs tools aren't available, the MCP server isn't connected. See
https://developers.filespin.io/api-reference/tools/mcp-server/ for setup, or
read the same docs as plain markdown: every page has a `.md` copy (for example
https://developers.filespin.io/image-transformations/crop.md), and
https://developers.filespin.io/llms-full.txt has them all.

## How-to questions: answer with the API, not the action tools

The same MCP server also has action tools that work on the user's account
(for example `create_video_clip`, `upload_asset`, `update_metadata`). Those
are for doing something in the account when the user asks you to. They are not the answer to "how do I…" questions.

When the user asks how to do something with FileSpin, answer with the
documented REST API or URL: look it up with `search_api_operations` and
`get_api_operation` (or the docs tools for URLs, widgets and guides), give the
request and cite the page. An action tool's own description is not
documentation; don't quote its defaults or field names as API facts. You can
then offer to run the matching action tool for them.

## Two kinds of auth: keep them apart

- **Your MCP session** signs in with OAuth (the browser opens on first use).
  This is only for you, the agent, to read the docs and the user's account.
- **The developer's application code** authenticates on its own, usually with
  an API key in the `X-FileSpin-Api-Key` header (server side only), or a JWT.
  Confirm the scheme with `get_api_operation` and
  https://developers.filespin.io/authentication/overview/.
  - Read the key from configuration, such as `os.environ["FILESPIN_API_KEY"]`
    or `process.env.FILESPIN_API_KEY`. Never paste a real key into code, and
    never put an API key in browser or mobile code.
  - API keys are created in the FileSpin Dashboard. The docs say where.

## Use the user's real account, not made-up values

The same MCP server can read the signed-in account. Use it so examples work
the first time:

- **`get_account_info`**: the account's role, plan and addons. Check that an
  addon the code depends on (for example `ON_DEMAND_IMAGE` for on-demand image
  URLs, or `REMOVE_BACKGROUND`) is enabled before building on it.
- **`get_thumbnail`** (for a real asset): returns a public CDN URL. Take the
  account's CDN host from it for image and video URLs, instead of a
  placeholder domain.
- **`get_schemas`**: the account's custom metadata schemas and field names.
  Use these exact field names when code sets or reads metadata.
- **`search_assets`**: real asset IDs, for example code and tests.

These read real data. Ask before using a write tool (upload, update metadata,
delete) against the user's account while developing.

## Where the core patterns are documented

Look these up with the tools above. The pages are listed so you can cite them.

| Task | Start here |
|---|---|
| API overview, response codes, rate limits | https://developers.filespin.io/api/overview/ |
| Asset data format (what an asset's JSON looks like) | https://developers.filespin.io/api/asset-data-format/ |
| Upload and asset metadata | `search_api_operations` ("upload", "update asset data") |
| Search | https://developers.filespin.io/api-reference/asset-search/advanced-search-options/ |
| On-demand image URLs (resize, crop, format, watermark…) | https://developers.filespin.io/image-transformations/ |
| Video transcoding and HLS playback | https://developers.filespin.io/video-streaming/ |
| Upload widget for web pages | https://developers.filespin.io/file-picker-widget/ |
| Webhooks | https://developers.filespin.io/api-reference/tools/webhooks/ |
| Automating a processing pipeline | https://developers.filespin.io/automated-media-pipeline/ |

## Before you finish

- Every FileSpin request in your code came from `get_api_operation` or a docs
  page, and you cited it.
- No real API key is in the code. The key is read from configuration.
- The CDN host, schema fields and asset IDs came from the user's account, not
  from invented examples.
- Anything undocumented is flagged to the user rather than guessed.
