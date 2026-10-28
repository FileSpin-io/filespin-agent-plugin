# Changelog

All notable changes to the FileSpin plugin are listed here. Versions follow
[Semantic Versioning](https://semver.org/).

## 0.1.1 - 2026-10-28

First public release.

- The FileSpin MCP server (`https://mcp.filespin.io/mcp`) for Claude Code,
  Codex and Cursor, with OAuth sign-in to your FileSpin account.
- The `filespin-developer` skill. It looks up FileSpin endpoints, parameters
  and URL syntax with the docs tools instead of writing them from memory,
  carries over the allowed values, defaults and limits the docs state, cites
  the page it used, and keeps API keys out of code.
- How-to questions are answered with the documented REST API or URL, not with
  the MCP server's account tools.
