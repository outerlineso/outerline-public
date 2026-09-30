---
name: outerline
description: "Use Outerline MCP to work with markdown notes, folders, tags, comments, links, and version history, or to recover tag metadata with authorization. Trigger for Outerline vault content, current notes/selections, or outerline:// links."
---

# Outerline MCP

Use the `outerline` MCP server for Outerline notes. Do not treat `outerline://` links as browser URLs or generic files. Resolve and operate on them through MCP.

## Activation Rules

Use this skill when the user says any of:

- "use Outerline", "read this in Outerline", "my Outerline notes", "Outerline vault/library"
- "current document", "current note", "selection", or "cursor" in an Outerline context
- "search/list/create/edit/move/rename/tag/star/comment in Outerline" or Outerline version history or tag metadata recovery
- an `outerline://...` URL
- any question about note content, document organization, writing, tags, comments, or version history when working inside the Outerline repository, even if "Outerline" is not named explicitly

Do not use this skill for Swift source code, the app codebase, or build/test operations unless the user is asking about Outerline note content related to that work.

If the Outerline MCP tools are not visible, explain that the `outerline` MCP server needs to be configured. Setup examples:

```bash
codex mcp add outerline -- /Applications/Outerline.app/Contents/Resources/OuterlineMCP --stdio
claude mcp add outerline -- /Applications/Outerline.app/Contents/Resources/OuterlineMCP --stdio
```

Outerline must have been opened at least once with a library selected. Live editor tools require the app to be running, and so do all vault tools when the library is a local (non-iCloud) folder.

## Core Workflow

1. Identify the target using MCP, not filesystem guesses.
2. For pasted `outerline://` links, call `resolve_deep_link` first, then use the returned `path` or `uuid`.
3. For "current note/document", call `get_active_document`.
4. For "current selection", call `get_selection`; use `replace_selection` only when the user wants the selected text rewritten.
5. For unknown notes, call `search_documents`, `list_documents`, `find_folder`, or read `vault://recent`/`vault://folders` before choosing a path.
6. Read before writing unless the user explicitly provided the full target path and content.
7. After writing, report the tool used and the affected note, folder, or metadata. For tag metadata repair, include the returned backup location.

## Tool Selection

Fast defaults:

- Find notes: `search_documents` for text, `list_documents` for folders/tags/starred, `find_folder` for folder names.
- Read note content: `get_document`.
- Understand structure: `get_outline`, `get_links`, `get_backlinks`, `get_graph`, `get_tags`, `get_annotations`.
- Review or leave margin feedback: `list_comments` to read, `create_comment` to add one anchored to exact selected text.
- Follow a deeplink: `resolve_deep_link`, then `get_document` or `get_outline` as needed.
- Create a deeplink: `get_deep_link` with `path` or `uuid`; add `line`, `heading`, `annotation`, or `folder: true` when needed.
- Current editor state: `get_active_document`, `get_selection`, `insert_at_cursor`, `replace_selection`.
- Create folders: `create_folder` with a vault-relative `path`; missing parents are created and existing folders are preserved.
- Rename or move folders: `edit_folder` with `fromPath` and a complete `toPath`, including the folder name. Create the destination parent first if needed.
- Small additions: `append_to_document`.
- Section edits: `replace_section` with an exact heading after checking `get_outline`.
- Full-note rewrites: `edit_document`; it preserves existing tags and is safer than many stitched section edits.
- Display titles: `edit_document_title` with a document `path` or `uuid` and `title`; blank `title` clears the customization without renaming the file.
- Tags, colors, and stars: `get_tags` or `get_tag_colors` to inspect, `set_tags` to add tags, `set_tag_color` to assign or clear a color, and `star_document` or `unstar_document` to change stars.
- Tag rename/delete: `rename_tag` and `delete_tag` check tag color metadata before saving affected open notes or changing tags. If the check fails, follow the tag metadata recovery guidance below before retrying.
- Vault overview: read `vault://folders`, `vault://tags`, `vault://starred`, `vault://recent`, or `vault://graph` for quick structure and activity.
- Version history: `list_versions` to find snapshot IDs, `get_version` or `get_version_annotations` to inspect them, `diff_versions`, `search_version_history`, `find_deleted_content`, or `get_writing_timeline` to compare history. Use `restore_version` only when explicitly requested.

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
- Before calling `repair_tag_metadata`, get explicit user authorization to reset existing tag color assignments. Explain that the tool backs up the exact original metadata bytes, leaves notes unchanged, and returns the backup location. Use `confirm_reset: true` only after that authorization; retry the original rename or delete in a separate call.
- Use `repair_tag_metadata` only for corrupt tag metadata. For a newer schema, ask the user to update Outerline; for a file access or sync error, resolve that problem first. If a tag batch reports completed note paths, inspect those notes before any retry because the batch may be partial.
- For writes based on fuzzy language, first discover the exact document/folder and state the resolved path.
- If a live editor tool says the app is not running, ask the user to open Outerline. Path-based vault tools can work with the app closed only for the built-in iCloud Library; a local library still requires the app to be open.

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

Create a folder hierarchy, then rename and move an existing folder:

```text
create_folder({ "path": "Archive/2026" })
edit_folder({ "fromPath": "Projects/Ideas", "toPath": "Archive/2026/Research" })
```

Create a link for an external task:

```text
get_deep_link({ "path": "Projects/Roadmap.md", "heading": "Milestones" })
```
