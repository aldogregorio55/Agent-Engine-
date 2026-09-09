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
