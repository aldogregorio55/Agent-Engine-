# Claude Engine — System Documentation (Work Machine)

## What This Is

A personal Claude Code engine living at `~/.claude/` — available across every project on this machine.
This folder is the source of truth, build workspace, and iteration space for the engine itself.

When starting a session here: check Build Status before doing anything. Do not re-execute completed steps.

---

## System Architecture

Three layers. Each has one job and must stay in its lane.

```
~/.claude/
├── CLAUDE.md          ← Layer 1: Orchestrator brain
│                         Loads every session globally.
│                         Contains: routing table, working norms, model selection.
│                         Does NOT contain: workflow steps, task logic, agent instructions.
│
├── agents/            ← Layer 2: Specialist identities
│                         Each agent = a lean system prompt defining WHO it is.
│                         Role, expertise, communication style, hard constraints.
│                         Does NOT contain: how-to steps (those live in skills).
│                         Frontmatter: name, model, color, skills, context: fork
│
└── skills/            ← Layer 3: Detailed playbooks
                          SKILL.md files that load on-demand when an agent is invoked.
                          Contains: numbered steps, edge cases, output format, examples.
                          Does NOT load until the agent fires — zero token cost until needed.
```

### Why three layers?

Each layer has a different load frequency and cost:
- `CLAUDE.md` — loads on every session (keep it lean, ~50-80 lines)
- Agent files — load when routed to (medium weight, identity only)
- Skill files — load only when the specific skill triggers (full playbook, no upfront cost)

This is progressive disclosure: detail only loads when it's actually needed.

---

## The Three Core Patterns

### 1. Fork Isolation
Every wired agent has `context: fork` in its frontmatter. When invoked, Claude Code spins up an isolated throwaway subcontext. The agent does all its work there — tool calls, reasoning, intermediate output — and only a structured summary returns to the main session.

**What this protects:** The main context window. Without forking, a session running 5 research + strategy tasks would accumulate 50k+ tokens of intermediate work, degrade in quality, and eventually hit limits.

**What agents cannot self-report:** An agent running in a fork experiences itself as the main thread. It cannot confirm "I am forked." Fork isolation is enforced at the runtime level by the frontmatter field, not by the agent's awareness.

### 2. Side-Channel File Persistence
Agents write substantive outputs to files on disk. The main context receives only the file path and a brief summary — not the file contents. Files cost zero tokens until explicitly read.

**The intended return format from every agent:**
```
Task complete. Output written to: /path/to/output-file.md
Summary: [3-5 sentences of key findings/decisions]
```

### 3. Progressive Disclosure
Skill files only load when their agent is invoked and the skill description matches the task. Workflow steps, edge cases, and output formats stay in SKILL.md files — never in agent prompts or CLAUDE.md. This keeps every layer lean and token-efficient by default.

---

## Orchestration Logic

### The orchestrator's role
The main Claude Code session is the orchestrator. Its job is:
- Receive intent from the user
- Route to the right agent(s)
- Launch tasks (parallel where independent, sequential where dependent)
- Receive summaries + file paths
- Surface findings to the user for judgment

The orchestrator does NOT do the work. It directs, receives, and synthesizes.

### The user's role
Output checker and verifier. The user operates at the judgment layer:
- Describes intent
- Reviews agent summaries
- Decides what's good, what needs refinement, what to discard
- Asks follow-up questions or triggers next-stage tasks

This is the shift: Claude assists the user → the user directs agents through Claude.

### Parallel task execution
Independent tasks launch simultaneously as background agents. Each runs in its own isolated context. Results notify when complete. The orchestrator accumulates summaries without blocking.

```
User: "Research X and design a workflow for Y"
          ↓
Orchestrator launches:
  ├── research-analyst [background] → researching X
  └── strategic-design [background] → designing Y
          ↓
Both complete independently
          ↓
Orchestrator receives two summaries + two file paths
          ↓
User reviews outputs
```

### Sequential task execution
Used when one task's output feeds the next. The orchestrator waits for the first to complete, passes the file path (not contents) to the next agent.

### Task briefing quality
Agent output quality is directly proportional to brief quality. A good brief includes:
- Exact task and scope
- Relevant context (not everything — just what the agent needs)
- Output file path and format
- Word/token cap on the return summary

---

## Agent → Skill Priority Recommendations

Every agent (and the main orchestrator) can call any skill in the library — the only functional binding is that every agent's `skills:` frontmatter now carries the full skill set. The table below is priority guidance: which skills each agent should reach for first, based on domain fit. It is not a restriction.

