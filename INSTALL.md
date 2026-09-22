# WPS Skills Installation Guide

[English](INSTALL.md) | [Tiếng Việt](INSTALL_vi.md) | [中文](INSTALL_zh.md)

> **This document is intended for AI coding assistants** (Claude Code, Cursor, Codex CLI, etc.). After reading this document, you should fully understand the installation process and automatically execute all steps in sequence without skipping.

---

## Pre-Installation Status Check

Before executing the installation, first determine whether it has already been installed. Run the following checks in sequence:

```bash
# 1. Check if MCP Server is registered
claude mcp list 2>/dev/null | grep -i wps

# 2. Check if Skills are registered
ls ~/.claude/skills/wps-excel/SKILL.md 2>/dev/null

# 3. Check if build artifact exists
ls wps-office-mcp/dist/index.js 2>/dev/null
```

**Evaluation Logic**:
- All 3 items exist → Already installed, no need to reinstall. Inform user: "WPS Skills is already installed. Restart Claude Code and WPS Office to use it."
- Partially exists → Resume installation from the missing steps.
- None exists → Execute full installation workflow.

---

## macOS Installation

### Step 1: Check Prerequisites

```bash
# Check operating system (must be Darwin)
uname -s

# Check if WPS Office is installed
ls /Applications/wpsoffice.app

# Check Node.js version (must be >= 18.0.0)
node -v
```

**If prerequisites are not met**:
- WPS Office not found → Ask user to install: https://www.wps.com/
- Node.js not found or version < 18 → Ask user to install: https://nodejs.org/
- Stop installation immediately if prerequisites are not met.

### Step 2: Run One-Click Installation Script

```bash
# Enter project root directory
cd <PROJECT_ROOT>

# Grant execute permissions and run
chmod +x scripts/auto-install-mac.sh
./scripts/auto-install-mac.sh
```

The script automatically performs all of the following operations:
- Double-checks prerequisites (macOS, WPS Office, Node.js 18+)
- Creates WPS add-on directory if it does not exist
- Installs add-on files to `~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/` (trailing `_` in directory name is mandatory)
- Updates publish.xml to register the add-on
- Runs `npm install` to install dependencies
- Runs `npm run build` to compile TypeScript
- Runs `claude mcp add wps-office` to register MCP Server
- Creates symlinks to register Skills in `~/.claude/skills/`

### Step 3: Post-Installation Verification

Run the following checks; all must pass for installation to be successful:

```bash
# 1. MCP Server registered (output should contain wps-office)
claude mcp list

# 2. Skills registered (should list wps-excel, wps-word, wps-ppt, wps-office)
ls ~/.claude/skills/

# 3. Build artifact exists
ls wps-office-mcp/dist/index.js

# 4. WPS add-on installed (directory name ends with _)
ls ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/

# 5. publish.xml registered (output should contain claude-assistant)
grep "claude-assistant" ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/publish.xml
```

### Step 4: Inform User

After installation completes, you must inform the user to:
1. **Restart Claude Code** (Mandatory! Skills are only loaded on startup)
2. Restart WPS Office
3. Open any Excel/Word/PPT document
4. Check the "Claude Assistant" tab in WPS Office toolbar and verify status shows "Polling"

### ⚠️ Known Issues

**macOS Sandbox Directory Permission Dialog**
- Symptom: When running verification accessing `~/Library/Containers/com.kingsoft.wpsoffice.mac/...` for the first time, macOS will prompt for file access permission.
- Action: Simply click "Allow". It will not prompt again.
- Impact: Normal macOS sandbox mechanism, does not affect installation.

---

## Linux Installation

### Step 1: Check Prerequisites

```bash
# Check operating system (must be Linux)
uname -s

# Check if WPS Office is installed
which wps || ls /opt/kingsoft/wps-office

# Check Node.js version (must be >= 18.0.0)
node -v
```

**If prerequisites are not met**:
- WPS Office not found -> Ask user to install: https://linux.wps.com
- Node.js not found or version < 18 -> Ask user to install: https://nodejs.org/

