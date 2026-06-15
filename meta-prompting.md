# Meta-Prompting

Meta-prompting uses AI to write, critique, and iterate on prompts. Instead of crafting prompts manually from scratch, you describe what you want and let the model draft the prompt — then refine it.

---

## Why It Works

Writing good prompts requires thinking like the model. The model already thinks like the model. Meta-prompting closes that gap.

It's especially useful when:
- You know what output you want but can't articulate the right instructions
- Your current prompt is underperforming and you've hit a wall
- You're building a prompt that needs to work reliably across many runs

---

## Core Patterns

### 1. Prompt Generator

Start here. Describe what you want; get a draft prompt back.

```
Write a prompt I can use to [goal].

The prompt will be used with [Claude / GPT-4 / etc.].
The user of this prompt is [describe who].
The output should be [format/length/style].
The most important constraint is [key constraint].

Return only the prompt — no explanation, no preamble.
```

---

### 2. Prompt Critiquer

Give it your existing prompt and have it find the weaknesses.

```
Critique this prompt. Identify the top 3 problems that would cause inconsistent or low-quality output.

<prompt>
[YOUR PROMPT]
</prompt>

For each problem:
- What's wrong
- Why it causes bad output
- How to fix it

Then rewrite the full prompt incorporating all fixes.
```

---

### 3. Prompt Expander

Take a minimal prompt and flesh it out.

```
Expand this rough prompt into a complete, high-quality prompt.

Rough prompt: "[YOUR ROUGH PROMPT]"

Add: clear role, specific action, output format, tone, and relevant constraints.
Do not change the core intent.
Return only the expanded prompt.
```

---

### 4. Prompt Stress-Tester

Find edge cases before your prompt goes into production.

```
You are a prompt stress-tester. Given this prompt, generate 5 example inputs that would cause it to produce bad, inconsistent, or off-target output.

<prompt>
[YOUR PROMPT]
</prompt>

For each bad input:
- Show the input
- Explain what would go wrong
- Suggest how to fix the prompt to handle it
```

---

### 5. Prompt Variants

Generate a family of prompts for A/B testing.

```
Write 3 different versions of a prompt for this task:

Task: [DESCRIBE THE TASK]

Version 1: Direct and minimal
Version 2: Detailed with examples
Version 3: Role-based with constraints

Label each version clearly. Return only the prompts — no commentary between them.
```

---

### 6. Few-Shot Example Generator

Have the model write the examples for your few-shot prompt.

```
I'm building a few-shot prompt for this task: [DESCRIBE TASK]

Write 3 high-quality input/output pairs that would teach the model the right pattern.

Format each pair as:
Input: [example input]
Output: [ideal output]

The output style should be [describe style].
```

---

## Full Meta-Prompting Workflow

1. **Describe** your goal to the model → get a draft prompt
2. **Stress-test** the draft → find edge cases
3. **Critique** the draft → get specific fixes
4. **Rewrite** with fixes applied
5. **Test** on 3-5 real inputs
6. **Iterate** until output is consistent

---

## Prompt for Building System Prompts

For Claude API / Claude.ai projects:

```
Write a system prompt for an AI assistant with these characteristics:

Role: [what the assistant does]
Audience: [who it serves]
Tone: [how it communicates]
Key behaviors: [what it always does]
Prohibited behaviors: [what it never does]
Output format defaults: [default structure of responses]

The system prompt should be under 300 words. Write it in second person ("You are..."). Return only the system prompt.
```
