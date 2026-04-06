# Zsh / Oh My Zsh Safety — Pipe & Secret Traps

> Load when: writing shell scripts, piping secrets or API keys, configuring env vars.

---

## Core Problem

zsh mutates values **before** they enter a pipe, via shell expansion or Oh My Zsh hook functions injecting stray output into stdout. The pipe itself is innocent.

---

## Five Traps

**Trap 1 — Command substitution strips trailing newlines**
`$(command)` silently deletes all trailing newlines.
```zsh
# BAD
MY_KEY=$(cat ~/.config/secure_keys.sh)
# SAFE
printf '%s' "${MY_KEY}" | wrangler secret put MY_SECRET
```

**Trap 2 — Glob expansion on special characters**
Oh My Zsh enables `EXTENDED_GLOB`. Characters `#`, `~`, `^`, `*`, `?`, `[` in unquoted values are treated as glob patterns.
```zsh
# BAD
echo $MY_KEY | some_command
# SAFE
echo "${MY_KEY}" | some_command
```

**Trap 3 — History expansion eats `!`**
Inside double quotes, `!` triggers history expansion. Keys containing `abc!xyz` get mangled.
```zsh
set +o histexpand  # disable before piping
# ... pipe the value ...
set -o histexpand  # re-enable after
```

**Trap 4 — Oh My Zsh hooks pollute stdout**
`precmd`, `preexec`, and theme hooks can write to stdout and contaminate the pipe.
```zsh
# SAFE — bypass all Oh My Zsh config entirely
zsh -f -c 'source ~/.config/secure_keys.sh && printf "%s" "${MY_KEY}" | wrangler secret put MY_SECRET'
```

**Trap 5 — SH_WORD_SPLIT splits base64 keys**
Some plugins toggle `SH_WORD_SPLIT`, splitting values at whitespace.
```zsh
[[ -o shwordsplit ]] && echo "SH_WORD_SPLIT is ON — danger"
```

---

## Safe Patterns (in order of preference)

**Best — file redirect (avoids shell expansion entirely):**
```zsh
printf '%s' "${MY_KEY}" > /tmp/.secret_val
wrangler secret put MY_SECRET < /tmp/.secret_val
rm /tmp/.secret_val
```

**Good — print -r:**
```zsh
print -rn -- "${MY_KEY}" | wrangler secret put MY_SECRET
```

**Nuclear — clean-room zsh:**
```zsh
zsh -f -c 'source ~/.config/secure_keys.sh && print -rn -- "${MY_KEY}" | wrangler secret put MY_SECRET'
```

---

## Quoting Quick Reference

| Syntax | Glob-safe | History-safe | Newline-safe | Escape-safe |
|---|---|---|---|---|
| `$MY_KEY` | ✗ | ✗ | ✗ | ✗ |
| `"${MY_KEY}"` | ✓ | ✗ | ✓ | ✗ |
| `printf '%s' "${MY_KEY}"` | ✓ | ✓ | ✓ | ✓ |
| `print -rn -- "${MY_KEY}"` | ✓ | ✓ | ✓ | ✓ |

---

## Diagnostics
```zsh
printf '%s' "${MY_KEY}" | xxd | head
xxd ~/.config/secure_keys.sh | head
```

---

## Rules (always apply)
- Always `"${VAR}"` — never bare `$VAR`
- Prefer `printf '%s'` or `print -rn --` over `echo`
- Use `zsh -f` when piping secrets in any context where plugins may interfere
- Never assume a pipe is transparent — expansion happens before the pipe
- Test with `xxd` when values look correct but don't work
