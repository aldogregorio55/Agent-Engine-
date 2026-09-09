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
