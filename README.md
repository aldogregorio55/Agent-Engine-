# Agent Engine

**Purpose.** This repo is a **general home base for my AI work**: the agent and skill libraries, prompting standards, knowledge modules, certification study, projects, and per-engagement workspaces.

**Tool-agnostic.** The system is plain markdown, written so any LLM can read and use it, mostly coding agents such as GitHub Copilot and Claude Code.

**Prompting reference: [prompting/prompt-engineering-master-context.md](prompting/prompt-engineering-master-context.md).** The house standard for how prompts are structured and written. For prompt work, it wins on form, structure, tone and technique.

## Projects

| Project | Status | What it is |
|---|---|---|
| [local-engine/](local-engine/) | Phase 0 complete, next Phase 1 | A personal open-weight LLM system: a backup to Claude Code, a way to learn how local models work, and freedom to use any model provider. Ollama + OpenCode + promptfoo. Start at [local-engine/README.md](local-engine/README.md). |

## What's Here

| Path | Purpose |
|---|---|
| [work-engine-orchestrator.md](work-engine-orchestrator.md) | Session baseline behaviour, agent directory, data security rules. **Load this first.** |
| [prompting/](prompting/) | Prompt-engineering craft. Contains the master context (house standard) plus dissected reference prompts and skill-creation guides. |
| [prompting/prompt-engineering-master-context.md](prompting/prompt-engineering-master-context.md) | **Canonical reference** for how we structure and write prompts. |
| [prompting/skill-creator.md](prompting/skill-creator.md) / [skill-creator-dissected.md](prompting/skill-creator-dissected.md) | Skill-authoring reference and annotated breakdown. |
| [planning/path-b-api-agent-future.md](planning/path-b-api-agent-future.md) | Parked design for API-native automation. |
| [session-notes/](session-notes/) — [session-index.md](session-notes/session-index.md) / [session-note-template.md](session-notes/session-note-template.md) | Session tracking. |
| [Claude Code Engine/agents/](Claude%20Code%20Engine/agents/) | Agent context modules (research, prompt-architect, senior-engineer, etc.). Load with `#file:` in Copilot; also the source-of-truth for Claude Code agents installed to `~/.claude/agents/`. |
| [Claude Code Engine/skills-transfer-pack.md](Claude%20Code%20Engine/skills-transfer-pack.md) | Single-file installer bundle of 12 Claude Code SKILL.md files with agent-wiring table. |
| [Claude Code Engine/memory-system.md](Claude%20Code%20Engine/memory-system.md) / [claude-engine-transfer.md](Claude%20Code%20Engine/claude-engine-transfer.md) | Memory-system design and portable transfer notes for the Claude Code engine. |
| [knowledge/](knowledge/) | Reusable frameworks and playbooks (professional skills, technical skills, github). |
| [certifications/](certifications/) | Certification study material and course notes (e.g. Claude Architect Foundations, AI Fluency). |
| [.github/](.github/) | Copilot custom instructions, prompts, skills. |
| [engagements/](engagements/) | Per-engagement workspaces (isolated, gitignored). |
| [outputs/](outputs/) | Produced deliverables (gitignored). |
| [demo/](demo/) | Sample outputs. |

## How to Use

1. Open a new chat in any coding agent (GitHub Copilot, Claude Code, etc.).
2. Load [work-engine-orchestrator.md](work-engine-orchestrator.md) + the relevant [Claude Code Engine/agents/](Claude%20Code%20Engine/agents/) module + any needed [knowledge/](knowledge/) files.
3. If the task is prompt authoring, also load [prompting/prompt-engineering-master-context.md](prompting/prompt-engineering-master-context.md) as the structural reference.
4. State the task — loaded context sets the operating mode.

Do not carry context across agent modes. Start fresh per task.

## Data Security

All engagement data stays within its engagement directory. No cross-engagement contamination. No external sharing. See the orchestrator file for full rules.

[engagements/](engagements/) and [outputs/](outputs/) contents are gitignored by default — only their READMEs are tracked. This prevents accidental push of client-sensitive material.

## Status

Path A active. Agent modules built. Knowledge base seeded. Master context relocated to [prompting/](prompting/). Certification study underway in [certifications/](certifications/). No live engagements yet.
