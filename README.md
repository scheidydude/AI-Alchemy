# AI Alchemy

A curated collection of prompting frameworks, templates, and tools for getting high-quality results from AI models — particularly Claude and Claude Code.

## Contents

### Prompting Frameworks

| File | What It Is |
|------|-----------|
| [`ai-craft-framework.md`](ai-craft-framework.md) | The **CRAFT** method — Context, Role, Action, Format, Tone/Target. Core framework for precise prompt engineering. |
| [`ai-prompting-techniques.md`](ai-prompting-techniques.md) | Comprehensive breakdown of 10+ frameworks (RTF, RACE, CO-STAR, CRISPE, and more) organized by skill level with recommendations. |
| [`the-contrarian-prompt.md`](the-contrarian-prompt.md) | System prompt that turns AI into an intellectual sparring partner — challenges assumptions, surfaces blind spots, pushes back. |
| [`prompts-to-learn-anything-10x-faster.md`](prompts-to-learn-anything-10x-faster.md) | A toolkit of learning-focused prompts: ELI5, visual maps, chunking, analogies, and more. |
| [`ai-teach-me.md`](ai-teach-me.md) | Prompts for accelerated personal learning and mental model building. |
| [`ai-agi.md`](ai-agi.md) | Quick explainer on AGI — useful as context when prompting around AI capabilities. |
| [`prompt-debugging.md`](prompt-debugging.md) | Diagnostic checklist for underperforming prompts — common failure modes, fixes, and a structured debugging prompt. |
| [`chain-of-thought-patterns.md`](chain-of-thought-patterns.md) | When and how to use CoT — zero-shot, few-shot, self-consistency, tree of thought, and reflection patterns. |
| [`prompt-personas.md`](prompt-personas.md) | Reusable persona blocks: expert editor, devil's advocate, Socratic tutor, strategist, and more. Drop into any prompt. |
| [`meta-prompting.md`](meta-prompting.md) | Use AI to write, critique, stress-test, and iterate on your own prompts. |

### Claude Code Tools

| File | What It Is |
|------|-----------|
| [`context-handoff.md`](context-handoff.md) | Prompt that generates a `HANDOFF.md` so a fresh Claude Code session picks up exactly where you left off — no lost momentum. |
| [`claude-status-line.md`](claude-status-line.md) | Setup prompt for a Claude Code statusline showing model, token usage, directory, and git state. |
| [`claude-code-getting-started.md`](claude-code-getting-started.md) | Getting started guide for Claude Code. |
| [`claude-code-hooks.md`](claude-code-hooks.md) | Hook recipes for auto-lint, auto-format, test-on-save, desktop notifications, audit logging, and blocking dangerous commands. |
| [`claude-code-memory.md`](claude-code-memory.md) | What to put in `CLAUDE.md`, how to structure it, and how to combine project memory with session handoffs. |
| [`claude-code-slash-commands.md`](claude-code-slash-commands.md) | Custom slash command recipes: `/review`, `/standup`, `/deploy-check`, `/summarize-pr`, `/explain`, `/todo`, `/changelog`. |
| [`claude-code-mcp-setup.md`](claude-code-mcp-setup.md) | Quick-start for connecting MCP servers — GitHub, PostgreSQL, Puppeteer, Brave Search, Slack, and more. |

### Evaluation

| File | What It Is |
|------|-----------|
| [`ai-eval-framework.md`](ai-eval-framework.md) | Five-dimension rubric (accuracy, completeness, relevance, clarity, calibration) for scoring AI output quality. |

### Workflow Templates

Multi-step prompt sequences in [`workflow-templates/`](workflow-templates/):

| File | What It Is |
|------|-----------|
| [`research-draft-review.md`](workflow-templates/research-draft-review.md) | Five-phase writing workflow: research → outline → draft → review → polish. |
| [`code-test-document.md`](workflow-templates/code-test-document.md) | Four-phase development workflow: implement → test → document → integrate. |

### After-Action Review Templates

Structured reflection templates in [`after-action-reviews/`](after-action-reviews/):

- **Personal** — weekly, monthly, yearly personal reviews
- **Writer** — writing practice retrospectives
- **Creative** — creative project retrospectives
- **Project** — end-of-project retrospective with scope, decisions, and lessons learned
- **Learning** — reflect on courses, books, or skill acquisition — what stuck, what didn't, what's next

## Usage

Each file is a standalone resource — copy the prompt text directly into your AI session, or use as a reference when crafting your own prompts.

For Claude Code tools, follow the instructions at the top of each file.

## License

MIT — see [LICENSE](LICENSE).
