# Confluence MCP Workflow

## Expected MCP implementation

This skill is designed around the `mcp-atlassian` Confluence tools. Tool availability depends on the MCP server configuration.

Expected read tools:

- `confluence_search`
- `confluence_get_page`
- `confluence_get_page_children`
- `confluence_get_page_history`
- `confluence_get_page_diff`
- `confluence_get_page_restrictions` when enabled

Expected writer-test tools used by this workflow:

- `confluence_copy_page`
- `confluence_update_page_section`
- `confluence_update_page` only for the reviewed storage-aware panel update path

The copy-only and heading-section writer-test workflow does not require full-page replacement, delete, move, restriction-change, comment, attachment, or Jira write tools. Panel-macro templates require a reviewed storage-aware use of `confluence_update_page` because `confluence_update_page_section` targets headings, not panel macro titles.

## Recommended read-only profile

Normal documentation review should use `READ_ONLY_MODE=true`.

With read-only mode enabled, the skill can retrieve the template and target pages, evaluate them, and produce a standardized draft without changing Confluence.

## Recommended writer-test profile

If the organization authorizes writer tests, use a separate controlled MCP profile named `acfConfluenceWriterTest`. Keep the working read-only reader profile unchanged.

The first writer-test allowlist is:

```text
confluence_search,
confluence_get_page,
confluence_copy_page,
confluence_update_page_section,
confluence_get_page_history,
confluence_get_page_diff
```

For storage-aware panel updates, the reviewed allowlist may add only:

```text
confluence_update_page
```

Do not expose delete, move, restriction-change, comment-writing, attachment-writing/deleting, or Jira write tools for controlled writer testing.

See `examples/acf-confluence-writer-test.mcp.json`, `INSTALL.md`, and this workflow before enabling the writer-test profile.

If the publisher profile is scoped to one Confluence space, use `CONFLUENCE_SPACES_FILTER` after confirming the actual space key.

## Authentication

For Confluence Server/Data Center using a Personal Access Token, the MCP server expects the Confluence connection URL and `CONFLUENCE_PERSONAL_TOKEN`.

Never place the token in this skill, a repository, a prompt, a runbook, a screenshot, or generated documentation. Use the approved secret-handling mechanism configured for the MCP server.

## Exact-format retrieval

For source analysis and normal drafting, Markdown retrieval is convenient.

For exact format preservation, retrieve the template and target with:

```text
convert_to_markdown=false
```

Raw Confluence storage XHTML can preserve macros and task metadata that Markdown conversion may not reproduce exactly.

## Panel macro update workflow

Use this workflow when the live template represents Brief, Prerequisites, Procedure, or Version as Confluence panel macros rather than headings.

1. Retrieve the copied test page with `convert_to_markdown=false`.
2. Locate panel macros by `ac:structured-macro ac:name="panel"`.
3. Identify each panel by its `ac:parameter ac:name="title"` value.
4. Confirm exactly one intended panel body will change for a single-section test.
5. Replace only that panel's `ac:rich-text-body` content.
6. Preserve panel macro IDs, panel styling parameters, page layout, images, links, and all unapproved panel bodies.
7. Preserve the `Version` panel and nested `change-history` macro unchanged.
8. Generate a before/after summary that names the changed panel title and confirms protected macros are unchanged.
9. Stop for explicit approval that names `confluence_update_page` before writing.
10. After writing, retrieve raw storage and markdown, verify the changed panel, verify page location, verify the Version widget, and inspect page history/diff when available.

Do not use loose global find/replace across the full storage body. If the storage cannot be parsed or the panel match is ambiguous, stop without writing.

## Image transfer diagnostic workflow

Use a separate read-only profile for image transfer diagnostics. Do not add attachment tools to the standard writer-test profile while diagnosing copied-page images.

Recommended read-only image diagnostic tools:

```text
confluence_get_page,
confluence_get_attachments,
confluence_download_attachment,
confluence_download_content_attachments,
confluence_get_page_images
```

Diagnostic sequence:

1. Retrieve the source page and copied page with metadata.
2. Compare attachment lists for the source and copied pages.
3. Compare image references in raw storage.
4. Confirm source images are discoverable through read-only attachment tools.
5. Confirm copied-page image references target filenames that are not attached to the copied page.
6. Report whether the failure is missing copied attachments or malformed storage.

Stop before upload or delete operations. If attachment restoration is required, create a separately approved attachment-write profile and limit it to the target test page.

## Portable image handling strategy

Use two separate image paths.

