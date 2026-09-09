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
