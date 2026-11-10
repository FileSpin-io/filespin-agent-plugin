# Changelog

All notable changes to the FileSpin plugin are listed here. Versions follow
[Semantic Versioning](https://semver.org/).

## 0.3.0 - 2026-11-09

- New `filespin` skill for working with your media library from your agent:
  find images, videos and documents and see thumbnails; save files and the
  things your agent made to FileSpin with clear names, metadata, tags and a
  collection; prepare social sizes, formats, watermarks and video renditions;
  and create review and share links, notify a teammate, and report back what
  the reviewer decided. It confirms before changing many assets, says plainly
  that it can't delete files, and points you to Agent activity and Connected
  apps in the dashboard when you ask what it did.
- It includes a workflow for a finished render ("save the launch video to
  FileSpin and send it to Priya for review"): upload it, add its metadata,
  link it to the source assets it was made from, and send it for review.
- Uploads from your computer: ask your agent to upload files or folders
  ("upload ~/Desktop/shoot to FileSpin"). Each file goes straight from your
  disk to your FileSpin storage over a short-lived signed URL, using the MCP
  server's `create_upload_url`, `complete_upload` and `get_upload_status`
  tools. Hidden files, keys and credentials are skipped unless you name
  them, and each file keeps its folder path so you can find it in FileSpin by
  folder name. When your agent can't see your files, or you'd rather pick
  them yourself, it gives you a FileSpin upload page instead.
- From the computer, the agent can also put a file into an asset that is
  already in FileSpin: add captions, a transcript or a cut to it as a named
  format, or replace its file while it keeps its ID, links and metadata (a
  replacement may wait for approval when your account uses content
  versioning). Replacing a file removes the formats added for the old one,
  so the agent tells you which and offers to add them again. It uses the MCP server's `create_conversion_upload_url`,
  `complete_conversion_upload`, `create_replace_upload_url` and
  `complete_replace_upload` tools.
- The skill is also a download for chat apps that accept skills: each release
  attaches `filespin.zip`, a single `filespin/` skill folder. See "Install the
  skill without the plugin" in the README.
- The Claude Code install steps now show one command per step, so each can be
  pasted on its own, plus the same install from a terminal.

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
