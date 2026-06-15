# Claude Code Memory

Claude Code reads `CLAUDE.md` files to carry project context across sessions — no re-explaining, no drift. This guide covers what to put in them and how to structure them for maximum impact.

---

## How Claude Reads Memory

Claude Code automatically reads `CLAUDE.md` from:
1. `~/.claude/CLAUDE.md` — global, applies to all projects
2. `<repo-root>/CLAUDE.md` — project-level
3. `<subdirectory>/CLAUDE.md` — scoped to that directory

All three merge. More specific files take precedence.

---

## What Belongs in CLAUDE.md

### Include: Non-obvious, durable facts

- Architecture decisions and the reasons behind them
- Commands Claude should always use (or never use)
- Naming conventions, file organization patterns
- Testing approach and what counts as "done"
- Known footguns and gotchas
- Personas or constraints ("always respond in Spanish", "never use `any` in TypeScript")

### Exclude: What Claude can derive from the code

- File contents (Claude can read them)
- Git history (Claude can run `git log`)
- Library documentation (Claude can look it up)
- Task state (use a plan or task list instead)

---

## CLAUDE.md Template

```markdown
# [Project Name]

## What This Is
One paragraph. What the project does and why it exists.

## Stack
- Language: 
- Framework: 
- Database: 
- Key dependencies: 

## Dev Commands
- Start dev server: `npm run dev`
- Run tests: `npm test`
- Lint: `npm run lint`
- Build: `npm run build`

## Architecture
Key decisions that aren't obvious from the code:
- Why we chose X over Y
- What layer owns what responsibility
- Any unusual patterns in use

## Conventions
- File naming: 
- Component structure: 
- API response format: 
- Error handling approach: 

## Testing
- What requires a test vs. what doesn't
- Where test fixtures live
- How to run a single test

## Do Not Touch
- Files/directories that are off-limits and why
- Third-party code that looks editable but isn't
- Configs managed externally

## Known Gotchas
- Footguns, non-obvious behaviors, past mistakes
- Environment-specific quirks
- Dependencies with known issues

## External Systems
- Auth: 
- Database: 
- CI/CD: 
- Monitoring: 
```

---

## Global CLAUDE.md (~/.claude/CLAUDE.md)

Your global file should encode preferences that apply everywhere.

```markdown
# Global Claude Code Preferences

## Communication Style
- Be direct. Skip pleasantries.
- When uncertain, say so — don't guess silently.
- Short responses preferred unless depth is needed.

## Coding Defaults
- TypeScript over JavaScript when both are options
- Prefer explicit types over inference at module boundaries
- No `any` unless unavoidable and commented
- Tests required for all new functions

## Git
- Conventional commits: feat/fix/chore/docs/refactor
- Commit message body only when the "why" isn't obvious
- Never force push to main

## What I Don't Want
- Don't rewrite working code to match your style preferences
- Don't add comments explaining what code does — only why
- Don't create files to track in-progress work (use the task system)
```

---

## Memory for Multi-Agent / Long Sessions

For sessions that run long or hand off between instances, combine `CLAUDE.md` with a `HANDOFF.md` (see `context-handoff.md`):

- `CLAUDE.md` → durable truths about the project
- `HANDOFF.md` → current state of this session's work

Keep them separate. `CLAUDE.md` should never need updating mid-session; `HANDOFF.md` is ephemeral.

---

## Tips

**Write it to survive a week.** Assume future-you has forgotten everything except what's in the file.

**Update after surprises.** Every time you hit a gotcha that cost you 20 minutes, add it to `CLAUDE.md` immediately.

**Keep it under 500 lines.** Longer = diluted. Claude weights everything roughly equally — a 2000-line file buries the critical facts.

**Use headers aggressively.** Claude scans, it doesn't read linearly. Headers help it find what's relevant to the current task.
