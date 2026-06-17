# Claude Code CLI — Statusline

The **status line** is a customizable bar pinned to the bottom of Claude Code that runs any shell script you configure. It receives JSON session data on stdin and displays whatever your script prints, giving you a persistent, at-a-glance view of context usage, costs, git status, model, working directory, or anything else you want to track.

It runs locally, does **not** consume API tokens, and hides temporarily during UI interactions like autocomplete suggestions, the help menu, and permission prompts.

---

## TL;DR — My statusline prompt

Paste this into Claude Code (`/statusline` will pick it up, or just send it as a message) to generate the script:

```markdown
Set up my Claude Code statusline

Configure my Claude Code statusline script at ~/.claude/statusline-command.sh to display:

<model> [<progress-bar> <ctx%> <used>k / <total>k tokens] <dir> git:<reponame> branch:<branch> <git-status> | 5h:<5h%> 7d:<7d%>

Behavior:

- Model: display name from statusline JSON input

- Context window: read context_window.used_percentage (fall back to 0 if null).
  - Render a 10-segment progress bar using ▓ for filled and ░ for empty
    (filled = round(pct/10)).
  - Follow it with the percentage and token counts: <used>k / <total>k, where
    used = context_window.total_input_tokens and total = context_window.context_window_size.
  - Color the bar + percentage green / yellow (≥60%) / red (≥85%).
  - Note: used_percentage is input-only (input + cache_creation + cache_read,
    excluding output) — if you compute it yourself, match that formula.

- Directory: basename of cwd (workspace.current_dir)

- Git section (when in a git repo):
  - Format: git:<reponame> branch:<branch>, where reponame = basename of
    $(git rev-parse --show-toplevel) and branch = git branch --show-current.
  - Append a status indicator: count staged changes (git diff --cached --numstat)
    shown as green +N, and modified/unstaged changes (git diff --numstat) shown as
    yellow ~N.
  - Overall git color: green = clean + pushed, yellow = unpushed commits
    (check git rev-list @{upstream}..HEAD), red = dirty working tree.
  - Use -c core.fsmonitor=false on every git invocation to avoid interfering with
    running processes.
  - Falls back to dir-only when not in a git repo.

- Rate limits (I'm on the Claude Pro plan):
  - Read rate_limits.five_hour.used_percentage and rate_limits.seven_day.used_percentage.
  - Display as: 5h:<n>% 7d:<n>%, color-coded green / yellow (≥60%) / red (≥85%).
  - IMPORTANT: the rate_limits object is only present after the first API response of
    a session. Guard every read with // empty and render NOTHING for this segment when
    the field is absent (do not show 0% or an error) — it will populate on the next update.
  - Optional: if rate_limits.five_hour.resets_at (Unix epoch seconds) is present, you
    may append a short reset time, e.g. (resets 3:15pm), using date -d @<ts> (GNU) or
    date -r <ts> (BSD/macOS).

- Performance: the script runs on every update, so cache the git results to a temp
  file keyed on session_id (from the JSON, NOT $$ — process IDs change each run) with
  a ~5 second TTL, so git status/diff don't lag in large repos.

- Robustness: guard all jq reads with // 0 or // empty so missing/null fields never
  error. Use printf '%b' rather than echo -e so ANSI escapes render reliably over
  SSH/tmux.

Wire it up in ~/.claude/settings.json under the statusLine field
(type: "command", command: path to the script). The key is "statusLine", not "statusCommand".
```

---

## When to use it

A status line is useful when you:

- Want to monitor context-window usage as you work
- Need to track session costs
- Work across multiple sessions and need to distinguish them
- Want git branch and status always visible

---

## Setting it up

There are two paths: let Claude Code generate the script for you, or wire one up manually.

### Option 1 — the `/statusline` command (easy)

The `/statusline` command accepts natural-language instructions describing what you want displayed. Claude Code generates a script file in `~/.claude/` and updates your settings automatically:

```text
/statusline show model name and context percentage with a progress bar
```

To remove it, run `/statusline` and ask it to delete or clear the status line (e.g. `/statusline delete`), or manually remove the `statusLine` field from `settings.json`.

### Option 2 — manual configuration

