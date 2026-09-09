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
