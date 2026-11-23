# Changing many assets at once

Read this before a job that changes, tags, collects or shares more than a
handful of assets.

## Plan first

- Before you call any tool, send the user a short plan: how many assets
  (and how many the search matched, when it found more), what will change,
  and which tools you'll use. Then wait for a yes.
- Many apps stop a turn after about 30 tool calls. For a big job, say so in
  the plan ("145 assets in about 5 batches") and that you'll stop and ask
  before going on.
- Never page through every result applying changes without telling the user
  the total first. Read-only work (search, lists, links) can go ahead, but
  give the total and offer the next page.

## Try one first

For metadata or tags across many assets, change one asset first, show the
user the result, and go on only after they confirm. Use `tag_assets_bulk`
for the rest rather than calling `update_metadata` in a loop. Keep the
`rollback_op_id` each call returns.

## Limits

- `create_share`, `create_collection` and `update_collection_assets` take at
  most 300 assets. Confirm with the user before adding or removing more
  than 300 in a collection.
- To share everything a search matched, pass `search_result_id` (or the
  search itself) to `create_share` instead of IDs copied from one results
  page.
- If a tool answers `share_cap_exceeded` or `collection_cap_exceeded`, tell
  the user the counts and ask them to narrow it. Don't retry with a cut-down
  list.

## Real IDs only

- Asset IDs come only from a tool result in this conversation, from the
  app, or from the user. They are random: never make one up or work one out
  from another.
- When the user says "continue" after you stopped at a limit, search again
  for the IDs before any change. Don't reuse IDs you remember.
- If a result has `abort_reason` (`all_assets_unreadable` or
  `consecutive_id_failures`), stop: the IDs were wrong. Search for real ones
  before trying again.

## When something fails

- **Not found (404):** check the ID, or find the asset with `search_assets`.
  A new upload may still be processing.
- **Not allowed (403):** check the account with `get_account_info`. Some
  actions need permissions the user may have to ask an admin for.
- **Too many requests (429):** wait, then use smaller batches. If a bulk
  tool reports many 429 errors, stop, say what worked and what failed (with
  the asset IDs), and ask before retrying.
- **Server error (500):** retry once. If it fails again, tell the user and
  suggest FileSpin support.
