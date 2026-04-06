# Provenance Fix — com.apple.provenance & launchd

> Load when: writing launchd plists, install scripts, or any file launchd will execute.

---

## The Problem

Every file Claude Code writes gets the `com.apple.provenance` extended attribute. macOS uses this to track files created by sandboxed apps. **launchd refuses to execute files that carry it** — `launchctl bootstrap` rejects tainted plists; `execve()` blocks tainted scripts/binaries.

| What has provenance | What breaks |
|---|---|
| plist file | `launchctl bootstrap` rejects it |
| script in ProgramArguments | launchd can't `execve()` it |
| binary in Program | launchd can't `execve()` it |

Scripts run from the terminal work fine — the terminal app is trusted. Only launchd enforces the provenance gate.

---

## Fix 1 — Generate plists from install scripts (not Claude)

A file written by `cat >` or a heredoc in a bash session has **no provenance** — bash does the write, not Claude Code:

```bash
cat > "${TARGET_PLIST}" <<PLIST
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" ...>
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>${LABEL}</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>${SCRIPT_PATH}</string>
    </array>
</dict>
</plist>
PLIST
```

## Fix 2 — Use `/bin/bash` as the Program

launchd checks the binary it forks. `/bin/bash` is a system binary with no provenance. The script is just text bash reads — no `execve()`, no provenance check:

```xml
<key>ProgramArguments</key>
<array>
    <string>/bin/bash</string>
    <string>/path/to/claude-written-script.sh</string>
</array>
```

Fix 2 does **not** help if the plist itself has provenance — Fix 1 is required for that.

## Fix 3 — Strip provenance after writing

```bash
for f in "${PLIST}" "${SCRIPT}" "${BINARY}"; do
    xattr -d com.apple.provenance "${f}" 2>/dev/null
done
# Or nuke all xattrs recursively:
xattr -cr /path/to/project/
```

---

## Rules

- **Never** symlink or copy a Claude-written plist into `~/Library/LaunchAgents/` — always generate from a heredoc
- **Always** use `/bin/bash` or `/bin/zsh` as `Program` in launchd plists, with scripts as arguments
- **Always** strip `com.apple.provenance` from deployed scripts/binaries in install scripts
- **Diagnose** with `xattr -l <file>` — presence of `com.apple.provenance` on a launchd target is the problem
- **Test** with `launchctl bootstrap gui/$(id -u) <plist>` — success is silent
