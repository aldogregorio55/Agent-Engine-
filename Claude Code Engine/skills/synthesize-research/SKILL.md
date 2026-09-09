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
