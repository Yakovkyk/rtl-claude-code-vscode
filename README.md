# RTL Support for Claude Code in VS Code

Hebrew (RTL) support for Claude Code extension in Visual Studio Code.

## What it does

- Automatically detects Hebrew text and applies right-to-left direction
- Keeps code blocks left-to-right (as they should be)
- Works with the Claude Code chat interface in VS Code

## Installation

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
