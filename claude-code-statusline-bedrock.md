# Claude Code CLI — Statusline (AWS Bedrock)

The **status line** is a customizable bar pinned to the bottom of Claude Code that runs any shell script you configure. It receives JSON session data on stdin and displays whatever your script prints, giving you a persistent, at-a-glance view of context usage, costs, git status, model, working directory, or anything else you want to track.

It runs locally, does **not** consume API tokens, and hides temporarily during UI interactions like autocomplete suggestions, the help menu, and permission prompts.

This document is for environments where Claude Code runs **exclusively against Amazon Bedrock** (e.g. `CLAUDE_CODE_USE_BEDROCK=1`). The feature works identically to the first-party API, with two Bedrock-specific differences: **cost reporting is an estimate only** and the **`rate_limits` object is never sent**. Both are covered below.

---

## TL;DR — My statusline prompt (Bedrock)

Paste this into Claude Code (`/statusline` will pick it up, or just send it as a message) to generate the script:

```markdown
Set up my Claude Code statusline for an AWS Bedrock environment

Configure my Claude Code statusline script at ~/.claude/statusline-command.sh to display:

<model> [<ctx%> <in>k↑ / <out>k↓ tokens] <dir> git:<reponame> branch:<branch> (~$<cost> est)

Behavior:
- Model: display name from statusline JSON input
- Context: percentage + input/output token counts, color-coded green/yellow(≥60%)/red(≥85%)
- Cost: this is AWS Bedrock, so cost.total_cost_usd is a CLIENT-SIDE ESTIMATE, not the
  AWS bill. Render it dimmed and prefixed with "~" and suffixed with "est" so it's never
  mistaken for actual billing. If you'd rather omit it, leave it out — tokens are the
  honest unit on Bedrock.
- Rate limits: DO NOT include any rate-limit section. The rate_limits object is only sent
  for Claude.ai Pro/Max subscribers and is always absent on Bedrock. Guard all jq reads
  with // empty or // 0 so the script never errors on missing fields.
- Directory: basename of cwd
- Git section (when in a git repo):
  - Format: git:<reponame> branch:<branch> where reponame = basename $(git rev-parse --show-toplevel)
  - Color: green = clean+pushed, yellow = unpushed commits, red = dirty working tree
  - Uses -c core.fsmonitor=false to avoid interfering with running processes
- Falls back to dir-only when not in a git repo

Wire it up in ~/.claude/settings.json under the statusLine field (type: "command",
command: path to the script). Note the key is "statusLine", not "statusCommand".
```

---

## ⚠️ Are the cost estimates accurate on Bedrock?

**No — not as authoritative billing.** The `cost.total_cost_usd` field (the same value `/cost` reports, and the same value the statusline reads) is a **client-side estimate computed locally from a price table bundled into Claude Code at build time**. It is explicitly *not* authoritative billing data. On Bedrock this matters more than usual because **Claude Code does not send or receive any billing metrics from your AWS account** — all real billing flows through AWS, and the client has no way to reconcile against it.

The estimate is derived purely from token counts × the bundled price table. It can drift from your actual AWS invoice for several Bedrock-specific reasons:

| Cause of drift | Effect on the estimate |
| --- | --- |
| **Legacy model IDs** | If your Bedrock config points at an older model (e.g. Opus 4.0/4.1 at $15/$75 per 1M), but the bundled price table assumes current rates, the estimate is wrong. The client can't know which exact model ARN you resolved to. |
| **Regional pricing** | Bedrock rates can vary by region; the bundled table reflects a single reference price, not your region's. |
| **Negotiated / Marketplace / EDP pricing** | Private pricing agreements, AWS Marketplace billing, or enterprise discounts are invisible to the client. Your real per-token cost may be well below the list price the estimate uses. |
| **Provisioned Throughput** | If you reserve capacity rather than pay on-demand, you're billed for the reservation regardless of tokens used. A token-count estimate is meaningless against a capacity reservation. |
| **Cross-region inference** | Routing across regions can change the effective rate the estimate doesn't model. |
| **Price-table staleness** | The table is fixed at the Claude Code build version you have installed; Bedrock price changes after that build won't be reflected until you upgrade. |

> **Bottom line:** On Bedrock, treat the statusline cost as a *directional* signal ("this session is getting expensive") rather than an accounting figure. Do not bill end users, trigger budget enforcement, or reconcile invoices from it.

