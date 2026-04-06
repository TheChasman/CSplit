# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Inherits account defaults from ~/.claude/CLAUDE.md.
> This file contains project-specific overrides only.

## Project
**CSplit** — splits bloated CLAUDE.md files into a lean router +
external detail docs, suppressing context reload cost across Claude Code sessions.

## Stack
- Rust CLI binary (no UI in v1), requires Rust 1.75+
- Account defaults apply (Svelte, SQLite3, D1 reserved for later versions)

## Build & Test (once implementation exists)
```
cargo build                    # compile
cargo run -- <path> [--flags]  # run locally
cargo test                     # all tests
cargo test <name>              # single test
cargo clippy                   # lint
cargo fmt --check              # format check
```

## CLI Interface (planned)
```
csplit <path> --output <dir>   # split CLAUDE.md into router + detail docs
csplit <path> --dry-run        # preview without writing
csplit <path> --validate       # check router format and line count
csplit <path> --cap <n>        # max output lines (default 80)
csplit <path> --force          # overwrite existing detail files
```

## Architecture (planned pipeline)
1. **Parser** — reads CLAUDE.md, identifies logical sections by heading structure
2. **Scorer** — classifies each section: always-load / sometimes-load / rarely-load
3. **Splitter** — keeps always-load inline, moves rest to `docs/` as detail files
4. **Router** — rewrites CLAUDE.md with `topic -> path` entries (one line per topic)
5. **Index Generator** — creates `docs/00-INDEX.md` with file map and load rules

Output constraints: router CLAUDE.md under 80 lines, detail files in `docs/`.

## Project Docs (load on demand only)
Document suite index       -> docs/00-INDEX.md
Agent routing rules        -> AGENTS.md
Context reload playbook    -> docs/context-reloading-playbook.md
Full README                -> docs/README.md

## Key Rules
- Never auto-load detail files -- explicit fetch only
- Account CLAUDE.md cap: 80 lines; project CLAUDE.md cap: 60 lines
- Router entries: `topic -> path`, one line per topic, no inline content

## Applied Learning
One-line bullets, under 15 words, no explanations. Prune monthly.
-
