# Workflow: Code → Test → Document

A three-phase prompt sequence for shipping well-tested, documented code. Run phases in order.

---

## When to Use This

- Building a new feature or function from a spec
- Writing library or utility code that others will use
- Any code that needs tests and documentation alongside it

---

## Phase 1: Code

**Goal:** Implement the feature cleanly before touching tests or docs.

```
Implement [FEATURE / FUNCTION / MODULE].

Context:
- Language/stack: [LANGUAGE AND FRAMEWORK]
- This will be used by: [WHO CALLS THIS CODE]
- Constraints: [performance requirements, dependencies to use/avoid, etc.]

Spec:
[DESCRIBE WHAT IT SHOULD DO — inputs, outputs, behavior, edge cases]

Requirements:
- No unnecessary abstractions — solve exactly this problem
- Prefer explicit over clever
- Handle these edge cases: [LIST]
- Do not handle these cases (caller's responsibility): [LIST]

Write the implementation only. No tests, no documentation yet.
```

---

## Phase 2: Tests

**Goal:** Write tests against the implementation — not against your assumptions.

```
Write tests for this implementation:

<code>
[PASTE IMPLEMENTATION]
</code>

Test framework: [Jest / pytest / Go test / etc.]

Cover:
1. Happy path — standard inputs produce correct outputs
2. Edge cases — [list the edge cases from Phase 1]
3. Error cases — invalid inputs, null/undefined, empty collections
4. Boundary conditions — min/max values, empty vs null, etc.

Rules:
- Each test should have one clear assertion
- Test names should read as: "it [does X] when [condition Y]"
- No mocking unless the thing being mocked is a network call or DB
- Don't test implementation details — test observable behavior

Write all tests now.
```

---

## Phase 3: Document

**Goal:** Write docs that help the next developer use this correctly.

```
Write documentation for this code. The audience is a developer who hasn't seen this before.

<code>
[PASTE IMPLEMENTATION]
</code>

<tests>
[PASTE TESTS — they reveal edge cases and usage patterns]
</tests>

Include:
1. **Purpose** — one sentence: what does this do and why does it exist?
2. **Usage** — minimal working example (copy-pasteable)
3. **Parameters** — name, type, description, whether required
4. **Returns** — what comes back, including error states
5. **Edge cases** — non-obvious behaviors the caller should know about
6. **Don't use this when** — cases where a different approach is better

Format: inline docstring in the language's standard style ([JSDoc / docstring / GoDoc / etc.])
Do not restate what is obvious from the type signatures.
```

---

## Phase 4: Integrate (Optional)

**Goal:** Check that the new code fits the surrounding codebase.

```
Review this new code in the context of the existing codebase.

<new-code>
[PASTE IMPLEMENTATION + TESTS]
</new-code>

<relevant-existing-code>
[PASTE RELATED FILES]
</relevant-existing-code>

Check:
1. Does it follow the existing naming and style conventions?
2. Does it duplicate anything that already exists?
3. Are there existing utilities it should use instead of reimplementing?
4. Does it fit naturally into the existing module structure?
5. Any API surface changes that would break existing callers?

Flag issues only — don't rewrite anything unless asked.
```