**What *is* reliable on Bedrock:** token counts (`context_window.*`, `cost.total_lines_added/removed`) and durations are measured locally and accurate. A token-based statusline is trustworthy; a dollar-based one is not. The common pattern is to **show tokens instead of (or alongside) dollars** on Bedrock, since tokens are the honest unit there.

**Where to get *actual* Bedrock cost** (the statusline cannot — use AWS-native tooling):

- **AWS Cost Explorer** — filter by the Bedrock service and model usage type for actual billed cost.
- **AWS Budgets** — set spend thresholds with alerts to catch a heavy session before it becomes a heavy bill.
- **Amazon CloudWatch** — Bedrock invocation and token metrics for near-real-time monitoring.
- **Per-key / per-user attribution** — several large enterprises use **LiteLLM** (an open-source proxy) to track spend per key when running Claude Code on Bedrock. Note: LiteLLM is unaffiliated with Anthropic and not audited by them.

A practical split: use the statusline for in-session context % and token counts (trustworthy) plus an optional flagged cost *estimate*; use AWS Cost Explorer / Budgets / CloudWatch for actual accounting and budget enforcement.

---

## When to use it

A status line is useful when you:

- Want to monitor context-window usage as you work
- Need a rough sense of token consumption (and an approximate cost gauge — see the Bedrock caveat above)
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

> The settings key is **`statusLine`** (an object), not `statusCommand`. The `/statusline` command writes this for you correctly; only relevant if you hand-edit settings.

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

Claude Code sends a JSON object on stdin. Key fields below — the **Bedrock notes** column flags anything that behaves differently than the first-party API.

### Model & workspace

| Field | Description | Bedrock notes |
| --- | --- | --- |
| `model.id`, `model.display_name` | Current model identifier and display name | Reflect the Bedrock-resolved model; display name may render differently depending on the model ID/ARN configured. |
| `cwd`, `workspace.current_dir` | Current working directory (same value; `workspace.current_dir` preferred) | Same |
| `workspace.project_dir` | Directory where Claude Code was launched (may differ from `cwd`) | Same |
| `workspace.added_dirs` | Directories added via `/add-dir` or `--add-dir`; empty array if none | Same |
| `workspace.git_worktree` | Worktree name when inside a linked git worktree; absent in the main tree | Same |
| `workspace.repo.host/owner/name` | Repo identity parsed from the `origin` remote; absent outside a repo or with no `origin` | Same |

### Cost & duration

| Field | Description | Bedrock notes |
| --- | --- | --- |
| `cost.total_cost_usd` | Estimated session cost in USD, computed client-side | **Estimate only — not the AWS bill.** See the cost section above. |
| `cost.total_duration_ms` | Wall-clock time since session start | Accurate |
| `cost.total_api_duration_ms` | Time spent waiting for API responses | Accurate |
| `cost.total_lines_added`, `cost.total_lines_removed` | Lines of code changed | Accurate |

### Context window

| Field | Description | Bedrock notes |
| --- | --- | --- |
| `context_window.total_input_tokens`, `total_output_tokens` | Token counts currently in context, from the most recent API response (cumulative before v2.1.132) | Accurate — the trustworthy unit on Bedrock |
| `context_window.context_window_size` | Max context size (200000 default, 1000000 for extended-context models) | Same |
| `context_window.used_percentage`, `remaining_percentage` | Pre-calculated context usage/remaining percentages | Accurate |
| `context_window.current_usage` | Per-component token counts: `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` | Accurate |
| `exceeds_200k_tokens` | Whether combined tokens from the latest response exceed 200k (fixed threshold) | Same |

> **Note on `used_percentage`:** it is calculated from input tokens only (`input_tokens + cache_creation_input_tokens + cache_read_input_tokens`) and excludes `output_tokens`. If you compute the percentage yourself, use the same input-only formula to match.

### Rate limits — **absent on Bedrock**

| Field | Description | Bedrock notes |
| --- | --- | --- |
| `rate_limits.five_hour.*`, `rate_limits.seven_day.*` | Percent of the 5-hour / 7-day limit consumed, plus `resets_at` reset times | **Never sent on Bedrock.** This object only appears for Claude.ai Pro/Max subscribers. Bedrock is pay-per-token with no subscription windows. Guard reads with `// empty` and omit any rate-limit UI. |

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

