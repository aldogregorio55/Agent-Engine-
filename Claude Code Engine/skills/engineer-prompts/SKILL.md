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
