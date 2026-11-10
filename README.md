# FileSpin plugin for coding agents

Build with [FileSpin](https://filespin.io) from Claude Code, Codex or Cursor.
The plugin adds three things:

- **The FileSpin MCP server** (`https://mcp.filespin.io/mcp`). It has docs tools
  for the public developer docs and API reference (`search_developer_docs`,
  `search_api_operations`, `get_api_operation`, `get_doc_page`), plus tools that
  work on your own FileSpin account (search assets, read schemas, thumbnails and
  more). You sign in with your FileSpin account through OAuth the first time a
  tool is used.
- **The `filespin-developer` skill.** It tells the agent to look up FileSpin
  endpoints, parameters and URL syntax with those tools, never write them from
  memory, cite the docs page, keep API keys out of code, and use your account's
  real schema fields and CDN host in examples.
- **The `filespin` skill.** For working with your media library: find
  images, videos and documents, save files and the things your agent made to
  FileSpin with names, metadata, tags and a collection, prepare social sizes,
  formats and video renditions, and create review and share links. It also
  uploads files or folders from your computer ("upload ~/Desktop/shoot to
  FileSpin"): each file goes straight from your disk to your FileSpin storage
  over a short-lived signed URL. Hidden files, keys and credentials are
  skipped unless you name them.

## Install

### Claude Code

Run these one at a time inside Claude Code. Paste each command on its own:
pasting both at once puts them into a single prompt, which Claude Code
rejects as an invalid marketplace source.

1. Add the marketplace:

   ```
   /plugin marketplace add FileSpin-io/filespin-agent-plugin
   ```

2. Install the plugin:

   ```
   /plugin install filespin@filespin
   ```

3. Run `/mcp`, choose `filespin`, and sign in.

Or install from your terminal, outside Claude Code:

```bash
claude plugin marketplace add FileSpin-io/filespin-agent-plugin
claude plugin install filespin@filespin
```

### Codex

```
codex plugin marketplace add FileSpin-io/filespin-agent-plugin
```

Then run `codex`, open `/plugins`, install **filespin**, and start a new
session.

### Cursor

Cursor reads this repository as an [Agent Plugin](https://agent-plugins.org)
(`plugin.json`, `mcp.json` and `skills/` at the root). Until it's listed in the
Cursor Marketplace, connect the MCP server from the
[MCP server setup page](https://developers.filespin.io/api-reference/tools/mcp-server/).

### Install the skill without the plugin

Chat apps don't install plugins, but some accept skills. Every
[release](https://github.com/FileSpin-io/filespin-agent-plugin/releases/latest)
attaches the `filespin` skill as one file:

```
https://github.com/FileSpin-io/filespin-agent-plugin/releases/latest/download/filespin.zip
```

The skill uses the FileSpin MCP server, so connect that first: see the
[MCP server setup page](https://developers.filespin.io/api-reference/tools/mcp-server/).

**Claude (web and desktop).** Skills need code execution, which you can turn
on in Settings > Capabilities. Download `filespin.zip`, go to **Customize >
Skills**, choose **+ Add > Upload skill** and pick the zip. Don't unzip it
first.

**Claude Code, without the plugin.** Unzip it into your skills folder:

```bash
unzip filespin.zip -d ~/.claude/skills/
```

**Codex, without the plugin.** Unzip it into your skills folder, then start a
new session:

```bash
unzip filespin.zip -d ~/.agents/skills/
```

The `filespin-developer` skill comes with the plugin only. It's for writing
code against the FileSpin API, so install the plugin in your coding agent to
get it.

### Any other MCP client

Add `https://mcp.filespin.io/mcp` as a remote (streamable HTTP) MCP server.
The [MCP server setup page](https://developers.filespin.io/api-reference/tools/mcp-server/)
has instructions for each client.

## What's in this repository

| Path | For |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code |
| `plugin.json`, `mcp.json` | The Agent Plugins format (Codex, Cursor) |
| `.agents/plugins/marketplace.json` | Codex marketplace |
| `skills/filespin-developer/SKILL.md` | Building with the FileSpin API, shared by every client |
| `skills/filespin/` | Working with your media library, including uploads from your computer, shared by every client. Also attached to each release as `filespin.zip` |

## Docs and support

- Developer docs: https://developers.filespin.io/
- Every docs page as markdown: https://developers.filespin.io/llms-full.txt
- Support: support@filespin.io
- Security issues: see [SECURITY.md](SECURITY.md)

## Contributing

This repository is published from FileSpin's internal source with each
release. Bug reports and suggestions are welcome as
[issues](https://github.com/FileSpin-io/filespin-agent-plugin/issues); see
[CONTRIBUTING.md](CONTRIBUTING.md). Changes are listed in
[CHANGELOG.md](CHANGELOG.md).

## License

MIT. See [LICENSE](LICENSE).
