# CSplit

**Split your bloated `CLAUDE.md` into a lean router — suppress context reload cost across every Claude Code session.**

---

## The Problem

`CLAUDE.md` reloads on **every single message**. Every line you store there costs tokens on turn 1, turn 2, turn 30. A 300-line `CLAUDE.md` carrying shell guides, UI mandates, and architectural notes isn't documentation — it's a per-turn tax that compounds exponentially as sessions grow.

One connected MCP server adds ~18,000 tokens per message. A bloated `CLAUDE.md` adds hundreds more. The context window fills faster, quality degrades earlier, and costs climb.

## The Solution

CSplit reads your `CLAUDE.md`, classifies each section by how often Claude actually needs it, and splits it into:

- **A lean router** — under 80 lines, loaded every turn, containing only routing entries and always-needed rules
- **Detail files** — full content, fetched on demand, invisible to turns that don't need them

```
Before:
  CLAUDE.md  ─── 340 lines reloaded every turn

After:
  CLAUDE.md              ─── 72 lines (router only)
  docs/zsh-safety.md     ─── loaded when writing shell scripts
  docs/ui-affordance.md  ─── loaded when building UI
  docs/provenance-fix.md ─── loaded when writing launchd plists
  ...
```

---

## How It Works

1. **Parse** — CSplit reads your `CLAUDE.md` and identifies logical sections
2. **Score** — each section is scored by load-frequency heuristic (always / sometimes / rarely)
3. **Split** — always-needed content stays in the router; everything else moves to `docs/`
4. **Route** — the rewritten `CLAUDE.md` gains one router entry per extracted section: `topic → path`
5. **Index** — `docs/00-INDEX.md` is generated as the authoritative map of all files and load rules

Claude Code fetches detail files only when the task needs them. Turns that don't need your Playwright MCP rules don't pay for them.

---

## Installation

```bash
cargo install csplit
```

Requires Rust 1.75+.

---

## Usage

```bash
# Split your account-level CLAUDE.md
csplit ~/.claude/CLAUDE.md --output ~/.claude/docs/

# Split a project CLAUDE.md
csplit ./CLAUDE.md --output ./docs/

# Dry run — show what would be split without writing
csplit ./CLAUDE.md --dry-run

# Validate an existing router (check line count, router format)
csplit ./CLAUDE.md --validate
```

### Options

| Flag | Default | Description |
|---|---|---|
| `--output` | `./docs/` | Directory for extracted detail files |
| `--dry-run` | false | Preview splits without writing |
| `--validate` | false | Check router line count and format only |
| `--cap` | 80 | Maximum lines for output CLAUDE.md |
| `--force` | false | Overwrite existing detail files |

---

## Router Format

CSplit rewrites section headers into single-line router entries:

```markdown
## Reference Docs
Zsh/Oh My Zsh pipe traps   → docs/zsh-safety.md
UI affordance mandate       → docs/ui-affordance.md
Playwright MCP wrapper      → docs/playwright-mcp.md
Provenance / launchd fix    → docs/provenance-fix.md
```

Claude Code reads the path reference when it needs the detail. Turns that don't need it see only one line.

---

## What Stays in the Router

CSplit keeps these categories inline — they're needed every turn:

- Stack declaration
- Planning / confidence rules
- Shell command aliases (the short versions)
- File reference discipline rules
- Sub-agent routing summary
- Session hygiene rules
- Applied Learning bullets (short)

Everything else — long guides, pattern tables, code blocks, architectural history — moves to detail files.

---

## Account vs Project Split

CSplit manages both levels:

```
~/.claude/CLAUDE.md          ← Account router — identity, stack, universal rules
~/projects/myapp/CLAUDE.md   ← Project router — inherits account, adds project specifics
```

Project-level files reference project-local `docs/`. Account-level files reference `~/.claude/docs/`. No cross-contamination.

---

## Context Savings

Based on the Context Reload Suppression Playbook:

| Optimisation | Typical saving |
|---|---|
| Lean CLAUDE.md (300 → 80 lines) | ~220 tokens × every turn |
| MCP disconnection | ~18,000 tokens × per server × every turn |
| On-demand detail loading | Detail tokens charged only when needed |

In a 30-message session with 3 MCP servers and a 300-line `CLAUDE.md`, this can represent tens of thousands of tokens saved before a single line of code is written.

---

## Compatibility

- Claude Code (all versions)
- Any project using `CLAUDE.md` / `.claude/CLAUDE.md` conventions
- macOS, Linux, ChromeOS

---

## Licence

MIT
