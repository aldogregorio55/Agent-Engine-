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
