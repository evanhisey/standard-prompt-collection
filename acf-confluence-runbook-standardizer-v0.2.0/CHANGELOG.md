# Changelog

## 1.0.0 - Validated writer and image workflow

- Added a copy-ready `acf-confluence-runbook-standardizer/` skill directory containing only the runtime skill payload files and removed duplicate runtime files from the package root.
- Marked the writer and image handling stages as complete after human verification.
- Added post-write cleanup guidance for local scratch storage files, temporary `content_file` inputs, and generated page body files.
- Added required VS Code Local harness settings for Confluence MCP validation and writer/image workflows.
- Added a storage-write guard to prevent UTF-8 BOM/mojibake text from being published before the first Confluence element.
- Added `change-history` macro `limit` preservation guidance, with missing limits defaulting to `3` displayed versions.
- Removed the development-roadmap dependency from the skill package and consolidated future diagnostic guidance into the diagnostic prompt.
- Updated installation guidance from review draft wording to the final validated workflow.
- Added explicit VS Code Copilot skill installation instructions.
- Documented the validated two-path image model: preserve existing Confluence-hosted image references and upload user-designated local images to the approved target page.
- Removed the redundant draft-template file from the deployable package; `runbook-standard.md` remains the skill's normative runbook reference.

## 0.3.2 - Image read validation and upload-test planning

- Added the separate read-only `acfConfluenceImageReadTest` MCP profile to the initial installation path.
- Made image and attachment retrieval part of successful read-only runbook validation.
- Documented the validated image-read result for template page `115220454`: six images found and six retrieved.
- Added gated image upload test planning that keeps upload tools out of the default reader and writer-test profiles.
- Documented a minimum temporary upload-test allowlist and stop conditions for attachment restoration testing.
- Added initial `acfConfluenceImageUploadTest` profile guidance and required testing restart at Stage 2 after upload MCP development.
- Made local-file staging the primary image upload path for runbooks that use local screenshots, diagrams, exported images, or transient generated images.
- Split portable image handling into Confluence-hosted image preservation for existing-page updates and user-designated local directory upload for new pages or new images.
- Clarified that automated Confluence export into local staging is optional future MCP work and must come from an approved upstream release or packaged fork, not local user cache patching.
- Added operational checks for direct existing-page image preservation tests and user-designated local directory upload tests.

## 0.3.1 - Writer MCP installation guidance

- Added initial end-user installation guidance for the separate `acfConfluenceWriterTest` MCP profile.
- Documented writer-test entry requirements, global MCP merge instructions, startup verification, validation prompt, and stop conditions.
- Changed writer-test examples to reuse the existing Confluence PAT input by default, with a documented option for separate reader/writer tokens.
- Made destination parent/title checks, pre/post child-page checks, and post-write location verification standard writer controls.
- Added a reviewed Stage 3A storage-aware panel update path for templates that use Confluence panel macros instead of headings, including Version/change-history preservation checks.
- Updated the package README to point users to the writer-test installation path without changing the default read-only profile.

## 0.3.0 - Writer development roadmap

- Added `Writer-Development-Roadmap/README.md` for staged writer-test development.
- Added `examples/acf-confluence-writer-test.mcp.json` with a separate writer-test MCP profile and minimum write-tool allowlist.
- Updated the skill instructions to require writer-test entry criteria, separate profile use, explicit approval before writes, copy-first testing, and post-write validation.
- Updated MCP workflow guidance and validation checklist with writer-test controls.
- Kept the default end-user path read-only and preserved the known-good reader profile.

## 0.2.0 - End-user MCP installation and diagnostics

- Added `INSTALL.md` with complete native Windows and native Linux setup procedures.
- Added explicit `uv` / `uvx` verification and `mcp-atlassian --help` preflight checks.
- Added session-only PAT entry examples that avoid placing the PAT directly in command history or Codex configuration.
- Added a defense-in-depth read-only Codex MCP configuration using `READ_ONLY_MODE=true` plus an `enabled_tools` allowlist.
- Added first functional test against Confluence template page `115220454`.
- Added first skill test against page `219712327`.
- Added troubleshooting for PATH, runtime, environment propagation, authentication, authorization, TLS, missing write tools, and skill discovery.
- Added `diagnostic-improvement-prompt.md` for safe diagnosis and future documentation/skill improvements.
- Added diagnostic regression/evaluation cases.

## 0.1.0 - Initial draft

- Added the runbook-standardization skill.
- Added PDF-derived runbook structure and validation references.
- Added read-only and controlled-publishing MCP workflows.