For existing Confluence page updates, preserve the page's existing Confluence-hosted image references and attachments in place. Retrieve raw storage before updating, keep image elements unchanged unless the user explicitly approves an image change, and validate after the update that the same image references and attachments remain present. This is the Confluence-to-Confluence image path: it does not require exporting images to the local filesystem.

For new Confluence pages or new images added to a page, use a user-designated local image directory. The user or source process places the required screenshots, diagrams, exported images, or generated files in that directory. The MCP workflow validates the local files, then uploads them with `confluence_upload_attachment` or `confluence_upload_attachments` after explicit approval.

Stock `mcp-atlassian 0.23.1` can retrieve images through `confluence_download_attachment`, `confluence_download_content_attachments`, and `confluence_get_page_images`, but those tools return inline MCP resources rather than local files. That is acceptable for this workflow because Confluence-to-Confluence updates preserve existing image storage, and new-page image uploads use a user-designated local directory. If fully automated export from Confluence to local staging is needed later, deliver it through an approved upstream `mcp-atlassian` release or an approved internally packaged MCP fork. Do not require end users to patch their local `uvx` package cache.

Existing-page image preservation checks:

1. Retrieve target page metadata, attachments, and raw storage before the update.
2. Record image filenames, attachment IDs when available, and storage references.
3. Preserve image tags, links, macros, and attachment references while editing text sections.
4. Re-retrieve the page after update.
5. Confirm the same existing image references still resolve and no attachment upload/delete occurred unless separately approved.

New-page local image upload checks:

1. Require the user to designate the local image directory.
2. Copy only approved image files into a page-specific staging directory when needed.
3. Verify filenames, extensions, byte sizes, and image signatures before upload.
4. Show the destination page ID, directory, filenames, and upload tool.
5. Stop for explicit approval before calling an upload tool.

## Existing-page Confluence image preservation workflow

Use this workflow when updating an existing Confluence page that already contains images. This path uses only Confluence page retrieval/update behavior and does not require local image downloads.

Required tools:

```text
confluence_get_page,
confluence_get_attachments,
confluence_get_page_images,
confluence_update_page
```

Use `confluence_update_page` only when a reviewed storage-aware update is required. If a smaller section-level update can preserve the existing image storage safely, prefer that smaller write.

Required sequence:

1. Retrieve the target page with metadata.
2. Retrieve target attachments.
3. Retrieve target page images when available.
4. Retrieve target raw storage with `convert_to_markdown=false`.
5. Record existing image filenames, attachment IDs when available, image URLs, `ri:attachment` references, and any surrounding macros/layout storage.
6. Prepare the text or panel-body update while preserving every existing image element exactly.
7. Generate a before/after summary that states image storage is unchanged.
8. Stop for explicit approval naming the target page ID and write tool.
9. Apply only the approved text update.
10. Retrieve raw storage, attachments, and page images again.
11. Confirm the same image references and attachments remain present.
12. Confirm no attachment upload or delete occurred unless separately approved.

Stop if raw storage cannot be inspected, if image references would need to be rewritten, or if the update would require replacing unparsed storage around images.

## Image upload test workflow

Use a separate profile named `acfConfluenceImageUploadTest` for image upload tests. Do not add upload tools to `acf_confluence`, `acfConfluenceImageReadTest`, or `acfConfluenceWriterTest`.

Use local-file upload as the primary path for new page images. This best supports runbooks that need locally captured screenshots, exported diagrams, or transient generated images uploaded to Confluence. Use `content_base64` only as a fallback for environments where the MCP server cannot read local files.

The user must designate the local image source directory or exact local image file before upload testing. Stage upload files in a page-specific temporary directory, for example `.image-upload-staging/<page-id>/`, when staging is needed before invoking an upload tool. Do not upload directly from broad locations such as a user Downloads folder or repository root.

Minimum upload-test tools:

```text
confluence_get_page,
confluence_get_attachments,
confluence_download_attachment,
confluence_download_content_attachments,
confluence_get_page_images,
confluence_upload_attachment,
confluence_upload_attachments
```

After the upload-test profile is added or changed, re-confirm the target and source evidence before upload:

