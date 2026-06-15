 I'll search for information about AI prompting frameworks beyond CRAFT, including their complexity levels and strategies for managing context in long-running projects.
Great question! Beyond CRAFT, there are numerous frameworks ranging from minimalist 3-component structures to sophisticated multi-agent architectures. Here's a comprehensive breakdown organized by skill level, plus strategies for managing context in long-running projects.

---

## **Prompt Frameworks by Skill Level**

### **Beginner-Friendly (3-4 Components)**

| Framework | Components | Best For | Why It Works |
|-----------|-----------|----------|--------------|
| **RTF** | Role, Task, Format | Quick prototyping, code generation, debugging | Dead simple—gets you 80% there with minimal cognitive load  |
| **RACE** | Role, Action, Context, Expectation | Fast deployments, internal agents, templated responses | Agile-friendly; just enough structure without ceremony  |
| **APE** | Action, Purpose, Expectation | Clarifying specific tasks with clear deliverables | Forces you to articulate *why* you need something  |

**Recommendation for beginners:** Start with **RTF**. It's memorable, fast, and produces predictable outputs. Once comfortable, graduate to **RACE** for slightly more nuance without overwhelming complexity.

---

### **Intermediate (5-6 Components)**

| Framework | Components | Best For | Key Advantage |
|-----------|-----------|----------|---------------|
| **CO-STAR** | Context, Objective, Style, Tone, Audience, Response | Content creation, customer-facing outputs, documentation | Won Singapore's GPT-4 competition; treats prompting as full-stack design  |
| **CRISPE** | Capacity, Insight, Statement, Personality, Experiment | Exploration, A/B testing, iterative refinement | Built at OpenAI; explicitly acknowledges prompting as experimental  |
| **CARE** | Context, Action, Result, Example | Teaching, best practices, before/after comparisons | Excellent for illustrating patterns with concrete examples  |
| **PECRA** | Purpose, Expectation, Context, Request, Audience | Audience-tailored outputs, training materials, compliance | Comprehensive contextual understanding  |

**Recommendation for intermediate users:** **CO-STAR** is the gold standard here. It adds the crucial "Audience" and "Tone" dimensions that separate generic AI output from professional-grade content. Use **CRISPE** when you're still discovering the optimal approach for a new task type.

---

### **Advanced (Multi-Step & Agentic)**

| Framework/Approach | Structure | Best For | Complexity Level |
|-------------------|-----------|----------|----------------|
| **RISEN** | Role, Instructions, Steps, End goal, Narrowing | Multi-step workflows, CI/CD setups, migrations, structured tutorials | High—explicit step sequencing  |
| **Chain-of-Thought (CoT)** | "Let's think step by step" + reasoning | Debugging, problem-solving, transparency in reasoning | Medium-High; encourages explicit reasoning  |
| **Tree of Thoughts (ToT)** | Decision tree with multiple options, recursive exploration, ranking | Technical troubleshooting, architectural decisions, logic-heavy workflows | Very High—explores multiple paths simultaneously  |
| **MPT (Modules-Pathways-Triggers)** | Self-contained modules, strategic pathways, activation triggers | Dynamic, adaptive interactions; complex scenarios requiring conditional logic | Expert—transforms prompts into "living systems"  |

**Recommendation for advanced users:** Master **Tree of Thoughts** for high-stakes decisions where you need to explore multiple implementation paths. For production systems, adopt the **MPT framework**—it creates modular, conditionally-activated prompt architectures that scale beyond single-turn interactions.

---

## **Framework Selection Matrix**

| Use Case | Recommended Framework | Level |
|----------|----------------------|-------|
| Quick code snippet | RTF | Beginner |
| Debug an error | RACE or CoT | Beginner/Intermediate |
| Write documentation | CO-STAR | Intermediate |
| Design system architecture | ToT | Advanced |
| Customer service automation | CO-STAR or PECRA | Intermediate |
| Multi-agent coding workflows | MPT + Context Engineering | Advanced |
| A/B testing prompts | CRISPE | Intermediate |

---

## **Managing Context in Long-Running Projects**

This is where "Context Engineering" becomes critical. For projects spanning hours, days, or multiple tasks, you'll hit the context window ceiling and face "context drift"—where accumulated noise degrades performance. Here are the proven strategies:

### **1. Compaction & Summarization**
When approaching token limits, summarize the conversation history into a high-fidelity compressed form that preserves:
- Architectural decisions
- Unresolved bugs/issues  
- Implementation details
- Recent file accesses

Discard redundant tool outputs and intermediate steps. Anthropic's Claude Code implements this by passing message history to the model for intelligent compression .

### **2. External Memory with Selective Retrieval**
Don't carry everything in-context. Store specs, decisions, and architectural notes in a central knowledge base (vector DB, files, or structured docs). Agents query only what they need for the current task. This keeps prompts lean and creates audit trails .

**Implementation:**
- Store: Design docs, API contracts, decision logs, code patterns
- Retrieve: Relevant sections based on current task + 2-3 representative examples
- Refresh: Update external memory with new decisions at milestones

### **3. Scoped, Role-Specific Context**
Break projects into specialized agents rather than one generalist:
- **Test Author Agent:** Receives only test frameworks and acceptance criteria
- **Refactor Specialist:** Gets code patterns and anti-pattern guidelines
- **Documentation Agent:** Pulls terminology guidelines and customer-facing examples

Each agent sees only what it needs, limiting drift surface area .

### **4. Checkpoint & Refresh Pattern**
At natural boundaries (milestones, phase completions):
1. Have the model summarize: "What have we accomplished? What's the current state? What decisions were made?"
2. Start a fresh context with that summary + essential carry-over only
3. Archive the full history for audit purposes

This is "garbage collection for your context"—refactoring accumulated cruft .

### **5. Structured Context Boundaries**
Use explicit markers in prompts:
```
[PREVIOUS ATTEMPTS - REFERENCE ONLY]
...old attempts...

[CURRENT WORKING CONTEXT]
...active task...

[CONSTRAINTS]
...invariant rules...
```
This helps the model prioritize what matters now versus historical noise .

### **6. Multi-Agent Architecture with Coordinator**
For complex initiatives, implement:
- **Worker Agents:** Execute specific tasks with scoped context
- **Coordinator/Arbiter Agent:** Reads worker outputs, compares against central spec, flags conflicts, tracks completion status

The coordinator focuses on alignment, not generation—creating a control layer that maintains coherence across parallel workstreams .

---

## **Practical Workflow for Long Projects**

| Phase | Action | Context Strategy |
|-------|--------|------------------|
| **Kickoff** | Use CO-STAR or RISEN for initial spec | Full context, comprehensive background |
| **Execution** | Switch to RTF for discrete tasks | Minimal, task-focused context |
| **Milestones** | Generate structured summary | Compaction—extract decisions and state |
| **Context Refresh** | Start new session with summary + constraints | Clean slate with essential carry-over |
| **Parallel Work** | Deploy scoped agents with coordinator | Role-specific context slices |
| **Integration** | Coordinator validates against central spec | External memory retrieval for cross-cutting concerns |

The key insight from Anthropic's research: **waiting for larger context windows isn't the solution**—even infinite windows suffer from context pollution. The discipline of *what to exclude* matters as much as what to include .

---

**Bottom line:** Beginners should master RTF/CO-STAR for immediate quality gains. Intermediate users should add CRISPE for iterative refinement. Advanced practitioners must shift from "prompt engineering" to **context engineering**—designing systems that manage state, memory, and multi-agent coordination for sustained coherence across complex projects.