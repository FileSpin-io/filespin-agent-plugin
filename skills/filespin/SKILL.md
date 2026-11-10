---
name: filespin
description: >-
  Use this skill whenever the user works with their FileSpin media library from
  a chat or agent: finding images, videos or documents ("find our product
  images", "show me last month's event photos"), saving files or things you
  made to FileSpin ("save the launch video to FileSpin", "upload this folder"),
  keeping captions, transcripts or cuts with an asset, replacing an asset's file,
  organising them with names, tags, metadata and collections, preparing
  outputs (social sizes, formats, watermarks, video renditions), and sharing
  for review ("create a review link for Priya", "did the client approve it?").
  Also for questions about what FileSpin can do. It uses the FileSpin MCP tools.
---

# Working with FileSpin

FileSpin is the user's digital asset management (DAM) library: their images,
videos and documents, with metadata, collections, shares and a CDN. You work
on it through the FileSpin MCP tools. If those tools are missing, FileSpin
isn't connected: point the user to
https://developers.filespin.io/api-reference/tools/mcp-server/ and stop.

Use the tools' real names and parameters as they are listed. Don't guess
values: look them up with the tool that lists them (`get_schemas`,
`list_share_templates`, `list_channel_presets`, `get_account_info`).

## Ground rules

- **Confirm before changing many assets.** Before tagging, updating metadata,
  changing public access or adding to a collection for more than 10 assets,
  say how many and what will change, and wait for a yes.
- **You can't delete files.** No tool deletes assets. If asked, say so and
  point the user to the FileSpin dashboard. Deleting a share
  (`delete_share`) is permanent, so confirm it first.
- **Metadata changes can be undone for 24 hours.** `update_metadata` and
  `tag_assets_bulk` return a `rollback_op_id`. Keep it, and offer
  `rollback_metadata_update` if the user changes their mind.
- **Show, don't describe.** Show images with markdown, using URLs from
  `get_thumbnail`: `![name](url)`. Don't build HTML pages to show assets or
  galleries. For a gallery someone else will see, make a share.
- **Don't invent a person's email.** Find teammates with `find_group_member`;
  for "me", use `get_account_info`. Ask for anyone else's address.
- **What did you do?** Every action you take is listed in the FileSpin
  dashboard under **Agent activity**. **Profile > Connected apps** shows the
  AI apps signed in to the account and can sign them out. Point the user
  there when they ask what you did or how to disconnect you.
- **Questions about FileSpin itself** (what it can do, limits, formats,
  security): call `search_developer_docs` and answer from what it returns,
  with the link. If it finds nothing, say the docs don't cover it.

## Find assets

- `search_assets` with `keyword`, `content_type`, `date_range` or
  `file_name`. To search custom fields ("approved", "Spring 2026 campaign"),
  call `get_schemas` first and pass `schema_criteria` with the real field
  names and values.
- Inside one collection, use `search_within_collection`. Find collections with
  `search_collections`.
- `get_asset_info` gives one asset's details, metadata and existing formats.
- Show results as a short list with names and a thumbnail each
  (`get_thumbnail`, size `small` or `medium`). Offer the next page rather than
  listing hundreds.

## Save files to FileSpin

Where the file is decides how it gets there:

| The file is… | Do this |
|---|---|
| On the user's computer, and you have a shell that can read it | Upload it straight from disk. Follow [references/upload.md](references/upload.md). |
| At a public link (https) | `import_from_url` |
| Somewhere you can't reach, or the user would rather pick the files | Give them an upload page: `create_upload_link`, then `get_upload_link_status` when they say they're done. See [references/upload.md](references/upload.md). |
| Something you made (a report, a CSV) | Write it to a file, then upload that file. |
| A file that belongs with an asset already in FileSpin (captions or a transcript for a video, a cut made in another tool, a print file) | Add it to that asset as a format. See "Keep files with an asset" below. |

If an upload fails because your environment can't reach the storage address,
switch to the upload page and say why.

Then organise what was saved:

- **Name.** The file name becomes the asset name. If the file is called
  `final_v3 (2).mp4`, suggest a clearer name before uploading.
- **Metadata and tags.** Use the asset's schema fields (`get_schemas`). Add
  what the user told you or what's clear from the request. Ask rather than
  guess. Use `update_metadata` for one asset, `tag_assets_bulk` for several.
- **Collection.** If the user names one, find it with `search_collections` or
  make it with `create_collection`, then pass `collection_id` when completing
  the upload, or use `update_collection_assets`.

## Save a finished render and send it for review

For "save the launch video to FileSpin and send it to Priya for review":

1. **Upload the render** as above.
2. **Add metadata** the user gave or the project makes clear: product,
   launch or campaign, version. Ask if any of it is unclear.
3. **Link it to its sources.** If the images, clips or footage it was made from
   are in FileSpin, link each one with `create_asset_relationship`:
   `asset_id` the source, `target_asset_id` the render, `relationship_type`
   `parent_child`. People can then find the render from its sources and the
   sources from the render.
4. **Replacing an earlier render?** Ask whether this render replaces one
   already in FileSpin, and confirm which asset before you change it.
   Replace that asset's file instead of uploading a new asset: it keeps its
   ID, links and metadata. From the computer, use `create_replace_upload_url`
   then `complete_replace_upload` (see
   [references/upload.md](references/upload.md)); from a public link, use
   `replace_asset`. If the result says `pending_approval`, tell the user the
   new version is waiting for approval and the asset shows the old file until
   someone releases it. **A replace removes the formats added to the asset**
   (captions, a transcript, a cut), because they belong to the old file:
   check `get_asset_info` before replacing, tell the user which ones will go,
   and once the new file is live, add back the ones that still apply (for
   example new captions for the new cut). Without a shell, you can't replace from the computer:
   upload it as a new asset, link it to the earlier one (`relationship_type`
   `related`, `subtype` `new_version`) and say the earlier links still show
   the old file.
5. **Create a review share** with `create_share`: the render's `file_ids`, a
   clear `title`, `allow_approval` and `allow_feedback` set to true, and the
   reviewer in `email_to`. Show the `share_url`.
6. **Later, report the review.** When asked "did Priya approve it?", call
   `get_share_approvals` (find the share with `search_shares` if needed) and
   give each asset's decision and comments.

Making or editing the video or image stays with the user's own tools. You
store it, organise it and get it reviewed.

## Keep files with an asset

Captions, a transcript, a cut made in another tool or a hand-made print file
belong with the asset they were made for, not as separate assets. Add each one
to that asset as a named format:

- From the computer: `create_conversion_upload_url` with the asset, a format
  name and the file, then `complete_conversion_upload`. See
  [references/upload.md](references/upload.md).
- Name formats plainly: lowercase letters, digits, `-` and `_`, up to 30
  characters, for example `captions`, `captions-fr`, `transcript`,
  `cut-30s`. A format with the same name is replaced, so check
  `get_asset_info` first and ask before overwriting one.
- Replacing the asset's file later removes the formats added to it: they
  belong to that file. Add them again for the new file if they still apply.
- Formats are private unless the user wants a permanent public link (for
  example captions for a public video): then pass `public` true.
- FileSpin stores the file as given. It doesn't write captions or
  transcripts: if the user wants them made, that's their own tool's job, or
  yours if you can make the file.
- To make a format from the asset itself, use `create_video_clip`,
  `transcode_video` or `print_export` instead.

## Prepare outputs

- **Social sizes and watermarks:** `get_branded_image_urls` with `platforms`,
  and `text_watermark` or `image_watermark`.
- **Tagged links for a campaign:** `list_channel_presets`, then
  `generate_campaign_links`.
- **Any size or format:** `get_thumbnail` with a size, crop and `format`.
- **Video renditions:** `transcode_video` (for example `720p-video`, `HLS`).
  These are public by default. Pass on the notice the tool returns, and offer
  to keep them private.
- **Clips:** `create_video_clip` with `start` and `length`.
- **Print (CMYK):** `print_export`, when the account has it
  (`get_account_info`).
- **Downloads:** `get_asset_download_url` for one file,
  `get_collection_download_url` for a collection as a zip.

## Share and notify

- **Review or client links:** `create_share`. Use `list_share_templates` when
  the user mentions a template or brand look. Always show the `share_url`.
- **Change a share:** `update_share`. Its `download_options` replaces the old
  ones, so include those you want to keep.
- **Who viewed or downloaded it:** `get_share_analytics`.
- **Tell someone:** `send_notification` with a subject and a short body.
  Recipients must be people the user named or found with `find_group_member`.
