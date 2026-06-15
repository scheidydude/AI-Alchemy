Set up my Claude Code statusline

Configure my Claude Code statusline script at ~/.claude/statusline-command.sh to display:

<model> [<ctx%> <used>k / <total>k tokens] <dir> git:<reponame> branch:<branch>

Behavior:
- Model: display name from statusline JSON input
- Context: percentage + token counts, color-coded green/yellow(≥60%)/red(≥85%)
- Directory: basename of cwd
- Git section (when in a git repo):
  - Format: git:<reponame> branch:<branch> where reponame = basename $(git rev-parse --show-toplevel)
  - Color: green = clean+pushed, yellow = unpushed commits, red = dirty working tree
  - Uses -c core.fsmonitor=false to avoid interfering with running processes
- Falls back to dir-only when not in a git repo

Wire it up in ~/.claude/settings.json as the statusCommand.