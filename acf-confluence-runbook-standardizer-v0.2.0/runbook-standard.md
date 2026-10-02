# NGSC Operation RunBook Standard

## Source

This reference was derived from the user-provided PDF export of `NGSC Operation RunBook Template`, corresponding to Confluence page ID `115220454`.

The PDF is a visual/content reference. The live Confluence template should be retrieved through MCP when available because the export does not expose the exact Confluence storage representation, macros, or page-layout constructs.

## What the supplied template establishes

The exported template visibly contains these major sections, in this order:

1. Brief
2. Prerequisites
3. Procedure
4. Version

The page title appears above the sectioned content.

### Visual intent

The PDF shows the primary sections as bordered content areas with a gray header band and light blue/cyan outline. The skill must not guess which Confluence macro, table style, or storage markup creates this appearance. Preserve/copy the live template's storage representation instead.

### Brief

The template describes itself as an example and template used to create a standardized Run Book for ACF technical tasks. It says the document demonstrates copying the template to the correct location and then updating it into an operational runbook.

Derived rule: every runbook needs a short Brief that explains its operational purpose and scope.

### Prerequisites

The example prerequisite block begins with a short explanatory sentence and then lists prerequisites as bullets.

The example includes:

- Confluence access with write permissions
- a process or procedure to document
- the runbook/template itself

Derived rule: actual runbooks should replace these example items with the real prerequisites for the operational procedure.

### Procedure

The example procedure is a numbered workflow with nested lettered steps and screenshots placed near the steps they illustrate.

The template's own creation procedure instructs the author to:

- determine the destination/parent location;
- copy the template page;
- select the correct space and parent page;
- complete the copy;
- update Brief;
- update Prerequisites;
- update the procedure/process content;
- add a change comment;
- publish the document.

Derived rule: new runbooks should normally be created by copying the live template rather than manually recreating its layout.

### Procedure versus Process wording

The visible section heading is `Procedure`. The example instructions later say `Update the Process section`.

For standardization, use `Procedure` as the canonical heading and treat `Process` in the example instructions as referring to the same content area. Do not create both headings unless a later approved template explicitly does so.

### Version

The exported page shows a `Version` section with columns such as Version, Published, Changed By, and Comment.

Do not manually invent this table's underlying implementation. When creating a runbook, preserve it by copying the live template page. When updating an existing page, preserve the existing storage/macro representation unless the user specifically requests a reviewed change to the template mechanism.

## Content quality rules added for end-user friendliness

These rules are implementation recommendations for this skill; they are not claimed to be text from the PDF:

- Use clear, action-oriented numbered steps.
- Expand acronyms on first use when the source supports the expansion.
- Keep one operator action per main numbered step where practical.
- Put commands in code blocks and preserve command text exactly.
- Put screenshots immediately after the step they explain.
- State expected results only when the source supports them.
- Put safety/security warnings before the action they govern.
- Flag missing information instead of inventing it.
- Prefer short paragraphs and scannable lists for working engineers.
