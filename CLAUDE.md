# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project Overview

Prompt Engineering Engine — the main hub for prompt-engineering work. **Primary output is prompts** (chat, agent, skill, automation). Everything else — agents, skills, knowledge modules, certification notes, orchestrator rules — exists to make those prompts sharper, more consistent, and more reusable.

This is a markdown-only repo. No build, no tests, no runtime.

## Canonical Reference

[prompting/prompt-engineering-master-context.md](prompting/prompt-engineering-master-context.md) is the house standard for prompt structure and writing. When form, structure, tone, or technique is in question, it wins. Read it before any prompt-authoring task.

## Repository Structure

| Path | Purpose |
|---|---|
| `work-engine-orchestrator.md` | Copilot session baseline: session memory, agent directory, data-security rules |
| `prompting/` | Master context (house standard), skill-authoring reference and dissection |
| `Claude Code Engine/` | Source of truth for Claude Code agents (`agents/`), skills (`skills/`), and the global `CLAUDE.md` master |
| `.github/` | Copilot custom instructions and `.agent.md` mirrors of the agent roster |
| `firm-skills-library/` | Skills written as work deliverables for colleagues — versioned (`v0/`…`vN/`), not part of the live roster |
| `knowledge/` | Reusable frameworks and playbooks (professional, technical, github) |
| `certifications/` | Certification study material and course notes |
| `session-notes/` | Session index and note template |
| `planning/` | Parked/future designs (e.g. Path B API-native automation) |
| `engagements/` | Per-engagement workspaces — gitignored except READMEs |
| `outputs/` | Produced deliverables — gitignored except READMEs |
| `agent-outputs/` | Transient agent working files — fully gitignored |
| `demo/` | Sample outputs |

## Operating Modes

- **Path A (active):** GitHub Copilot in VS Code, manual context loading via `#file:` — orchestrator + one agent module + needed knowledge files.
- **Path B (parked):** API-native orchestration. Design lives in [planning/path-b-api-agent-future.md](planning/path-b-api-agent-future.md). Do not build without explicit go-ahead.

## Prompt Conventions

- Identify the target model (standard vs. reasoning) before writing a prompt.
- House XML template: `<role>`, `<instructions>`, `<input>`, `<example>`.
- Process steps are directives, not explanations.
- Constraints as a standalone "do NOT" list.
- Variables use `{{UPPERCASE_PLACEHOLDER}}`.
- Cut every section that can be cut. Token efficiency always counts.
- Exacting language — no loose verbs, relative terms, or interpretable phrasing.

## Editing Rules

- **Agents and skills:** edit in `Claude Code Engine/`. These are copied (not symlinked) to `~/.claude/agents/` and `~/.claude/skills/` and mirrored in `.github/agents/` — flag the sync when changing them.
- **`Claude Code Engine/CLAUDE.md`** is the global master, not this file. It carries routing/norms only — no persona or personal context.
- **`firm-skills-library/`:** never copy into the live skill roster without an explicit decision. Add a new version folder rather than overwriting a prior one.
- **Session notes:** use `session-notes/session-note-template.md`; follow the naming convention in `session-notes/session-index.md` and add an entry to the index.
- **Orchestrator session memory:** update `work-engine-orchestrator.md` § Session Memory when material state changes. Keep it a state tracker, not a log.

## Data Security — Hard Rules

1. Engagement data stays inside its `engagements/[name]/` directory.
2. No cross-engagement contamination.
3. No external sharing of engagement content — no uploads, pastes, or sends to outside services.
4. Never commit anything from `engagements/` or `outputs/` beyond their READMEs. Do not weaken `.gitignore` rules.
5. Flag content that looks pasted from outside the workspace without sanitization.

## Output Locations

| Output | Path |
|---|---|
| Agent working files | `agent-outputs/` |
| General long-form outputs | `outputs/[topic]-[YYYY-MM-DD].md` |
| Engagement deliverables | `engagements/[name]/outputs/` |
| Eval records | `engagements/[name]/eval/` |
