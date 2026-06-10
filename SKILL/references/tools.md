# Outerline MCP Tool Reference

Outerline MCP server name: `outerline`.

The exact callable namespace depends on the client. Claude and Codex may expose the same MCP operation with different visible tool names. Use the actual MCP tool exposed by the client rather than shelling into the vault.

## Access Model

- Vault filesystem tools read and write configured Outerline `.md` documents and `.outerline-data` metadata. They usually work without the app running.
- Write tools modify files on disk and notify the app to refresh when the app is running.
- Live editor tools use the running app bridge and fail cleanly if Outerline is not open.
- Version tools read snapshots under `.outerline-data`.

## Read Tools

| Tool | Inputs | Use |
|---|---|---|
| `list_documents` | `folder?`, `tag?`, `starred?` | List documents, sorted newest first. |
| `get_document` | `path?`, `uuid?` | Read full document content by relative path or UUID. |
| `search_documents` | `query` | Full-text case-insensitive search across the vault. |
| `get_outline` | `path` | Extract headings with levels and line numbers. |
| `get_backlinks` | `path` | Find documents linking to the target through wiki links. |
| `get_links` | `path` | Extract outbound wiki links and external URLs. |
| `get_annotations` | `path` | Get highlights and annotations for a document. |
| `get_tags` | `path?` | Get all vault tags with counts, or tags for one document. |
| `get_graph` | `center?` | Get full or local link graph. |
| `get_deep_link` | `path?`, `uuid?`, `folder?`, `line?`, `heading?`, `annotation?` | Generate an `outerline://` URL. |
| `resolve_deep_link` | `url` | Resolve an `outerline://` URL without opening it. |
| `list_folders` | `root?`, `depth?` | List folders with shallow document counts. |
| `find_folder` | `query` | Find folders by exact or substring match. |
| `get_tag_colors` | none | Get assigned tag colors. |

## Write Tools

| Tool | Inputs | Use |
|---|---|---|
| `create_document` | `path`, `content` | Create a new markdown note, creating intermediate folders. |
| `rename_document` | `fromPath`, `toPath` | Rename a note in place. |
| `move_document` | `fromPath`, `toPath` | Move a note to another folder. |
| `rename_folder` | `fromPath`, `toPath` | Rename a folder in place. |
| `move_folder` | `fromPath`, `toPath` | Move a folder to another location. |
| `append_to_document` | `path`, `content` | Append content to an existing note. |
| `replace_section` | `path`, `heading`, `content` | Replace content under a heading; heading line is preserved. |
| `edit_document` | `path`, `content` | Replace full note content; existing tags are preserved. |
| `create_annotation` | `path`, `highlighted_text`, `color?`, `note?` | Add highlight/annotation. Colors: `yellow`, `blue`, `green`, `pink`, `purple`. |
| `set_tags` | `path`, `tags` | Add tags, with or without `#`. Existing tags are skipped. |
| `rename_tag` | `old_tag`, `new_tag` | Rename a tag across all documents. |
| `delete_tag` | `tag` | Remove a tag from all documents. |
| `set_tag_color` | `tag`, `color?` | Assign or clear tag color. |
| `star_document` | `path` | Star a document. |
| `unstar_document` | `path` | Remove a document star. |

## Live Editor Tools

| Tool | Inputs | Use |
|---|---|---|
| `get_active_document` | none | Get current editor path and content. Requires app running. |
| `get_selection` | none | Get selected text. Requires app running. |
| `insert_at_cursor` | `text` | Insert text at cursor. Requires app running. |
| `replace_selection` | `text` | Replace selected text. Requires app running. |

## Version History Tools

| Tool | Inputs | Use |
|---|---|---|
| `list_versions` | `path?`, `uuid?`, `limit?` | List snapshots with timestamp, word count, save type. |
| `get_version` | `path?`, `uuid?`, `version_id` | Read a specific version snapshot. |
| `get_version_annotations` | `path?`, `uuid?`, `version_id` | Read annotations captured with a snapshot. |
| `diff_versions` | `path?`, `uuid?`, `version_id_a`, `version_id_b?` | Diff two snapshots, or snapshot vs current. |
| `search_version_history` | `path?`, `uuid?`, `query` | Search all snapshots for text. |
| `find_deleted_content` | `path?`, `uuid?`, `query` | Find matching text that existed historically but not now. |
| `restore_version` | `path?`, `uuid?`, `version_id` | Restore a snapshot after auto-snapshotting current state. |
| `get_writing_timeline` | `path?`, `uuid?` | Show word count progression over time. |

## Resources

| URI | Use |
|---|---|
| `vault://folders` | Folder tree with document counts. |
| `vault://tags` | Tags across the vault with counts. |
| `vault://starred` | Starred documents. |
| `vault://recent` | 20 recently modified documents. |
| `vault://graph` | Full link graph. |

## Error Handling

- `vault_not_loaded`: Outerline has not persisted a library location. Ask the user to open Outerline and choose a library.
- "Outerline app is not running": only live editor tools are unavailable. Use path-based tools or ask the user to open Outerline.
- Ambiguous wikilinks: `resolve_deep_link` may return `matches`; ask the user to choose unless context is decisive.
- Invalid path: use relative vault paths and avoid `../`.