Add a `statusLine` field to your user settings (`~/.claude/settings.json`) or project settings. Set `type` to `"command"` and point `command` at a script path or an inline shell command:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2
  }
}
```

The `command` field runs in a shell, so inline commands work too. For example, using `jq`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'"
  }
}
```

#### Optional config fields

| Field | Default | Purpose |
| --- | --- | --- |
| `padding` | `0` | Extra horizontal spacing (in characters), added on top of the interface's built-in spacing. Controls relative indentation. |
| `refreshInterval` | unset | Re-runs the command every N seconds in addition to event-driven updates (minimum `1`). Use for time-based data (clocks) or when background subagents change git state while the session is idle. |
| `hideVimModeIndicator` | `false` | Suppresses the built-in `-- INSERT --` text. Set `true` when your script renders `vim.mode` itself, to avoid showing it twice. |

---

## How it works

Claude Code runs your script and pipes JSON session data to it via stdin. Your script reads the JSON, extracts what it needs, and prints text to stdout. Claude Code displays whatever your script prints.

**When it updates:** after each new assistant message, after `/compact` finishes, when the permission mode changes, or when vim mode toggles. Updates are debounced at 300ms, so rapid changes batch together. If a new update fires while the script is still running, the in-flight execution is cancelled. Edits to your script don't appear until the next interaction triggers an update. Event triggers go quiet when the main session is idle (e.g. a coordinator waiting on background subagents) — set `refreshInterval` to keep time-based segments current during idle periods.

**What your script can output:**

- **Multiple lines** — each `echo`/`print` displays as a separate row.
- **Colors** — ANSI escape codes like `\033[32m` (green); the terminal must support them.
- **Links** — OSC 8 escape sequences make text clickable (Cmd+click on macOS, Ctrl+click on Windows/Linux). Requires a terminal with hyperlink support (iTerm2, Kitty, WezTerm).

**Sizing to the terminal:** Claude Code captures the script's output rather than connecting it to the terminal, so `tput cols` and language-level width detection won't work from inside the script. Read the `COLUMNS` and `LINES` environment variables instead (Claude Code sets these before running your script; requires v2.1.153+).

---

## Available data

Claude Code sends a JSON object on stdin. Key fields:

### Model & workspace

| Field | Description |
| --- | --- |
| `model.id`, `model.display_name` | Current model identifier and display name |
| `cwd`, `workspace.current_dir` | Current working directory (same value; `workspace.current_dir` preferred) |
| `workspace.project_dir` | Directory where Claude Code was launched (may differ from `cwd`) |
| `workspace.added_dirs` | Directories added via `/add-dir` or `--add-dir`; empty array if none |
| `workspace.git_worktree` | Worktree name when inside a linked git worktree; absent in the main tree |
| `workspace.repo.host/owner/name` | Repo identity parsed from the `origin` remote; absent outside a repo or with no `origin` |

### Cost & duration

| Field | Description |
| --- | --- |
| `cost.total_cost_usd` | Estimated session cost in USD, computed client-side (may differ from your bill) |
| `cost.total_duration_ms` | Wall-clock time since session start |
| `cost.total_api_duration_ms` | Time spent waiting for API responses |
| `cost.total_lines_added`, `cost.total_lines_removed` | Lines of code changed |

### Context window

| Field | Description |
| --- | --- |
| `context_window.total_input_tokens`, `total_output_tokens` | Token counts currently in context, from the most recent API response (cumulative before v2.1.132) |
| `context_window.context_window_size` | Max context size (200000 default, 1000000 for extended-context models) |
| `context_window.used_percentage`, `remaining_percentage` | Pre-calculated context usage/remaining percentages |
| `context_window.current_usage` | Per-component token counts: `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` |
| `exceeds_200k_tokens` | Whether combined tokens from the latest response exceed 200k (fixed threshold) |

> **Note on `used_percentage`:** it is calculated from input tokens only (`input_tokens + cache_creation_input_tokens + cache_read_input_tokens`) and excludes `output_tokens`. If you compute the percentage yourself, use the same input-only formula to match.

### Rate limits (Claude.ai Pro/Max only)

