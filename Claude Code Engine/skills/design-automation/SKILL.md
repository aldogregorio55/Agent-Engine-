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
