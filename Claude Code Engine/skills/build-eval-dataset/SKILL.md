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