### Step 2: Manual Installation

```bash
# Enter project root directory
cd <PROJECT_ROOT>

# Install dependencies and build
cd wps-office-mcp
npm install
rm -rf dist
npm run build
cd ..

# Copy add-on to WPS directory (directory name must end with _)
mkdir -p ~/.local/share/Kingsoft/wps/jsaddons
cp -R wps-claude-assistant ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_

# Create publish.xml
# Note: enable="enable_dev" is development mode (default); change to enable="true" (production mode) if loading fails
cat > ~/.local/share/Kingsoft/wps/jsaddons/publish.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<jsplugins>
  <jsplugin name="claude-assistant" type="wps,et,wpp" url="claude-assistant_/" enable="enable_dev"/>
</jsplugins>
EOF

# Register MCP Server
claude mcp add wps-office node $(pwd)/wps-office-mcp/dist/index.js

# Register Skills
mkdir -p ~/.claude/skills
ln -sf $(pwd)/skills/wps-excel ~/.claude/skills/wps-excel
ln -sf $(pwd)/skills/wps-word ~/.claude/skills/wps-word
ln -sf $(pwd)/skills/wps-ppt ~/.claude/skills/wps-ppt
ln -sf $(pwd)/skills/wps-office ~/.claude/skills/wps-office
```

### Step 3: Post-Installation Verification

```bash
# 1. MCP Server registered
claude mcp list

# 2. Skills registered
ls ~/.claude/skills/

# 3. Build artifact exists
ls wps-office-mcp/dist/index.js

# 4. WPS add-on installed (directory name ends with _)
ls ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_/

# 5. publish.xml registered
grep "claude-assistant" ~/.local/share/Kingsoft/wps/jsaddons/publish.xml
```

### Step 4: Inform User

1. **Restart Claude Code** (Mandatory!)
2. Restart WPS Office
3. Open any document, check "Claude Assistant" tab

### ⚠️ Linux Startup Order (Important)

Issue #17 report: On certain distributions such as Kylin Linux, if WPS is already running before Claude Code (MCP Server) starts, the WPS process may terminate unexpectedly.

**Correct startup order**:
1. Start **Claude Code** first (initiating MCP Server)
2. Then launch **WPS Office** (add-on HTTP polling will discover ready port 58891)
3. Use WPS MCP tools

If WPS is already running, run `pkill -9 wps && pkill -9 wpp && pkill -9 et` before following the startup order above.

### Linux Key Paths Reference

| Item | Path |
|------|------|
| WPS Add-ons Base Directory | `~/.local/share/Kingsoft/wps/jsaddons/` |
| Add-on Install Directory | `<Base Directory>/claude-assistant_/` (trailing `_` mandatory) |
| publish.xml | `<Base Directory>/publish.xml` |

---

## Windows Installation

### Step 1: Check Prerequisites

```powershell
# Check if WPS Office add-ons directory exists
Test-Path "$env:APPDATA\kingsoft\wps\jsaddons"

# Check Node.js version (must be >= 18.0.0)
node -v
```

**If prerequisites are not met**:
- WPS add-ons directory not found → Ask user to install WPS Office: https://www.wps.com/
- Node.js not found or version < 18 → Ask user to install: https://nodejs.org/
- Stop installation immediately if prerequisites are not met.

### Step 2: Run One-Click Installation Script

```powershell
# Enter project root directory
cd <PROJECT_ROOT>

# Execute installation script
powershell -ExecutionPolicy Bypass -File scripts/auto-install.ps1
```

The script automatically performs all of the following operations:
- Checks Node.js 18+ version
- Runs `npm install` to install dependencies
- Runs `npm run build` to compile TypeScript
- Configures Claude Code MCP (writes to `%USERPROFILE%\.claude\settings.json`)
- Copies Skills to `%USERPROFILE%\.claude\skills\`
- Installs WPS add-on to `%APPDATA%\kingsoft\wps\jsaddons\wps-claude-addon_\` (trailing `_` is mandatory)
- Updates publish.xml to register the add-on

### Step 3: Post-Installation Verification

```powershell
# 1. MCP Server registered
claude mcp list

