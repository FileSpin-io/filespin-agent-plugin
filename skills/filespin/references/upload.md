# Uploading local files to FileSpin

The FileSpin MCP server runs remotely and can't read the user's disk. You can:
your shell sends each file straight to the user's FileSpin storage over a
short-lived signed URL. The file's bytes never pass through you or the MCP
server.

You need the FileSpin MCP tools `create_upload_url`, `complete_upload` and
`get_upload_status`, and a shell. If the tools are missing, the MCP server
isn't connected or is out of date: see
https://developers.filespin.io/api-reference/tools/mcp-server/.

Use an upload page instead (`create_upload_link`, then `get_upload_link_status`
when the user says they're done) when:

- your shell can't see the user's files (for example it runs in a cloud
  sandbox), or you have no shell;
- the user asks for an upload page, or would rather pick the files themselves.

The page needs the user to sign in to FileSpin, works only for them, and is
safe to show. Don't look for open links first: just create one.

## 1. Decide exactly which files

Upload only what the user asked for. If they haven't said which files, ask
once and offer both: tell you the paths, or an upload page to pick them in the
browser.

- For a folder, list its files first. Don't descend into hidden folders.
- Always skip hidden files and OS clutter (`.DS_Store`, `Thumbs.db`,
  `desktop.ini`, `._*`).
- Never upload keys, credentials or config secrets (`.env`, `*.pem`, `*.key`,
  `id_rsa`, `~/.ssh/`, `~/.aws/`) unless the user names that exact file.
- If the request is vague, or there are more than 20 files or more than 1 GB in
  total, show the count, total size and a few names, and ask before uploading.

## 2. Get each file's size

```bash
stat -f%z "<path>"     # macOS
stat -c%s "<path>"     # Linux
```

`wc -c < "<path>"` works on both.

## 3. Get upload URLs

Call `create_upload_url` with up to 20 files per call: `filename` (the base
name, which becomes the asset's name) and the exact `size` in bytes. Ask for
the URLs just before you upload: they expire after 15 minutes.

Each file comes back with an `asset_id` and a ready-to-run `curl` command, or
with an `error` (for example, over the plan's size limit). If the error says
the upload quota is used up, stop and tell the user.

## 4. Upload each file

For each file:

1. Compute its MD5 with the matching command from `md5_commands`.
2. Run its `curl` command with `<LOCAL_PATH>` replaced by the file's path,
   keeping the single quotes. Upload one file at a time, or a few in parallel.

Rules:

- **Never show, log or repeat the URLs.** They are credentials.
- The URL is signed for that exact size and `Content-Type`. Don't change the
  command or the file.
- If `curl` fails with 403, the URL has expired or the file changed: get a new
  URL for that file and try again.
- If `curl` fails with error 55 ("Failed sending data to the peer"), run the
  same command again with `--http1.1 -H 'Expect: 100-continue'` added after
  `curl`. The URL and the file are fine; this makes curl wait for the
  storage redirect before it sends the file.

## 5. Complete each upload

After its `curl` succeeds, call `complete_upload` with:

- `asset_id`, `size` and `checksum` (the MD5, 32 hex characters);
- `source_path`: the file's full path with your home directory written as `~`,
  for example `~/Projects/Acme/Spring 2026/hero.jpg`. The user can then find
  the asset in FileSpin by these folder names;
- `metadata` and `collection_id`, only if the user asked for them. Find the
  collection with `search_collections`, or make one with `create_collection`,
  before you start.

## 6. Confirm

Call `get_upload_status` with the batch's `asset_id`s every few seconds until
none is `queued` or `processing`. Then tell the user:

- how many files are in FileSpin, with their names;
- which failed and why, using the `explanation` given. Offer to retry them.

Don't say the upload is done until `get_upload_status` reports the files as
`ready`.

## Into an asset that already exists

The same steps put a file from the computer into an existing asset. Confirm
which asset first.

- **As a format** (captions, a transcript, a cut): `create_conversion_upload_url`
  with `asset_id`, `name` (lowercase, digits, `-`, `_`, up to 30 characters),
  `filename` and `size`; compute the MD5; run its `curl` command; then
  `complete_conversion_upload` with `asset_id`, `name`, `size`, `checksum` and
  `public` if the user wants a public link. The format shows under conversions
  in `get_asset_info` a few seconds later.
- **As a replacement** of the asset's own file: `create_replace_upload_url`
  with `asset_id`, `filename` and `size`; compute the MD5; run its `curl`
  command; then `complete_replace_upload` with `asset_id`, `size` and
  `checksum`. The asset keeps its ID, links and metadata. If the result is
  `pending_approval`, the new version waits for approval and the asset shows
  the old file until it is released. If the URL step says a version is
  already waiting, tell the user it must be released or rejected first.
  **A replace removes the formats someone added** (captions, a transcript, a
  cut), because they belong to the old file; FileSpin's own renditions are
  made again. Before replacing, list them with `get_asset_info` and tell the
  user; afterwards, add back the ones that still apply to the new file.

The rules above apply: never show the URLs, and get a new URL if one has
expired. If `complete_*` says nothing was uploaded, run the `curl` command
first; if the checksum doesn't match, recompute it for the same file.