| Agent | Model | Recommended skills (priority) |
|---|---|---|
| prompt-architect | Sonnet | engineer-prompts, audit-prompt, refactor-prompt, chain-prompts |
| ai-evaluator | **Opus** | evaluate-ai-output, score-prompt-quality, build-eval-dataset, audit-prompt |
| strategic-design | Sonnet | design-automation, develop-strategy, design-agent-system, design-eval-framework, chain-prompts |
| research-analyst | Sonnet | synthesize-research, build-eval-dataset |
| business-analyst | Sonnet | develop-strategy |
| senior-engineer | Sonnet | design-agent-system |
| content-strategist | Sonnet | produce-deliverable |
| document-writer | Sonnet | — |
| project-manager | Sonnet | — |
| red-team | Sonnet | — |
| workspace-architect | Sonnet | — |

"—" = no priority preference; the agent still reaches every skill in the library.

**Next skills to build** (priority order):
1. `checkpoint-session` → general-assistant (fills the session memory/continuity gap)
2. `review-code` → senior-engineer (highest community adoption, most expensive without a playbook)
3. `implement-feature` → senior-engineer
4. `decompose-project` → project-manager
5. `craft-content` → content-strategist

---

## Known Gotchas (learned the hard way — do not relearn)

1. **Write permissions for forked subagents.** Path-scoped permissions (`Write(*.md)`, even `Write(/Users/<user>/**)`) may not resolve from an agent subprocess's working directory. Global `~/.claude/settings.json` needs bare unconditional `"Write"` and `"Edit"` entries in `permissions.allow`. Requires a Claude Code restart to take effect in agent subcontexts.
2. **When the side-channel breaks, context cost is 5-10x.** If agents can't write files, summaries flood back inline. This single failure pushed a session from ~35% to ~55% context. If agent returns start arriving inline, stop and fix permissions before running more tasks.
3. **Skills are not self-confirmed.** Agents report skill loading inconsistently. The only true test is a real task run — verify the output file lands in `agent-outputs/` with the expected format.
4. **Skill description budget is ~16,000 chars total.** Exceeding it silently drops skills from context, no warning. Keep each description under ~600 chars. Check with `/context` → Skills section if skills stop triggering.
5. **Subagents do not inherit skills.** Description-based triggering only works in the main thread. Inside an agent context, only skills listed in that agent's `skills:` frontmatter exist.

---

## Build Status (work machine)

| Step | Task | Status |
|---|---|---|
| 1 | Write `~/.claude/CLAUDE.md` (orchestrator brain: routing table, norms, model selection) | PENDING |
| 2 | Install skills from `skills-transfer-pack.md` into `~/.claude/skills/` | PENDING |
| 3 | Patch agent frontmatter: `skills:` field to the full skill library + `context: fork` | PENDING |
| 4 | Set permissions: bare `"Write"` + `"Edit"` in `~/.claude/settings.json`, restart | PENDING |
| 5 | Verify side-channel end-to-end: one real research task, file lands in `agent-outputs/` | PENDING |
| 6 | Build next skill wave (checkpoint-session, review-code) | PENDING |

---

## How to Write a Good Agent Brief

The quality of an agent's output is directly proportional to the quality of its brief. A brief is not a prompt — it is a complete operating context for an autonomous worker.

**Required elements:**

```
1. Identity activation
   "You are the [agent-name]. Use the [skill-name] skill."
   → Explicitly tells the agent which playbook to follow.

2. Context block
   Relevant background only. Not everything — just what changes
   the agent's decisions. Paste key excerpts, not entire documents.

3. Task definition
   One clear objective. What is being produced. For whom.

4. Output file path
   Exact path where the agent should write its output.
   Always in the agent-outputs/ directory or a project-specific equivalent.

5. Return format
   "Return: file path + [N]-sentence summary of [specific thing]"
   Cap the return. Uncapped returns flood the main context.
```

**What a bad brief looks like:**
> "Research AI tools and write a strategy."

**What a good brief looks like:**
> "You are the research-analyst. Use the synthesize-research skill.
> Context: [2-3 sentences of specific background]
> Task: Research [specific question]. Prioritize [source types].
> Write full synthesis to: [exact path]
> Return: file path + 4 sentences covering [specific angle]."

The difference is specificity and bounded output. Vague briefs produce vague outputs and unbounded returns.

**Trigger phrasing:** Never fire an agent with terse imperatives ("go", "do it", "start") — enterprise prompt shields pattern-match these as injection attempts. Use task-shaped phrasing: "perform your instructions", "execute the task defined in your instructions".

---

## How to Build a New Skill

Every skill follows the same structure. Use this template when adding to `~/.claude/skills/`.

**Directory:** `~/.claude/skills/[skill-name]/SKILL.md`

