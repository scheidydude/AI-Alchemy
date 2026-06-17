# Context Handoff Prompt

> **How to use:** Paste everything below the `---` into Claude Code when context is getting heavy. It will produce a `HANDOFF.md` in the repo. Then run `/clear` and start your next session: *"Read HANDOFF.md and confirm you're ready to continue."*

---
```text
# Create a Context Handoff Document

Write a handoff document so a fresh Claude Code session can pick up exactly where we left off — no lost momentum, no re-litigating settled decisions, no repeated mistakes.

This is an engineering handoff, not a summary. Write what matters for shipping.

## Output

Create `HANDOFF.md` in the repo root. If one already exists, read it, archive it as `HANDOFF-archive-YYYY-MM-DD-HHMM.md`, then write the new one. The new file must stand alone — assume the next session has zero memory of this conversation.

## Required Sections

Use these exact headings, in order. Skip a section only if it genuinely doesn't apply (say so briefly rather than omitting).

### 1. Mission
Two or three sentences. What are we building or fixing, and why does it matter?

### 2. Current State
Where things stand **right now**:
- What's working and verified
- What's half-built and in what state
- What's broken or blocked, and on what
- The exact next action a fresh session should take

Be specific. "Auth is working" is useless. "Clerk sign-in works end-to-end; `/api/protected` correctly 401s when unauthenticated, verified at commit `abc1234`" is useful.

### 3. Decisions Made (and Why)
Every non-obvious technical or product decision made this session:
- **Decision:** what we chose
- **Alternatives considered:** what we rejected
- **Reason:** why we chose it
- **Reversibility:** load-bearing, or easy to change?

This section is the biggest win over `/compact`. The next session will want to re-debate these. Make it unnecessary.

### 4. Architecture & Key Files
One line per important file: what it does and why it exists. Call out:
- Files created this session
- Files modified significantly (conceptual change, not line-by-line)
- Files that look touchable but **shouldn't be**, and why

### 5. Gotchas & Hard-Won Knowledge
Stuff that cost us time — dead ends, surprises, footguns. Example:
- "Supabase RLS policies don't apply to the service role key"

If a fresh session will repeat the mistake without this, write it down.

### 6. Conventions In Play
How we're writing code that isn't obvious from reading it: naming patterns, file organization, commit style, testing approach, what we're deliberately *not* doing. Reference `CLAUDE.md` or similar if it exists.

### 7. Open Questions
Deferred decisions and things needing user input. Phrase as clear questions, not vague topics.

### 8. Do Not Touch
Files, systems, or decisions that are settled. Prevents the next session from "helpfully" refactoring something we left alone on purpose.

### 9. Resume Command
A short, copy-pasteable instruction to get the next session rolling:

> "Read HANDOFF.md. Then [specific next action]. Do not [specific thing to avoid]. Confirm before making changes outside [scope]."

## Quality Bar

1. **Check ground truth.** Run `git status`, `git log --oneline -20`, inspect relevant files. Write from the filesystem, not memory. If they disagree, filesystem wins — note the discrepancy.
2. **Cut ruthlessly.** Every sentence should earn its place.
3. **Be honest about uncertainty.** If something might not work, say so. If a decision has reservations, note them in Decisions Made.

**The test:** Next session reads only `HANDOFF.md` and the repo, makes the next meaningful commit within five minutes, asks the user nothing that was already settled.

When done, show me the file path and a one-paragraph summary so I can sanity-check before clearing.
```
