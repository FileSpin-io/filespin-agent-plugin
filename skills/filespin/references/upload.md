# Uploading local files to FileSpin

The FileSpin MCP server runs remotely and can't read the user's disk. You can:
one script sends each file straight to the user's FileSpin storage. The file's
bytes never pass through you or the MCP server.

You need the FileSpin MCP tools `create_upload_command` and
`get_upload_status`, and a shell. If they are missing, the MCP server isn't
connected or is out of date: see
https://developers.filespin.io/api-reference/tools/mcp-server/.

Use an upload page instead (`create_upload_link`, then `get_upload_link_status`
when the user says they're done) when:

- your shell can't see the user's files (for example it runs in a cloud
  sandbox), or you have no shell;
- your shell can't reach FileSpin at all (see step 4);
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

## 2. Get the upload command

Call `create_upload_command` once for the whole batch, just before you upload:

- `max_files`: how many files (default 50, up to 500);
- `metadata` and `collection_id`, only if the user asked for them. They apply
  to every file. Find the collection with `search_collections`, or make one
  with `create_collection`, before you start.

No sizes or checksums are needed. If the result says the upload quota is used
up, stop and tell the user.

## 3. Fill in the metadata

Always fill in metadata, even when the user only said "upload this folder"
and didn't mention it. The result's `schema` lists the fields new uploads get:
name, type, whether it is required, and the allowed `options` for choice
fields. Fill what you can from what you know locally:

- folder and file names (`~/Shoots/Spring 2026/hero-red-dress.jpg`: project
  "Spring 2026", tags "red", "dress");
- documents next to the files (a shot list, a brief, a README);
- what the user said in this conversation.

Rules:

- Fill only fields you have evidence for. Leave the rest empty; never guess or
  invent values.
- A field with `options` takes one of them exactly.
- Don't copy secrets or personal data from files into metadata.

Don't stop to ask: upload with the values you have evidence for, then show the
user a short table (file, field, value, where it came from) and offer to change
any of them with `update_metadata`. Ask **before** uploading only when:

- a value would be a guess you can't support from what you see; or
- a required field has no evidence.

Skip metadata only when the user said not to add any.

Values every file shares can go in each file's JSON, or in
`create_upload_command`'s `metadata` when the user gave them up front. A
file's own values win.

## 4. Run it

Take `scripts.sh` (macOS, Linux) or `scripts.powershell` (Windows) and replace
its last line with one line per file: the file's absolute path and that
file's values as JSON (`'{}'` only when there is nothing to fill):

```bash
fs_up "/Users/jo/Shoots/Spring 2026/hero.jpg" '{"title": "Hero shot", "tags": ["spring", "dress"]}'
fs_up "/Users/jo/Shoots/Spring 2026/team.jpg" '{"title": "Team photo"}'
```

(PowerShell: `FsUp 'C:\Users\jo\Shoots\hero.jpg' '{"title": "Hero shot"}'`.)

Run the whole script as one command, for one file or many. Use absolute
paths: FileSpin records each file's folder (home directory as `~`) so the
user can find assets by folder names they know.

Each file goes straight to FileSpin's storage. If the storage address can't be
reached, or the file is bigger than 5 GB, the script sends that file through
FileSpin's upload server instead, on its own.

**Never show, log or repeat the command or its token.** The token is a
credential. If the command has expired (`expires_at`), get a new one.

## 5. Read the results

Each file prints one line of JSON:

- `{"files": [{"id": "...", "name": "hero.jpg", ...}]}`: the asset was made.
  `"duplicate": true` means the same file was already uploaded with this
  command (for example a retried run), and `id` is that asset.
  `"warning"` means its metadata doesn't fit the schema: the file is stored;
  correct the values with `update_metadata`;
- `{"status": "ERROR", "message": "..."}`: that file was refused. The
  messages are `CONNECT_FAILED` (see below), `PUT_FAILED_<n>` (storage
  refused the file; run the script again for it), `FILE_TOO_BIG` (over the
  plan's per-file size),
  `UPLOAD_LIMIT_EXCEEDED` (quota used up: stop), `UPLOADS_PAUSED` (the
  account can't upload right now: stop and tell the user),
  `UPLOAD_TOKEN_NOT_FOUND` (the command expired: get a new one) and
  `UPLOAD_TOKEN_USED_UP` (more files than `max_files`: get another command
  for the rest).

It is safe to run the script again: files already uploaded are answered with
their asset, not uploaded twice.

If every file fails with `CONNECT_FAILED`, your shell can't reach FileSpin at
all, for example in a sandbox that only allows some hosts. Don't retry: use
`import_from_url` if the files are at public links, otherwise give the user an
upload page (`create_upload_link`) and say why.

## 6. Confirm

Call `get_upload_status` with the batch's asset ids every few seconds until
none is `queued` or `processing`. Then tell the user:

- how many files are in FileSpin, with their names;
- which failed and why, using the `explanation` given. Offer to retry them.

Don't say the upload is done until `get_upload_status` reports the files as
`ready`.

## If `create_upload_command` isn't available

Older FileSpin servers only have `create_upload_url`. Call it with each file's
`filename`, exact `size` in bytes (`wc -c < "<path>"`) and `path` (up to 20
files), run its `upload_script`, which prints `<asset_id> <md5>` per file, and
call `complete_upload` for each with `asset_id`, `size`, `checksum` (the MD5)
and `source_path` (the path with your home directory as `~`). Then step 5.
Never show the URLs; if one has expired (403), ask for a new one.

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
