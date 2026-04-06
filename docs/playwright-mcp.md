# Playwright MCP — Wrapper Rules

> Load when: configuring or launching Playwright MCP.

---

## The Problem

Vanilla `npx @playwright/mcp@latest` spawns headless Chrome instances (`mcp-chrome-9bc5202`) that persist as ghost processes — invisible to CMD+Tab, accumulating across sessions.

---

## Always Use the Wrapper

The wrapper lives in `~/.zshrc`:

```zsh
playwright-mcp() {
  pkill -f "playwright-mcp" 2>/dev/null
  pkill -f "mcp-chrome-9bc5202" 2>/dev/null
  sleep 1
  trap 'pkill -f "mcp-chrome-9bc5202" 2>/dev/null' EXIT INT TERM
  npm exec @playwright/mcp@latest "$@"
}
```

What it does:
1. Kills stale instances from previous sessions
2. Registers a cleanup trap on EXIT, INT, TERM
3. Delegates all args to the real MCP

---

## Rules

- **Never** launch via bare `npx @playwright/mcp@latest`
- In `settings.json` or `.mcp.json`, use `playwright-mcp` as the command
- Ghost processes suspected? Run: `pkill -f "mcp-chrome-9bc5202"`
