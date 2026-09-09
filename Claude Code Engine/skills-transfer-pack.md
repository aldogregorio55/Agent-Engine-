# Claude Engine — Skills Transfer Pack
**Exported:** 2026-07-13
**Source machine:** personal (`~/.claude/skills/`)
**Contents:** SKILL.md files covering the skill library, verbatim, single-file transfer format

---

## Instructions for the receiving Claude Code instance

Perform the following installation task:

1. For each skill block below, create the directory `~/.claude/skills/<skill-name>/` where `<skill-name>` is the name in the `FILE:` marker.
2. Write the content between `=== BEGIN SKILL.md ===` and `=== END SKILL.md ===` to `~/.claude/skills/<skill-name>/SKILL.md`, byte-for-byte — do not reformat, summarize, or "improve" anything.
3. After all files are written, set every agent's `skills:` frontmatter in `~/.claude/agents/` to the full skill library — this is what grants functional reach; the recommendations table below is priority guidance, not a restriction.
4. Verify: list `~/.claude/skills/` and confirm one directory per skill, each containing exactly one SKILL.md.

## Agent → skill priority recommendations

Every agent (and the main orchestrator) can call any skill in the library — the only functional binding is that every agent's `skills:` frontmatter carries the full set. The table below is priority guidance only.

| Agent | Recommended skills (priority) |
|---|---|
| prompt-architect | engineer-prompts, audit-prompt, refactor-prompt, chain-prompts |
| ai-evaluator | evaluate-ai-output, score-prompt-quality, build-eval-dataset, audit-prompt |
| strategic-design | design-automation, develop-strategy, design-agent-system, design-eval-framework, chain-prompts |
| research-analyst | synthesize-research, build-eval-dataset |
| business-analyst | develop-strategy |
| senior-engineer | design-agent-system |
| content-strategist | produce-deliverable |
| document-writer | — |
| project-manager | — |
| red-team | — |
| workspace-architect | — |

"—" = no priority preference; the agent still reaches every skill in the library.

---

<!-- FILE: ~/.claude/skills/audit-prompt/SKILL.md -->
=== BEGIN SKILL.md (audit-prompt) ===
---
name: audit-prompt
description: >-
  Diagnoses an existing prompt's weaknesses, failure modes, and structural gaps.
  Use when the user says "audit this prompt", "what's wrong with this prompt",
  "why is this prompt failing", "diagnose this prompt", "review this prompt",
  or any request to assess an existing prompt before changing it.
  Do NOT use when: the user wants to write a new prompt from scratch (use engineer-prompts),
  or when the user wants to fix the prompt immediately (use refactor-prompt).
allowed-tools: Read, Write
context: fork
agent: prompt-architect
---

# Audit Prompt

## Your Task
Diagnose the provided prompt systematically. Identify structural gaps, ambiguity sources,
and failure modes across all five components. Write the audit report to a file.
Return file path + summary. Do NOT rewrite the prompt — diagnosis only.

## Steps

1. **Capture the prompt and context**
   - What model will run this prompt?
   - What task is it meant to accomplish?
   - What failure behavior has been observed? (if provided)
   - What is the deployment context? (API, chat, batch)

2. **Score each component (1-5)**
   Evaluate the prompt against the five-component framework:
   - **Role**: Is the persona specific, relevant, and at the right expertise level?
   - **Instructions**: Are steps numbered, unambiguous, and complete? Any gaps or contradictions?
   - **Input/Context**: Are data boundaries marked? Is injection format specified?
   - **Output format**: Is the schema exact — field names, types, required vs. optional?
   - **Examples**: Are there at least 1-2 demonstrations? Do they match the output format?

3. **Identify failure modes**
   For each component scored below 4, name the failure mode:
   - Vague role → inconsistent persona, tone drift
   - Missing steps → model improvises, fills gaps incorrectly
   - Unmarked input → model confuses injected data with instructions
   - Underspecified output → format inconsistency across runs
   - No examples → high variance on edge cases

4. **Assess cross-component conflicts**
   - Does the role definition conflict with the instructions?
   - Does the output format contradict the examples?
   - Are there instructions that can't be satisfied given the input spec?

5. **Rank issues by severity**
   - **Critical**: Causes incorrect or unsafe outputs (wrong facts, format failures, off-task responses)
   - **Major**: Causes inconsistency across runs or with edge inputs
   - **Minor**: Reduces efficiency or clarity but doesn't break outputs

6. **Write audit report to file**
   - Output path: provided in brief, or default to `agent-outputs/audit-[date].md`

## Output Format

```markdown
# Prompt Audit Report: [Name or short description]
**Date:** [Date]
**Target model:** [Model]
**Task type:** [Classify / Extract / Generate / Reason / Evaluate / Converse]

## Component Scores

| Component | Score (1-5) | Finding |
|-----------|-------------|---------|
| Role | [X] | [One-line finding] |
| Instructions | [X] | [One-line finding] |
| Input/Context | [X] | [One-line finding] |
| Output format | [X] | [One-line finding] |
| Examples | [X] | [One-line finding] |

**Overall health:** [X/5 average]

## Issues by Severity

### Critical
- **[Component]**: [Issue] → [Failure mode it causes]

### Major
- **[Component]**: [Issue] → [Failure mode it causes]

### Minor
- **[Component]**: [Issue] → [Failure mode it causes]

## Cross-Component Conflicts
[List any conflicts between components, or "None identified"]

## Root Cause Summary
[2-3 sentences: what is the core structural problem, what is it causing, what domain of fixes is needed]

## Recommended Next Step
[Single action: "Run refactor-prompt to address [specific issues]" or "Run engineer-prompts to rebuild from scratch if [condition]"]
```

## Edge Cases

- If no failure behavior is provided: audit against all five components anyway; flag that real failure data would sharpen the diagnosis
- If the prompt is very short (<50 tokens): note that brevity may be intentional; assess whether it's appropriate for the task type
- If the prompt scores 4+ on all components: return a clean bill of health with minor improvement suggestions; do not manufacture issues
- If the prompt is a system prompt vs. a user-turn prompt: note the difference; system prompts have different role/instruction norms
- Never rewrite the prompt in this skill — diagnosis only; refactoring belongs in refactor-prompt

