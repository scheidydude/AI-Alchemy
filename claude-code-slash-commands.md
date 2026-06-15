# Claude Code Custom Slash Commands

> **How to use:** Create `.md` files in `.claude/commands/` (project) or `~/.claude/commands/` (global). The filename becomes the command. Run with `/command-name` in any Claude Code session.

---

## Setup

```
.claude/
  commands/
    review.md        → /review
    deploy-check.md  → /deploy-check
    standup.md       → /standup
```

The contents of the `.md` file is the prompt that runs when you invoke the command. Use `$ARGUMENTS` to pass arguments from the slash command.

---

## Command Recipes

### /review

Code review the current diff.

**`.claude/commands/review.md`**
```
Review the current git diff for:
1. Correctness bugs — logic errors, off-by-one, null handling, race conditions
2. Security issues — injection, auth bypass, exposed secrets, insecure defaults
3. Simplification — anything that could be 30%+ simpler without losing clarity
4. Missing edge cases — inputs that would break this

Format output as a numbered list. Each item:
- Location (file:line)
- Problem (one sentence)
- Fix (specific, not vague)

Skip style nitpicks. Skip what's already correct. Be direct.
```

---

### /standup

Generate a standup summary from git log.

**`.claude/commands/standup.md`**
```
Generate a standup update based on my git activity since yesterday.

Run: git log --since="yesterday" --author="$(git config user.name)" --oneline

Format:
**Yesterday**
- [what you actually shipped, in plain English — not commit messages verbatim]

**Today**
- [infer from in-progress branches or uncommitted work if any]

**Blockers**
- [flag anything that looks stuck or has a TODO/FIXME nearby]

Keep it under 100 words. Write for a team audience, not a changelog.
```

---

### /deploy-check

Pre-deploy sanity check.

**`.claude/commands/deploy-check.md`**
```
Run a pre-deploy checklist on the current branch vs main.

Check:
1. git diff main --stat — what's changing
2. Any .env variables added to code but not to .env.example
3. Any TODO/FIXME/HACK comments in changed files
4. Any console.log or debug statements left in changed files
5. Whether tests pass: run the test command from CLAUDE.md
6. Whether there are any TypeScript errors: npx tsc --noEmit

Report findings as: ✅ Clear / ⚠️ Warning / ❌ Blocker

End with: "Safe to deploy" or "Do not deploy — [reason]"
```

---

### /summarize-pr

Summarize a PR for a reviewer.

**`.claude/commands/summarize-pr.md`**
```
Write a PR description for the current branch.

Run git diff main to see the changes.

Output:
## What
[1-3 sentences — what changed and why]

## How
[Key implementation decisions — what's non-obvious]

## Testing
[How to verify this works manually]

## Risks
[What could go wrong, what to watch in staging]

Be specific. Avoid vague phrases like "improves performance" or "fixes issues."
```

---

### /explain

Explain a function or file to a new team member.

**`.claude/commands/explain.md`**
```
Explain $ARGUMENTS to someone who is new to this codebase but is an experienced developer.

Cover:
- What it does (one sentence)
- Why it exists (the problem it solves)
- How it works (key logic, not line-by-line)
- What calls it and what it calls
- Any gotchas or non-obvious behaviors

Keep it under 200 words. Use plain language.
```

Usage: `/explain src/auth/middleware.ts`

---

### /todo

List all TODO/FIXME items in the codebase.

**`.claude/commands/todo.md`**
```
Find all TODO, FIXME, HACK, and XXX comments in the codebase.

Run: grep -rn "TODO\|FIXME\|HACK\|XXX" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" . | grep -v node_modules | grep -v .git

Group by file. For each item, note:
- The comment text
- File and line number
- Rough estimate of effort (quick / medium / large)

Sort by file. Output as a markdown checklist.
```

---

### /changelog

Generate a changelog entry from recent commits.

**`.claude/commands/changelog.md`**
```
Generate a changelog entry for the commits since $ARGUMENTS (or since last tag if no argument).

Run: git log $ARGUMENTS..HEAD --oneline (or git log $(git describe --tags --abbrev=0)..HEAD --oneline if no argument)

Group changes into:
### Added
### Changed
### Fixed
### Removed

Write for an end-user audience — what changed, not how. No commit hashes. No internal jargon.
```

Usage: `/changelog v1.2.0`

---

## Tips

- Commands are just prompts — write them the way you'd write any careful prompt.
- Use `$ARGUMENTS` to make commands take parameters (`/explain <file>`).
- Global commands in `~/.claude/commands/` work across all projects.
- Project commands in `.claude/commands/` override global ones of the same name.
- Version-control `.claude/commands/` — shared commands are a team resource.