- **Absent:** `session_name`, `workspace.git_worktree`, `workspace.repo`, `effort`, `vim`, `agent`, `pr`, `worktree`, and — on Bedrock — **always `rate_limits`**. Present only under their respective conditions.
- **Nullable:** `context_window.current_usage` (null before the first API call and again after `/compact` until repopulated); `used_percentage` / `remaining_percentage` (may be null early in a session).

Always handle missing fields with conditional access and null values with fallback defaults (`// 0`, `or 0`, `?.`).

---

## Example scripts

### Bedrock-tuned statusline (Bash)

Emphasizes tokens (the honest unit on Bedrock), flags the cost as an estimate, and omits rate limits:

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
IN=$(echo "$input"  | jq -r '.context_window.total_input_tokens // 0')
OUT=$(echo "$input" | jq -r '.context_window.total_output_tokens // 0')
COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')   # ESTIMATE only

GREEN='\033[32m'; YELLOW='\033[33m'; RED='\033[31m'; DIM='\033[2m'; RESET='\033[0m'

if   [ "$PCT" -ge 85 ]; then C="$RED"
elif [ "$PCT" -ge 60 ]; then C="$YELLOW"
else C="$GREEN"; fi

IN_K=$((IN / 1000)); OUT_K=$((OUT / 1000))

printf '%b\n' "[$MODEL] ${C}${PCT}%% ${IN_K}k↑/${OUT_K}k↓${RESET} 📁 ${DIR##*/} ${DIM}(~\$$(printf '%.2f' "$COST") est)${RESET}"
```

The `~` and `est` label on the dollar amount is deliberate — it signals to anyone reading the bar that it's not the AWS bill. To drop dollars entirely, delete the `COST` line and the trailing `(~$… est)` segment.

### Git status with color (Bash)

Uses `-c core.fsmonitor=false` on every git call to avoid interfering with a running fsmonitor process, and reports the repo name + branch:

```bash
#!/bin/bash
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')

GREEN='\033[32m'; YELLOW='\033[33m'; RESET='\033[0m'

if git -c core.fsmonitor=false rev-parse --git-dir > /dev/null 2>&1; then
    REPO=$(basename "$(git -c core.fsmonitor=false rev-parse --show-toplevel 2>/dev/null)")
    BRANCH=$(git -c core.fsmonitor=false branch --show-current 2>/dev/null)
    STAGED=$(git -c core.fsmonitor=false diff --cached --numstat 2>/dev/null | wc -l | tr -d ' ')
    MODIFIED=$(git -c core.fsmonitor=false diff --numstat 2>/dev/null | wc -l | tr -d ' ')
    GIT_STATUS=""
    [ "$STAGED" -gt 0 ] && GIT_STATUS="${GREEN}+${STAGED}${RESET}"
    [ "$MODIFIED" -gt 0 ] && GIT_STATUS="${GIT_STATUS}${YELLOW}~${MODIFIED}${RESET}"
    echo -e "[$MODEL] 📁 ${DIR##*/} | git:$REPO branch:$BRANCH $GIT_STATUS"
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
    if git -c core.fsmonitor=false rev-parse --git-dir > /dev/null 2>&1; then
        BRANCH=$(git -c core.fsmonitor=false branch --show-current 2>/dev/null)
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

- **Test with mock input** (note: no `rate_limits` block, mirroring Bedrock):
  ```bash
  echo '{"model":{"display_name":"Claude Sonnet 4.6"},"workspace":{"current_dir":"/home/user/project"},"context_window":{"used_percentage":25,"total_input_tokens":48000,"total_output_tokens":12000},"cost":{"total_cost_usd":0.42},"session_id":"test-session-abc"}' | ./statusline.sh
  ```
- **Keep output short** — the bar has limited width; long output truncates or wraps awkwardly.
- **Cache slow operations** — see the caching example above.
- **Prefer tokens over dollars** on Bedrock — tokens are measured locally and accurate; the dollar figure is an estimate.

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
- On Bedrock, `rate_limits` is always absent — guard those reads with `// empty` so the script doesn't error.
- Restart Claude Code if values stay empty after multiple messages.

**Cost looks wrong / doesn't match AWS**
- Expected on Bedrock. The statusline cost is a client-side *estimate*, not the AWS bill. Use Cost Explorer / CloudWatch / Budgets for actual spend. See the cost section above.

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

*Sources: [Claude Code — Customize your status line](https://code.claude.com/docs/en/statusline) · [Claude Code — Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) · [Claude Code — Manage costs](https://docs.anthropic.com/en/docs/claude-code/costs)*