=== END SKILL.md (audit-prompt) ===

---

<!-- FILE: ~/.claude/skills/build-eval-dataset/SKILL.md -->
=== BEGIN SKILL.md (build-eval-dataset) ===
---
name: build-eval-dataset
description: >-
  Generates a stratified labeled evaluation dataset covering happy path, boundary,
  adversarial, and regression cases for a prompt or AI system.
  Use when the user says "build an eval dataset", "create test cases for this prompt",
  "generate evaluation examples", "build a benchmark dataset", "create labeled examples
  for testing", or any request to produce a set of input-output pairs for systematic testing.
  Do NOT use when: the user wants to design the eval framework/infrastructure (use design-eval-framework),
  or when evaluating a single existing output (use evaluate-ai-output).
allowed-tools: Read, Write, WebSearch
context: fork
agent: ai-evaluator
---

# Build Eval Dataset

## Your Task
Generate a stratified labeled evaluation dataset for the specified prompt or AI system.
Cover all four case categories. Label every example with expected output and difficulty tier.
Write the dataset to a file. Return file path + dataset summary (count by category).

## Steps

1. **Profile the prompt or system**
   - What is the task? (classify, extract, generate, reason, evaluate)
   - What is the input format? What is the expected output format?
   - What are the known edge cases or failure modes? (if provided)
   - How many examples are needed? (minimum 10; recommend 30-50 for reliable evaluation)

2. **Define the four case categories**

   **Category 1: Happy Path (40% of dataset)**
   Standard inputs that the prompt is designed to handle well.
   - Representative of the most common real-world inputs
   - Varying complexity within the normal range
   - Clear, unambiguous expected outputs
   - Goal: verify the prompt works correctly on intended inputs

   **Category 2: Boundary Cases (25% of dataset)**
   Inputs at the edges of what the prompt is designed to handle.
   - Minimal inputs (empty, very short, single word)
   - Maximal inputs (very long, dense, complex)
   - Format variations (different structures, encodings, styles)
   - Goal: verify graceful behavior at the edges

   **Category 3: Adversarial Cases (20% of dataset)**
   Inputs designed to stress or break the prompt.
   - Ambiguous inputs (multiple valid interpretations)
   - Off-topic inputs (outside the intended domain)
   - Contradictory inputs (conflicting information)
   - Prompt injection attempts (if relevant to the deployment context)
   - Goal: verify the prompt fails gracefully and doesn't produce harmful outputs

   **Category 4: Regression Cases (15% of dataset)**
   Cases targeting previously observed or anticipated failure modes.
   - Inputs that mirror known failure patterns
   - Cases that test specific prompt components (role, output format, examples)
   - Cases designed to catch regressions after prompt changes
   - Goal: ensure changes don't re-introduce fixed failures

3. **Generate examples**
   For each example:
   - Write a realistic input (not obviously artificial)
   - Specify the expected output (exact or schema-level — be precise)
   - Label the difficulty: Easy / Medium / Hard
   - Label the category: happy-path / boundary / adversarial / regression
   - Add a brief annotation: what this case is testing and why

4. **Validate dataset balance**
   - Category distribution matches the targets above (±5%)
   - Difficulty distribution: at least 30% Hard cases
   - No duplicate or near-duplicate examples
   - Expected outputs are realistic and achievable by the prompt

5. **Write dataset to file**
   - Format: markdown table + JSON block (for programmatic use)
   - Output path: provided in brief, or default to `agent-outputs/eval-dataset-[date].md`

## Output Format

```markdown
# Eval Dataset: [Prompt or System Name]
**Date:** [Date]
**Task type:** [Classify / Extract / Generate / Reason / Evaluate]
**Total examples:** [N]
**Distribution:** Happy Path: [N] | Boundary: [N] | Adversarial: [N] | Regression: [N]

## Dataset

### Happy Path Examples

#### Example HP-01
**Difficulty:** Easy / Medium / Hard
**Input:**
```
[Input text]
```
**Expected output:**
```
[Expected output — exact or schema-level]
```
**Tests:** [What this case is testing]

---

[Repeat for all happy path examples]

### Boundary Examples

#### Example B-01
[Same structure]

### Adversarial Examples

#### Example A-01
[Same structure]

### Regression Examples

#### Example R-01
[Same structure]

---

## Dataset Summary

| Category | Count | % of total | Difficulty breakdown |
|----------|-------|-----------|----------------------|
| Happy Path | [N] | [%] | Easy: [N] / Med: [N] / Hard: [N] |
| Boundary | [N] | [%] | Easy: [N] / Med: [N] / Hard: [N] |
| Adversarial | [N] | [%] | Easy: [N] / Med: [N] / Hard: [N] |
| Regression | [N] | [%] | Easy: [N] / Med: [N] / Hard: [N] |

## JSON Dataset (for programmatic use)

```json
[
  {
    "id": "HP-01",
    "category": "happy-path",
    "difficulty": "medium",
    "input": "...",
    "expected_output": "...",
    "tests": "..."
  }
]
```

## Coverage Gaps
[Anything this dataset doesn't cover well — note explicitly]
```

## Edge Cases

- If the prompt's task is unclear: ask for a concrete example input-output pair before generating; use it to calibrate all other examples
- If ground truth expected outputs are impossible to specify exactly: use schema-level specifications ("must contain fields X and Y", "must be a valid JSON object") rather than exact strings
- If the user requests fewer than 10 examples: generate 10 minimum and note why fewer examples produce unreliable eval results
- If the task involves sensitive content (medical, legal, financial): flag that adversarial cases should be reviewed by a domain expert before use in production evals
- If regression cases are unknown: flag the gap; generate speculative regression cases based on common failure modes for the task type; label them as speculative

=== END SKILL.md (build-eval-dataset) ===

---

<!-- FILE: ~/.claude/skills/chain-prompts/SKILL.md -->
=== BEGIN SKILL.md (chain-prompts) ===
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

