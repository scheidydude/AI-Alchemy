# Claude Code CLI — Getting Started Guide
*Everything you need to be productive on day one*

Version 1.0 · May 2026

---

## Table of Contents

1. [What Is Claude Code?](#1--what-is-claude-code)
2. [First-Time Use](#2--first-time-use)
3. [CLAUDE.md — Your Project Memory](#3--claudemd--your-project-memory)
4. [The .claude/ Folder](#4--the-claude-folder)
5. [Slash Commands](#5--slash-commands)
6. [Context & Compaction](#6--context--compaction)
7. [Plan Mode](#7--plan-mode)
8. [Checkpoints & Branching](#8--checkpoints--branching)
9. [Subagents](#9--subagents)
10. [Permissions](#10--permissions)
11. [MCP Servers — Extending Claude](#11--mcp-servers--extending-claude)
12. [Model Selection & Effort](#12--model-selection--effort)
13. [Git Worktrees — Parallel Work](#13--git-worktrees--parallel-work)
14. [Keyboard Shortcuts](#14--keyboard-shortcuts)
15. [Additional Topics to Explore](#15--additional-topics-to-explore)
16. [Quick Reference Card](#16--quick-reference-card)

---

## 1 · What Is Claude Code?

Claude Code is Anthropic's terminal-native AI coding agent. Unlike browser-based chat tools, it runs **directly in your terminal** with full access to your filesystem, shell, and version control. It reads your codebase, edits files, runs commands, and manages Git — all within a single interactive session powered by a context window of up to 1 million tokens.

Think of it less like a chatbot and more like a **senior engineer pair-programming with you** who has already read your entire codebase and knows your project conventions.

---

## 2 · First-Time Use

### Installation

The recommended installation method is the native binary installer:

```bash
# Native binary (recommended)
curl -fsSL https://claude.ai/install.sh | bash

# macOS via Homebrew
brew install --cask claude-code

# NPM (deprecated — use native binary instead)
npm install -g @anthropic-ai/claude-code
```

> **Note:** NPM installation is officially deprecated. Migrate existing NPM installs with `claude install`.

### Authentication

After installation, authenticate with your Anthropic account:

```bash
claude auth login        # Log in or switch accounts
claude auth status       # Verify current auth state
claude auth logout       # Clear stored credentials
```

### Starting Your First Session

1. Open a terminal and navigate to your project directory.
2. Run **`/init`** to have Claude analyze your project and create an initial `CLAUDE.md`:
   ```bash
   claude
   /init
   ```
3. Ask Claude anything about your code or give it a task.
4. Check session health anytime with `/doctor`.

> **Tip:** Run `/doctor` when something isn't working right. It checks API key validity, permission config, MCP server connections, and hook registration.

### Useful First-Run CLI Flags

| Flag / Command | Purpose |
|---|---|
| `claude` | Start interactive session |
| `claude -c` | Resume last conversation |
| `claude "describe this codebase"` | Start with an immediate prompt |
| `claude -p "review README.md"` | Non-interactive / pipe mode |
| `claude --model sonnet` | Start with a specific model |
| `claude --max-budget-usd 2.00` | Set a per-session spending cap |
| `claude doctor` | Diagnose the installation |

---

## 3 · CLAUDE.md — Your Project Memory

**`CLAUDE.md`** is a Markdown file that Claude reads at the start of every session in that repository. It acts as a persistent project brief — your standing instructions, conventions, and context that Claude should always know.

### Where CLAUDE.md Lives

- **Project root:** `.claude/CLAUDE.md` (or project root `CLAUDE.md`) — scoped to that repo
- **Global:** `~/.claude/CLAUDE.md` — applies to all sessions on your machine

### What to Put in CLAUDE.md

- Tech stack and language versions
- Project structure overview (key directories and their purpose)
- Build, test, and lint commands
- Code style conventions and patterns to follow
- Branching and commit message conventions
- Things Claude should **never** do (e.g., never modify migrations directly)
- Pointers to key files Claude should read first

> **Best Practice:** Keep `CLAUDE.md` under 200 lines. Longer files load slower and dilute focus. Use `@file` mentions inside tasks to pull in deeper context on demand.

### Example CLAUDE.md

```markdown
# My API Service

## Stack
Python 3.12 · FastAPI · PostgreSQL · Redis

## Commands
- Run tests: `pytest -x -q`
- Lint: `ruff check . && mypy .`
- Start dev server: `uvicorn app.main:app --reload`

## Conventions
- Use type hints everywhere
- All new endpoints need integration tests
- Never edit files under migrations/ directly

## Key Files
- app/config.py — environment config
- app/dependencies.py — shared FastAPI deps
```

You can edit `CLAUDE.md` at any time mid-session with `/memory`.

---

## 4 · The .claude/ Folder

The `.claude/` directory is the control center for how Claude behaves in your project. Not every subdirectory needs to exist — Claude Code creates them on demand.

```
.claude/
├── CLAUDE.md               # Project memory loaded every session
├── settings.json           # Shared config — safe to commit to Git
├── settings.local.json     # Machine-local overrides — gitignore this
├── agents/                 # Custom subagent definitions (.md files)
├── commands/               # Custom slash commands (.md files)
├── hooks/                  # Event-driven automation scripts
├── plugins/                # Installed plugin manifests
├── rules/                  # Scoped behavior rules (.md files)
├── skills/                 # Reusable multi-step workflows
└── projects/               # Session transcripts and state
```

### agents/

Each `.md` file defines a specialized subagent with its own name, description, allowed tools, and model. Claude delegates to these via the Agent tool during complex tasks. Example: a `code-reviewer.md` agent focused solely on security review.

### commands/

Each `.md` file becomes a custom slash command. The filename is the command name: `commands/review.md` → `/review`. The file body is the prompt that runs when invoked. Commit these to Git so your whole team shares them.

> **Note:** As of 2026, slash commands and skills are unified. `.claude/commands/` still works, but the recommended pattern is `.claude/skills/` — every skill automatically gets a `/slash-command` interface.

### hooks/

Shell scripts (or any executable) triggered by Claude Code lifecycle events: `PreToolUse`, `PostToolUse`, `Stop`, and more. Wire them up in `settings.json` under the `hooks` key. Use them to block dangerous commands, auto-format on write, or send a Slack notification when Claude finishes.

### plugins/

Plugin manifests install bundles of agents, skills, hooks, and MCP servers in one shot. Plugins can be distributed via a marketplace and force-enabled by org admins through managed settings.

### rules/

Scoped behavior rules in Markdown files. Each rules file can be path-scoped (e.g., apply only to files matching a pattern), giving you fine-grained control over Claude's coding style per module or file type.

### skills/

Reusable multi-step workflows. Each skill is a folder containing a `SKILL.md` with YAML frontmatter controlling invocation: Claude can auto-invoke based on task context, you can trigger manually with `/skill-name`, or both. Skills support argument substitution (`$ARGUMENTS`, `$0`, etc.).

### Output Styles

You can define reusable output format instructions — JSON schemas, Markdown templates, table formats — as named styles and reference them in `CLAUDE.md` or invoke them explicitly. This ensures Claude always returns data in the shape your tooling expects.

---

## 5 · Slash Commands

Slash commands control the **session itself**, not the code. Type `/` inside an active session to see the full list with autocomplete. Unlike regular prompts (which tell Claude what to do), slash commands tell Claude *how to behave* or perform actions on the session state.

### Context Management

| Command | What it does |
|---|---|
| `/clear` | Wipe all conversation history (aliases: `/reset`, `/new`) |
| `/compact [instructions]` | Compress history into a summary — preserves thread without losing tokens |
| `/context` | Visualize current context window usage |
| `/branch` | Fork the conversation — safe to experiment |
| `/rewind` | Roll back to a previous checkpoint |

> **Rule of thumb:** Use `/compact` when context usage exceeds 80%, `/clear` when switching to a completely new task. Pass instructions to `/compact` to tell Claude what to preserve: `/compact retain the error handling patterns`.

### Code Actions

| Command | What it does |
|---|---|
| `/init` | Analyze project and create/update CLAUDE.md |
| `/plan` | Enter plan mode — Claude proposes before acting |
| `/review` | Request a code review of recent changes |
| `/diff` | View file diffs from the current session |
| `/security-review` | Scan for security vulnerabilities |
| `/batch` | Spawn parallel subagents for concurrent work |
| `/autofix-pr` | Auto-fix CI failures |

### Configuration

| Command | What it does |
|---|---|
| `/permissions` | Manage tool permissions interactively |
| `/memory` | Edit CLAUDE.md project memory in-session |
| `/config` | Open the settings panel |
| `/model` | Switch AI model (`opus` / `sonnet` / `haiku`) |
| `/effort` | Set reasoning level (`low` / `medium` / `high` / `xhigh`) |

### Utilities

| Command | What it does |
|---|---|
| `/cost` | View token usage and estimated spend |
| `/status` | Current git status, model, context, MCP connections |
| `/mcp` | List connected MCP servers and available tools |
| `/copy` | Copy last response to clipboard |
| `/export` | Export conversation history |
| `/help` | Show all available commands |
| `/doctor` | Diagnose installation and connectivity issues |

---

## 6 · Context & Compaction

Context is Claude's working memory for a session. Claude Code supports up to **1 million tokens** of context, but managing it well is the single biggest factor in getting consistent, high-quality output in long sessions.

### How Context Fills Up

- Every message you send and every file Claude reads consumes tokens
- Subagent results are returned into the parent context
- Long tool outputs (test results, diffs, logs) are the biggest consumers

### Compaction vs. Clear

| Action | Effect |
|---|---|
| `/compact` | Replaces history with a compressed summary — context shrinks, thread preserved |
| `/compact [instructions]` | Compact but keep specific information (e.g. `/compact retain API surface`) |
| `/clear` | Deletes all history — full reset, file edits remain on disk |

> **Note:** Compaction completes instantly as of v2.0.64. Auto-compaction triggers when the context window approaches its limit.

### Context Tips

- **Use `@file` mentions** to pull in specific files rather than asking Claude to search
- **Delegate verbose tasks to subagents** — their output summarizes back rather than flooding your context
- **Check context usage with `/context`** periodically in long sessions
- **Compact proactively at 80%**, not reactively after quality degrades

---

## 7 · Plan Mode

Plan mode changes Claude's behaviour: instead of immediately writing code or executing commands, Claude first proposes a **step-by-step plan** for you to review. You approve, modify, or reject the plan before any action is taken.

### Activating Plan Mode

```bash
/plan                        # Toggle plan mode on/off
claude --plan                # Start a session in plan mode
/model opusplan              # Switch to Opus with planning defaults
```

### When to Use Plan Mode

- Large refactors that touch many files
- Architectural changes with uncertain scope
- Any task where you want to verify Claude's approach before it executes
- Onboarding: reviewing what Claude intends to do builds familiarity

> **Tip:** Use `/branch` before any risky change, then `/plan` to review the approach before execution. If the plan looks wrong, you can rewind without consequences.

---

## 8 · Checkpoints & Branching

Claude Code automatically creates checkpoints as you work, letting you roll back to any prior state in the session.

### Key Commands

| Command | What it does |
|---|---|
| `/rewind` | Roll back to a previous checkpoint in the session |
| `/branch` | Fork the current conversation — safe to experiment |
| `Ctrl+Z` (in prompt) | Undo last input before sending |

Checkpoints are session-scoped. For persistent safety across sessions, use Git worktrees (see Section 13).

---

## 9 · Subagents

Subagents are **child instances of Claude Code** that Claude spawns to handle parallel or specialized tasks. The orchestrator (your main session) delegates work, the subagent executes it, and the result returns to the parent context.

### How Subagents Work

1. You describe a task that benefits from parallelism or specialization.
2. Claude spawns one or more subagents via the Agent tool.
3. Each subagent runs independently with its own context.
4. Results are summarized back into the orchestrator's context.

### Using Subagents

```bash
# Spawn parallel agents via slash command
/batch

# Define custom agents in .claude/agents/code-reviewer.md
# Claude will delegate matching tasks automatically
```

### Agent Definition Example (`agents/security-reviewer.md`)

```markdown
---
name: security-reviewer
description: Scans code for security vulnerabilities and OWASP issues
tools: [read_file, search_files, bash]
model: claude-opus-4-6
---

You are a security-focused code reviewer. When invoked, scan the
provided code for OWASP Top 10 vulnerabilities and summarize findings.
```

> **Tip:** Subagents are ideal for long, verbose tasks (running full test suites, scanning large codebases). Their output summarizes back rather than flooding your main context.

---

## 10 · Permissions

Claude Code operates with a layered permission system that controls which tools it can use and under what conditions. Understanding this prevents surprises and keeps your environment safe.

### Permission Levels

- **Ask (default):** Claude prompts you before each tool use
- **Auto-approve:** Claude executes tools without prompting (use carefully)
- **Blocked:** tool is disabled entirely

### Configuring Permissions

```bash
/permissions             # Open interactive permissions panel
/config                  # Open full settings panel
```

Or edit `.claude/settings.json` directly:

```json
{
  "permissions": {
    "allow": ["read_file", "bash:git *", "write_file"],
    "deny": ["bash:rm -rf *"]
  }
}
```

### Permission Modes

- **Default mode:** Claude asks before any file write or shell command
- **Auto mode (`--dangerously-skip-permissions`):** skip all prompts — only use in trusted CI environments, never on prod

> **Warning:** Auto-approve is powerful but risky. Scope it tightly using allow/deny patterns, and never enable it in environments with access to production systems or sensitive data.

### Hooks for Deterministic Guards

Use `PreToolUse` hooks for rules that must **never** be overridden by Claude — e.g., blocking `rm -rf` or preventing writes to production config files. Hook scripts exit with code `0` (allow) or non-zero (block).

---

## 11 · MCP Servers — Extending Claude

MCP (Model Context Protocol) servers connect Claude Code to external tools and data sources: GitHub, Jira, databases, Slack, Datadog, and more. MCP is what separates a capable Claude Code setup from a truly integrated one.

### Adding MCP Servers

```bash
# Add GitHub MCP server
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=your-token \
  -- npx -y @modelcontextprotocol/server-github

# List connected servers
claude mcp list

# Check status in-session
/mcp
/status
```

### Configuration in `.mcp.json`

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

> **Tip:** Commit `settings.json` (shared MCP config) but gitignore `settings.local.json` (machine-specific tokens and paths). Disable unused MCP servers to reduce context overhead.

---

## 12 · Model Selection & Effort

Claude Code supports multiple models. Choose based on task complexity and cost sensitivity.

### Quick Decision Guide

- **Haiku 4.5:** Simple exploration, quick lookups, lightweight tasks
- **Sonnet 4.6:** Cost-sensitive daily coding — the default for most plans
- **Opus 4.7:** Hard reasoning, architecture decisions, agentic loops, security analysis

```bash
/model haiku         # Switch to Haiku for lightweight tasks
/model sonnet        # Switch to Sonnet (cost-efficient daily driver)
/model opus          # Switch to Opus for complex reasoning
/effort low          # Less thinking — faster, cheaper
/effort xhigh        # Maximum reasoning (Opus 4.7 only)
```

> **Note:** Bedrock and Vertex deployments default to Sonnet 4.5. Pin newer models via the `ANTHROPIC_DEFAULT_OPUS_MODEL` environment variable if your admin hasn't configured this.

---

## 13 · Git Worktrees — Parallel Work

Git worktrees let you run **multiple Claude Code sessions simultaneously** on different branches of the same repo — without stashing or losing work. Each worktree is an independent checkout with its own session.

```bash
# Create a worktree for a feature branch
git worktree add ../my-repo-feature feature/auth-refactor

# Open a Claude Code session in it
cd ../my-repo-feature && claude
```

> **Use case:** Run one session doing a large refactor on main while a second session on a feature branch handles a parallel bug fix. Neither session interferes with the other.

---

## 14 · Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `↑` / `↓` arrows | Navigate prompt history |
| `Ctrl+C` | Cancel current response (keeps session open) |
| `Ctrl+D` | Exit Claude Code cleanly |
| `Ctrl+Z` (in prompt) | Undo last prompt before sending |
| `Shift+Enter` | Insert newline in prompt (multi-line input) |
| `Tab` | Autocomplete slash commands |
| `Esc` | Clear current prompt |
| `Option+P` / `Alt+P` | Switch model mid-session without clearing context |

---

## 15 · Additional Topics to Explore

These topics weren't covered in depth above but are worth learning as you grow into Claude Code:

### IDE Integrations

Claude Code integrates with VS Code, JetBrains IDEs, and others. The IDE plugin adds key-bound versions of `/compact` and `/clear`, inline diff views, and a persistent side panel.

### Auto Mode & CI/CD

Run Claude Code non-interactively in pipelines with `claude -p` (print mode). Combine with `--max-budget-usd` and `--allowedTools` to scope what it can do. Use `--output-format json` for structured output.

```bash
claude -p "summarize changes in this PR" --output-format json \
  --max-budget-usd 0.50 --allowedTools read_file,bash:git
```

### /loop — Recurring Tasks

The `/loop` command (and the `scheduledTasks` YAML config) lets Claude run recurring tasks on a schedule — e.g., a nightly test-failure summary or daily dependency audit.

### OpenTelemetry & Cost Tracking

Claude Code emits OpenTelemetry traces. If your team uses Datadog or another APM tool, configure the `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable to pipe usage metrics into your observability stack. Use `/cost` for in-session spending and the Admin API `GET /v1/organizations/usage_report/claude_code` for org-wide analytics.

### Managed Settings (Enterprise)

Admins can distribute `managed-settings.json` via Intune, GPO, or config management to enforce model selection, permission policies, approved MCP servers, and Bedrock endpoint configuration across all users — without users needing to configure anything manually.

### The .mcp.json File

Project-level MCP configuration lives in `.mcp.json` at the repo root (separate from `.claude/settings.json`). Commit this file so all contributors automatically connect to the same MCP servers.

### Diagnostics: /doctor and /insights

`/doctor` checks installation health. `/insights` provides session analytics: tool usage, acceptance rates, token burn rate, and estimated cost breakdown. Run these when something feels off or when justifying ROI to your team.

---

## 16 · Quick Reference Card

### The 5 Core Systems

| System | Where it lives |
|---|---|
| Configuration | `.claude/settings.json` + `CLAUDE.md` |
| Permissions | `/permissions`, `settings.json` allow/deny |
| Hooks | `.claude/hooks/` + `settings.json` hooks key |
| MCP Servers | `.mcp.json` + `claude mcp add` |
| Subagents | `.claude/agents/*.md` |

### Essential First Commands

```bash
/init          # Analyze project, create CLAUDE.md
/doctor        # Check installation health
/help          # See all available commands
/compact       # Compress context when > 80% full
/plan          # Review approach before Claude acts
/cost          # Check session token spend
/permissions   # Review and adjust tool access
```

### Top CLAUDE.md Tips

- Keep it under 200 lines
- Include run/test/lint commands — Claude uses them automatically
- Document things Claude should **never** do
- Commit it to Git so your whole team benefits

---

*Claude Code Documentation: [docs.claude.com](https://docs.claude.com)  ·  Support: [support.claude.com](https://support.claude.com)*
