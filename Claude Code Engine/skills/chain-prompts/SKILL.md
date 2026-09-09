---
name: chain-prompts
description: >-
  Architects multi-step prompt pipelines where the output of one prompt feeds the next.
  Use when the user says "chain these prompts", "build a prompt pipeline", "design a
  multi-step prompt workflow", "connect prompt A to prompt B", "orchestrate prompts",
  or any request to design a sequence of prompts with explicit handoffs.
  Do NOT use when: the task can be accomplished in a single prompt (use engineer-prompts),
  or when designing a full agent system with tool use (use design-agent-system).
allowed-tools: Read, Write
context: fork
agent: prompt-architect
---

# Chain Prompts

## Your Task
Design a multi-step prompt pipeline with explicit handoff contracts between each stage.
Each prompt in the chain is a standalone unit. Handoff contracts define exactly what
passes between them. Write the full pipeline spec to a file. Return file path + summary.

## Steps

1. **Decompose the task into stages**
   - What is the end-to-end goal?
   - What are the natural logical stages? (extract → classify → generate → validate, etc.)
   - Which stages are sequential (output A feeds B) vs. parallel (independent, merge later)?
   - What is the minimum number of prompts? Eliminate stages that don't add value.

2. **Define each stage**
   For every stage in the chain, specify:
   - **Stage name**: verb-noun format (e.g., `extract-entities`, `classify-intent`)
   - **Input**: exact format and source (upstream output, user input, external data)
   - **Task**: single, well-scoped action — one stage, one job
   - **Output**: exact schema — fields, types, required vs. optional
   - **Model**: which model runs this stage (and why, if different from others)
   - **Failure mode**: what happens if this stage produces bad output

3. **Write handoff contracts**
   A handoff contract defines what Stage N guarantees to Stage N+1:
   - Output schema (exact fields and types)
   - Validation rules (what must be true for the output to be safe to pass on)
   - Error output format (what Stage N returns when it cannot complete successfully)
   - Null/empty case handling (what Stage N+1 does if the field is missing or empty)

4. **Design the failure policy**
   - **Retry**: under what conditions should a stage retry? (max attempts, backoff)
   - **Skip**: can a stage be skipped if it fails? (e.g., optional enrichment stage)
   - **Abort**: when should the pipeline stop entirely?
   - **Fallback**: is there a degraded output path if a stage fails?

5. **Write the full pipeline spec**
   - Diagram the chain (text-based: `Stage A → Stage B → Stage C`)
   - Write each stage prompt (complete, ready-to-use)
   - Write each handoff contract
   - Document the failure policy
   - Output path: provided in brief, or default to `agent-outputs/chain-[date].md`

## Output Format

```markdown
# Prompt Chain: [Pipeline Name]
**Date:** [Date]
**Goal:** [End-to-end task this pipeline accomplishes]
**Stages:** [N]
**Model(s):** [Model(s) used]

## Pipeline Diagram

[Stage 1: name] → [Stage 2: name] → [Stage 3: name]
                       ↓
               [Stage 2b: parallel branch, if any]

## Stage Specifications

### Stage 1: [Name]
**Input:** [Source and format]
**Task:** [What this stage does — one sentence]
**Model:** [Model]

[COMPLETE PROMPT — ready to copy-paste]

**Output schema:**
```json
{
  "field_name": "type — description",
  "field_name": "type — description"
}
```

---

### Stage 2: [Name]
[Same structure]

---

## Handoff Contracts

### Stage 1 → Stage 2
**Guaranteed fields:** [List]
**Validation rules:** [What must be true]
**Error format:** `{ "error": "string", "stage": "stage-1", "input_received": "..." }`
**Null handling:** [What Stage 2 does if field is null]

## Failure Policy
| Condition | Action |
|-----------|--------|
| Stage N times out | [Retry N times, then abort / fallback] |
| Stage N returns error format | [Abort / skip / fallback] |
| Stage N output fails validation | [Retry with clarification / abort] |

## Known Limitations
[What this pipeline won't handle well — multi-stage failure cascades, latency cost, etc.]
```

## Edge Cases

- If only two prompts are provided: design the chain and the handoff contract; note that a two-stage chain may be collapsible to one prompt — flag if so
- If stages have circular dependencies: flag as a design error; propose a resolution (loop with exit condition, or restructure)
- If the user wants parallel branches: design a fork/merge pattern with an explicit merge stage
- If model selection is not specified: default to the same model for all stages; note where a cheaper model is viable for simpler stages
- If an intermediate stage's output format is ambiguous: define it explicitly even if the user didn't — annotate it as a design decision
