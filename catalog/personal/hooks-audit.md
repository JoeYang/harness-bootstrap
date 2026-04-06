# Hooks Audit

Source: `~/.claude/settings.json` → `hooks{}`

---

## Notification Hook

**Event**: `Notification` — fires when Claude Code has a message for the user (task complete, waiting, error).
**Command**: `notify-send 'Claude Code' "$CLAUDE_NOTIFICATION_MESSAGE" && paplay /usr/share/sounds/freedesktop/stereo/message-new-instant.oga &`
**What it does**: Desktop notification (notify-send) + plays a "new message" chime. Runs in background (`&`) to avoid blocking.

**Why a hook, not CLAUDE.md**: This is a system-level side effect (desktop notification + audio). CLAUDE.md instructions cannot invoke OS commands — only hooks can. Deterministic execution guaranteed on every notification.
**When useful**: Always. Essential for long-running tasks where the user switches to another window.
**When NOT**: Headless/SSH sessions without a display server (notify-send and paplay fail silently).
**Candidate for**: Global only. Personal desktop preference, not project-enforceable.

---

## PermissionRequest Hook

**Event**: `PermissionRequest` — fires when Claude Code needs permission approval for a blocked action.
**Command**: `notify-send 'Claude Code' 'Waiting for permission approval' && paplay /usr/share/sounds/freedesktop/stereo/window-attention.oga &`
**What it does**: Desktop notification + plays an "attention" sound (different from the Notification chime). Alerts the user that Claude is blocked waiting.

**Why a hook, not CLAUDE.md**: Same as Notification — OS-level side effect. The distinct sound (window-attention vs message-new-instant) lets the user distinguish "done" from "blocked" by ear alone.
**When useful**: Always. Permission blocks are time-sensitive — the agent is idle until approved.
**When NOT**: Sessions with `skipDangerousModePermissionPrompt: true` and a broad allow list rarely trigger this. Still harmless to keep.
**Candidate for**: Global only. Personal desktop preference.

---

## Stop Hook

**Event**: `Stop` — fires when Claude Code finishes a turn (completes task or stops for any reason).
**Command**: `if git diff --quiet HEAD 2>/dev/null && git diff --cached --quiet 2>/dev/null; then exit 0; fi; echo 'Code changes detected...' >&2; exit 0`
**What it does**: Checks for uncommitted changes (both staged and unstaged). If changes exist, prints a warning to stderr. Always exits 0 (never blocks).

**Why a hook, not CLAUDE.md**: CLAUDE.md can say "check for uncommitted changes" but the agent may skip it, especially when it believes the task is complete. A hook runs deterministically — no skipping possible.
**Why exit 0**: The Stop hook fires after the agent has already finished its turn. A non-zero exit signals a hook execution failure to the harness (may surface as an error), not an agent block. Exit 0 ensures the advisory stderr message is injected into context cleanly without triggering hook error handling.
**When useful**: Every git project. Catches the common failure mode where the agent says "done" with uncommitted work.
**When NOT**: Non-git directories (the `2>/dev/null` silences errors gracefully). Harmless everywhere.
**Candidate for**: Global only. Universal safety net.

---

## Summary

| Hook | Event | Purpose | Scope |
|---|---|---|---|
| Notification | Task message | Desktop alert + chime | Global (personal) |
| PermissionRequest | Permission needed | Desktop alert + attention sound | Global (personal) |
| Stop | Turn complete | Warn about uncommitted changes | Global (universal) |

**Key insight**: The two notification hooks are pure personal preference (desktop environment dependent). The Stop hook is a universal safety net that belongs in every session. All three are correctly global — none are project-specific.
