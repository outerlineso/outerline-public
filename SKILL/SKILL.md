---
name: outerline
description: "Use Outerline MCP to read, search, edit, organize, tag, star, and deep-link markdown notes. Trigger for Outerline vault content, current notes/selections, or outerline:// links."
---

# Outerline MCP

Use the `outerline` MCP server for Outerline notes. Do not treat `outerline://` links as browser URLs or generic files. Resolve and operate on them through MCP.

## Activation Rules

Use this skill when the user says any of:

- "use Outerline", "read this in Outerline", "my Outerline notes", "Outerline vault/library"
- "current document", "current note", "selection", or "cursor" in an Outerline context
- "search/list/create/edit/move/rename/tag/star in Outerline"
- an `outerline://...` URL
- any question about note content, document organization, writing, or tags when working inside the Outerline repository, even if "Outerline" is not named explicitly

Do not use this skill for Swift source code, the app codebase, or build/test operations unless the user is asking about Outerline note content related to that work.

If the Outerline MCP tools are not visible, explain that the `outerline` MCP server needs to be configured. Setup examples:

```bash
codex mcp add outerline -- /Applications/Outerline.app/Contents/Resources/OuterlineMCP --stdio
claude mcp add outerline -- /Applications/Outerline.app/Contents/Resources/OuterlineMCP --stdio
```

Outerline must have been opened at least once with a library selected. Live editor tools require the app to be running.

## Core Workflow

1. Identify the target using MCP, not filesystem guesses.
2. For pasted `outerline://` links, call `resolve_deep_link` first, then use the returned `path` or `uuid`.
3. For "current note/document", call `get_active_document`.
4. For "current selection", call `get_selection`; use `replace_selection` only when the user wants the selected text rewritten.
5. For unknown notes, call `search_documents`, `list_documents`, `find_folder`, or read `vault://recent`/`vault://folders` before choosing a path.
6. Read before writing unless the user explicitly provided the full target path and content.
7. After writing, report the exact note path and tool used.

## Tool Selection

Fast defaults:

- Find notes: `search_documents` for text, `list_documents` for folders/tags/starred, `find_folder` for folder names.
- Read note content: `get_document`.
- Understand structure: `get_outline`, `get_links`, `get_backlinks`, `get_graph`, `get_tags`, `get_annotations`.
- Follow a deeplink: `resolve_deep_link`, then `get_document` or `get_outline` as needed.
- Create a deeplink: `get_deep_link` with `path` or `uuid`; add `line`, `heading`, `annotation`, or `folder: true` when needed.
- Current editor state: `get_active_document`, `get_selection`, `insert_at_cursor`, `replace_selection`.
- Small additions: `append_to_document`.
- Section edits: `replace_section` with an exact heading after checking `get_outline`.
- Full-note rewrites: `edit_document`; it preserves existing tags and is safer than many stitched section edits.
- Folder/tag overview or recent activity: read `vault://folders`, `vault://tags`, or `vault://recent` before making assumptions about vault structure.
- Version recovery: `list_versions`, `diff_versions`, `search_version_history`, `find_deleted_content`; use `restore_version` only when explicitly requested.

For the full tool and resource catalog, read [references/tools.md](references/tools.md).

## Deeplink Handling

Supported URL shapes include:

- `outerline://open/<document-uuid>`
- `outerline://open/<document-uuid>?line=5`
- `outerline://open/<document-uuid>?heading=Overview`
- `outerline://open/<document-uuid>?annotation=<annotation-uuid>`
- `outerline://folder/<folder-uuid>`
- `outerline://path/Notes/Planning.md`
- `outerline://wikilink/Planning`

Always call `resolve_deep_link` for pasted URLs. It returns the target kind, path, UUID when known, title, position metadata, or `matches` for ambiguous wikilinks. If ambiguous, ask the user which match to use or pick only when context makes the target unambiguous.

## Safety Rules

- Use relative vault paths such as `Projects/Roadmap.md`; do not use absolute local paths as MCP document paths.
- Do not bypass MCP path validation by editing vault files directly.
- Do not run shell commands to open `outerline://` links unless the user explicitly asks to open the app.
- Do not use broad write tools (`rename_tag`, `delete_tag`, folder moves, version restore) without clear user intent.
- For writes based on fuzzy language, first discover the exact document/folder and state the resolved path.
- If `get_active_document`, `get_selection`, `insert_at_cursor`, or `replace_selection` says the app is not running, ask the user to open Outerline or switch to path-based tools.

## Common Patterns

Resolve and read a pasted link:

```text
resolve_deep_link({ "url": "outerline://..." })
get_document({ "path": "<resolved path>" })
```

Answer a question about the current note:

```text
get_active_document()
get_outline({ "path": "<active path>" })
```

Rewrite the selected text:

```text
get_selection()
replace_selection({ "text": "<new selected text>" })
```

Edit a section safely:

```text
get_outline({ "path": "Notes/Plan.md" })
replace_section({ "path": "Notes/Plan.md", "heading": "Risks", "content": "<new section body>" })
```

Create a link for an external task:

```text
get_deep_link({ "path": "Projects/Roadmap.md", "heading": "Milestones" })
```