| Field | Description |
| --- | --- |
| `rate_limits.five_hour.used_percentage`, `rate_limits.seven_day.used_percentage` | Percent of the 5-hour / 7-day limit consumed (0–100) |
| `rate_limits.five_hour.resets_at`, `rate_limits.seven_day.resets_at` | Unix epoch seconds when the window resets |

These appear only for Claude.ai subscribers (Pro/Max) after the first API response. Each window may be independently absent — handle with `// empty` in jq.

### Session & misc

| Field | Description |
| --- | --- |
| `session_id` | Unique session identifier (stable per session — use it for cache keys) |
| `session_name` | Custom name set with `--name` or `/rename`; absent otherwise |
| `transcript_path` | Path to the conversation transcript file |
| `version` | Claude Code version |
| `output_style.name` | Current output style |
| `effort.level` | Reasoning effort (`low`/`medium`/`high`/`xhigh`/`max`); absent if model doesn't support it |
| `thinking.enabled` | Whether extended thinking is enabled |
| `vim.mode` | Vim mode (`NORMAL`/`INSERT`/`VISUAL`/`VISUAL LINE`) when vim mode is on |
| `agent.name` | Agent name when running with `--agent` or agent settings |
| `pr.number`, `pr.url`, `pr.review_state` | Open PR for the current branch (`review_state`: `approved`/`pending`/`changes_requested`/`draft`) |
| `worktree.name/path/branch/original_cwd/original_branch` | Worktree details during `--worktree` sessions |

### Conditionally absent / nullable

- **Absent:** `session_name`, `workspace.git_worktree`, `workspace.repo`, `effort`, `vim`, `agent`, `pr`, `worktree`, `rate_limits` — present only under their respective conditions.
- **Nullable:** `context_window.current_usage` (null before the first API call and again after `/compact` until repopulated); `used_percentage` / `remaining_percentage` (may be null early in a session).

Always handle missing fields with conditional access and null values with fallback defaults (`// 0`, `or 0`, `?.`).

---

## Example scripts

### Context window progress bar (Bash)

```bash
#!/bin/bash
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

BAR_WIDTH=10
FILLED=$((PCT * BAR_WIDTH / 100))
EMPTY=$((BAR_WIDTH - FILLED))
BAR=""
[ "$FILLED" -gt 0 ] && printf -v FILL "%${FILLED}s" && BAR="${FILL// /▓}"
[ "$EMPTY" -gt 0 ] && printf -v PAD "%${EMPTY}s" && BAR="${BAR}${PAD// /░}"

echo "[$MODEL] $BAR $PCT%"
```

### Git status with color (Bash)

```bash
#!/bin/bash
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')

GREEN='\033[32m'; YELLOW='\033[33m'; RESET='\033[0m'

if git rev-parse --git-dir > /dev/null 2>&1; then
    BRANCH=$(git branch --show-current 2>/dev/null)
    STAGED=$(git diff --cached --numstat 2>/dev/null | wc -l | tr -d ' ')
    MODIFIED=$(git diff --numstat 2>/dev/null | wc -l | tr -d ' ')
    GIT_STATUS=""
    [ "$STAGED" -gt 0 ] && GIT_STATUS="${GREEN}+${STAGED}${RESET}"
    [ "$MODIFIED" -gt 0 ] && GIT_STATUS="${GIT_STATUS}${YELLOW}~${MODIFIED}${RESET}"
    echo -e "[$MODEL] 📁 ${DIR##*/} | 🌿 $BRANCH $GIT_STATUS"
else
    echo "[$MODEL] 📁 ${DIR##*/}"
fi
```

### Cache expensive git operations

The script runs frequently, so commands like `git status`/`git diff` can cause lag in large repos. Cache to a temp file keyed on `session_id` (stable per session, unique across concurrent sessions — process IDs like `$$` change every invocation and defeat the cache):

