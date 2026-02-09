# Overleaf MCP Server

## Project Overview
This project is a **Model Context Protocol (MCP) Server** designed to integrate **Overleaf** projects with LLM interfaces like Claude or Gemini. It enables AI assistants to directly read, analyze, and extract content from LaTeX documents hosted on Overleaf using their Git integration.

## Architecture
- **Runtime:** Node.js (ES Modules).
- **Protocol:** [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol/protocol).
- **Integration:** Uses the system's `git` command-line tool to clone and pull Overleaf repositories.
- **Data Source:** Overleaf Git interface (`https://git.overleaf.com/...`).
- **Configuration:** Local JSON file (`projects.json`) stores credentials and project mappings.

## Key Files
- **`overleaf-mcp-server.js`**: The main entry point and server implementation.
  - Defines the MCP tools (`list_projects`, `read_file`, etc.).
  - Contains an internal `OverleafGitClient` class for Git operations (Note: logic is currently inlined here).
  - Handles the Stdio transport for communicating with the MCP client.
- **`overleaf-git-client.js`**: A CommonJS module providing a standalone `OverleafGitClient` class.
  - *Note:* Currently, the server uses its own inlined ESM version of the client, not this file. This file appears to be a separate or legacy library implementation.
- **`projects.json`**: (User-created) Stores sensitive configuration like Project IDs and Git Tokens. **Gitignored**.
- **`projects.example.json`**: Template for the configuration file.

## Setup & Configuration

### 1. Installation
```bash
npm install
```

### 2. Configuration (`projects.json`)
Create a `projects.json` file based on `projects.example.json`:
```json
{
  "projects": {
    "default": {
      "name": "My Paper",
      "projectId": "YOUR_OVERLEAF_PROJECT_ID",
      "gitToken": "YOUR_OVERLEAF_GIT_TOKEN"
    }
  }
}
```
*   **Git Token:** Generate via Overleaf Account Settings -> Git Integration.
*   **Project ID:** Found in the project URL (`overleaf.com/project/<ID>`).

### 3. Running the Server
The server is designed to be run by an MCP client (like Claude Desktop or Gemini CLI) via stdio.

**Manual Start:**
```bash
node overleaf-mcp-server.js
```

**MCP Configuration Example:**
```json
{
  "mcpServers": {
    "overleaf": {
      "command": "node",
      "args": ["/path/to/OverleafMCP/overleaf-mcp-server.js"]
    }
  }
}
```

## Available Tools
The server exposes the following tools to the LLM:
- `list_projects`: Lists configured projects.
- `list_files`: Lists `.tex` files in a project.
- `read_file`: Reads the full content of a file.
- `get_sections`: Parses LaTeX structure (sections, subsections) without content.
- `get_section_content`: Extracts text from a specific LaTeX section.
- `status_summary`: Provides an overview of the project (files, main file, sections).

## Development Conventions
- **Modules:** The server uses **ES Modules** (`import`/`export`).
- **Styles:**
  - `child_process.exec` (promisified) is used for Git commands.
  - Temporary directories are created in `os.tmpdir()` with a prefix `overleaf-<projectID>`.
- **Security:**
  - Credentials are passed via the Git URL or environment variables (`GIT_PASSWORD`).
  - `projects.json` is strictly ignored to prevent leaking credentials.
