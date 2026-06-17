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