1. Confirm source template page `115220454` when source-hosted images are involved.
2. Confirm approved target page ID, title, location, current version, and attachment list.
3. Confirm target page storage has the expected existing image references.
4. Ask the user for the local image source directory or exact file path, or use the already-designated path.
5. Copy only the approved image files from that location into the page-specific staging directory when staging is needed.
6. Verify filenames, file sizes, extensions, count, and image signatures.
7. Show destination page ID, source path, staging directory when used, filenames, and file sizes.
8. Stop for explicit approval naming `acfConfluenceImageUploadTest` and the upload tool.
9. Upload only the approved image files.
10. Append or update page storage only when separately approved.
11. Re-run image diagnostics and page retrieval.
12. Confirm existing image references remain intact and the uploaded local image resolves.
13. Remove staged files after validation, or report that they were retained for audit.

Never expose `confluence_delete_attachment` during the initial image upload test.

## New-runbook workflow

1. Get template page `115220454` with metadata and raw storage.
2. Resolve the destination space key, parent page ID, and new title.
3. Retrieve the destination parent page and current children when `confluence_get_page_children` or search can provide them.
4. Confirm the proposed page title is not already present under the destination parent unless the user explicitly approves updating that exact existing page ID.
5. Show the proposed destination space, parent page title/ID, new title, and pre-write child-page check.
6. Stop and wait for explicit human approval.
7. Copy the template using `confluence_copy_page`.
8. Retrieve the new copy and confirm the source template is unchanged.
9. Retrieve the destination parent children again and confirm exactly the expected new page exists under the approved parent.
10. Show the proposed Brief, Prerequisites, and Procedure content.
11. Stop and wait for explicit human approval before updates.
12. Update Brief, Prerequisites, and Procedure with the smallest safe writes. Use heading-section updates only for true headings; use the reviewed panel macro workflow for panel-backed sections.
13. Supply a meaningful version comment on update operations when supported.
14. Retrieve the result, inspect history/diff when available, verify the page location again, and validate it.

## Existing-page workflow

1. Get the page with metadata.
2. Get raw storage if the page contains macros/layouts or exact preservation matters.
3. Compare it with live template `115220454`.
4. Prepare a draft and change summary.
5. Confirm the exact target page ID, title, space, parent, and current version before requesting approval.
6. If the user authorizes an in-place update, prefer `confluence_update_page_section` for discrete sections.
7. Use a full `confluence_update_page` only when necessary and only after ensuring the full stored content can be safely preserved.
8. Re-fetch and inspect the updated page.
9. Confirm the page ID, title, space, and parent still match the approved target.
10. Use page history/diff tools when available to verify the change.

## Version comments

When writing, use a concise version comment that explains the documentation change, for example:

```text
Standardized runbook structure against NGSC Operation RunBook Template; preserved technical procedure content.
```

Do not claim that a version comment provides approval or technical validation.

## Writer-test stop conditions

Stop without writing when:

- the approved test parent is missing or ambiguous;
- the parent-child check cannot confirm the approved destination for a new page;
- a page with the proposed title already exists under the destination parent and the user has not explicitly approved updating that exact page ID;
- post-write retrieval shows the page was created or updated under an unexpected parent, space, title, or page ID;
- the active write profile is not `acfConfluenceWriterTest` or another explicitly approved writer-test profile;
- the user has not approved the exact destination, title, content, and write tool;
- the live template cannot be retrieved;
- raw storage is needed to preserve macros but cannot be inspected;
- section boundaries are not safe for `confluence_update_page_section`;
- panel macro storage cannot be parsed or exactly one intended panel body cannot be isolated;
- the Version panel or nested `change-history` macro would be modified by an unapproved content update;
- the requested change would require `confluence_update_page` that has not been separately reviewed and approved;
- a tool outside the approved writer-test allowlist is required.

## Failure behavior

If authentication fails, MCP is unavailable, the template cannot be retrieved, the destination is ambiguous, or the write tools are disabled:

- do not guess;
- do not weaken TLS/security settings automatically;
- do not request the PAT in chat;
- remain in draft mode when possible;
- report the specific missing capability or input needed.

## End-user installation and diagnostics

For complete workstation setup instructions, use the root `INSTALL.md` document. It documents the validated VS Code Chat + Credal path using the global VS Code user `mcp.json` file and a read-only `acf_confluence` server definition.

When installation or startup fails, use `diagnostic-improvement-prompt.md`. Troubleshoot in this order:

1. operating system/shell;
2. VS Code Chat and Credal model availability;
3. `uv` / `uvx` PATH;
4. `uvx mcp-atlassian --help` startup;
5. environment-variable presence/propagation;
6. global VS Code MCP configuration;
7. network/TLS;
8. Confluence authentication/authorization;
9. read-only page retrieval;
10. skill discovery.

Do not request or display the PAT during diagnostics, and do not disable TLS verification as a troubleshooting shortcut.
