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
