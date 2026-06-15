# Workflow: Research → Draft → Review

A three-phase prompt sequence for producing high-quality written content. Run each phase in order — output from one feeds the next.

---

## When to Use This

- Writing articles, essays, or reports
- Creating documentation from scratch
- Producing content that needs to be accurate and well-structured
- Any writing task where quality matters more than speed

---

## Phase 1: Research

**Goal:** Build a solid knowledge base before touching the draft.

```
You are a research analyst. I'm writing about: [TOPIC]

My target audience: [WHO WILL READ THIS]
My angle / thesis: [WHAT POINT AM I MAKING]

Do the following:
1. Identify the 5-7 most important things a reader needs to know about this topic
2. Note where common misconceptions exist
3. List the strongest counterarguments to my angle
4. Flag anything that's commonly overstated or understated on this topic
5. Suggest 3 concrete examples or case studies that would make this more grounded

Format: numbered lists. Be specific, not vague.
```

---

## Phase 2: Outline

**Goal:** Structure before writing — prevents meandering drafts.

```
Based on the research above, create an outline for [FORMAT: article / essay / report / email] on [TOPIC].

Constraints:
- Target length: [WORD COUNT]
- Tone: [formal / conversational / technical / plain]
- Must include: [any required sections or points]
- Must avoid: [anything off-limits]

Outline format:
## Section title (estimated word count)
- Key point 1
- Key point 2
- What makes this section earn its place

End with: "Strongest opening hook" and "Strongest closing line" as suggestions.
```

---

## Phase 3: Draft

**Goal:** Write the full piece from the outline.

```
Write [FORMAT] based on this outline:

[PASTE OUTLINE]

Audience: [WHO]
Tone: [TONE]
Length: approximately [WORD COUNT] words

Rules:
- Every paragraph must earn its place — cut anything that's just filler
- No hedging phrases ("it's worth noting that", "in conclusion")
- No passive voice unless unavoidable
- Specific over general — use examples, not abstractions
- Opening line must hook immediately — no throat-clearing

Write the full draft now.
```

---

## Phase 4: Review

**Goal:** Critique before finalizing.

```
Review this draft as a ruthless senior editor. You care about:
- Clarity (does every sentence land?)
- Accuracy (any claims that could be wrong?)
- Structure (does it flow? does anything feel out of place?)
- Opening and closing (are they strong enough?)
- Cuts (what 20% could be removed to make it 50% better?)

<draft>
[PASTE DRAFT]
</draft>

Output:
1. Top 3 problems (specific, with line references)
2. Top 3 strengths (so I know what to preserve)
3. Suggested rewrite for the weakest paragraph
4. One-line verdict: publish / revise / rethink
```

---

## Phase 5: Polish (Optional)

**Goal:** Final line edits.

```
Polish this draft for final publication. Fix:
- Weak verbs → strong verbs
- Passive voice → active voice
- Wordy phrases → tighter equivalents
- Any repeated words within 3 sentences of each other

Do not change the meaning, structure, or examples. Only improve the sentence-level writing.

Return the full polished draft.

<draft>
[PASTE DRAFT]
</draft>
```
