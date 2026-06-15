# AI Output Evaluation Framework

A rubric for assessing AI-generated output quality. Use it to audit outputs before using them, to calibrate your prompts, or to compare model performance on the same task.

---

## The Five Dimensions

Score each dimension 1–5. A total of 20+ is production-ready. Below 15, revise the prompt.

---

### 1. Accuracy (Did it get the facts right?)

| Score | Meaning |
|-------|---------|
| 5 | All claims verifiable and correct |
| 4 | Minor errors that don't affect the substance |
| 3 | Some incorrect claims, but core answer is right |
| 2 | Significant factual errors |
| 1 | Fabricated or fundamentally wrong |

**Red flags:** Specific statistics without a source, proper nouns that sound plausible but are unverifiable, confident claims about recent events.

---

### 2. Completeness (Did it answer the whole question?)

| Score | Meaning |
|-------|---------|
| 5 | Nothing missing; anticipates follow-up questions |
| 4 | Covers all main points, minor gaps |
| 3 | Answers the surface question, misses important nuance |
| 2 | Partial answer only |
| 1 | Doesn't address the question |

**Red flags:** Answer ends abruptly, skips the hard part of the question, gives the obvious answer without addressing the implicit harder question.

---

### 3. Relevance (Did it stay on task?)

| Score | Meaning |
|-------|---------|
| 5 | Every sentence earns its place |
| 4 | Mostly on-point, minor tangents |
| 3 | Useful content buried in filler |
| 2 | Significant off-topic content |
| 1 | Didn't address the actual request |

**Red flags:** Lengthy preamble restating the question, excessive caveats, tangents the user didn't ask for.

---

### 4. Clarity (Is it easy to use?)

| Score | Meaning |
|-------|---------|
| 5 | Clear, well-structured, immediately actionable |
| 4 | Clear, minor structure issues |
| 3 | Understandable but requires effort to parse |
| 2 | Confusing structure or language |
| 1 | Unclear or unusable |

**Red flags:** Passive voice throughout, jargon without definition, walls of text with no structure, ambiguous pronouns.

---

### 5. Calibration (Does confidence match certainty?)

| Score | Meaning |
|-------|---------|
| 5 | Hedges exactly where uncertainty exists; confident where warranted |
| 4 | Mostly well-calibrated, minor over/under-confidence |
| 3 | Slightly overconfident or over-hedged |
| 2 | Significant confidence mismatch |
| 1 | Hallucination presented as fact, or so hedged it's useless |

**Red flags:** No hedging anywhere (overconfident), hedging on everything (useless), "I think" on provable facts.

---

## Scoring Sheet

```
Task: _______________________________________________
Model: _____________  Date: ____________

Accuracy:      /5
Completeness:  /5
Relevance:     /5
Clarity:       /5
Calibration:   /5

Total:         /25

Notes:
- What worked:
- What failed:
- Prompt change to try:
```

---

## Quick Hallucination Check

Run this after any output containing specific facts, names, dates, or statistics:

```
Review your previous response. For each factual claim:
1. State the claim
2. Rate your confidence: High / Medium / Low
3. Note if you cannot verify it from your training data

Flag anything I should independently verify.
```

---

## Comparative Eval Prompt

Test two prompts against each other on the same task:

```
I'm going to show you two AI responses to the same task. Evaluate them on:
- Accuracy (which has fewer errors?)
- Completeness (which answers more of the question?)
- Clarity (which is easier to use?)
- Conciseness (which wastes less space?)

Task: [ORIGINAL TASK]

Response A:
[RESPONSE A]

Response B:
[RESPONSE B]

For each dimension, pick a winner and explain why in one sentence. Then give an overall winner.
```
