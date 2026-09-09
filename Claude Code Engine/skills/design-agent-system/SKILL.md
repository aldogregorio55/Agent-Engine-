---
name: design-agent-system
description: >-
  Architects multi-agent orchestration systems — agent rosters, handoff contracts,
  tool assignments, context budgets, and failure policies.
  Use when the user says "design an agent system", "architect a multi-agent workflow",
  "build an agent pipeline", "how should these agents be structured", "design the
  agent architecture", or any request to design a system where multiple agents
  collaborate to complete a task.
  Do NOT use when: the task involves a single agent only (use engineer-prompts or
  design-automation), or when designing prompt chains without agents (use chain-prompts).
allowed-tools: Read, Write
context: fork
agent: strategic-design
---

# Design Agent System

## Your Task
Architect a complete multi-agent system. Define the agent roster, orchestration pattern,
tool assignments, context budget strategy, handoff contracts, and failure policy.
Write the full architecture spec to a file. Return file path + summary.

## Steps

1. **Clarify the end-to-end goal**
   - What is the system trying to accomplish? (One sentence)
   - What is the input to the system? What is the output?
   - Are there latency, cost, or reliability constraints?
   - What tools/APIs/data sources are available?

2. **Choose the orchestration pattern**
   Evaluate and select one:
   - **Sequential pipeline**: Agent A → Agent B → Agent C (simple, predictable, easy to debug)
   - **Parallel fan-out**: Orchestrator launches N agents simultaneously, merges results
   - **Hierarchical**: Orchestrator delegates to sub-orchestrators who manage sub-agents
   - **Loop/reflection**: Agent runs, evaluator scores, agent revises until threshold met
   - **Hybrid**: Combination — parallel research, sequential synthesis, eval loop on output
   Document the reason for the chosen pattern.

3. **Define the agent roster**
   For each agent in the system:
   - **Name**: role-based (e.g., `research-analyst`, `data-extractor`, `output-validator`)
   - **Responsibility**: one job, one sentence
   - **Tools**: exact tools this agent is allowed to use
   - **Context strategy**: fork (isolated) or inline (shared context)
   - **Model**: which model — justify if not default
   - **Input**: what it receives (source, format, schema)
   - **Output**: what it produces (format, schema, file path if applicable)

4. **Write handoff contracts**
   For every agent-to-agent connection:
   - What does Agent A guarantee to pass to Agent B?
   - What schema/format must the output conform to?
   - What does Agent B do if Agent A's output is malformed or empty?

5. **Design the context budget strategy**
   - Which agents run in forks? (All agents with 3+ tool calls or >500 token outputs)
   - What do agents return to the orchestrator? (File path + summary — never raw output)
   - What is the max return token cap per agent? (200-300 tokens)
   - How does the orchestrator accumulate results without flooding its context?

6. **Define the failure policy**
   For each agent:
   - **Retry**: conditions and max attempts
   - **Skip**: is this agent optional? What is the degraded output if skipped?
   - **Abort**: when does the orchestrator stop the entire system?
   - **Escalate**: when does failure surface to the user for judgment?

7. **Write the architecture spec to file**
   - Output path: provided in brief, or default to `agent-outputs/agent-system-[date].md`

## Output Format

```markdown
# Agent System Architecture: [System Name]
**Date:** [Date]
**Goal:** [End-to-end task]
**Orchestration pattern:** [Sequential / Parallel / Hierarchical / Loop / Hybrid]
**Agent count:** [N]

## System Diagram

[Orchestrator]
    ├── [Agent 1: name] → writes to: [file path]
    ├── [Agent 2: name] → writes to: [file path]
    └── [Agent 3: name] → writes to: [file path]
         ↓
    [Merge/synthesis step if applicable]

## Agent Roster

### Agent: [Name]
**Responsibility:** [One sentence]
**Tools:** [List]
**Context strategy:** fork / inline
**Model:** [Model — rationale if non-default]
**Input:** [Source and schema]
**Output:** [File path + summary schema]

---

[Repeat for each agent]

## Handoff Contracts

### [Agent A] → [Agent B]
**Passes:** [Field list with types]
**Validation:** [What must be true]
**On failure:** [What Agent B does]

## Context Budget Strategy
| Agent | Context mode | Return format | Token cap |
|-------|-------------|---------------|-----------|
| [Name] | fork | file path + summary | 200 tokens |

## Failure Policy
| Agent | Retry | Skip allowed | Abort condition | Escalate condition |
|-------|-------|-------------|-----------------|-------------------|
| [Name] | [N times] | [Yes/No] | [Condition] | [Condition] |

## Implementation Notes
[Key decisions, tradeoffs accepted, known limitations]
```

## Edge Cases

- If only one agent is needed: flag that the system may be over-engineered; recommend a single-agent approach with a skill
- If the user wants agents to share a context: flag the context cost risk; recommend fork isolation unless there is a specific reason for shared context
- If tool access is not specified: list the minimal tools each agent needs; note assumptions
- If the system is recursive (agent calls itself or loops back): design an explicit exit condition and max iteration limit
- If cost is a constraint: identify which agents can use lighter models (Haiku for simple extraction, Sonnet for reasoning, Opus for evaluation)
