# Hooks Cookbook

Copy-paste recipes for `.claude/settings.json` hooks. Merge each snippet into the `"hooks"` object.

### Auto-format on Edit

**Event**: `PostToolUse` (matcher: `Edit|Write`) -- Format files after Claude edits them.

```json
"PostToolUse": [{
  "matcher": "Edit|Write",
  "hooks": [{
    "type": "command",
    "command": "FILE=$(echo $CLAUDE_TOOL_INPUT | jq -r '.file_path // .filePath // empty') && [ -n \"$FILE\" ] && ruff format --quiet \"$FILE\"",
    "timeout": 10000
  }]
}]
```

Swap the formatter command per language:
- **prettier**: `npx prettier --write \"$FILE\" 2>/dev/null`
- **clang-format**: `clang-format -i \"$FILE\"`

`$CLAUDE_TOOL_INPUT` is JSON; `jq` extracts the file path. Exit code is ignored for PostToolUse (advisory). Set `timeout` to avoid hanging.

### Branch Protection

**Event**: `PreToolUse` (matcher: `Edit|Write`) -- Block edits on main/master.

```json
"PreToolUse": [{
  "matcher": "Edit|Write",
  "hooks": [{
    "type": "command",
    "command": "BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null) && case \"$BRANCH\" in main|master) echo \"Blocked: you are on $BRANCH. Create a feature branch first.\" >&2; exit 2;; esac; exit 0"
  }]
}]
```

Exit 2 blocks the tool and feeds stderr back to Claude. Exit 0 allows. Other exit codes signal hook errors (not blocks). Based on the Trail of Bits defensive pattern.

### File Protection

**Event**: `PreToolUse` (matcher: `Edit|Write`) -- Block edits to .env, lock files, CI configs.

```json
"PreToolUse": [{
  "matcher": "Edit|Write",
  "hooks": [{
    "type": "command",
    "command": "FILE=$(echo $CLAUDE_TOOL_INPUT | jq -r '.file_path // .filePath // empty') && case \"$FILE\" in *.env|*.env.*|*lock.json|*lock.yaml|*.lock|*/.github/workflows/*) echo \"Blocked: $FILE is protected.\" >&2; exit 2;; esac; exit 0"
  }]
}]
```

Adjust the `case` patterns to suit. Combine with branch protection by adding both to the `PreToolUse` array -- all must pass.

### Auto-lint After Write

**Event**: `PostToolUse` (matcher: `Write`) -- Lint new files so Claude sees warnings on the next turn.

```json
"PostToolUse": [{
  "matcher": "Write",
  "hooks": [{
    "type": "command",
    "command": "FILE=$(echo $CLAUDE_TOOL_INPUT | jq -r '.file_path // .filePath // empty') && [ -n \"$FILE\" ] && case \"$FILE\" in *.py) ruff check --quiet \"$FILE\" >&2;; *.ts|*.tsx|*.js|*.jsx) npx eslint \"$FILE\" >&2;; esac; exit 0",
    "timeout": 15000
  }]
}]
```

Lint output goes to stderr so Claude sees it. Always exits 0 -- PostToolUse cannot block. Claude self-corrects on the next turn.

### Uncommitted Changes Warning

**Event**: `Stop` -- Warn when Claude finishes a turn with uncommitted changes.

```json
"Stop": [{
  "hooks": [{
    "type": "command",
    "command": "if git diff --quiet HEAD 2>/dev/null && git diff --cached --quiet 2>/dev/null; then exit 0; fi; echo 'Code changes detected. Run git diff to review uncommitted changes.' >&2; exit 0",
    "statusMessage": "Checking for code changes..."
  }]
}]
```

Always exit 0. Stop hooks fire after the turn -- non-zero signals a hook error, not a block.

### Desktop Notifications

**Event**: `Notification` + `PermissionRequest` -- Desktop alerts with distinct sounds.

```json
"Notification": [{
  "hooks": [{
    "type": "command",
    "command": "notify-send 'Claude Code' \"$CLAUDE_NOTIFICATION_MESSAGE\" && paplay /usr/share/sounds/freedesktop/stereo/message-new-instant.oga &"
  }]
}],
"PermissionRequest": [{
  "hooks": [{
    "type": "command",
    "command": "notify-send 'Claude Code' 'Waiting for permission approval' && paplay /usr/share/sounds/freedesktop/stereo/window-attention.oga &"
  }]
}]
```

Distinct sounds let you tell "done" from "blocked" by ear. The `&` backgrounds playback. Linux only (notify-send + PulseAudio). On macOS use `osascript` + `afplay`.

### Context Re-injection After Compaction

**Event**: `PostCompact` -- Re-inject critical rules after `/compact` compresses context.

```json
"PostCompact": [{
  "hooks": [{
    "type": "command",
    "command": "echo 'REMINDER: Always run tests before committing. Never edit files on main branch. Check CLAUDE.md for project boundaries.' >&2; exit 0"
  }]
}]
```

Keep the message short -- you are burning context tokens. Focus on rules the agent most often forgets after compaction. Exit 0 always.
