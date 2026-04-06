# CSplit — Document Suite Index

> **What this is:** The reference map for every file in the CSplit context-management system.
> Claude Code reads this only when explicitly asked. It is never auto-loaded.

---

## File Hierarchy

```
~/.claude/
  CLAUDE.md                   ← Account-level router (auto-loaded every turn)

~/projects/csplit/
  CLAUDE.md                   ← Project-level router (auto-loaded every turn)
  AGENTS.md                   ← Sub-agent rules (load when spawning agents)
  README.md                   ← GitHub public-facing description
  docs/
    00-INDEX.md               ← This file
    zsh-safety.md             ← Oh My Zsh pipe/secret traps + safe patterns
    ui-affordance.md          ← Skeuomorphism mandate + Tailwind patterns
    playwright-mcp.md         ← Playwright MCP wrapper rules
    provenance-fix.md         ← com.apple.provenance launchd fix
    context-reload-playbook.md ← Full context suppression reference
    claude-md-routing-rules.md ← CLAUDE.md architecture reference
```

---

## Load Rules

| File | When to load |
|---|---|
| `~/.claude/CLAUDE.md` | Auto — every turn (keep lean) |
| `project/CLAUDE.md` | Auto — every turn (keep lean) |
| `AGENTS.md` | When spawning or configuring sub-agents |
| `docs/zsh-safety.md` | When writing shell scripts, piping secrets |
| `docs/ui-affordance.md` | When building any UI component |
| `docs/playwright-mcp.md` | When configuring or launching Playwright MCP |
| `docs/provenance-fix.md` | When writing launchd plists or install scripts |
| `docs/context-reload-playbook.md` | When diagnosing context cost or planning sessions |
| `docs/claude-md-routing-rules.md` | When editing any CLAUDE.md file |

---

## Maintenance Rules

- Account `CLAUDE.md` hard cap: **under 80 lines**
- Project `CLAUDE.md` hard cap: **under 60 lines**
- No inline content in either CLAUDE.md — router entries only
- Applied Learning bullets: under 15 words, prune monthly
- This index: update whenever a doc is added or renamed