=== END SKILL.md (chain-prompts) ===

---

<!-- FILE: ~/.claude/skills/design-agent-system/SKILL.md -->
=== BEGIN SKILL.md (design-agent-system) ===
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

=== END SKILL.md (design-agent-system) ===

---

<!-- FILE: ~/.claude/skills/design-automation/SKILL.md -->
=== BEGIN SKILL.md (design-automation) ===
---
name: design-automation
description: >-
  Designs automation workflows, integration pipelines, and scheduled processes.
  Use when the user says "design an automation", "plan a workflow", "build an
  integration", "n8n workflow for", "automate this process", "connect X to Y",
  or any request involving automated data movement, triggers, or tool integrations.
allowed-tools: Read, Write, WebSearch
context: fork
agent: strategic-design
---

# Design Automation

## Your Task
Design a complete automation workflow spec from trigger to output.
Write the spec to a file. Return file path + brief summary to the orchestrator.

## Steps

1. **Map trigger and end state**
   - What event starts this automation? (webhook, schedule, manual, another workflow)
   - What is the desired end state? (data in system X, notification sent, file created)
   - What is the happy path? (describe in plain English, no tool names yet)

2. **Identify data sources and destinations**
   - What systems does data come from? (CRMs, calendars, forms, APIs, databases)
   - Where does data go? (Sheets, Slack, email, CRM, webhook)
   - What transformations happen in between? (filter, enrich, format, route)

3. **Design step sequence**
   - Break the happy path into discrete, atomic steps
   - For each step: input, operation, output, tool/node type
   - Number steps. Show data flow between them.

4. **Define conditional logic and error handling**
   - What branches exist? (if data missing → route to fallback, if API fails → retry then notify)
   - What are the failure modes? (empty response, rate limit, authentication failure)
   - Document every "if/then" explicitly — no implicit assumptions

5. **Write the spec doc**
   - Include: trigger, data flow diagram (text), step table, error handling table, tool list
   - Note any data that must be validated or cleaned before use
   - Flag any steps that require human review or approval

## Output Format

```markdown
# Automation Spec: [Name]
**Date:** [Date]
**Trigger:** [What starts it]
**End State:** [What success looks like]

## Data Flow

[Trigger] → [Step 1] → [Step 2] → ... → [Output]

## Steps

| # | Operation | Input | Output | Tool/Node | Notes |
|---|-----------|-------|--------|-----------|-------|
| 1 | [Operation] | [Data] | [Data] | [n8n node / API] | |
| 2 | | | | | |

## Conditional Logic

| Condition | Branch | Action |
|-----------|--------|--------|
| [If X] | [Then Y] | [Result] |

## Error Handling

| Failure Mode | Impact | Mitigation |
|---|---|---|
| [API timeout] | [Data loss] | [Retry 3x → notify Slack] |

## Tool / Integration List
- [Tool 1]: [Purpose]
- [Tool 2]: [Purpose]

## Open Questions
- [Anything that requires user decision before build]
```

## Edge Cases

- If the user hasn't defined a trigger: ask before proceeding — no trigger = no automation
- If a required integration doesn't have a native n8n node: note it, suggest HTTP Request node or webhook alternative
- If PII or sensitive data is in the flow: flag data handling requirements explicitly
- If the scope is too large for one workflow: recommend splitting into sub-workflows with handoff points

=== END SKILL.md (design-automation) ===

---

<!-- FILE: ~/.claude/skills/design-eval-framework/SKILL.md -->
=== BEGIN SKILL.md (design-eval-framework) ===
---
name: design-eval-framework
description: >-
  Designs a reusable evaluation infrastructure for an AI system — metrics selection,
  rubric architecture, dataset requirements, and measurement pipeline.
  Use when the user says "design an eval framework", "build an eval system",
  "how should I measure this AI system", "design the measurement infrastructure",
  "set up evals for this agent", or any request to design how an AI system will be
  measured over time (not just evaluate one output).
  Do NOT use when: the user wants to evaluate a single output right now (use evaluate-ai-output),
  or when building an eval dataset of examples (use build-eval-dataset).
allowed-tools: Read, Write
context: fork
agent: strategic-design
---

# Design Eval Framework

## Your Task
Design a complete, reusable evaluation framework for the specified AI system.
Define what gets measured, how it gets measured, who measures it, and how results feed back
into improvement. Write the framework spec to a file. Return file path + summary.

## Steps

1. **Profile the AI system**
   - What type of system is it? (RAG, agent, chatbot, prompt pipeline, classifier)
   - What task does it perform?
   - What are the known or suspected failure modes?
   - Who are the users and what do they care about most?
   - Is there ground truth available? (labeled data, expert reference outputs)

2. **Define evaluation dimensions**
   Select 3-5 dimensions that matter for this system. Common dimensions:
   - **Correctness**: Is the output factually or functionally right?
   - **Faithfulness**: Is the output grounded in the provided context?
   - **Completeness**: Does the output cover everything required?
   - **Relevance**: Is the output on-task and appropriate to the input?
   - **Format compliance**: Does the output match the specified schema?
   - **Safety**: Does the output avoid harmful, biased, or off-policy content?
   - **Efficiency**: Token cost, latency, tool call count (for agents)
   - **User satisfaction**: Proxy or direct measure of whether users got what they needed

3. **Assign measurement methods per dimension**
   For each dimension, select the appropriate method:
   - **Exact match**: for factual QA, entity extraction, classification
   - **Semantic similarity (BERTScore)**: for paraphrase-tolerant outputs
   - **LLM-as-judge (G-Eval)**: for subjective quality, complex reasoning, custom criteria
   - **Human evaluation**: for subjective, high-stakes, or novel task types
   - **Rule-based checks**: for format compliance, required fields, length constraints
   - **Reference comparison**: for outputs where ground truth exists
   Document why each method was selected and its limitations.

4. **Define scoring rubrics**
   For each LLM-as-judge dimension:
   - Score range (1-5 or pass/fail)
   - Anchor examples for each score level (concrete, not vague)
   - Judge prompt template (temperature=0, CoT before score)
   - Calibration requirement: 3-5 examples spanning full score range

