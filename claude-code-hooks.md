# Claude Code Hooks

> **How to use:** Hooks run shell commands automatically in response to Claude Code events. Configure them in `.claude/settings.json` (project-level) or `~/.claude/settings.json` (global).

---

## Hook Anatomy

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolName",
        "hooks": [
          {
            "type": "command",
            "command": "your shell command here"
          }
        ]
      }
    ]
  }
}
```

### Events

| Event | Fires |
|-------|-------|
| `PreToolUse` | Before Claude calls a tool |
| `PostToolUse` | After Claude calls a tool |
| `Notification` | When Claude sends a notification |
| `Stop` | When Claude finishes a response |

### Matchers

Match on tool name: `"Write"`, `"Edit"`, `"Bash"`, `"*"` (all tools).

---

## Recipes

### Auto-lint after file edits

Run ESLint after any file write or edit.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{ "type": "command", "command": "npm run lint --silent 2>&1 | head -20" }]
      },
      {
        "matcher": "Edit",
        "hooks": [{ "type": "command", "command": "npm run lint --silent 2>&1 | head -20" }]
      }
    ]
  }
}
```

---

### Auto-format on save

Run Prettier after writes.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{ "type": "command", "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null" }]
      }
    ]
  }
}
```

> `$CLAUDE_TOOL_INPUT_FILE_PATH` is set automatically to the file path passed to the tool.

---

### Run tests after edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{ "type": "command", "command": "npm test --silent 2>&1 | tail -20" }]
      }
    ]
  }
}
```

---

### Desktop notification on completion

Get notified when a long task finishes (macOS).

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "*",
        "hooks": [{ "type": "command", "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'" }]
      }
    ]
  }
}
```

Linux (requires `notify-send`):
```json
{ "type": "command", "command": "notify-send 'Claude Code' 'Task complete'" }
```

---

### Log all Bash commands

Audit trail of every shell command Claude runs.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) $CLAUDE_TOOL_INPUT_COMMAND\" >> ~/.claude/bash-audit.log" }]
      }
    ]
  }
}
```

---

### Block dangerous commands

Intercept destructive Bash calls (exit non-zero to block).

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "echo \"$CLAUDE_TOOL_INPUT_COMMAND\" | grep -qE '(rm -rf|DROP TABLE|git push --force)' && exit 1 || exit 0" }]
      }
    ]
  }
}
```

> Hook exits with non-zero → Claude Code blocks the tool call and surfaces the error.

---

### Type-check after TypeScript edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{ "type": "command", "command": "npx tsc --noEmit 2>&1 | head -30" }]
      }
    ]
  }
}
```

---

## Environment Variables Available in Hooks

| Variable | Value |
|----------|-------|
| `$CLAUDE_TOOL_NAME` | Name of the tool being called |
| `$CLAUDE_TOOL_INPUT_COMMAND` | The Bash command (Bash tool only) |
| `$CLAUDE_TOOL_INPUT_FILE_PATH` | File path (Write/Read/Edit tools) |
| `$CLAUDE_TOOL_INPUT_CONTENT` | File content being written |

---

## Tips

- **Output is shown to Claude.** Hook stdout feeds back into context — use it for validation messages.
- **Exit 1 to block.** A non-zero exit in `PreToolUse` stops the tool call.
- **Keep hooks fast.** Slow hooks block every tool call.
- **Project hooks beat global hooks.** `.claude/settings.json` overrides `~/.claude/settings.json` for same-named events.
