# RTL Support for Claude Code in VS Code

Hebrew (RTL) support for Claude Code extension in Visual Studio Code.

## What it does

- Automatically detects Hebrew text and applies right-to-left direction
- Keeps code blocks left-to-right (as they should be)
- Works with the Claude Code chat interface in VS Code

## Installation

### Option A — Manual install (one-time)

1. Find your Claude Code extension folder:
   ```
   C:\Users\<username>\.vscode\extensions\anthropic.claude-code-*\webview\
   ```

2. Create a backup of `index.js`:
   ```
   cp index.js index.js.backup
   ```

3. Append the RTL script to `index.js`:
   ```
   cat rtl-claude-code.js >> index.js
   ```

4. Restart VS Code

### Option B — Claude Code skill (auto-maintain)

**Why this exists:** every time Claude Code updates, two things break the RTL fix:
1. The injection into `webview/index.js` is wiped (the file is replaced clean).
2. The CSS class names inside Claude Code are minified and **change between versions** (e.g. `.U.N` in one version, `.Q.Z` in the next) — so even reinjecting the old script won't work. The selectors must be updated to match the new version.

The skill automates both:
- **Diagnoses** which Claude Code versions are installed and whether RTL is currently active in each.
- **Scans** the new CSS class names from the current Claude Code build.
- **Updates** the selectors in `rtl-claude-code.js` if they changed.
- **Reinstalls** (re-injects into `webview/index.js`).

So the skill is both the *installer* and the *maintainer* — use it for first install AND every time after Claude Code updates.

**Setup:**

1. Copy the `skill/` folder into your Claude Code skills directory, renaming it to `rtl-fix`:
   ```
   cp -r skill ~/.claude/skills/rtl-fix
   ```

2. Whenever Hebrew text looks wrong (or after a Claude Code update), just tell Claude Code:
   > "תקן RTL" (or "fix RTL")

   The skill will run end-to-end and report what it found and fixed. After it's done, reload the VS Code window (`Ctrl+Shift+P` → `Reload Window`).

## Uninstall

Restore the backup:
```
cp index.js.backup index.js
```

## Credits

Based on [RTL for VS Code Agents](https://github.com/GuyRonnen/rtl-for-vs-code-agents) by GuyRonnen.

This is a simplified version for Claude Code only (without Copilot/Antigravity support).

## License

GPL-3.0 (same as original project)