# 2. Skills registered (should list wps-excel, wps-word, wps-ppt, wps-office)
Get-ChildItem "$env:USERPROFILE\.claude\skills"

# 3. Build artifact exists
Test-Path "wps-office-mcp\dist\index.js"

# 4. WPS add-on installed
Test-Path "$env:APPDATA\kingsoft\wps\jsaddons\wps-claude-addon_"

# 5. publish.xml registered add-on
Select-String -Path "$env:APPDATA\kingsoft\wps\jsaddons\publish.xml" -Pattern "wps-claude-addon"
```

### Step 4: Inform User

After installation completes, you must inform the user to:
1. **Restart Claude Code** (Mandatory!)
2. Restart WPS Office
3. Check the "Claude Assistant" tab in WPS

---

## Key Paths Reference

### macOS

| Item | Path |
|------|------|
| WPS Add-on Base Directory | `~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/` |
| Add-on Install Directory | `<Base Directory>/claude-assistant_/` (trailing `_` mandatory) |
| publish.xml | `<Base Directory>/publish.xml` |
| Skills Registration Directory | `~/.claude/skills/` (4 symlinks) |
| MCP Server Entrypoint | `<PROJECT_ROOT>/wps-office-mcp/dist/index.js` |
| HTTP Polling Port | `58891` |

### Windows

| Item | Path |
|------|------|
| WPS Add-on Base Directory | `%APPDATA%\kingsoft\wps\jsaddons\` |
| Add-on Install Directory | `<Base Directory>\wps-claude-addon_\` (trailing `_` mandatory) |
| publish.xml | `<Base Directory>\publish.xml` |
| Skills Registration Directory | `%USERPROFILE%\.claude\skills\` (copied, not symlinked) |
| MCP Server Configuration | `%USERPROFILE%\.claude\settings.json` |

---

## Troubleshooting & Error Handling

Follow the reference table below when encountering installation errors:

### npm install fails

```bash
# Clear cache and retry
cd wps-office-mcp
rm -rf node_modules package-lock.json
npm install
```

If it still fails, verify Node.js version:
```bash
node -v
# Must be >= 18.0.0, otherwise upgrade Node.js
```

### npm run build (TypeScript compilation) fails

```bash
cd wps-office-mcp
rm -rf dist node_modules
npm install
npm run build
```

If you see `tsc: command not found`, typescript is not installed as a dependency. Check if typescript is listed in package.json devDependencies.

### MCP Server registration fails

Manual registration:
```bash
claude mcp add wps-office node <ABSOLUTE_PROJECT_PATH>/wps-office-mcp/dist/index.js
```

Note: `<ABSOLUTE_PROJECT_PATH>` must be replaced with the actual path; do not use relative paths or environment variables.

### Skills symlink creation fails

```bash
PROJECT_DIR=<ABSOLUTE_PROJECT_PATH>
mkdir -p ~/.claude/skills
ln -sf "$PROJECT_DIR/skills/wps-excel" ~/.claude/skills/wps-excel
ln -sf "$PROJECT_DIR/skills/wps-word" ~/.claude/skills/wps-word
ln -sf "$PROJECT_DIR/skills/wps-ppt" ~/.claude/skills/wps-ppt
ln -sf "$PROJECT_DIR/skills/wps-office" ~/.claude/skills/wps-office
```

Verification:
```bash
ls -la ~/.claude/skills/
# Should display 4 symlinks pointing to skills/ subdirectories in the project
```

### WPS add-on does not show "Claude Assistant" tab

1. Confirm add-on directory was copied correctly and ends with `_`:
```bash
# macOS
ls ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/
# Should contain main.js, manifest.xml, ribbon.xml, etc.
```

2. Confirm publish.xml contains registration entry:
```bash
# macOS
cat ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/publish.xml
# Should contain <jsplugin name="claude-assistant" .../>
```

3. Force quit and restart WPS:
```bash
# macOS
pkill -f wpsoffice
sleep 2
open /Applications/wpsoffice.app
```

### HTTP polling port 58891 in use (macOS)

```bash
# Check port usage
lsof -i :58891

