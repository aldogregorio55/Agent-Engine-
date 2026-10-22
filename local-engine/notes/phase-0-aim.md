# Phase 0: Aim

**Started:** 2026-10-22
**Status:** Complete
**Goal:** Lock the plan, scan the open-source landscape, pick a stack.

---

## Notes

Dated entries, newest at the bottom. One entry per session, so these notes double as session notes.

### 2026-10-22: Planning session (start to Phase 0 complete)

**Starting point.** Wanted to run open-weight models locally, assuming the Agent Engine was already "wired up for any model." It's only half true: the agent and skill *content* is portable markdown, but the *harness* features (`context: fork`, the Skill tool, the Agent tool, `model: sonnet`) belong to Claude Code.

**The real goal surfaced mid-session.** Not just learning: getting out of total dependence on Anthropic. The system should outlast both the vendor and this machine. That turned a learning toy into an exit plan with provider tiers.

**How the plan evolved:**
1. Started with 4 phases (research → Ollama harness → rebuild agents → apply). Reordered: the agents can't be adapted for local models before seeing how local models fail.
2. Added eval and a recovery test (both missing). Eval was briefly folded into every phase's gate, which was a silent change, caught and reversed. It's now a dedicated Phase 3, so building comes first.
3. "Adopt before build" replaced the custom `orchestrator.py`. The aim is to integrate open source, not reinvent it.
4. A research-analyst scan (`agent-outputs/local-engine-oss-scan.md`) picked the stack. I read the full report before accepting it.
5. The plan was promoted to `README.md` as the main file. Rules went into `AGENTS.md` (vendor-neutral), and `CLAUDE.md` imports it. Full folder scaffolding, Safety.md style.

**Hardware reality.** M2, 16GB unified memory. The GPU gets about 10–11GB, so the ceiling is about 14B at Q4. Kimi K2 (about 1T parameters) and full DeepSeek (671B) are out of reach locally. The `deepseek-r1:8b/14b` tags are distills (Qwen/Llama trained on R1 outputs).

**Engine-level check.** Found 3 copies of the Claude Code engine (repo master, `~/.claude/`, backup). Canon = repo master `Claude Code Engine/`. Live and backup match each other but lag behind canon. The drift: `red-team`, `safety-run-logger`, `detect-injection` and `threat-detection` aren't live, `general-assistant` is missing from canon, and 7 skills have old names in their `agent:` field.

**Commits:** `5ab0bad` (local-engine scaffold), `356ea12` (root README reframed as a general, tool-agnostic home base).

---

## Decisions

| Date | Decision | Why |
|---|---|---|
| 2026-10-22 | Path B (own stack) over Path A (Claude Code pointed at Ollama) | Claude Code's ~20k-token system prompt and tools are too heavy for 8–14B models |
| 2026-10-22 | Ollama as the runtime | MIT, built on llama.cpp, easiest to learn. The OpenAI-compatible API keeps the exit open. |
| 2026-10-22 | Adopt before build. No custom orchestrator. | Integrating open source teaches more and is less to maintain |
| 2026-10-22 | Stack: **Ollama + OpenCode + promptfoo** | OpenCode reads `SKILL.md` as is and runs one agent per command without the interactive interface. promptfoo has YAML config and built-in Ollama support. All MIT. |
| 2026-10-22 | Code/harness orchestrates, the model doesn't | Small models fail at routing and tool loops |
| 2026-10-22 | Phase 2 keeps agent functionality as close as possible; losing tool use is acceptable | Content stays the same; frontmatter is generated from canon |
| 2026-10-22 | Phases 0–6, with a dedicated eval phase (3). Build first, then eval. | Focus on building, then measure |
| 2026-10-22 | No numeric gate thresholds yet | Judge on evidence first |
| 2026-10-22 | Keep Hybrid (Phase 5). Next steps after Phase 6: TBD | |
| 2026-10-22 | Models: Qwen + DeepSeek only (qwen3.5:9b, qwen3:14b, deepseek-r1:8b) | Preference. Gemma dropped; gpt-oss:20b is over the GPU budget. |
| 2026-10-22 | Optionality via provider tiers: 0 Claude Code → 1 cloud API (e.g. Kimi) → 2 local | Independence means not *depending* on any vendor, not refusing them |
| 2026-10-22 | Personal project, no restrictions | Chinese-origin models are fine |
| 2026-10-22 | Tools installed on the machine, documented in the repo (`Brewfile`, `package.json`, `STACK.md`) | Repo = recipe, machine = kitchen. Model digests pin the weights that were evaluated. |
| 2026-10-22 | `README.md` = main file, `AGENTS.md` = rules, `CLAUDE.md` = `@AGENTS.md` | AGENTS.md is read by any harness. Kept thin because OpenCode loads it into every local call. |
| 2026-10-22 | Phase notes double as session notes; templates folder added | Documenting is key. A pattern carried over from the Safe AI Agents project. |
| 2026-10-22 | Scaffold every folder now; track `.opencode/agents/` (generated) and `playground/` | The tree matches the disk from day one; documenting is key |
| 2026-10-22 | `local-engine/agent-outputs/` tracked in git (`.gitignore` exception) | Project outputs are part of the record |
| 2026-10-22 | Commit straight to main. No branch for docs. | Solo project. Branch later for risky Phase 2 experiments. |
| 2026-10-22 | `red-team`, `safety-run-logger` (and the injection/threat skills) stay out of the live `~/.claude/` setup | Not needed live |
| 2026-10-22 | Root README reframed: a general home base, tool-agnostic, prompt-engineer framing dropped | The repo is broader than prompt engineering |

---

## Learned

- **Ollama** = the runtime/backend. A server on `localhost:11434` wrapping llama.cpp. The CLI is only a client of it.
- **OpenCode** = the harness: the frontend *and* the agent logic. It decides what to send; Ollama only answers. Same split as Claude Code ↔ Anthropic's servers.
- **Weights** = a GGUF file: a metadata header plus named tensors (matrices of numbers). Data only, nothing executable. Q4 shrinks 8B from ~16GB to ~5GB.
- **The context window costs RAM** (the KV cache). Ollama's default is 4k and **cuts off the overflow without warning.**
- **The OpenAI Chat Completions format** is the industry's shared API shape, not a dependency on OpenAI. Swapping providers = `base_url` + `api_key` + `model`.
- **CLAUDE.md** = standing orders loaded every session. The test is "does every session need this?" **AGENTS.md** = the same thing, for any harness.
- **Agent Skills (`SKILL.md`)** is an open standard, so the skill library is already portable.
- **The real risk is small-model tool calls, not tooling.** OpenCode's issue tracker shows repeated Ollama tool-call failures.

---

## Open questions

- [ ] How OpenCode reads canon skills: symlink `.opencode/skills` → canon, or fix the `~/.claude` drift? → Phase 2
- [ ] Baseline both paths, restricted OpenCode vs a raw API call, to measure harness overhead? → Phase 3
- [ ] Claude as eval judge, local models as the systems tested? → Phase 3
- [ ] Reword the Phase 6 gate to "run offline, grade afterwards"? → before Phase 6
- [ ] Re-verify model tags and harness facts → before Phase 2
- [ ] Does OpenCode also load the repo-root files above `local-engine/`? Would bloat local context. → Phase 2
- [ ] Which 5–10 real tasks go in the eval set? → Phase 3
- [ ] `work-engine-orchestrator.md` is stale (April) and still marked "Load this first" in the root README. Update, archive or leave? → engine housekeeping