5. **Design the evaluation pipeline**
   - **Trigger**: when does eval run? (every deploy, weekly sample, on regression alert)
   - **Sample strategy**: how many examples? (minimum 10 for reliability; 50+ for production)
   - **Automation**: which metrics can run automatically vs. require human review
   - **Baseline**: what is the current performance benchmark?
   - **Alert threshold**: at what score drop does an alert fire?

6. **Define the feedback loop**
   - How do eval results surface to the team?
   - Which failure modes feed back into prompt iteration?
   - Which failure modes indicate architecture changes are needed?
   - How is the eval dataset updated as the system evolves?

7. **Write framework spec to file**
   - Output path: provided in brief, or default to `agent-outputs/eval-framework-[date].md`

## Output Format

```markdown
# Eval Framework: [System Name]
**Date:** [Date]
**System type:** [RAG / Agent / Chatbot / Prompt pipeline / Classifier]
**Task:** [What the system does]

## Evaluation Dimensions

| Dimension | Weight | Method | Rationale |
|-----------|--------|--------|-----------|
| [Dim 1] | [%] | [Method] | [Why] |
| [Dim 2] | [%] | [Method] | [Why] |
| [Dim 3] | [%] | [Method] | [Why] |

**Composite score:** Weighted average of above dimensions

## Rubrics

### [Dimension]: [LLM-as-judge dimensions only]
**Judge prompt:**
```
[Complete judge prompt — ready to use]
```
**Score anchors:**
| Score | Description | Example |
|-------|-------------|---------|
| 5 | [Concrete] | [Example] |
| 3 | [Concrete] | [Example] |
| 1 | [Concrete] | [Example] |

## Measurement Pipeline

| Step | Method | Automation | Frequency |
|------|--------|-----------|-----------|
| Sample selection | [Stratified / Random / Triggered] | [Auto/Manual] | [When] |
| Automated metrics | [List] | Auto | [When] |
| LLM judge | [List] | Auto | [When] |
| Human review | [Spot check N%] | Manual | [When] |
| Report | [Format] | Auto | [When] |

## Baselines and Thresholds
| Dimension | Current baseline | Alert threshold | Target |
|-----------|-----------------|----------------|--------|
| [Dim 1] | [Score] | [Score] | [Score] |

## Feedback Loop
- **Prompt iteration trigger:** [Condition → Action]
- **Architecture review trigger:** [Condition → Action]
- **Dataset update protocol:** [When and how new examples are added]

## Known Limitations
[What this framework won't catch — blind spots, metric limitations, coverage gaps]
```

## Edge Cases

- If no ground truth exists: design an LLM-as-judge framework; note that human calibration of the judge is required before trusting scores
- If the system is too new to have baselines: define the framework first, run one baseline pass, then set thresholds
- If the team has no eval tooling: recommend starting with a simple spreadsheet + LLM judge before investing in DeepEval/RAGAS
- If multiple user personas have different success criteria: design per-persona dimensions and weight them separately
- If the system is in production: prioritize lightweight automated metrics first; reserve human evaluation for spot checks and regressions

=== END SKILL.md (design-eval-framework) ===

---

<!-- FILE: ~/.claude/skills/develop-strategy/SKILL.md -->
=== BEGIN SKILL.md (develop-strategy) ===
---
name: develop-strategy
description: >-
  Develops strategic plans, business analyses, and framework-based recommendations.
  Use when the user says "develop strategy for", "analyze this business problem",
  "create a plan for", "strategic options for", "framework analysis of",
  "what should we do about", or any request requiring structured strategic thinking
  and a recommendation.
allowed-tools: Read, Write, WebSearch
context: fork
agent: strategic-design
---

# Develop Strategy

## Your Task
Analyze the strategic problem and produce a structured strategy document with
a clear recommendation. Write output to a file. Return file path + summary.

## Steps

1. **Define the problem or opportunity**
   - What is the specific decision, challenge, or opportunity?
   - What is the context? (company size, market, constraints, timeline)
   - What does success look like? (measurable outcome)
   - What is explicitly out of scope?

2. **Gather relevant context**
   - Research any facts needed to ground the analysis (market data, precedents, benchmarks)
   - Identify the key stakeholders and their interests
   - Note any constraints: budget, timeline, resources, political

3. **Select and apply a framework**
   - Choose the most relevant framework for this problem type:
     - Competitive positioning: Porter's Five Forces, Blue Ocean
     - Internal assessment: SWOT, McKinsey 7S
     - Go-to-market: ICP + PQS, Jobs-to-be-Done, Crossing the Chasm
     - Decision-making: Decision matrix, scenario planning, pre-mortem
   - Apply rigorously — don't just list framework categories, fill them with real analysis

4. **Generate options (minimum 2)**
   - Never recommend a single path without alternatives
   - For each option: description, pros, cons, resource requirements, risk level
   - Options should be meaningfully different — not slight variations

5. **Recommend with rationale**
   - State the recommended option clearly
   - Explain why it fits better than alternatives given the constraints
   - Identify the top 3 risks and mitigations
   - Define next steps with owners and timeframes

## Output Format

```markdown
# Strategy: [Topic]
**Date:** [Date]
**Decision owner:** [Who decides]

## Situation
[2-3 sentences: what's happening, why it matters, what must be decided]

## Analysis
[Framework applied + findings — use the framework's structure]

## Options

### Option 1: [Name]
**Description:** ...
**Pros:** ...
**Cons:** ...
**Resources:** ...
**Risk:** High / Medium / Low

### Option 2: [Name]
...

## Recommendation
**Recommended:** Option [X]
**Rationale:** [Why this beats alternatives given the constraints]

## Top Risks + Mitigations
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|

## Next Steps
| Action | Owner | By when |
|--------|-------|---------|
```

## Edge Cases

- If the user's problem is too vague to analyze: ask for the specific decision to be made before proceeding
- If critical data is unavailable: note assumptions explicitly, flag confidence as Low
- If the problem requires domain expertise the agent lacks: state the limitation, recommend external input
- Never recommend only one option — always generate at least two alternatives

