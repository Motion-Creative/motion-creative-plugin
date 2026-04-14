---
name: update
description: >
  Update the motion-creative plugin to the latest version from GitHub.
  Pulls latest changes and refreshes the local cache so new skills and
  fixes take effect. Use when: "update plugin", "get latest version",
  "refresh plugin", "update motion creative", "check for updates".
allowed-tools:
  - Bash
  - Read
---

# Update Motion Creative Plugin

Pull the latest version of the motion-creative plugin from GitHub and refresh the local cache.

---

## Step 1: Locate Plugin Paths

Find the marketplace clone and cache directories:

```bash
MARKETPLACE_DIR="$HOME/.claude/plugins/marketplaces/motion-creative"
CACHE_BASE="$HOME/.claude/plugins/cache/motion-creative/motion-creative"
```

If `$MARKETPLACE_DIR` doesn't exist, tell the user the plugin isn't installed via marketplace and stop.

## Step 2: Check Current State

```bash
# Current cached version
cat "$CACHE_BASE"/*/. claude-plugin/plugin.json 2>/dev/null | jq -r '.version'

# Current marketplace commit
git -C "$MARKETPLACE_DIR" rev-parse --short HEAD
```

## Step 3: Pull Latest

```bash
git -C "$MARKETPLACE_DIR" pull origin main --ff-only
```

If the pull fails (e.g., network error, dirty state), report the error and stop. Do not force-reset.

## Step 4: Refresh Cache

Copy the updated plugin files from the marketplace clone into the cache:

```bash
# Find the active cache version directory
INSTALL_DIR=$(ls -d "$CACHE_BASE"/*/ 2>/dev/null | head -1)

# Sync plugin contents (skills, config, MCP server)
rsync -a --delete "$MARKETPLACE_DIR/Plugin/" "$INSTALL_DIR/"
```

`rsync --delete` ensures removed files don't linger in the cache. The source is the `Plugin/` directory from the repo — it contains `.claude-plugin/`, `.mcp.json`, `skills/`, and any other plugin files.

## Step 5: Report

Show the user:

1. **Version**: previous version -> new version (read from the updated `plugin.json`)
2. **Changes**: `git -C "$MARKETPLACE_DIR" log --oneline <old-sha>..HEAD` to show what changed
3. **Action needed**: "Restart your Claude Code session or run `/reload-plugins` for changes to take effect."

If already up to date (no new commits), say so: "You're on the latest version (vX.Y.Z)."
