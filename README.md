# notion-skills

<p align="center">
  <strong>Git-style operations for Notion.</strong>
</p>

<p align="center">
  Manage your Notion workspace exactly like a git repository.<br>
  <strong>Clone, Diff, Pull, and Push your Notion blocks directly to local Markdown.</strong>
</p>

<p align="center">
  <a href="https://github.com/aytoast/notion-skills/stargazers"><img src="https://img.shields.io/github/stars/aytoast/notion-skills?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/aytoast/notion-skills/commits/master"><img src="https://img.shields.io/github/last-commit/aytoast/notion-skills?style=flat" alt="Last commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/aytoast/notion-skills?style=flat" alt="License"></a>
</p>

<p align="center">
  <a href="#the-git-for-notion-workflow">The Workflow</a> ·
  <a href="#why-use-it">Why use it</a> ·
  <a href="#install">Install</a> ·
  <a href="#architecture">Architecture</a>
</p>

---

`notion-skills` brings the robust, frictionless mental model of Git to Notion. Instead of relying on buggy MCP connections or constantly querying the Notion API directly from an agent, you maintain a local Markdown replica of your workspace. 

Your IDE or AI agent can instantly index your local files with zero latency. When you are ready, you use Git-style Notion commands to push and pull changes between your local working directory and the remote Notion server.

---

## The Git for Notion Workflow

Forget about configuring complex MCP gateways. Treat Notion as your remote source of truth and your local Markdown files as your working directory.

1. **`notion clone`**: Downloads the remote workspace tree from Notion to your local markdown directory.
2. **`notion diff`**: Compares your local markdown edits directly against the live Notion API to preview what will change.
3. **`notion pull`**: Fetches remote Notion changes and synchronizes them into your local markdown files, overwriting outdated local state.
4. **`notion push`**: Translates local markdown edits into Notion API payloads and patches the remote blocks directly.

*(Note: Currently, `notion pull` is fully implemented as `notion-pull`, with the other commands acting as the conceptual framework for upcoming releases).*

---

## Why use it

*   **Frictionless Sync**: By treating Notion as a remote branch, you eliminate the need for complex, persistent state management. Edit locally, sync globally.
*   **Persistent Auth**: Uses a persistent integration token (`NOTION_PAT`) in `.env`. No OAuth expiration loops or connection timeouts common in other protocols.
*   **IDE Context Indexing**: Syncs database items and page structures to local Markdown. Your IDE indexer and LLM agent can scan and reference your Notion workspace instantly, costing you 80 tokens instead of 1,500+.
*   **Decoupled Notion API Wrappers**: Standalone low-level commands isolate raw Notion API logic from the high-level sync workflows, acting as the "plumbing" for our Git-style operations.

---

## Install

### Quick Start

1. Clone this repository into your workspace or agent plugin directory:

   ```bash
   git clone https://github.com/aytoast/notion-skills.git
   ```

2. Define your integration token in a `.env` file at the root of your workspace:

   ```env
   NOTION_PAT=your_internal_integration_token_here
   ```

3. Initialize the tracking root by creating a directory named `notion` at your workspace root, and inside it create a `notion.yaml` configuration file specifying the page or database ID to track:

   ```yaml
   type: "page" # or "database"
   id: "your_notion_page_or_database_id_here"
   ```

---

## Architecture

The plugin separates high-level Git-style synchronization logic ("porcelain") from low-level API commands ("plumbing"):

```text
               +-----------------------------------+
               |    Git-style Sync Operations      |  <-- e.g., notion-pull.py
               +-----------------+-----------------+
                                 |
                                 v
               +-----------------------------------+
               |        Notion API Wrappers        |  <-- e.g., update-notion-page.py
               +-----------------------------------+
```

### 1. High-Level Sync Operations (Porcelain)
Located under `skills/`. Builds complex orchestration logic on top of base API commands.
*   `notion-pull` — Recursively pulls pages and databases, compiling them into a structured local markdown directory.

### 2. Notion API Wrappers (Plumbing)
Located under `api/`. Clean, isolated wrappers around Notion endpoints that execute the actual HTTP requests for pushes and pulls.

| Command | Endpoint | Purpose |
| :--- | :--- | :--- |
| `append-block-children` | `PATCH /v1/blocks/{id}/children` | Appends content blocks. |
| `archive-block` | `DELETE /v1/blocks/{id}` | Archives or deletes a block. |
| `create-page` | `POST /v1/pages` | Creates a new page. |
| `get-block` | `GET /v1/blocks/{id}` | Retrieves block metadata. |
| `get-block-children` | `GET /v1/blocks/{id}/children` | Lists child blocks. |
| `get-database` | `GET /v1/databases/{id}` | Retrieves database details. |
| `get-page` | `GET /v1/pages/{id}` | Fetches page metadata. |
| `get-user` | `GET /v1/users/me` | Verifies integration user. |
| `query-database` | `POST /v1/databases/{id}/query` | Filters and retrieves database pages. |
| `update-block` | `PATCH /v1/blocks/{id}` | Modifies block content. |
| `update-page` | `PATCH /v1/pages/{id}` | Patches page properties. |

---

## Running Sync Operations

To execute the synchronization (the equivalent of `notion pull`), run the sync script from your workspace root:

```bash
python skills/notion-pull/scripts/notion-pull.py
```
