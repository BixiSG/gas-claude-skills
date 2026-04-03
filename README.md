# GAS (Google App Script) Claude Skills

Private collection of custom Claude Code skills.

## Skills

| Skill | Description |
|-------|-------------|
| [gas-workflow](gas-workflow/SKILL.md) | Workflow for creating, storing, editing, and deploying Google Apps Script projects. Handles folder setup, local/GitHub storage, clasp CLI integration, and push-with-notes discipline. |

## Usage

These skills auto-trigger in Claude Code when relevant context is detected, or can be invoked manually via slash command (e.g., `/gas-workflow`).

## Installation

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Git installed

### Option 1: Install a single skill (recommended)

1. Clone this repo:
   ```bash
   git clone https://github.com/BixiSG/dark-claude-skills.git
   ```

2. Copy the skill folder you want into your Claude skills directory:
   ```bash
   # Linux / macOS
   cp -r dark-claude-skills/gas-workflow ~/.claude/skills/

   # Windows (PowerShell)
   Copy-Item -Recurse dark-claude-skills\gas-workflow $env:APPDATA\Claude\skills\
   ```

3. Restart Claude Code (or start a new session). The skill is now active.

### Option 2: Install all skills at once

```bash
# Linux / macOS
cp -r dark-claude-skills/*/ ~/.claude/skills/

# Windows (PowerShell)
Get-ChildItem -Directory dark-claude-skills | Where-Object { $_.Name -ne '.git' } | Copy-Item -Recurse -Destination $env:APPDATA\Claude\skills\
```

### Verify installation

In Claude Code, type `/gas-workflow` — if it triggers, you're good. Skills also auto-trigger when you mention relevant topics (e.g., "write me an Apps Script").

### Updating skills

Pull the latest changes and re-copy:
```bash
cd dark-claude-skills
git pull
cp -r gas-workflow ~/.claude/skills/
```

### Uninstalling a skill

Delete the skill folder from your skills directory:
```bash
# Linux / macOS
rm -rf ~/.claude/skills/gas-workflow

# Windows (PowerShell)
Remove-Item -Recurse $env:APPDATA\Claude\skills\gas-workflow
```
