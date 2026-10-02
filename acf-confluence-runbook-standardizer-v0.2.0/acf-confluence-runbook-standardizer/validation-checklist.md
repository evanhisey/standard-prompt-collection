# Runbook Validation Checklist

Use this checklist before presenting a final draft and again after any Confluence write.

## Structure

- [ ] Page has a clear runbook title.
- [ ] `Brief` exists and appears before Prerequisites.
- [ ] `Prerequisites` exists and appears before Procedure.
- [ ] `Procedure` exists and contains the operator workflow.
- [ ] `Version` presentation is preserved from the template when applicable.
- [ ] No duplicate `Process` section was introduced solely because the source template instructions used that word.

## Brief

- [ ] Brief states what the runbook accomplishes.
- [ ] Brief is concise and does not contain the detailed procedure.
- [ ] Scope/boundaries are included only when supported by the source.

## Prerequisites

- [ ] Required access/permissions are listed.
- [ ] Required software/tools are listed when supported.
- [ ] Required connectivity/environment state is listed when supported.
- [ ] Template example prerequisites were not left behind as if they apply to the target procedure.
- [ ] Missing prerequisites are flagged instead of invented.

## Procedure

- [ ] Main operator actions are numbered and in execution order.
- [ ] Nested details are subordinate to the correct step.
- [ ] Commands and configuration values match the source exactly unless an authorized technical change was requested.
- [ ] Screenshots/images, if present, support the nearby step.
- [ ] Warnings appear before the action they govern.
- [ ] Expected results are not invented.
- [ ] The procedure does not silently cross environments/accounts or authorization boundaries.

## Preservation

- [ ] Existing technical meaning is preserved.
- [ ] URLs, hostnames, identifiers, and ticket references were not accidentally altered.
- [ ] Security warnings and required permissions were preserved.
- [ ] Confluence macros/layout outside the intended edit were preserved.
- [ ] Version/history presentation was not manually fabricated.
- [ ] If the page uses panel macros, only the approved panel body changed.
- [ ] The Version panel and nested `change-history` macro were preserved unchanged unless a separate template change was explicitly approved.
- [ ] The `change-history` macro retained its existing `limit` parameter, or a missing `limit` parameter was added with value `3`.
- [ ] Storage updates did not introduce leading BOM, mojibake, or other visible garbage text before the first Confluence element.

## Publishing

- [ ] The user explicitly requested the write/publish action.
- [ ] Destination page, space, parent, and title are unambiguous.
- [ ] For new pages, the destination parent was retrieved before writing.
- [ ] For new pages, existing children under the destination parent were checked before writing when supported.
- [ ] For new pages, no same-title child existed unless the user explicitly approved updating that exact existing page ID.
- [ ] Current page metadata/version was retrieved before the write when updating an existing page.
- [ ] A meaningful version comment was supplied when supported.
- [ ] The page was retrieved again after the write.
- [ ] Post-write rendered content has no visible `ï»¿`, `Ã¯Â»Â¿`, or similar encoded BOM text before the first panel or heading.
- [ ] Post-write page ID, title, space, and parent match the approved destination or target.
- [ ] Parent children were checked after a create/copy operation when supported.
- [ ] Post-write content matches the intended standardized draft.
- [ ] Local scratch storage files, temporary `content_file` inputs, and generated page body files used for the write were removed, unless the user explicitly requested audit retention.
- [ ] Any retained scratch or generated write files were reported with exact path and reason.
- [ ] Human technical review is still required before treating the runbook as approved.

## Writer-Test Controls

- [ ] The read-only reader profile remained unchanged.
- [ ] The active writer profile is the approved `acfConfluenceWriterTest` profile.
- [ ] The test destination space and parent page were approved before any write.
- [ ] The destination parent and current children were retrieved before the copy when supported.
- [ ] The approved new title did not already exist under the destination parent, unless updating that exact existing page ID was explicitly approved.
- [ ] The source template page `115220454` was retrieved before copying.
- [ ] Raw Confluence storage was inspected when macro/layout preservation mattered.
- [ ] Panel-backed sections were not treated as normal headings.
- [ ] For panel-backed sections, the storage-aware panel update path isolated exactly one approved panel body before writing.
- [ ] The proposed destination, title, content, and exact write tool were shown before writing.
- [ ] Explicit human approval was received before each write.
- [ ] The first write copied the template rather than editing an existing operational page.
- [ ] Only the new test copy was populated.
- [ ] The copied page ID, title, space, and parent matched the approved destination.
- [ ] The destination parent children were checked after copy to confirm exactly the expected new page was created.
- [ ] `confluence_update_page_section` was used only for true headings; panel-backed sections used the reviewed storage-aware panel workflow.
- [ ] Any use of `confluence_update_page` was explicitly approved for a panel-body update and validated with a panel-only before/after summary.
- [ ] No full-page replacement, delete, move, restriction, comment, attachment, or Jira write tool was used.
- [ ] Page history/version and diff were checked when available.
- [ ] The Version panel `change-history` macro still uses the intended displayed-version `limit` after the write.
- [ ] Local scratch storage files, temporary `content_file` inputs, and generated page body files used for the write were removed, unless the user explicitly requested audit retention.
- [ ] A human inspected the rendered Confluence page in the browser.

## Image Transfer Diagnostics

- [ ] Image diagnostics used a separate read-only profile.
- [ ] Source page attachments were listed before any upload test.
- [ ] Copied-page attachments were listed and compared with source attachments.
- [ ] Source and copied-page image references were compared in raw storage.
- [ ] Source images were tested with read-only image/download tools when available.
- [ ] Existing-page updates preserved existing Confluence-hosted image references and attachments without requiring local export.
- [ ] Existing-page updates retrieved metadata, attachments, page images, and raw storage before and after the write.
- [ ] Existing-page update summaries explicitly stated whether image storage changed.
- [ ] Missing copied-page attachments were distinguished from malformed page storage.
- [ ] No attachment upload/delete tool was enabled or used unless separately approved.

## Image Upload Test Controls

- [ ] Image upload testing used a separate `acfConfluenceImageUploadTest` profile.
- [ ] Local-file upload was used as the primary path unless a fallback was explicitly approved.
- [ ] Target page ID, title, space, current version, current attachments, and current image references were reverified before upload.
- [ ] Source template images were verified when source-hosted image references were involved.
- [ ] The user designated the local image source directory or exact local image file before upload.
- [ ] Images came from that user-designated location and were copied into a page-specific staging directory before upload when staging was needed.
- [ ] Filenames, extensions, file sizes, and count were verified before upload.
- [ ] Image signatures were verified before upload.
- [ ] The exact destination page ID, source path, staging directory when used, upload filenames, and file sizes were shown before upload.
- [ ] Explicit approval named `acfConfluenceImageUploadTest` and the upload tool before upload.
- [ ] Uploaded filenames matched the approved local files and target image references when applicable.
- [ ] Any page body update to append or reference the uploaded image was separately approved.
- [ ] Existing image references remained intact after the upload and page-body update.
- [ ] Staged files were removed after validation or explicitly retained for audit.
- [ ] No attachment delete tool was enabled or used.
