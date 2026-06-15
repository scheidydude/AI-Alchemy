# Chain-of-Thought Prompting Patterns

Chain-of-thought (CoT) prompting tells the model to reason step by step before answering. It dramatically improves accuracy on complex tasks — math, logic, multi-step analysis, code debugging.

---

## When to Use CoT

Use CoT when:
- The answer requires multiple reasoning steps
- Errors compound (math, logic, planning)
- You need to audit the reasoning, not just the answer
- The task involves weighing competing factors

Skip CoT when:
- The task is simple retrieval or formatting
- Speed matters more than accuracy
- You're generating creative content (CoT can over-constrain it)

---

## Patterns

### 1. Zero-Shot CoT

The simplest trigger. Just append the instruction to think first.

```
[Your question or task here]

Think step by step before answering.
```

Or: `Let's think through this carefully.`
Or: `Walk me through your reasoning before giving the final answer.`

**Best for:** Quick boost on analytical tasks with no examples handy.

---

### 2. Few-Shot CoT

Provide examples that show the reasoning process, not just the answer.

```
Q: A store sells apples for $0.50 each and bananas for $0.30 each.
   If I buy 4 apples and 6 bananas, how much do I spend?

A: First, calculate apples: 4 × $0.50 = $2.00
   Then, calculate bananas: 6 × $0.30 = $1.80
   Total: $2.00 + $1.80 = $3.80

Q: [Your actual question]
A:
```

**Best for:** Consistent reasoning on a class of similar problems (math, classification, structured analysis).

---

### 3. Self-Consistency

Run the same prompt multiple times (or ask explicitly), then pick the most common answer. Reduces single-run errors.

```
Solve this problem three different ways, then state which answer you're most confident in and why.

[Problem]
```

**Best for:** High-stakes decisions where a single wrong answer has real cost.

---

### 4. Step-Back Prompting

Ask for a higher-level principle before solving the specific case.

```
Before answering, identify the general principle or concept that applies here.
Then use that principle to answer the specific question.

[Question]
```

**Best for:** Problems that require domain knowledge the model might apply inconsistently.

---

### 5. Tree of Thought (ToT)

Explore multiple solution paths explicitly before committing.

```
Consider three different approaches to solving this problem.
For each approach: outline the steps, identify risks, and rate likelihood of success (1-10).
Then choose the best approach and execute it.

[Problem]
```

**Best for:** Complex planning, architecture decisions, strategy problems with many valid paths.

---

### 6. Reflection / Critique

Ask the model to check its own work.

```
[Task or question]

After answering, review your response and identify:
- Any errors or gaps
- Anything you're uncertain about
- Whether the answer fully addresses the question

Then revise if needed.
```

**Best for:** High-accuracy needs — summaries, code, factual claims.

---

## CoT for Code Debugging

```
Here is a bug report and the code:

<bug>
[Description of the bug]
</bug>

<code>
[Code block]
</code>

Before writing a fix:
1. Identify all the places the bug could originate
2. Trace the execution path that leads to the error
3. State your hypothesis for the root cause
4. Then write the fix

Explain each step as you go.
```

---

## Quick Reference

| Pattern | Trigger Phrase | Best For |
|---------|---------------|----------|
| Zero-Shot CoT | "Think step by step" | General analytical tasks |
| Few-Shot CoT | Show examples with reasoning | Repeatable problem types |
| Self-Consistency | "Solve three ways" | High-stakes single answers |
| Step-Back | "What principle applies?" | Domain-knowledge problems |
| Tree of Thought | "Consider three approaches" | Planning and strategy |
| Reflection | "Review and revise" | Accuracy-critical outputs |
