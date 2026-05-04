# NASCOP Reporting DWH — MCP Setup Guide

This repository contains configuration files for connecting AI coding tools to the NASCOP Reporting Data Warehouse MCP (Model Context Protocol) server.

**Server endpoint:** `https://insights.nascop.org/mcp`

> **Before you start:** Replace every occurrence of `YOUR_BEARER_TOKEN_HERE` in the config files with the Bearer token provided to you by the NASCOP DWH administrator. Never commit the real token to a repository.

> **Data protection notice:** By configuring and using this MCP server you confirm that you are an authorised NASCOP programme user, that you will use the data solely for HIV programme monitoring and reporting purposes, and that you will handle all data in accordance with the **Kenya Data Protection Act, 2019**. Unauthorised access or misuse may result in revocation of access and legal liability.

---

## What is MCP?

MCP (Model Context Protocol) lets AI assistants query live data sources — in this case, the NASCOP reporting database — directly from your editor or terminal. Once configured, you can ask questions in plain English and the AI will fetch real data to answer.

---

## Setting your token

Each config file contains the placeholder `YOUR_BEARER_TOKEN_HERE`. Replace it in all four files before use:

| File | Where to replace |
|------|-----------------|
| [.vscode/mcp.json](.vscode/mcp.json) | `"Authorization"` header value |
| [.mcp.json](.mcp.json) | `"Authorization"` header value |
| [.codex/config.toml](.codex/config.toml) | `Authorization` header value |

`.claude/settings.json` does not contain a token and does not need editing.

---

## Config 1: VS Code — GitHub Copilot / Claude extension (`.vscode/mcp.json`)

**File:** [.vscode/mcp.json](.vscode/mcp.json)

This registers the MCP server inside VS Code so that any MCP-compatible AI extension can use it.

### Setup steps

1. **Clone or copy this folder** to your machine and open it in VS Code:
   ```
   code .
   ```

2. **Replace the token** in `.vscode/mcp.json` — change `YOUR_BEARER_TOKEN_HERE` to the real token.

