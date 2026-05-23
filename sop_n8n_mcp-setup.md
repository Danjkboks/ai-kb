---
type: sop
domain: n8n
source: session
status: active
tags: [n8n, mcp, claude-code, api, jwt, integration, workflow-builder]
created: 2026-05-19
updated: 2026-05-21
---

# SOP — n8n Instance-Level MCP Setup

**Purpose:** Configure and use n8n's built-in MCP server so Claude Code can create, validate, and manage workflows directly.

---

## What This Enables

With n8n MCP configured, Claude Code gains access to 1,650+ n8n nodes and can:
- `search_nodes` — find any integration by keyword
- `get_node` — get node details, properties, operations
- `validate_node` / `validate_workflow` — validate before deploying
- `n8n_create_workflow` — create workflows from code
- `n8n_update_partial_workflow` — incremental workflow edits
- `n8n_list_workflows` — browse existing workflows
- `n8n_deploy_template`, `n8n_autofix_workflow`, and more

---

## Current Configuration

| Setting | Value |
|---|---|
| MCP Server URL | `http://localhost:5678/mcp-server/http` |
| Auth | JWT token (stored in `D:\aidirectory\.env` as `N8N_MCP_TOKEN`) |
| Status | ✅ Enabled (toggle ON at `localhost:5678/settings/mcp`) |
| Config file | `D:\lab\workflow-lab\.mcp.json` |

---

## 1. Enable in n8n UI

1. Open `http://localhost:5678`
2. Go to **Settings → Instance-level MCP**
3. Toggle ON
4. Copy the Server URL: `http://localhost:5678/mcp-server/http`

---

## 2. Generate JWT Token

In n8n UI → Settings → Instance-level MCP → **Access token** tab:
1. Click "Generate new token"
2. Copy it immediately (shown only once)
3. Save to `D:\aidirectory\.env`:
   ```
   N8N_MCP_TOKEN=your-jwt-token-here
   ```

⚠️ Generating a new token **immediately revokes** the old one.

---

## 3. Configure Claude Code MCP

Create / update `D:\lab\workflow-lab\.mcp.json`:

```json
{
  "mcpServers": {
    "n8n": {
      "type": "http",
      "url": "http://localhost:5678/mcp-server/http",
      "headers": {
        "Authorization": "Bearer YOUR_JWT_TOKEN_HERE"
      }
    }
  }
}
```

Claude Code reads `.mcp.json` from the project root at session startup — no install required beyond having Node.js.

---

## 4. Verify Connection

In a Claude Code session:
```
Use the n8n MCP to search for the Slack node.
```

If configured correctly, Claude will call `search_nodes({query: "slack"})` and return results.

---

## n8n-skills Plugin (Separate from MCP)

The **n8n-skills** Claude Code plugin (at `D:\aidirectory\skills\n8n-skills\`) provides expert guidance on HOW to use the MCP tools — it's the "brain" that works alongside the MCP "hands". Together they form the complete workflow-building system:

- MCP = data access (nodes, validation, workflow CRUD)
- n8n-skills = expertise (patterns, gotchas, best practices)

---

## Key Distinction: n8n Built-in MCP vs czlonkowski/n8n-mcp

| | n8n Built-in MCP | czlonkowski/n8n-mcp |
|---|---|---|
| Auth | JWT (n8n UI) | REST API key |
| Runtime | No install | Requires `npx` |
| Nodes | 1,650+ | 1,650+ |
| **Verdict** | ✅ Use this | Not needed |

The built-in HTTP MCP server exposes all required tools — the GitHub package is unnecessary.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| MCP not responding | Verify n8n is running: `docker start n8n` |
| 401 Unauthorized | Token expired or revoked — regenerate in n8n UI |
| Tools not available in Claude | Check `.mcp.json` path and restart Claude Code session |
| n8n MCP toggle grayed out | Ensure n8n version supports it (2024.x+) |
