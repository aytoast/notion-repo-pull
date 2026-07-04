# notion-skills


<p align="center">
  <strong>why manually query API when local workspace do trick</strong>
</p>


<p align="center">
  Notion workspace content, direct in your IDE.<br>
  <strong>Zero sync latency. Zero context decay.</strong>
</p>


<p align="center">
  <a href="https://github.com/aytoast/notion-skills/stargazers"><img src="https://img.shields.io/github/stars/aytoast/notion-skills?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/aytoast/notion-skills/commits/master"><img src="https://img.shields.io/github/last-commit/aytoast/notion-skills?style=flat" alt="Last commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/aytoast/notion-skills?style=flat" alt="License"></a>
</p>


<p align="center">
  <a href="#why-use-it">Why use it</a> ·
  <a href="#before--after">Before / After</a> ·
  <a href="#install">Install</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#what-you-get">What you get</a>
</p>


---


`notion-skills` is a modular Notion plugin and skill directory for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Codex, Gemini, Cursor, and other agentic environments. It synchronizes remote Notion pages and databases into local Markdown files, allowing your IDE to index them for zero-latency, context-rich agent interactions.


---


## Why use it


*   **Persistent Auth**: Uses a persistent integration token (`NOTION_PAT`) in `.env`. No OAuth expiration.

*   **IDE Context Indexing**: Syncs database items and page structures to local Markdown. Your IDE indexer and LLM agent can scan and reference your Notion workspace instantly.

*   **Decoupled Notion API Wrappers**: Standalone commands isolate raw Notion API request and authentication logic from high-level synchronization workflows. This decoupling provides:
    *   **Modular Reusability**: Other automated agent workflows (e.g., automated issue creation, documentation updates) can execute base API commands directly without duplicating authentication or curl invocation logic.
    *   **Strict Payload Enforcement**: Notion's API requires deeply nested JSON payloads. Standalone wrappers act as a validation boundary. Agents refer to `references/schema.md` to formulate payload structures, eliminating formatting and schema mismatches.
    *   **Centralized Bug Fixing**: Changes to Notion's API layout only require a single fix in the wrapper script. All high-level sync workflows inherit the fix immediately, preventing widespread script breakages.

*   **Constrained Agent Guidance**: Wraps Notion API calls with input/output schemas (`references/schema.md`). Prevents agents from sending malformed payloads and lets them log bug fixes directly inside `SKILL.md` rules.


---


## Before / After


<table>
<tr>
<th width="50%">🔄 Manual API Agent — 1500+ input tokens</th>
<th width="50%">⚡ Indexed Local Workspace — 80 input tokens</th>
</tr>
<tr>
<td valign="top">

> *Agent has no view of the Notion page...*
>
> 1. Runs `get-page` to fetch metadata.
> 2. Parses page properties.
> 3. Runs `get-block-children` to fetch text blocks.
> 4. Discovers a child database, runs `query-database`.
>
> **Time spent: 15s. Tokens wasted: 1500+**

</td>
<td valign="top">

> *Agent uses local index...*
>
> 1. Directly reads the local Markdown file `notion/docs/onepager/page.md`.
>
> **Time spent: <0.1s. Tokens wasted: <80**

</td>
</tr>
</table>


```text
┌────────────────────────────────────────────┐
│   indexing speed        █████████     100x │
│   context resolution    █████████     100% │
│   api validation errors ░░░░░░░░░       0% │
│   agent productivity    █████████     9000 │
└────────────────────────────────────────────┘
```


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


The plugin separates high-level synchronization logic from low-level API commands:


```text
               +-----------------------------------+
               |  Repository Sync Workflow Script  |  <-- e.g., pull-notion.py
               +-----------------+-----------------+
                                 |
                                 v
               +-----------------------------------+
               |        Notion API Wrappers        |  <-- e.g., query-notion-database.py
               +-----------------------------------+
```


### 1. Notion API Wrappers
Located under `api/`. Clean, isolated wrappers around Notion endpoints.


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


### 2. Repository Sync Workflow
Located under `skills/`. Builds complex orchestration logic on top of base API commands:
*   `repo-pull` — Recursively pulls pages and databases, compiling them into a structured local directory.


---


## What you get


Each integration tool directory contains:
*   `scripts/`: Python wrapper executing the API requests.

*   `SKILL.md`: Configuration and execution instructions.

*   `references/schema.md`: Strict input schemas and response JSON structures.


### Running Repository Sync

To execute the synchronization, run the sync script from your workspace root:

```bash
python skills/repo-pull/scripts/pull-notion.py
```
