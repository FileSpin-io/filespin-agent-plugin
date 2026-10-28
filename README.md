# FileSpin plugin for coding agents

Build with [FileSpin](https://filespin.io) from Claude Code, Codex or Cursor.
The plugin adds two things:

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

## Install

### Claude Code

```
/plugin marketplace add FileSpin-io/filespin-agent-plugin
/plugin install filespin@filespin
```

Then run `/mcp`, choose `filespin`, and sign in.

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
| `skills/filespin-developer/SKILL.md` | The skill, shared by every client |

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