**Frontmatter (required fields):**
```yaml
---
name: skill-name                    ← matches directory name, verb form
description: >-
  [Trigger phrase 1], [Trigger phrase 2].   ← phrases users actually say
  Use when: [positive condition].
  Do NOT use when: [negative boundary].     ← prevents misrouting
allowed-tools: Read, Write, WebSearch       ← only what the skill needs
context: fork                               ← always
agent: agent-name                           ← recommended primary agent (non-binding)
---
```

**Body structure:**
```markdown
# Skill Name

## Your Task
[One paragraph. What is produced. Where it goes. What comes back.]

## Steps
1. [Concrete step with sub-bullets if needed]
2. ...

## Output Format
[Exact template the agent should populate]

## Edge Cases
- [If X happens: do Y]
- [Never do Z]
```

**Before writing the skill, write one eval scenario:**
> Input: [what the user would say]
> Expected routing: [which agent + skill]
> Expected output: [what a good result looks like]
> Failure mode: [what a bad result looks like]

Build the eval first, then the skill. This prevents skills that look right but don't work.

---

## Context Budget Rules

| Condition | Mode |
|---|---|
| 3+ tool calls required | Fork (mandatory) |
| Output likely >500 tokens | Fork (mandatory) |
| Multi-step chain (agent A feeds agent B) | Fork all steps |
| Single lookup, <3 tool calls | Inline acceptable |
| Clarifying question | Inline acceptable |

**Agent return cap:** 200-300 tokens maximum. If an agent needs more to communicate results, it means the output belongs in a file, not the summary.

**Main session health signals — investigate if you see:**
- Agent summaries growing longer each run
- Orchestrator asking follow-up questions inline instead of re-running with tighter briefs
- Context above 70% mid-session without a clear reason

---

## Operating Procedures

### Starting a new session
1. Open Claude Code in this directory
2. Read this file — check Build Status
3. Check `agent-outputs/` for recent files from prior sessions
4. Do not re-run completed steps

### Running a workflow
1. Describe intent to the orchestrator
2. For independent tasks: launch in parallel (`run_in_background: true`)
3. For dependent tasks: launch sequentially, pass file paths between steps
4. When agents complete: read summaries, open output files if detail is needed
5. Make decisions at the judgment layer — you are the verifier, not the executor

### When an agent fails to write a file
It means the side-channel pattern broke. The agent will return content inline.
- Accept the inline content for this run
- Write the file manually from the returned content (orchestrator can do this)
- Do not re-run the agent just for the file write — capture what came back
- Diagnose the permission issue before the next run

### Adding a new agent
1. Create `~/.claude/agents/[name].md`
2. Write frontmatter: `name`, `description`, `model`, `color`, `skills:` set to the full skill library, `context: fork`
3. Write body: identity only — role, expertise, style, hard constraints
4. Propose which existing skills it should prioritize (domain match against its role) and add its row to the routing table in `~/.claude/CLAUDE.md` — a recommendation for the user to confirm, not a restriction

### Adding a new skill
1. Create `~/.claude/skills/[skill-name]/SKILL.md` following the skill template above; set `agent:` to the recommended primary agent (non-binding)
2. Add the skill to every agent's `skills:` frontmatter — this is what grants functional reach, not the `agent:` field
3. Propose which agent(s) should feature it as a priority recommendation and update the routing table in this file and in `~/.claude/CLAUDE.md`
4. Run one real task to verify trigger and output format

---

## Portable Injection Target

The long-term goal is a ~15-line block that drops into any project's CLAUDE.md, giving that project access to the global engine without re-explaining the architecture.

**Current working draft:**

```markdown
## Claude Engine

Global agents and skills available at `~/.claude/`. Agents run in isolated
forks (context: fork) — delegate tasks, receive summary + file path only.
Do not read output files unless explicitly asked.

| Agent | Best for |
|---|---|
| research-analyst | Research, synthesis, source gathering |
| prompt-architect | Prompt design, optimization, debugging |
| strategic-design | Automation design, workflow architecture, GTM strategy |
| business-analyst | Business analysis, financial modeling, strategic options |
| ai-evaluator | Evaluating LLM outputs, designing eval rubrics |
| senior-engineer | Code, debugging, architecture, review |
| project-manager | Project planning, task decomposition, risk |
| content-strategist | Writing, editing, tone adaptation |
| document-writer | Work docs, specs, docs for a cold reader |
| red-team | Prompt injection payload design, guardrail bypass validation, adversarial safety testing |
| workspace-architect | Directory audits, file location, project scaffolding, workspace hygiene |

Output files go to: [project]/agent-outputs/
```

Refine this block with each project. Remove what's noise. Add what's missing.
Every iteration should be shorter than the last without losing routing fidelity.