3. **Install an MCP-compatible extension**, for example:
   - [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) (v1.99+)
   - [Claude for VS Code](https://marketplace.visualstudio.com/items?itemName=Anthropic.claude-code)

4. **Trust the workspace** when VS Code prompts — workspace-scoped MCP configs require a trusted workspace to load.

5. **Verify the server is active:**
   - Open the Command Palette (`Ctrl+Shift+P`)
   - Run **MCP: List Servers** (GitHub Copilot) or **Claude: Show MCP Servers**
   - You should see `reporting-dwh` listed with status **Connected**

6. **Start querying** — see the [Using the MCP](#using-the-mcp) section below.

### Config reference

```json
{
  "servers": {
    "reporting-dwh": {
      "type": "http",
      "url": "https://insights.nascop.org/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BEARER_TOKEN_HERE"
      }
    }
  }
}
```

| Field | Value |
|-------|-------|
| `type` | `http` — uses HTTP/SSE MCP transport |
| `url` | NASCOP MCP server endpoint |
| `headers.Authorization` | Your Bearer token |

---

## Config 2: Claude Code CLI (`.mcp.json` + `.claude/settings.json`)

Two files work together for Claude Code CLI:

| File | Purpose |
|------|---------|
| [.mcp.json](.mcp.json) | Registers the MCP server (equivalent of `.vscode/mcp.json`) |
| [.claude/settings.json](.claude/settings.json) | Pre-approves `curl` commands so Claude does not prompt for permission |

### Setup steps

1. **Install Claude Code CLI** if you have not already:
   ```
   npm install -g @anthropic-ai/claude-code
   ```

2. **Clone or copy this folder** to your machine and open a terminal inside it.

3. **Replace the token** in `.mcp.json` — change `YOUR_BEARER_TOKEN_HERE` to the real token.

4. **Launch Claude Code** from inside the folder:
   ```
   claude
   ```
   Both `.mcp.json` and `.claude/settings.json` are loaded automatically from the project folder.

5. **Confirm MCP connectivity** — type `/mcp` in the Claude prompt and check that `reporting-dwh` appears with status connected.

6. **Start querying** — see the [Using the MCP](#using-the-mcp) section below.

### Manual connection test

Run this to confirm the server responds before launching Claude:

```bash
curl -s -i -X POST https://insights.nascop.org/mcp \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

A successful response returns HTTP `200 OK` and a JSON body containing `"protocolVersion"`.

### Config reference

**.mcp.json**
```json
{
  "mcpServers": {
    "reporting-dwh": {
      "type": "http",
      "url": "https://insights.nascop.org/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BEARER_TOKEN_HERE"
      }
    }
  }
}
```

**.claude/settings.json**
```json
{
  "permissions": {
    "allow": [
      "Bash(curl:*)"
    ]
  }
}
```

The `Bash(curl:*)` permission allows Claude to run `curl` commands (e.g. to test the MCP connection) without prompting you each time. All other commands still require your approval.

---

## Config 3: OpenAI Codex CLI (`.codex/config.toml`)

**File:** [.codex/config.toml](.codex/config.toml)

### Setup steps

1. **Install the Codex CLI** if you have not already:
   ```
   npm install -g @openai/codex
   ```

2. **Clone or copy this folder** to your machine and open a terminal inside it.

3. **Replace the token** in `.codex/config.toml` — change `YOUR_BEARER_TOKEN_HERE` to the real token.

4. **Launch Codex** from inside the folder:
   ```
   codex
   ```
   The `.codex/config.toml` file is loaded automatically.

5. **Verify the server** — Codex lists active MCP servers on startup. Look for `reporting-dwh: enabled`.

6. **Start querying** — see the [Using the MCP](#using-the-mcp) section below.

### Config reference

```toml
[mcp_servers.reporting-dwh]
enabled = true
url = "https://insights.nascop.org/mcp"

[mcp_servers.reporting-dwh.http_headers]
Authorization = "Bearer YOUR_BEARER_TOKEN_HERE"
```

---

## Using the MCP

Once connected, ask questions in plain English — the AI will call the MCP server automatically when it needs data.

### With Claude Code VS Code extension

1. Open the **Claude** panel from the Activity Bar or press `Ctrl+Shift+P` → **Claude: Open Chat**.
2. Type your question. Claude shows a **tool call badge** when fetching from the MCP server — click it to see the query sent.
3. Follow up in the same thread: *"Break that down by age band"* or *"Show as a table."*

### With GitHub Copilot Chat

1. Open Copilot Chat: `Ctrl+Shift+P` → **GitHub Copilot: Open Chat**.
2. Ask naturally, or prefix with `#reporting-dwh` to target the tool explicitly.
3. Copilot shows a **tool invocation card** before the answer.

### With Claude Code CLI

1. Just type your question at the `claude` prompt.
2. Claude will call the MCP tool and show the result inline.

### Example prompts by topic

| Topic | Example prompt |
|-------|---------------|
| Active clients on ART | *"How many clients are currently on ART by county?"* |
| Viral load suppression | *"What is the VL suppression rate by partner for active clients?"* |
| Unsuppressed clients | *"List counties with unsuppression rates above 10%."* |
| IIT risk | *"What proportion of active clients are at high interruption-in-treatment risk?"* |
| HIV testing | *"What was the HIV positivity rate by testing strategy in 2024?"* |
| PrEP | *"What proportion of eligible clients were started on PrEP by county?"* |
| PBFW | *"What proportion of pregnant and breastfeeding women have unsuppressed viral loads?"* |
| HEI | *"What proportion of HIV-exposed infants were uninfected at 24 months?"* |
| OTZ | *"What proportion of enrolled OTZ clients have completed training modules?"* |
| OVC | *"What proportion of OVC clients are virally suppressed?"* |
| PNS | *"How many index-client partners were elicited and linked to treatment?"* |
| COVID | *"What proportion of clients on treatment have received a COVID booster?"* |

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Server shows **Disconnected** | Network or wrong token | Run the manual `curl` test above |
| `401 Unauthorized` | Token expired or incorrect | Contact the NASCOP DWH admin for a new token |
| MCP server not listed | Wrong working directory | Open VS Code / terminal from inside this folder |
| No tools available after connect | Server-side issue | Confirm `https://insights.nascop.org/mcp` is reachable |

---

## Security

- **Never commit the real token** to any repository.
- The token grants read access to NASCOP programme data — treat it like a password.
- If you suspect the token has been exposed, contact the NASCOP DWH administrator immediately to rotate it.
- Share the token only via a secure channel (password manager, encrypted message) — not email or chat.