=== END SKILL.md (develop-strategy) ===

---

<!-- FILE: ~/.claude/skills/engineer-prompts/SKILL.md -->
=== BEGIN SKILL.md (engineer-prompts) ===
---
name: engineer-prompts
description: >-
  Designs, optimizes, or debugs prompts and system prompts for production use.
  Use when the user says "create a prompt for", "write a prompt that", "optimize
  this prompt", "debug this prompt", "design a system prompt", "improve this
  prompt", or any request to produce or refine an AI prompt.
allowed-tools: Read, Write
context: fork
agent: prompt-architect
---

# Engineer Prompts

## Your Task
Design, optimize, or debug the requested prompt using the 5-component framework.
Return the completed prompt with component annotations and a brief rationale.
For long prompts, write to a file and return the file path + summary.

## Steps

1. **Identify task type and target model**
   - What is the model being asked to do? (classify, extract, generate, reason, evaluate, converse)
   - Which model will run this prompt? (Claude, GPT, reasoning model)
   - What are the constraints? (token budget, latency, output format requirements)

2. **Apply the 5-component framework**
   - **Role**: Define who the model is. Be specific — expertise level, mindset, domain.
   - **Instructions**: Numbered steps or clear directives. Use CoT markers where reasoning matters.
   - **Input/Context**: What information is injected at runtime. Use XML tags to separate sections.
   - **Output format**: Exact schema — field names, types, required vs. optional, error states.
   - **Examples**: At least 1 input-output demonstration. More for complex or ambiguous tasks.

3. **Apply model-specific optimizations**
   - *Claude*: Use XML tags (`<context>`, `<task>`, `<output>`). Explicit "think step by step" if CoT needed.
   - *GPT / reasoning models*: System/user message split matters. Keep system prompt focused.
   - *All models*: Concrete over abstract. Testable assertions over vague quality goals.

4. **Define output format precisely**
   - If JSON: include schema with field types, required fields, example
   - If markdown: specify structure, headers, table format
   - Include explicit error states: what to return when input is missing, ambiguous, or out of scope

5. **Draft test cases**
   - Write 2-3 test inputs spanning the expected input range
   - Include at least 1 edge case (empty input, ambiguous input, out-of-scope input)
   - Describe the expected output for each (not necessary to run them)

6. **Deliver with rationale**
   - Annotate key decisions (why this role definition, why this output format)
   - Note any tradeoffs made (brevity vs. completeness, prescriptiveness vs. flexibility)

## Output Format

```
## Prompt: [Name]

**Target model:** [Model]
**Task type:** [Classify / Extract / Generate / Reason / Evaluate / Converse]
**Token estimate:** [Rough count]

---

[COMPLETE PROMPT — ready to copy-paste]

---

## Component Annotations

**Role:** [Why this definition]
**Instructions:** [Key decisions]
**Output format:** [Schema rationale]

## Test Cases

| Input | Expected output |
|-------|----------------|
| [Case 1] | [Expected] |
| [Edge case] | [Expected] |

## Known Limitations
[What this prompt won't handle well]
```

## Edge Cases

- If the user provides an existing prompt to optimize: read it first, diagnose failure mode before rewriting
- If no target model is specified: default to Claude Sonnet, note the assumption
- If the output format is not specified: propose JSON and explain why, let user override
- If the task is too complex for one prompt: recommend a chain and spec each component

=== END SKILL.md (engineer-prompts) ===

---

<!-- FILE: ~/.claude/skills/evaluate-ai-output/SKILL.md -->
=== BEGIN SKILL.md (evaluate-ai-output) ===
---
name: evaluate-ai-output
description: >-
  Evaluates LLM outputs, designs evaluation rubrics, and measures prompt quality.
  Use when the user says "evaluate this prompt", "assess output quality", "design
  an eval", "build a rubric for", "measure LLM performance", "how good is this
  output", or any request to systematically assess AI-generated content.
allowed-tools: Read, Write
context: fork
agent: ai-evaluator
---

# Evaluate AI Output

## Your Task
Design and run a systematic evaluation of the LLM output or prompt.
Write the evaluation report to a file. Return file path + summary.

## Steps

1. **Classify the query type (L1-L4)**
   Before selecting metrics, determine what type of task this is:

   | Level | Name | What it tests |
   |-------|------|---------------|
   | L1 | Explicit Facts | Direct retrieval — answer is verbatim in source |
   | L2 | Implicit Facts | Multi-hop reasoning — combining facts across sources |
   | L3 | Interpretable Rationales | Following documented reasoning (guidelines, workflows) |
   | L4 | Hidden Rationales | Inferring undocumented reasoning from patterns |

2. **Select metrics (max 5)**
   Choose 2-3 level-specific + 1-2 operational:

   | Level | Primary metrics |
   |-------|----------------|
   | L1 | Exact Match, F1, Retrieval Precision/Recall, Semantic Similarity |
   | L2 | Multi-hop Accuracy, Reasoning Chain Correctness, Supporting Facts F1 |
   | L3 | Workflow Adherence, Decision Tree Compliance, Protocol Fidelity |
   | L4 | Expert Alignment, Analogical Reasoning Quality, Pattern Recognition Accuracy |
   | Operational | Latency, Cost, Token count |

3. **Design rubric**
   - For each metric: define score levels (1-5 or pass/fail)
   - Anchor every level with a concrete example — no vague descriptors
   - For LLM-as-judge: specify temperature=0, chain-of-thought before score
   - Calibration requirement: 3-5 anchor examples spanning full score range

4. **Run evaluation**
   - Apply rubric to the provided output(s)
   - Score each metric independently
   - Show reasoning before score (chain-of-thought)
   - Note any scoring uncertainty

5. **Write evaluation report**
   - Include L1-L4 classification rationale, metrics selected, rubric, scores, findings
   - Flag any metric with inter-rater reliability concern
   - Recommend specific improvements based on lowest-scoring metrics

## Output Format

