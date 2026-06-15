# Prompt Debugging

When AI output misses the mark, the problem is almost always in the prompt — not the model. Use this checklist to diagnose and fix.

---

## Diagnostic Checklist

Work through these in order. Most bad outputs fail one of the first three.

### 1. Role / Anchor
- Did you assign a specific persona or expertise?
- Is the role relevant to the task? ("expert editor" vs. "expert Python dev" matters)
- Did you accidentally give a conflicting role?

### 2. Context
- Does the AI have everything it needs to answer well?
- Are you assuming background knowledge the AI can't infer?
- Is any context ambiguous or contradictory?

### 3. Action / Task
- Is the task a single, specific verb? (write, summarize, list, analyze, compare)
- If multiple tasks: did you sequence them, or pile them in one sentence?
- Is success clearly defined?

### 4. Format
- Did you specify output format? (bullets, table, JSON, prose, word count)
- If no format: the AI picks one — it may not pick the right one.
- Are you getting too much or too little? Add/remove a length constraint.

### 5. Tone / Audience
- Who is the intended reader?
- Did you define the voice? (formal, plain, technical, conversational)
- Is the AI writing for itself instead of your audience?

### 6. Constraints
- What should the AI *not* do? Explicit bans reduce hallucination and drift.
- Examples: "Do not invent facts," "Do not use jargon," "Do not summarize — analyze."

---

## Common Failure Modes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Output too generic | No role, no context | Add persona + background |
| Output too long | No length constraint | Add "in X words" or "3 bullets max" |
| Output too short | No depth signal | Add "be thorough" or increase scope |
| Wrong format | No format specified | Specify format explicitly |
| Hallucinated facts | No grounding constraint | Add "only use information I provide" |
| Off-topic | Task too vague | Rewrite action as single specific verb |
| Repeating the question | Prompt too short | Add context and examples |
| Ignoring instructions | Instructions buried in prose | Use headers or XML tags to separate |

---

## Structured Debugging Prompt

Paste this to have Claude diagnose your own prompt:

```
Analyze this prompt and identify the top 3 weaknesses:

<prompt>
[YOUR PROMPT HERE]
</prompt>

For each weakness:
- What's missing or broken
- Why it causes bad output
- Specific rewrite to fix it

Then rewrite the full prompt incorporating all fixes.
```

---

## Quick Fixes

**Add few-shot examples.** One or two examples of desired output beats a paragraph of instructions.

```
Bad:  Write a catchy subject line.
Good: Write a catchy subject line. Example of good: "You're leaving money on the table." Example of bad: "Newsletter Issue #47."
```

**Use XML tags for structure.** Helps the model distinguish context from instructions.

```xml
<context>We're a B2B SaaS for mid-market HR teams.</context>
<task>Write a cold email subject line targeting HR Directors.</task>
<constraints>Under 8 words. No emojis. No questions.</constraints>
```

**Iterate in rounds.** After first output, ask: *"What would make this 2x better?"* Then apply the answer.
