# Multi-Agent Workflow Development — Practices & Findings

**Purpose:** Conceptual reference for building and debugging multi-agent workflows on declarative platforms (e.g. Azure AI Foundry, Copilot Studio). Grounded in real build experience.
**Last Updated:** 2026-05-05

---

## 1. Platform Mental Model — The Most Important Thing

**Agents are nodes, not orchestrators.**

On declarative platforms, the workflow YAML owns all routing and sequencing. Agents are invoked as nodes — they receive input and produce output. They cannot invoke other agents, transfer control, or know where they sit in the pipeline.

This has a critical implication for prompt writing: **any orchestration language in an agent's system prompt will cause it to simulate orchestration in natural language.** It will describe what it's about to do, output fake tool calls as text, or ask meta-questions about the system architecture — because it believes it is responsible for routing.

**Rule:** Each agent's system prompt should describe only its own task and its own output. Routing language, agent names, and transfer commands do not belong in agent prompts on declarative platforms.

---

## 2. Data Delivery Is the Foundation

Before debugging prompt logic, verify that each agent is actually receiving the data it needs.

**On declarative platforms, conversation history is not the same as file content.** Passing a message that references a file is not the same as the agent having access to that file. Data sources (transcripts, frameworks, knowledge bases) must be explicitly attached to each agent that needs them — the platform does not pass file content between agents automatically.

**The failure mode:** An agent without proper data access will produce plausible-looking output by drawing on training data. The output will be thematically coherent but ungrounded. This is hard to detect without comparing against a known ground truth.

**Rule:** Before running end-to-end, verify each agent's data sources in isolation. Confirm the agent can retrieve what it needs before wiring the pipeline together.

---

## 3. Test One Variable at a Time

Multi-agent systems have many variables: prompt content, tool configuration, data sources, model choice, platform configuration. When something goes wrong, it is almost never obvious which variable is responsible.

**The discipline:** Change one thing, test, observe, record. Then change the next thing.

**In practice:**
- Test each agent in isolation (agent chat window) before running the full workflow
- When a failure occurs, resist the urge to change prompts and configuration simultaneously
- If you need to verify a platform or configuration hypothesis, use a minimal test input — not a full workflow run
- Record every test result, including the exact conditions and configuration at the time

**The cost of not doing this:** You end up with multiple changed variables and no clean signal. You can fix things accidentally and not know why it worked, or break things and not know why it broke.

---

## 4. Tool Configuration

### `tool_choice: required` is dangerous

Setting `tool_choice: required` forces the agent to make a tool call on every single invocation, regardless of whether the task requires it. This causes two problems:

1. **In review loops:** An agent receiving feedback and revising output does not need to re-retrieve files. Forced retrieval wastes tokens and can contaminate the revision with stale context.
2. **On platforms with content scanning:** Forced, unguided tool calls look anomalous to content safety scanners. A tool call with no context or query structure is more likely to be flagged than a purposeful retrieval with a well-formed query.

**Default to `tool_choice: auto`.** Only force tool use if there is a specific reason every single invocation requires retrieval.

### Tool invocations are a content safety surface

On platforms that scan tool calls (inputs and outputs), every tool invocation passes through the content safety layer. At high rates of forced invocation across multiple agents, probabilistic false-positive rates compound rapidly.

For a 4-agent workflow where 3 agents each have a ~30% false-positive rate per invocation:
- Per-agent pass probability: 70%
- End-to-end pass probability: 0.7³ ≈ 34%

Reducing forced tool invocations (via `tool_choice: auto`) is the primary lever for reducing this compounding effect.

---

## 5. Model-Specific Platform Behaviour

Not all models behave identically on the same platform. Platform-level features (guardrails, tool configuration, content filters) may be implemented differently per model.

**Before committing to a model choice:**
- Test the model's compatibility with the specific platform features you depend on (tool use, structured output, guardrails)
- Run isolation tests on each model under your actual configuration — not just the baseline
- Use the model with the higher sensitivity as your canary when diagnosing platform issues (a more sensitive model surfaces problems faster with smaller sample sizes)

**Do not extrapolate from a small sample.** A model appearing fully blocked in 3–4 consecutive tests may be experiencing the same probabilistic issue as other models — just hitting it more frequently. Run at least 10 controlled tests before drawing a categorical conclusion.

---

## 6. Isolation Testing Protocol

When diagnosing a failure, work from the simplest possible configuration toward complexity. Each step eliminates one variable.

**For guardrail / platform block issues:**

| Test | What It Eliminates |
|------|--------------------|
| Empty agent (no prompt, no tools) | If 0 flags: platform and model are not the root cause |
| Add tools, keep prompt empty | If flags increase: tool invocation is the trigger |
| Add real prompt, keep tools | If flags decrease vs. empty prompt: prompt structure helps but tools are still the cause |
| Change `tool_choice: required` → `auto` | If flags drop to near zero: forced tool calls are the specific mechanism |
| Swap model | Isolates model-specific sensitivity |

**Always record:** model, tool configuration, `tool_choice` setting, prompt state (empty / real), input used, and all 10 results.

---

## 7. Documentation Practices for Multi-Agent Builds

### What to record per session
- **Design decisions:** What was decided, what options were considered, why this one was chosen. Record these once — they prevent re-litigation in future sessions.
- **Test results:** Structured tables, not narrative. Exact conditions, exact results.
- **Root cause corrections:** When a root cause is found to be wrong, correct it with an explanation of why it was wrong and what the real cause is. Do not delete or overwrite the original — the correction is part of the record.
- **Timeline of events:** When failures were discovered, what was changed, what was reverted and why. This prevents future confusion about which configuration was in place during which run.

### Session structure
- **Index file:** One entry per session — summary, outcome, blockers. Scannable in under a minute.
- **Detail log:** All sessions in one file, in order. Full test data, issue analysis, design decisions, outstanding items attached to the session that generated them.
- **Platform reference:** Running log of confirmed platform behaviours. Updated as new quirks are discovered — not reconstructed from memory later.

### What belongs where
- Decisions and rationale → detail log
- Confirmed platform behaviour → platform reference
- Outstanding items → attached to the session that generated them, not a floating to-do list
- Resolved items → marked explicitly as resolved, not deleted

---

## 8. Common Failure Patterns

| Pattern | Symptom | Root Cause | Fix |
|---|---|---|---|
| Agent simulates full workflow | Single agent produces output for all steps | Orchestration language in system prompt | Remove all routing/transfer/delegation language |
| Plausible but ungrounded output | Agent produces coherent results that don't match source material | Agent lacks access to the data source | Attach data source directly to the agent's tool configuration |
| Pseudo-handoff narration | Agent outputs `` `transfer_to_x` `` as plain text | SDK-derived handoff language in prompt | Replace with output descriptions — "output your completed analysis" |
| Process narration | Agent announces what it's about to do before doing it | No explicit rule against it | Add: "Do not describe your process — produce your output directly" |
| Review loop rubber-stamp | Reviewer approves everything without scrutiny | Reviewer has no mandate to independently verify | Explicit verification mandate per item type (Y items AND N/A items) |
| Intermittent platform blocks | Agent blocked on some runs, not others, with identical input | Forced tool invocations hitting content safety scanner | Change `tool_choice: required` → `auto`; audit which agents actually need forced retrieval |