```markdown
# Evaluation Report: [Subject]
**Date:** [Date]
**Query Level:** L[1/2/3/4] — [Name]
**Overall Score:** [X/5 or Pass/Fail]

## Classification Rationale
[Why this is L1/L2/L3/L4]

## Metrics Selected
1. [Metric 1] — [Why chosen]
2. [Metric 2] — [Why chosen]
3. [Metric 3] — [Why chosen]

## Rubric

### [Metric 1]
| Score | Description | Anchor Example |
|-------|-------------|----------------|
| 5 | [Concrete description] | [Example] |
| 3 | [Concrete description] | [Example] |
| 1 | [Concrete description] | [Example] |

## Evaluation Results

### [Metric 1]
**Score:** [X/5]
**Reasoning:** [CoT before score]

## Summary
**Strengths:** ...
**Weaknesses:** ...
**Top recommendation:** [Most impactful improvement]

## Confidence
**Evaluation confidence:** High / Moderate / Low
**Note:** [Any reliability concerns]
```

## Edge Cases

- If no ground truth is available: use LLM-as-judge with explicit rubric; note limitation
- If the task type is ambiguous (could be L2 or L3): evaluate at both levels, report both
- If sample size is 1: note that single-sample evaluations are unreliable; recommend at least 10 samples
- If the output is very long: sample representative sections, note sampling approach

=== END SKILL.md (evaluate-ai-output) ===

---

<!-- FILE: ~/.claude/skills/produce-deliverable/SKILL.md -->
=== BEGIN SKILL.md (produce-deliverable) ===
---
name: produce-deliverable
description: >-
  Produces a high-stakes deliverable (slide deck, report, spec, eval set) by orchestrating
  specialist agents through a research → draft → verify → build pipeline, gating on an independent
  fact-verification pass before the artifact is built.
  Use when the user says "build a deck/report/spec from this", "turn this repo/project into a
  presentation", "produce a polished deliverable where the numbers have to be right", or any request
  to create a fact-heavy artifact for an external or high-scrutiny audience.
  Do NOT use when: the task is a single-pass write with no facts to verify (use the content-strategist
  directly), or when designing the agent system itself rather than running it (use design-agent-system).
allowed-tools: Agent, Read, Write, Edit
context: inline
agent: content-strategist
---

# Produce Deliverable

## Your Task
Orchestrate a fact-safe production pipeline that turns source material into a finished deliverable.
Separate *finding the truth* from *making it pretty*: gather source-traced facts first, draft from
those facts only, verify every claim against the original source independently, then build the
artifact last from verified facts. **Never let the build stage decide what is true.**

Run the four stages as separate agents in isolated contexts. Pass file paths between stages — never
raw content. Return the final artifact path plus an explicit list of what still needs a human eye.

## When to reach for this vs. a single agent
Use the full pipeline when the deliverable is **fact-heavy and high-scrutiny** — numbers in front of
managers, external audiences, anything expensive to walk back. For a low-stakes or fact-light artifact,
skip to a single content-strategist pass; the pipeline's overhead only pays off when a wrong figure is
costly.

## Steps

1. **Research → a source-traced dossier (not the artifact)**
   - Spawn a research agent (`research-analyst`, or `content-strategist` for lighter work).
   - Brief: produce a dossier where every fact and number is traced to a `file:line` or source URL,
     caveats and ambiguities flagged, nothing polished.
   - Output: `agent-outputs/[project]-dossier.md`.
   - Gate: any claim the dossier cannot source is a known gap — it does not enter the draft as fact.

2. **Draft → copy and structure**
   - Spawn the drafting agent (`content-strategist`).
   - Its ONLY source of truth is the dossier — it may shape and structure, never invent.
   - Match the target template / house style. Put depth in speaker notes or appendices; keep the
     on-slide / on-page text tight.
   - Output: `[project]/[deliverable]-draft.md`.

3. **Verify → an independent fact audit (this is the gate)**
   - Spawn an evaluation agent (`ai-evaluator`).
   - Instruct it explicitly: **do NOT trust the dossier or the draft — verify every quantitative and
     factual claim against the ORIGINAL primary sources directly.** A different source of truth is
     what catches errors propagated from the dossier into the draft.
   - Output: a claim-by-claim table (claim → `source:line` → MATCH / MISMATCH / UNVERIFIABLE), a
     corrections list, and a bottom-line pass/fail.
   - Gate: MISMATCHes must be corrected before build. UNVERIFIABLE claims are flagged for human
     review, not silently shipped.

4. **Build → the artifact, from verified facts**
   - Spawn the build agent (`senior-engineer` for HTML/code artifacts; `content-strategist` or
     `document-writer` for prose).
   - Fold the verified corrections into the build brief so it builds from clean data.
   - Invoke the relevant craft skill (e.g. `dataviz` for charts; the Slide Decks knowledge for decks).
   - Output: the final artifact, and apply the corrections back to the draft source so the two stay
     in sync.

## Briefing rules (apply to every stage)
- Identity + skill activation, tight relevant context (not everything), one task, an exact output path.
- Cap the return: "file path + N-sentence summary." Substantive output goes to the file, not the reply.
- Sequential by dependency (each stage needs the last). Parallelize only genuinely independent work.

## Output Format

```
Deliverable: [final artifact path]
Pipeline:    research → draft → verify (N claims, M mismatches fixed) → build
Needs a human eye: [what couldn't be verified in-process — rendering, tools not run, out-of-scope claims]
```

## Edge Cases

- **Fact-light deliverable:** skip stages 1 and 3; a single content-strategist pass is enough — say so
  rather than running the full pipeline for show.
- **No independent source to verify against:** the verify stage cannot do its job — flag that the numbers
  rest on a single unverified source, and make that limitation explicit in the deliverable.
- **Verification finds the dossier is the error source:** correct the dossier too, not just the draft, or
  the next build re-poisons from it.
- **Build environment can't render/validate the artifact (no browser, no runtime):** hand-verify what you
  can and name the unrun checks as required human follow-up. Never imply zero remaining review.