# Terminate process occupying port
kill <PID>
```

### macOS add-on directory permissions error

```bash
# Manually create directory
mkdir -p ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons

# Fix permissions
chmod -R 755 ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons
```

---

## Known Issues (GitHub Issues)

### Issue #6: MCP Connection Failed ("Failed to connect")

**Symptom**: `claude mcp list` shows `wps-office: Failed to connect`.

**Root cause**: Outdated build artifacts in dist directory where duplicate tool name registrations cause the MCP Server to crash on startup.

**Resolution**:
```bash
cd wps-office-mcp
# Clean old build artifacts and recompile (critical step!)
rm -rf dist
npm run build
# Verify server starts cleanly (should see "Server started successfully")
node dist/index.js 2>&1 | head -5
# Press Ctrl+C to exit
```

If it still fails, complete clean rebuild:
```bash
cd wps-office-mcp
rm -rf dist node_modules
npm install
npm run build
```

Then re-register MCP:
```bash
claude mcp remove wps-office
claude mcp add wps-office node <ABSOLUTE_PROJECT_PATH>/wps-office-mcp/dist/index.js
```

### Issue #5: WPS cannot find add-on on Linux

**Symptom**: Add-on installs successfully on Linux (e.g. Arch Linux), but the Claude Assistant tab does not appear in WPS.

**Root cause**: Installation script lacked trailing `_` suffix on the Linux add-on directory, and publish.xml was missing. WPS jsaddons specification requires the directory name to end with `_` to be recognized.

**Resolution**:
```bash
# 1. Check current install path (incorrect old path)
ls ~/.local/share/Kingsoft/wps/jsaddons/wps-claude-addon 2>/dev/null

# 2. If old directory exists, remove it
rm -rf ~/.local/share/Kingsoft/wps/jsaddons/wps-claude-addon

# 3. Copy manually to correct path (directory name MUST end with _)
cp -R <PROJECT_ROOT>/wps-claude-assistant ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_

# 4. Create publish.xml (also required on Linux)
cat > ~/.local/share/Kingsoft/wps/jsaddons/publish.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<jsplugins>
  <jsplugin name="claude-assistant" type="wps,et,wpp" url="claude-assistant_/" enable="enable_dev"/>
</jsplugins>
EOF

# 5. Restart WPS
pkill -f wps
# Then reopen WPS
```

**Note**: Add-on directories may vary across Linux distributions. Common paths:
- Standard: `~/.local/share/Kingsoft/wps/jsaddons/`
- Some distros: `~/.kingsoft/wps/jsaddons/`

If neither works, search for the directory:
```bash
find / -path "*/Kingsoft/wps/jsaddons" -type d 2>/dev/null
find / -path "*kingsoft/wps/jsaddons" -type d 2>/dev/null
```

### Issue #4: WPS Add-on startup error "arguments error"

**Symptom**: WPS displays popup `ERROR: arguments error at <anonymous>:1:89`.

**Root cause**: Missing `<ribbon>` and `<scripts>` tag declarations in manifest.xml, causing WPS to fail resolving the add-on entrypoint.

**Resolution**:

`wps-claude-assistant/manifest.xml` has been patched with complete ribbon and scripts declarations. If you still encounter this error:

```bash
# 1. Check if manifest.xml has ribbon and scripts declarations
grep -E "ribbon|scripts" <ADDON_INSTALL_DIR>/manifest.xml
# You should see:
#   <ribbon src="ribbon.xml"/>
#   <script src="main.js"/>

# 2. Confirm ribbon.xml and main.js exist
ls <ADDON_INSTALL_DIR>/ribbon.xml
ls <ADDON_INSTALL_DIR>/main.js

# 3. Re-copy add-on using the latest version
# macOS:
rm -rf ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_
cp -R <PROJECT_ROOT>/wps-claude-assistant ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_

# 4. Restart WPS
pkill -f wpsoffice
sleep 2
open /Applications/wpsoffice.app
```
