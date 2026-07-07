---
name: notion-pull
description: sync notion pages and databases to markdown files recursively
---
# notion pull

## workflow
1. accept a Notion page id, database id, or URL from the user as the synchronization root.
2. fetch the remote metadata (properties) using the Notion API.
3. initialize local state: create the directory, `notion.yaml`, and `page.md`/`database.md` with properties serialized as YAML frontmatter if they do not exist locally.
4. compute a diff between the local markdown file and the remote Notion properties/blocks.
5. if the local file has changes, update the remote Notion page via API to mirror local modifications.
6. if the remote Notion page has changes, update the local markdown file to mirror remote modifications. do not duplicate the entity title as a markdown heading.
7. fetch the entity's content (blocks or child pages) using the Notion API.
8. for any nested child pages or inline databases encountered, recursively execute steps 2-8, creating subdirectories to mirror the remote tree structure.
9. represent inline child databases and subpages as relative markdown links in the parent file body.
10. continue recursion until the entire tree is initialized and bidirectionally synchronized.
11. when fetching a page's blocks, you MUST check for `has_children: true` on structural blocks (like `column_list` and `column`). traverse these blocks recursively to ensure no nested text or `child_database` links are omitted from the final `page.md`.

## automation
for bulk synchronization, you can run the sync script:
```powershell
python skills/notion-pull/scripts/notion-pull.py
```
the script must be run from the root of the workspace (where the `notion` directory is located) and requires `.env` with the `NOTION` token.