- **Slide decks specifically:** pair with the `knowledge/Slide Decks/` reference — `slide-deck-creation.md`
  (craft) and `slide-deck-production-pipeline.md` (this pattern, worked example).

=== END SKILL.md (produce-deliverable) ===

---

<!-- FILE: ~/.claude/skills/refactor-prompt/SKILL.md -->
=== BEGIN SKILL.md (refactor-prompt) ===
---
name: refactor-prompt
description: >-
  Transforms a diagnosed or underperforming prompt into an improved version.
  Use when the user says "fix this prompt", "rewrite this prompt", "improve this prompt",
  "refactor this prompt", "this prompt isn't working", or provides a prompt and asks
  for an improved version.
  Do NOT use when: the user wants diagnosis only without changes (use audit-prompt),
  or when building a net-new prompt from scratch (use engineer-prompts).
allowed-tools: Read, Write
context: fork
agent: prompt-architect
---

# Refactor Prompt

## Your Task
Transform the provided prompt into a stronger version using the five-component framework.
Diagnose before rewriting. Annotate every change with a rationale.
Write the refactored prompt to a file. Return file path + summary of changes.

## Steps

1. **Read and diagnose first**
   - If an audit report already exists: read it; use it as the change brief
   - If no audit exists: run a rapid internal audit (score each component 1-5 silently)
   - Identify the top 3 issues to fix before touching the prompt

2. **Preserve intent, fix structure**
   - What is the prompt trying to accomplish? Lock this in before refactoring.
   - Do not change the task or goal — only improve how it is expressed
   - Flag if the original intent seems wrong (and note it, but still refactor as requested)

3. **Refactor component by component**
   - **Role**: Make it specific — domain, seniority, mindset. Remove generic labels.
   - **Instructions**: Number the steps. Remove ambiguity. Add any missing logical steps.
   - **Input/Context**: Add XML tags around injected data. Clarify what is static vs. dynamic.
   - **Output format**: Specify exact schema. Add field types, required fields, error states.
   - **Examples**: Add at least one input-output demonstration if missing. Ensure it matches the output format exactly.

4. **Apply model-specific optimizations**
   - *Claude*: XML tags, explicit CoT markers where reasoning matters, `<thinking>` for complex tasks
   - *GPT / reasoning models*: clean system/user split; concise system prompt
   - *All models*: concrete over abstract; testable assertions over vague quality goals

5. **Check for regressions**
   - Does the refactored prompt still accomplish the original task?
   - Have any capabilities from the original been accidentally removed?
   - Are the examples consistent with the new output format?

6. **Write to file**
   - Include: original prompt, refactored prompt, change log with rationale per change
   - Output path: provided in brief, or default to `agent-outputs/refactor-[date].md`

## Output Format

```markdown
# Prompt Refactor: [Name or short description]
**Date:** [Date]
**Target model:** [Model]
**Issues addressed:** [List from audit or rapid diagnosis]

---

## Original Prompt

[Verbatim original — do not edit this section]

---

## Refactored Prompt

[Complete refactored prompt — ready to copy-paste]

---

## Change Log

| Component | Change | Rationale |
|-----------|--------|-----------|
| Role | [What changed] | [Why] |
| Instructions | [What changed] | [Why] |
| Input/Context | [What changed] | [Why] |
| Output format | [What changed] | [Why] |
| Examples | [What changed] | [Why] |

## Regression Check
- Original task preserved: Yes / No — [note if No]
- Capabilities removed: None / [List if any]
- Example consistency: Pass / [Issue if fail]

## Known Limitations After Refactor
[What this prompt still won't handle well — be honest]
```

## Edge Cases

- If the user provides an audit report: read it first and use it to scope the refactor
- If the original prompt is very short and seems intentionally minimal: refactor conservatively; note what was preserved and why
- If multiple prompts are provided: refactor each independently; do not merge them
- If the prompt has hardcoded values that should be variables: flag them, add placeholders, note the assumption
- If the refactored prompt is significantly longer than the original: note the token cost increase and explain why it is justified

=== END SKILL.md (refactor-prompt) ===

---

<!-- FILE: ~/.claude/skills/score-prompt-quality/SKILL.md -->
=== BEGIN SKILL.md (score-prompt-quality) ===
---
name: score-prompt-quality
description: >-
  Applies a 5-dimension weighted rubric to a prompt and produces a numerical quality score.
  Use when the user says "score this prompt", "rate this prompt", "how good is this prompt",
  "give this prompt a quality score", "benchmark this prompt", or any request to produce
  a quantitative quality assessment of a prompt.
  Do NOT use when: the user wants diagnosis and recommendations without a score (use audit-prompt),
  or when evaluating LLM outputs rather than prompts themselves (use evaluate-ai-output).
allowed-tools: Read, Write
context: fork
agent: ai-evaluator
---

# Score Prompt Quality

## Your Task
Apply a 5-dimension weighted rubric to the provided prompt and produce a numerical score.
Show reasoning before every score (chain-of-thought). Write the scorecard to a file.
Return file path + overall score + one-sentence top finding.

## Steps

1. **Read and classify the prompt**
   - What task is this prompt designed to accomplish?
   - What model will it run on? (if known)
   - Is this a system prompt, user-turn prompt, or few-shot example set?

2. **Apply the 5-dimension rubric**
   Score each dimension 1-5. Show CoT reasoning before the score.

   **Dimension 1: Clarity (weight: 25%)**
   Is every instruction unambiguous? Could a model interpret it differently than intended?
   - 5: Every instruction has one interpretation; no ambiguity
   - 3: Some ambiguity in edge cases but core instructions are clear
   - 1: Multiple competing interpretations; likely to produce inconsistent outputs

   **Dimension 2: Completeness (weight: 25%)**
   Are all necessary components present? Role, instructions, input spec, output format, examples.
   - 5: All five components present and well-formed
   - 3: Most components present; one missing or underdeveloped
   - 1: Two or more components absent; model must fill in significant gaps

   **Dimension 3: Output Specificity (weight: 20%)**
   How precisely is the output format defined? Fields, types, required vs. optional, error states.
   - 5: Exact schema with all fields, types, and error states specified
   - 3: Format described but not schema-precise; some fields implied
   - 1: No output format specified; model decides format

   **Dimension 4: Robustness (weight: 20%)**
   Does the prompt handle edge cases, empty inputs, and off-topic inputs?
   - 5: Explicit handling for edge cases, null inputs, and out-of-scope queries
   - 3: Handles some edge cases; main happy path well-specified
   - 1: No edge case handling; likely to fail on non-standard inputs

   **Dimension 5: Token Efficiency (weight: 10%)**
   Is every word doing work? Are there redundancies, unnecessary qualifications, or padding?
   - 5: No waste; every sentence adds instructional value
   - 3: Some redundancy but not harmful to performance
   - 1: Significant padding; instructions repeated; format overhead exceeds content

