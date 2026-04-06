# Context Reload Suppression Playbook

> **Core insight:** Every message re-ingests the entire conversation history plus all static overhead. Cost compounds exponentially, not linearly. This is a context hygiene problem, not a plan-size problem.

---

## What Reloads on Every Message

| Source | Notes |
|---|---|
| Full conversation history | Compounds exponentially — message 30 can cost 30× message 1 |
| `CLAUDE.md` (global + project) | Loaded before every single turn |
| All connected MCP server definitions | ~18,000 tokens per server per message |
| System prompts and skills | Invisible overhead, always present |
| Explicitly loaded files | Stays in context until cleared |

---

## Layer 1 — Static Configuration

### Lean `CLAUDE.md` Architecture

`CLAUDE.md` reloads on every message — it is the highest-leverage file to optimise.

- Hard cap: **under 200 lines**
- Include only: tech stack, build commands, coding conventions, the 95%-confidence rule, key architectural decisions
- For everything else, store the *path or URL* to the detail — not the detail itself

**Example router entry:**
```
Auth conventions → see /docs/auth-conventions.md
DB schema decisions → see /docs/schema-decisions.md
```

### Codebase Map File (on-demand only)

Create a separate `CODEBASE.md` — loaded only when explicitly requested, never auto-loaded:

```markdown
# Codebase Map
src/
  auth.rs         → JWT verification, session management
  db/mod.rs       → SQLite connection pool, migrations
  ui/App.svelte   → Root component, routing
  api/routes.rs   → HTTP handlers, middleware
```

Claude fetches this only when navigating unfamiliar code. Keep it out of `CLAUDE.md`.

### Project Config: Deny Noisy Commands

In your project config, deny commands with large repetitive output:

```json
{
  "denied_commands": ["git log", "find . -name", "ls -la"]
}
```

Replace with scoped alternatives — see Layer 5 shell aliases below.

---

## Layer 2 — MCP (Model Context Protocol) Hygiene

One connected MCP server ≈ 18,000 tokens added to **every message**.

**Session-start routine:**
1. Run `/mcp`
2. Disconnect every server not needed for this specific task

**Task-based server checklist:**

| Session Type | Keep | Disconnect |
|---|---|---|
| Coding | Filesystem MCP | Calendar, email, Drive |
| Writing | Drive MCP | Filesystem, calendar |
| Research | Web search | Everything else |

**Prefer CLI equivalents** wherever they exist — e.g., `gcal` CLI instead of Google Calendar MCP. A CLI call costs zero tokens in static overhead; an MCP server costs ~18K per turn.

---

## Layer 3 — Session Lifecycle

### The 60% Compact Rule

Auto-compact fires at 95% — by then, output quality is already degraded ("lost in the middle" effect).

**Manual protocol:**
1. Monitor via status line or run `/context` periodically
2. At ~60% fill → run `/compact` with explicit preservation instructions:
   ```
   Keep: current task goal, auth module decisions, schema changes made today.
   Drop: all file exploration output, old error messages, resolved tangents.
   ```
3. After 3–4 sequential compacts, quality drifts → extract session summary → `/clear` → paste summary → continue fresh

### The 5-Minute Cache Window

Prompt caching has a hard **5-minute timeout**. Step away longer than that and the next message reprocesses everything from scratch at full cost.

- Run `/compact` or `/clear` before any break over 5 minutes
- Never leave an active session idle — return cost is a full reload

### The Edit-Not-Correct Habit

If Claude's response is wrong:

| Action | Effect |
|---|---|
| Send a correction follow-up | Stacks permanently into history — reloaded on every future message |
| Edit original message + regenerate | Replaces the bad exchange entirely — zero residual cost |

**Default behaviour: always edit, never correct.**

---

## Layer 4 — Input Discipline

### Surgical File References

| Avoid | Prefer |
|---|---|
| "Here's the whole repo, find the issue" | `@auth.rs → check verify_session, lines 45–80` |
| Pasting an entire file | Paste only the specific function |
| Pasting with comments and docs intact | Strip comments and blank lines before pasting |

Use `@filename` references rather than pastes where possible. Let Claude read what it needs; don't pre-load what it doesn't.

### Batch All Sub-Tasks Into One Message

Three messages = 3× the history reload cost of one message.

**Instead of:**
```
Message 1: Summarise the current auth flow
Message 2: Identify where the session timeout bug could originate
Message 3: Suggest a fix with minimal changes
```

**Send once:**
```
1. Summarise the current auth flow
2. Identify where the session timeout bug could originate
3. Suggest a fix with minimal changes to existing structure
```

> **Trade-off:** Multi-part prompts can sometimes reduce quality on complex tasks where focus matters. Use judgement on highly intricate work.

