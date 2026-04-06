# AGENTS.md — CSplit Sub-Agent Rules

> Load this file when spawning or configuring sub-agents.
> Do not auto-load — this is not part of the session router.

---

## Model Allocation

| Model | Role | Usage Target |
|---|---|---|
| Sonnet | Default — coding, synthesis, decisions | Majority of turns |
| Haiku | Sub-agents — research, scanning, formatting, summarisation | As needed |
| Opus | Deep architecture only, when Sonnet is insufficient | < 20% |

---

## When to Spawn a Haiku Sub-Agent

Spawn a sub-agent when the task requires **any** of:
- 3 or more file reads
- Exploratory research across multiple sources
- Multi-file analysis or comparison
- Repetitive formatting / transformation tasks
- Summarisation of large output before returning to main session

Do **not** spawn a sub-agent for:
- Single-file edits
- Direct user questions answerable from context
- Tasks under 2 tool calls

---

## Sub-Agent Return Protocol

Sub-agents return **summaries only** — never raw output.

Mandatory return format:
```
TASK: <one-line description>
FINDINGS: <3–5 bullet summary>
FILES TOUCHED: <list if any>
DECISIONS NEEDED: <any blocker requiring Sonnet/user input>
```

Never carry exploratory output back into the main session context.

---

## MCP Server Allocation by Session Type

| Session | Keep | Disconnect |
|---|---|---|
| Coding | Filesystem MCP | Calendar, Gmail, Drive |
| Writing | Drive MCP | Filesystem, Calendar |
| Research | Web Search | Everything else |

Each connected MCP server costs ~18,000 tokens per message in static overhead.
Disconnect aggressively at session start.

---

## Context Hygiene During Agent Runs

- Sub-agent workflows cost 7–10× a standard session
- Each sub-agent reloads full context independently on wake-up
- Keep sub-agent prompts minimal — pass only what the sub-agent needs
- Never pass full conversation history to a sub-agent; summarise first
- After sub-agent returns, discard its raw working context immediately

---

## CSplit-Specific Agent Tasks

| Task | Model | Notes |
|---|---|---|
| Parse input CLAUDE.md | Haiku | Section classification only |
| Score section load-frequency | Haiku | Heuristic scoring, return table |
| Draft extracted detail doc | Sonnet | Needs quality judgement |
| Write router entry lines | Haiku | Mechanical transformation |
| Validate output CLAUDE.md line count | Haiku | Count + flag if > 80 lines |
| Architectural decisions | Sonnet / Opus | Design only, not exploration |
