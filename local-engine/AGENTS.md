# Local Engine

A personal open-weight LLM system: a backup to Claude Code, a way to learn how local models work, and freedom to use any model provider.

**Read first:** `README.md`, the main project file (status, decisions, phases, stack). Use its Quick Start table to decide what else to load.

## Current State

- **Phase:** 1, Runtime (Phase 0 complete)
- **Stack:** Ollama (runtime) + OpenCode (harness) + promptfoo (eval)
- **Models:** qwen3.5:9b (primary), qwen3:14b, deepseek-r1:8b

Update this section and the README status header whenever the phase changes.

## Rules

1. **All agent outputs go to `agent-outputs/`.** That folder is tracked in git (an exception in the root `.gitignore`).
2. **Canon agents and skills live in `../Claude Code Engine/`.** Read them there. Never copy them into this folder. If a tool needs another format, generate it from the canon files.
3. **Adopt before build.** Check for an existing open-source tool before writing any code.
4. **One API contract:** the OpenAI-compatible `/v1/chat/completions` API. Nothing Ollama-specific in glue code.
5. **Build first, then eval.** Phases 0–2 build. Phase 3 is the eval. From Phase 4 on, don't claim a phase is done without an eval result.
6. **Tools are documented, not stored.** Any install, upgrade or model pull updates `Brewfile`, `package.json` or `STACK.md` in the same change.
7. **Document everything.** Notes go in `notes/phase-N-*.md` (format: `templates/phase-note-template.md`), written as the work happens, one dated entry per session. They also serve as session notes. Decisions also go in `README.md`.
8. **Structure is fixed.** Put files in the folder the README's Folder Structure names. Superseded files move to `archive/`; never delete them.
9. **Keep this file thin.** OpenCode loads it into every local agent call, and a local model's context is small. Detail belongs in `README.md`.

## Environment

- Apple M2, 16GB unified memory. Models up to about 14B at Q4.
- Ollama not installed yet as of 2026-10-22.