### Plan Mode as Standard

The costliest token event is Claude building the wrong thing — all that wasted exchange history stays loaded.

Add to `CLAUDE.md`:
```
Do not make any changes until you have 95% confidence in what needs building.
Ask clarifying questions until you reach that confidence level.
```

---

## Layer 5 — Automated Sanitisers & Shell Tooling

### Context Monitor (zsh)

Polls usage and fires a macOS notification at 50% and 70%:

```zsh
#!/bin/zsh
while true; do
  pct=$(claude /cost 2>/dev/null | grep -oP '\d+(?=%)')
  [[ -n "$pct" && "$pct" -gt 70 ]] && \
    osascript -e 'display notification "Claude context >70% — consider /compact" with title "Claude Code"'
  [[ -n "$pct" && "$pct" -gt 50 ]] && \
    osascript -e 'display notification "Claude context >50%" with title "Claude Code"'
  sleep 300
done
```

### Pre-Paste Compressor (Rust/zsh)

Strip comments, blank lines, and doc strings before pasting:

```zsh
# Rust — strip line comments and blank lines
alias rustpaste='grep -v "^\s*//" | grep -v "^\s*$" | grep -v "^\s*//!"'

# Usage:
cat src/auth.rs | rustpaste | pbcopy
```

### Safe Git Aliases

Replace raw noisy commands with scoped token-safe versions:

```zsh
alias glog='git log --oneline -20'
alias gstat='git status --short'
alias gdiff='git diff --stat'
alias gbranch='git branch --list | head -20'
```

Add these to `~/.zshrc`. Never let Claude run raw `git log` — the full output enters context.

### Targeted `find` Wrapper

```zsh
# Scoped find — max depth 3, exclude node_modules/target dirs
alias sfind='find . -maxdepth 3 -not -path "*/target/*" -not -path "*/.git/*"'
```

---

## Layer 6 — Model Selection & Sub-Agent Routing

Sub-agent workflows cost **7–10× a standard session** — each sub-agent reloads full context independently on wake-up.

### Model Allocation

| Model | Use For | Target Share |
|---|---|---|
| Sonnet | Default coding, most tasks | Majority |
| Haiku | Sub-agents: research, formatting, file scanning, summarisation | As needed |
| Opus | Deep architectural decisions only, when Sonnet isn't sufficient | <20% |

### Sub-Agent Routing Rule

Add to `CLAUDE.md`:
```
For any task requiring 3+ file reads, exploratory research, or multi-file analysis:
spawn a Haiku sub-agent. Return only summarised findings — not raw output.
```

This keeps expensive Sonnet/Opus turns focused on synthesis and decision-making, not exploration.

---

## Layer 7 — Timing & Rhythm

### Peak vs Off-Peak

| Period | Hours (ET) | Effect |
|---|---|---|
| Peak | 08:00–14:00 weekdays | Sessions drain faster |
| Off-peak | Afternoons, evenings, weekends | Normal or extended usage |

Schedule heavy refactors and multi-agent sessions for off-peak only.

### Budget Tactics

- **Near a reset with allocation remaining** → go hard, run agents, get full value
- **Near your limit with time remaining** → stop entirely; don't burn the last 5% mid-task and stall

---

## Priority Order (Biggest Wins First)

1. **Lean `CLAUDE.md`** — reloads every message; biggest static overhead reduction
2. **MCP disconnection** — 18K tokens per server per message eliminated
3. **Manual compact at 60%** — prevents quality degradation before it starts
4. **Edit not correct** — stops bad exchanges compounding through all future history
5. **Surgical file references** — prevents exploratory token sprawl
6. **Codebase map on-demand** — navigation without permanent context load
7. **Batch prompts** — multiplies value of every token already in history
8. **5-minute cache discipline** — stops idle sessions reprocessing from scratch
9. **Haiku sub-agents** — reserves expensive models for high-value turns only
10. **Shell alias sanitisers** — prevents command output bloat at source

---

## Quick-Start Checklist (Every Session)

- [ ] Run `/context` — see current token overhead before typing anything
- [ ] Run `/mcp` — disconnect all servers not needed for this task
- [ ] Check status line — model, context %, token count visible
- [ ] Start complex tasks in plan mode — no changes until 95% confidence
- [ ] Set a 5-minute rule — `/compact` or `/clear` before stepping away
- [ ] Monitor at 60% fill — manual `/compact` with preservation instructions
- [ ] Use `/clear` between unrelated tasks — never carry Topic A context into Topic B

---

*Source: Nate Herk "18 Claude Code Token Hacks in 18 Minutes" + extended research*