3. **Calculate composite score**
   Weighted average: (Clarity × 0.25) + (Completeness × 0.25) + (Output Spec × 0.20) + (Robustness × 0.20) + (Efficiency × 0.10)

4. **Identify the top improvement lever**
   The dimension with the lowest weighted contribution = the highest-value fix.
   Name the exact change that would most improve the score.

5. **Write scorecard to file**
   - Output path: provided in brief, or default to `agent-outputs/score-[date].md`

## Output Format

```markdown
# Prompt Quality Scorecard
**Date:** [Date]
**Prompt:** [Name or first 80 chars]
**Model target:** [Model or "unspecified"]
**Prompt type:** [System / User-turn / Few-shot]

## Dimension Scores

| Dimension | Weight | Score (1-5) | Weighted |
|-----------|--------|-------------|---------|
| Clarity | 25% | [X] | [X × 0.25] |
| Completeness | 25% | [X] | [X × 0.25] |
| Output Specificity | 20% | [X] | [X × 0.20] |
| Robustness | 20% | [X] | [X × 0.20] |
| Token Efficiency | 10% | [X] | [X × 0.10] |
| **Composite** | 100% | — | **[Sum]** |

## Scoring Rationale

### Clarity — [Score]/5
[CoT reasoning — what was found, why this score]

### Completeness — [Score]/5
[CoT reasoning]

### Output Specificity — [Score]/5
[CoT reasoning]

### Robustness — [Score]/5
[CoT reasoning]

### Token Efficiency — [Score]/5
[CoT reasoning]

## Top Improvement Lever
**Dimension:** [Dimension name]
**Current score:** [X/5]
**Change:** [Exact, specific change that would most improve the score]
**Expected score after change:** [X/5]
**Composite score improvement:** [+X points]

## Summary
**Composite score:** [X.X / 5.0]
**Grade:** Excellent (4.5+) / Strong (3.5-4.4) / Acceptable (2.5-3.4) / Needs work (<2.5)
**One-line verdict:** [Single sentence on overall quality and the top issue]
```

## Edge Cases

- If the prompt is very short (<30 tokens): note that brief prompts may be intentionally lean; apply the rubric but flag that brevity can be a feature not a flaw
- If no model is specified: score against general best practices; note the assumption
- If the prompt is a system prompt: weight Clarity and Completeness more heavily; system prompts have higher standards for role definition
- If the prompt scores 4.5+: return an excellent verdict; do not manufacture issues to justify the analysis
- If the user provides multiple prompts: score each independently; produce a comparison table at the end

=== END SKILL.md (score-prompt-quality) ===

---

<!-- FILE: ~/.claude/skills/synthesize-research/SKILL.md -->
=== BEGIN SKILL.md (synthesize-research) ===
---
name: synthesize-research
description: >-
  Researches a topic and synthesizes findings into a structured report.
  Use when the user says "research X", "synthesize findings on", "compile
  information about", "I need a report on", "what do we know about",
  or any request to gather and organize information from multiple sources.
allowed-tools: WebSearch, Read, Write
context: fork
agent: research-analyst
---

# Synthesize Research

## Your Task
Research the given topic and produce a structured synthesis document.
Write the output to a file — do not return raw content to the main context.
Return only a brief summary (3-5 sentences + file path) to the orchestrator.

## Steps

1. **Define scope**
   - Clarify the topic boundaries: what is in scope, what is not
   - Identify the audience and required depth (overview vs. deep dive)
   - Determine whether the request is time-sensitive (current state) or historical

2. **Source discovery**
   - Identify 3-5 source categories relevant to this topic (academic, industry, official docs, news, community)
   - Search each category. Prioritize primary sources over aggregators.
   - Collect 6-10 sources minimum. Flag any topic where fewer than 4 credible sources exist.

3. **Cross-validate**
   - For each key claim, find at least 2 independent sources
   - Flag any claims supported by only 1 source as Low confidence
   - Identify conflicts between sources — do not silently resolve them

4. **Structure synthesis**
   - Organize findings by theme, not by source
   - Assign confidence levels per section: High / Moderate / Low / Contested
   - Include a "Knowledge Gaps" section for anything not found

5. **Write output file**
   - Save to a clearly named file (e.g., `research-[topic]-[date].md`)
   - Return file path + 3-5 sentence summary to the orchestrator

## Output Format

```markdown
# Research: [Topic]
**Date:** [Date]
**Confidence:** [Overall: High / Moderate / Low]
**Sources:** [Count]

## Executive Summary
[2-3 sentences]

## [Theme 1]
**Confidence:** High / Moderate / Low / Contested
[Findings with inline citations]

## [Theme 2]
...

## Knowledge Gaps
- [Gap 1] — not found / insufficient sources
- [Gap 2]

## Sources
| # | Source | Type | Credibility |
|---|--------|------|-------------|
| 1 | [URL or citation] | Academic / Industry / Official / News | High / Moderate |
```

## Edge Cases

- If fewer than 4 credible sources found: flag in Knowledge Gaps, note confidence as Low, do not fabricate
- If the topic is rapidly evolving: note the research date prominently and recommend re-synthesis within [timeframe]
- If the user's scope is too broad for one pass: synthesize at overview level, list sub-topics for follow-up

=== END SKILL.md (synthesize-research) ===

---
