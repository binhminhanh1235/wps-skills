# WPS Skills

[English](README.md) | [Tiếng Việt](README_vi.md) | [中文](README_zh.md)

> Any functional or architectural updates to this project must be synchronized with the relevant documentation upon completion. This is part of our contract.

An MCP toolset for AI coding assistants to control WPS Office via natural language.

## Project Scope

This project provides an MCP Server + Skills framework enabling AI assistants (Claude Code, Cursor, Augment, Codex CLI, Windsurf, etc.) to control WPS Office.
- 231 professional MCP tools + 12 built-in tools = 243 tools in total
- Supports Excel (82 tools) / Word (28 tools) / PPT (112 tools) / Common (9 tools)
- Supports macOS, Windows, and Linux

## Prerequisites

- WPS Office installed ([https://www.wps.com/](https://www.wps.com/) or [https://www.wps.cn/](https://www.wps.cn/))
- Node.js >= 18.0.0
- Git

## Automated Installation (AI Execution)

The following steps are designed to be executed automatically by AI assistants. Steps requiring human interaction are marked with ⚠️.

### Step 1: Clone the Repository

```bash
git clone https://github.com/lc2panda/wps-skills.git
cd wps-skills
```

If the project is already cloned, skip this step.

### Step 2: Install Dependencies and Build

```bash
cd wps-office-mcp
npm install
npm run build
cd ..
```

### Step 3: Configure MCP Server

Add the following configuration to the appropriate configuration file for your AI tool. Replace the path with the actual absolute path to your project.

**Claude Code** — add to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "wps-office": {
      "command": "node",
      "args": ["/YOUR_PATH/wps-skills/wps-office-mcp/dist/index.js"]
    }
  }
}
```

**Cursor** — add to project root `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "wps-office": {
      "command": "node",
      "args": ["/YOUR_PATH/wps-skills/wps-office-mcp/dist/index.js"]
    }
  }
}
```

**OpenAI Codex CLI** — add to `~/.codex/config.toml`:
```toml
[mcp_servers.wps-office]
command = "node"
args = ["/YOUR_PATH/wps-skills/wps-office-mcp/dist/index.js"]
```
Or register via CLI: `codex mcp add wps-office -- node /YOUR_PATH/wps-skills/wps-office-mcp/dist/index.js`

**Augment / Other MCP-compatible IDEs** — Refer to your IDE's MCP Server configuration guide using the same command and args. This MCP Server is a standard stdio implementation (spec 2025-11-25) compatible with all MCP first-class clients (Claude Code, Cursor, Codex CLI, GitHub Copilot CLI, Windsurf, etc.).

### Step 4: Install WPS Add-on

⚠️ Requires manual execution (AI cannot directly control WPS UI application):

```bash
# macOS
bash scripts/auto-install-mac.sh

# Windows (PowerShell)
powershell scripts/install.ps1

# Linux
bash scripts/install.sh
```

⚠️ You must restart WPS Office after installation for the add-on to take effect.

### Step 5: Install Skills (Required for Claude Code only)

```bash
# Create skills directory if it does not exist
mkdir -p ~/.claude/skills

# Create symbolic links
ln -sf "$(pwd)/skills/wps-excel" ~/.claude/skills/wps-excel
ln -sf "$(pwd)/skills/wps-word" ~/.claude/skills/wps-word
ln -sf "$(pwd)/skills/wps-ppt" ~/.claude/skills/wps-ppt
ln -sf "$(pwd)/skills/wps-office" ~/.claude/skills/wps-office
```

### Step 6: Verify Installation

```bash
# Verify MCP Server can start
node wps-office-mcp/dist/index.js &
# You should see the "MCP Server started successfully" log
kill %1 2>/dev/null
```

## Architecture

```
Skills Layer (SKILL.md natural language guidance)
  ↓ Invoked by Claude Code
MCP Server Layer (239 tools)
  ↓ wpsClient.executeMethod()
Execution Layer
  ├── macOS: wps-claude-assistant (227 actions, HTTP polling)
  └── Windows: wps-com.ps1 (231 actions, COM interface)
```

## Tool Inventory

| Application | Tool Count | Primary Capabilities |
|-------------|------------|----------------------|
| Excel | 82 | Formulas / Data manipulation / Charts / Pivot tables / Worksheets / Formatting / Workbooks / Rows & columns / Comments & protection / Image export |
| Word | 28 | Formatting / Content editing / Document management / Headers & footers / Comments / Template filling / Paragraph structure |
| PPT | 112 | Slides / Shapes / Images / Tables / Design & beautify / Animations / Charts / 3D / Data visualization / Image export |
| Common | 9 | File saving / Connection detection / Text selection / Format conversion |
| Built-in | 12 | Connection check / Universal method call / Data caching |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| MCP connection failed | Ensure `npm install && npm run build` has been executed; verify `dist/index.js` exists. |
| WPS not responding | Restart WPS Office; confirm the add-on is installed. |
| "arguments error" | Re-run the installation script and restart WPS. |
| Plugin not found on Linux | See the Linux-specific section in [INSTALL.md](INSTALL.md). |
| Tool invocation returns null | Confirm a document of the corresponding type is open in WPS Office. |

## License

MIT