```bash
#!/bin/bash
input=$(cat)
SESSION_ID=$(echo "$input" | jq -r '.session_id')
CACHE_FILE="/tmp/statusline-git-cache-$SESSION_ID"
CACHE_MAX_AGE=5  # seconds

cache_is_stale() {
    [ ! -f "$CACHE_FILE" ] || \
    [ $(($(date +%s) - $(stat -f %m "$CACHE_FILE" 2>/dev/null || stat -c %Y "$CACHE_FILE" 2>/dev/null || echo 0))) -gt $CACHE_MAX_AGE ]
}

if cache_is_stale; then
    if git rev-parse --git-dir > /dev/null 2>&1; then
        BRANCH=$(git branch --show-current 2>/dev/null)
        echo "$BRANCH" > "$CACHE_FILE"
    else
        echo "" > "$CACHE_FILE"
    fi
fi
# ...read $CACHE_FILE and render...
```

---

## Subagent status lines

The `subagentStatusLine` setting renders a custom row body for each subagent shown in the agent panel below the prompt, replacing the default `name · description · token count` row.

```json
{
  "subagentStatusLine": {
    "type": "command",
    "command": "~/.claude/subagent-statusline.sh"
  }
}
```

The command runs once per refresh tick with all visible subagent rows passed as a single JSON object on stdin. The input includes the base hook fields plus `columns` (usable row width) and a `tasks` array, where each task has `id`, `name`, `type`, `status`, `description`, `label`, `startTime`, `tokenCount`, `tokenSamples`, and `cwd`.

Write one JSON line to stdout per row you want to override: `{"id": "<task id>", "content": "<row body>"}`. The `content` string renders as-is (ANSI colors, OSC 8 links supported). Omit a task's `id` to keep its default rendering; emit an empty `content` to hide it. The same trust and `disableAllHooks` gates that apply to `statusLine` apply here.

---

## Windows configuration

On Windows, Claude Code runs status line commands through **Git Bash** when installed, otherwise through **PowerShell**.

Git Bash treats unquoted backslashes as escape characters, so a path like `C:\Users\username\script.mjs` arrives with separators removed and the command silently fails. **Use forward slashes** in the `command` path (`~` also works and expands to your Windows home directory).

To run a PowerShell script as your status line:

```json
{
  "statusLine": {
    "type": "command",
    "command": "powershell -NoProfile -File C:/Users/username/.claude/statusline.ps1"
  }
}
```

---

## Tips

- **Test with mock input:**
  ```bash
  echo '{"model":{"display_name":"Opus"},"workspace":{"current_dir":"/home/user/project"},"context_window":{"used_percentage":25},"session_id":"test-session-abc"}' | ./statusline.sh
  ```
- **Keep output short** — the bar has limited width; long output truncates or wraps awkwardly.
- **Cache slow operations** — see the caching example above.

Community projects like [ccstatusline](https://github.com/sirmalloc/ccstatusline) and [starship-claude](https://github.com/martinemde/starship-claude) provide pre-built configurations with themes and extra features.

---

## Troubleshooting

**Status line not appearing**
- Make the script executable: `chmod +x ~/.claude/statusline.sh`
- Ensure output goes to stdout, not stderr; run the script manually to confirm output.
- On Windows + Git Bash, use forward slashes in the `command` path.
- If `disableAllHooks` is `true`, the status line is also disabled — remove it or set `false`.
- Run `claude --debug` to log the exit code and stderr from the first invocation.

**Shows `--` or empty values**
- Fields may be null before the first API response. Handle nulls with fallbacks (`// 0`).
- Restart Claude Code if values stay empty after multiple messages.

**Context percentage shows unexpected values**
- Use `used_percentage` for the simplest accurate state. It may differ from `/context` output due to when each is calculated.

**OSC 8 links not clickable**
- Verify terminal support (iTerm2, Kitty, WezTerm; Terminal.app does not support them).
- Force detection with `FORCE_HYPERLINK=1 claude` if needed.
- SSH/tmux may strip OSC sequences; use `printf '%b'` instead of `echo -e` if escapes appear as literal text.

**Workspace trust required**
- Because `statusLine` executes a shell command, it runs only after you accept the workspace trust dialog. Otherwise you'll see `statusline skipped · restart to fix`.

**Script errors or hangs**
- Non-zero exits or no output cause the line to go blank. Slow scripts block updates and can be cancelled mid-run if a new update fires. Keep scripts fast and test independently with mock input.

---

*Source: [Claude Code — Customize your status line](https://code.claude.com/docs/en/statusline)*
