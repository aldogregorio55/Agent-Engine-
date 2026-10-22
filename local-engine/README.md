# Local Engine

**Owner:** Aldo Gregorio
**Started:** 2026-10-22
**Last Updated:** 2026-10-22
**Status:** Phase 0 complete (plan canonized, stack chosen). Next: Phase 1, Runtime.
**Machine:** Apple M2, 16GB unified memory (models up to about 14B at Q4)

A personal open-weight LLM system: a backup to Claude Code, a way to learn how local models work, and freedom to use any model provider. Ollama (runtime) + OpenCode (harness) + promptfoo (eval).

This is the main project file: status, decisions and reasons. Rules for AI sessions live in `AGENTS.md`.

---

## Quick Start: What to Load

| Task | Load these |
|---|---|
| **Any cold session** | `AGENTS.md` (auto-loaded), then this file |
| **Phase 1: Runtime** | This file (Stack, Tool Documentation, Known Gotchas), `notes/phase-1-runtime.md`, `STACK.md` once it exists |
| **Phase 2: Harness** | This file (Stack, Known Gotchas), `agent-outputs/local-engine-oss-scan.md` §1–2, `../Claude Code Engine/agents/` and `skills/` |
| **Phase 3: Eval** | This file (Eval Design), `notes/phase-3-eval.md`, `agent-outputs/local-engine-oss-scan.md` §3 |
| **Learning: how local models work** | `docs/concepts.md` |
| **Writing phase notes** | `templates/phase-note-template.md`, then the matching file in `notes/` |
| **Why a decision was made** | This file (Session Context → Decisions made) |
| **Tool landscape and evidence** | `agent-outputs/local-engine-oss-scan.md` |

---

## Purpose

1. **Learn** how local open-weight models work, from the runtime up.
2. **Independence.** Have a working backup if Claude Code becomes unavailable. The system must outlast both the vendor and this machine.
3. **Optionality.** Never depend on a vendor, but be free to use any of them. The same harness, agents and eval run on a local model or on any cloud API.

**Not the purpose:** matching Claude Code, or building tools from scratch. Claude stays primary. The aim is to integrate existing open source, not write new tools.

---

## Principles

| # | Principle | What it means in practice |
|---|---|---|
| 1 | **Adopt before build** | Use maintained open-source tools. Write code only to connect them. |
| 2 | **Plain text is the system** | Agents, skills, eval sets and outputs are markdown or YAML in git. |
| 3 | **One API contract** | Only the OpenAI-compatible `/v1/chat/completions` API. Switching backend or provider is a config change. |
| 4 | **Models are config** | New hardware or a new provider means a new model entry, then rerun the eval. |
| 5 | **Single source** | Read agents and skills from the repo master (`Claude Code Engine/`). If a tool needs another format, generate it from the master. Never hand-edit a copy. |
| 6 | **Build first, then eval** | Phases 0–2 build. Phase 3 builds the eval and runs the baseline. After that, nothing counts as better until the eval says so. |
| 7 | **The repo is the system** | Clone, install a runtime, pull a model, run. Which machine it's on doesn't matter. |

---

## Stack (chosen 2026-10-22)

| Layer | Tool | Role | Claude equivalent |
|---|---|---|---|
| Runtime (backend) | **Ollama** (MIT, built on llama.cpp) | Loads the weights, generates tokens, serves the API on `localhost:11434` | Anthropic's servers |
| Harness (frontend + agent logic) | **OpenCode** (MIT) | Terminal interface, builds the prompt from agents and `SKILL.md` files, runs the agent loop and tools, can run one agent per command (`opencode run --agent`) | Claude Code |
| Eval | **promptfoo** (MIT) | YAML-configured, side-by-side comparison of models against reference outputs | n/a |

Runners-up: Goose (harness), Inspect AI (eval). Evidence: `agent-outputs/local-engine-oss-scan.md`.

```
You ──► OpenCode (harness) ──► Ollama (localhost:11434/v1) ──► local model
```

### Tool Documentation

The tools are installed on the machine and run from the terminal, never stored in the repo. The repo documents them in three files at the `local-engine/` root:

| File | Job | Covers |
|---|---|---|
| `Brewfile` | Executable manifest: `brew bundle` installs everything in it | Ollama (and OpenCode if installed via brew) |
| `package.json` + `package-lock.json` | Executable and version-locked: `npm ci` reinstalls exact versions | promptfoo (and OpenCode if installed via npm) |
| `STACK.md` | Human-readable index: each tool's tested version, why it's there, how to install it; each model's tag plus its **digest** (the ID from `ollama list`) | Everything, including models |

Restore on a new machine: `brew bundle && npm ci`, then pull the models listed in `STACK.md`.

Limits: Homebrew installs the latest version, not the one you tested, so `STACK.md` records the tested version. Model tags can be re-pushed with new weights, so the digest shows whether the weights you have are the ones that were evaluated.

| File | Created in |
|---|---|
| `Brewfile`, `STACK.md` | Phase 1 |
| `package.json` | Phase 2 (if OpenCode is installed via npm) or Phase 3 (promptfoo) |
| Model digests in `STACK.md` | Updated on every eval rerun |

### Phase Notes

`local-engine/notes/` holds one file per phase (`phase-0-aim.md` … `phase-6-resilience.md`), built from `templates/phase-note-template.md`: a status header, dated Notes entries, a Decisions table, Learned, and Open questions. These are your notes, kept separate from agent outputs, and they also serve as session notes (no separate session index). Write them as the work happens, not afterwards.

---

## Provider Tiers (optionality)

Because of principle 3, the harness can point at any OpenAI-compatible provider. Each tier is a fallback for the one above it.

| Tier | Provider | When | Trade-off |
|---|---|---|---|
| 0 | **Claude Code** | Primary, every day | Depends on Anthropic |
| 1 | **Cloud alternative via API** (e.g. Kimi K2 through Moonshot's OpenAI-compatible API) | Anthropic unavailable | Close to full capability, and tool use likely works. Still cloud: costs money, needs the internet, data leaves the machine. |
| 2 | **Local** (Ollama + Qwen/DeepSeek) | All cloud access gone | Reduced capability, but fully yours and offline |

Same harness, same agents, same eval across Tiers 1 and 2. Adding a Tier 1 provider is a config entry, not a build. Exact Moonshot setup in OpenCode is unverified.

---

## Model Shortlist (local, Tier 2)

| Model | Role | Notes |
|---|---|---|
| **qwen3.5:9b** | Primary | Fits with room for context |
| **qwen3:14b** | Comparator | Near the 16GB ceiling once context is added |
| **deepseek-r1:8b** | Comparator | A distill: Qwen/Llama trained on R1 outputs, not DeepSeek's own model. Tool support unverified. |

Qwen and DeepSeek only. Gemma and gpt-oss are out. Confirm the tags with `ollama pull` in Phase 1.

---

## Phases

**Status:** Canonized 2026-10-22.

Phases 0–2 focus on building. Phase 3 is the dedicated eval phase. From Phase 4 on, the eval reruns as each phase's gate.

| # | Phase | Work | What you learn | Eval gate | Done when |
|---|---|---|---|---|---|
| **0** | **Aim** ✅ | Lock the plan. Scan open-source harnesses, eval tools and skill-format support. Pick a stack. | How to judge open-source tools (license, activity, fit) | n/a (build) | Plan canonized, stack chosen |
| **1** | **Runtime** | Install Ollama. Pull the shortlisted models. Use them from the CLI, `curl` and Python. Inspect the GGUF metadata. Measure the context budget for each model. Create the `Brewfile` and `STACK.md`. | The layers from the prompt to a token: runtime, weights, quantization, context, memory, the API | n/a (build) | You can explain every layer from the prompt to a token. `Brewfile` and `STACK.md` record Ollama and the model digests. |
| **2** | **Adopt harness** | Install OpenCode and connect it to Ollama. Load the canon agents and skills, keeping their **functionality as close to the original as possible**. Content stays the same; frontmatter is generated from canon. Losing tool use is acceptable. | How a harness loads agents and skills, and what a local model can and can't do with them | n/a (build) | Canon agents run end to end on a local model and write output files |
| **3** | **Eval** | Build the eval task set: 5–10 real tasks plus Claude reference outputs. Design the rubric. Set up promptfoo. Run the **baseline** on every shortlisted model. | How to define "good enough" and measure it: rubric design, reference outputs, model comparison | This phase *is* the eval | Scorecard exists. You know where it fails, with evidence. |
| **4** | **Adapt for local** | Add a local-mode override (no Claude-only tools), trimmed skills, context budgets, chains, routing. Only fix what the baseline showed. | How to adapt prompts and context for small models | Rerun the eval and compare with the baseline | Measurable improvement on the tasks that failed |
| **5** | **Hybrid** | Claude Code hands chosen tasks to the local harness as a worker. | How to split work between a cloud orchestrator and a local worker, routing by eval evidence | Local scores on the tasks you delegate | A clear list of which tasks go local |
| **6** | **Resilience** | Offline test (Wi-Fi off, no Claude). Write a one-page restore doc built on `brew bundle && npm ci` plus the models in `STACK.md`. Swap the backend (llama.cpp or Foundry Local). | Backend portability and recovery: proving the system survives losing a vendor | Full eval run offline on the swapped backend | A real task done with no vendor and no Ollama-specific code |

Gates have no numeric thresholds yet. Each gate is judged on the eval evidence.

### Next Steps (after Phase 6)

TBD. Candidate: add a Tier 1 cloud provider (e.g. Kimi) to the harness and eval.

---

## Eval Design

| Element | Decision |
|---|---|
| Task set | 5–10 tasks taken from real work, covering the agents you use most |
| Reference | Claude's output on the same task = reference answer |
| Scoring | Rubric per task. The ai-evaluator agent helps design it in Phase 3. |
| Tool | promptfoo |
| Built in | Phase 3 |
| Rerun when | Each phase gate from Phase 4 on, a new model, new hardware, a new provider, or a changed backend |

The eval set is the part that outlasts the machine. A new laptop = rerun it, and you know within an hour whether the setup holds.

---

## Folder Structure

Every folder exists from the start, with a one-line README. Files marked (P*n*) arrive in that phase.

```
local-engine/
  README.md                       ← this file: status, decisions, phases
  AGENTS.md                       ← rules for any harness (auto-loaded, keep thin)
  CLAUDE.md                       ← one line: @AGENTS.md
  STACK.md                        ← tools, tested versions, model digests            (P1)
  Brewfile                        ← brew bundle manifest                             (P1)
  package.json / package-lock.json← npm tools, version-locked                       (P2–3)
  opencode.json                   ← providers + models; must sit at the root         (P2)

  notes/                          ← phase notes, one per phase (also serve as session notes)
    phase-0-aim.md … phase-6-resilience.md

  templates/                      ← copy, never edit in place
    phase-note-template.md            ⭐ template for notes/

  docs/                           ← learning + operational docs, for you
    concepts.md                       ⭐ how local models work
    restore.md                        restore checklist                              (P6)

  agent-outputs/                  ← agent work, tracked in git
    local-engine-oss-scan.md          ⭐ Phase 0 open-source scan

  playground/                     ← Phase 1 experiments, tracked as a learning record (P1)

  .opencode/                      ← harness wiring                                   (P2)
    agents/                           GENERATED from canon, tracked, never hand-edit
    skills                            → canon skills (method: open decision 7)

  scripts/                        ← glue code                                         (P2)
    gen-agents.py                     canon frontmatter → OpenCode
    run.sh                            one agent call, or a chain

  eval/                           ← Phase 3 onward; every run frozen and kept
    promptfooconfig.yaml
    tasks/                            the 5–10 real tasks
    references/                       Claude's reference outputs
    rubrics/                          scoring rubrics
    results/                          one folder per run: phase-3-baseline/, phase-4-rerun/, …

  archive/                        ← superseded versions; move here, don't delete
```

`.opencode/agents/` has a `.gitkeep` instead of a README, because OpenCode would load a README there as an agent.

---

## Known Gotchas

1. **Context gets cut off without warning.** Ollama's default context is 4k tokens, and a long agent + skill prompt gets truncated with no error. OpenCode's docs recommend 16–32k. Set it explicitly.
2. **Memory ceiling.** The GPU can use about 10–11GB of the 16GB. A 14B Q4 model plus a long context is about the limit.
3. **Claude-only instructions.** Agent and skill text refers to the Skill tool, forking and file writes. Local models will ignore these or pretend to follow them. Fixed in Phase 4.
4. **Thinking models.** qwen3 and the R1 distills write out reasoning before answering. Decide whether to strip it.
5. **Tool calls on small models.** OpenCode's issue tracker has repeated Ollama tool-call failures, several closed without a fix. This is the main risk the scan found.
6. **Agent frontmatter doesn't map.** `model: sonnet`, `skills:` and `context: fork` aren't valid in OpenCode. A generator converts them (Phase 2). OpenCode may pass unknown keys through to the provider. Test this.
7. **OpenCode reads `~/.claude/skills` by default.** That's the stale live copy, not canon. Point it at canon explicitly.
8. **Ollama Inc. is a company.** The core is MIT. Principle 3 keeps the exit open.

---

## Open Decisions

| # | Decision | Status |
|---|---|---|
| 1 | Harness | ✅ OpenCode |
| 2 | Eval tool | ✅ promptfoo |
| 3 | Where config lives | ✅ `local-engine/`. Outputs go to `local-engine/agent-outputs/`, which is tracked in git. |
| 4 | Hybrid in scope | ✅ Kept, as Phase 5 |
| 5 | Model shortlist | ✅ qwen3.5:9b, qwen3:14b, deepseek-r1:8b |
| 6 | Eval task set (which real tasks) | Open. Phase 3, your pick. |
| 7 | How OpenCode reads canon skills (symlink `local-engine/.opencode/skills` → canon, or fix the `~/.claude` drift) | Open. Phase 2. |
| 8 | Two paths in the baseline: restricted OpenCode vs a raw API call, to measure harness overhead | Open (proposed). Phase 3. |
| 9 | Eval judge: Claude as judge, local models as the systems tested | Open (proposed). Phase 3. |
| 10 | Phase 6 gate: "run offline, grade afterwards", since a Claude judge can't run offline | Open (proposed). Before Phase 6. |
| 11 | Re-verify model tags and harness facts (they change monthly) | Open. Before Phase 2. |

---

## Session Context (2026-10-22)

Background for future sessions picking this up cold.

### How the phases evolved

| Original framing (user) | Became |
|---|---|
| 0: Research & planning | Phase 0 Aim, which now also includes the open-source scan |
| 1: Build harness to run local models (Ollama) | Phase 1 Runtime. Ollama is the runtime; the harness (OpenCode) is adopted in Phase 2. |
| 2: Rebuild agent engine for local, code as orchestrator | Split in two: Phase 2 adopts a harness and keeps the agents' functionality as close to the original as possible (losing tool use is acceptable), Phase 4 adapts them based on eval failures. Order flipped because you can't design for local failures you haven't seen yet. |
| 3: Apply the agent system to Ollama | Merged into Phase 2 |
| *(missing)* Eval | Phase 3, a dedicated eval phase after the build phases. It reruns as the gate for Phases 4–6. (It was briefly folded into every phase's gate, then restored as its own phase so building comes first.) |
| *(missing)* Recovery test | Phase 6 Resilience |

### Decisions made

| Decision | Why |
|---|---|
| Path B over Path A | Path A = Claude Code pointed at Ollama. Its system prompt and tools take about 20k tokens, and 8–14B models are slow at that size and unreliable at tool calls. Path B keeps the agent content and drops the Claude-specific harness. |
| Ollama as the runtime | MIT-licensed and built on llama.cpp (MIT). The easiest way to learn. The OpenAI-compatible API keeps the exit open. |
| Adopt, don't build | Writing a custom `orchestrator.py` was dropped. The aim is to learn to work with existing open source, not reinvent it. |
| OpenCode as the harness | Reads `SKILL.md` folders as they are (Agent Skills is an open standard), runs one agent per command without the interactive interface, MIT, very active. Its weakness is small-model tool calls. |
| promptfoo for eval | YAML config in git, built-in Ollama support, compares models side by side. |
| Code (or harness) as orchestrator, not the model | Small models can't do the routing, briefing and tool-calling loop well. The harness/code handles the flow; the model does the specialist work inside each call. |
| Qwen + DeepSeek shortlist | User preference. Gemma dropped; gpt-oss:20b (14GB) never made the plan and is over the GPU budget anyway. |
| Optionality via provider tiers | Independence doesn't mean avoiding vendors, only not depending on any of them. A cloud alternative (e.g. Kimi) is a config entry in the same harness. |
| Personal project, no firm restrictions | Chinese-origin models (Qwen, DeepSeek, Kimi) are fine. |
| `planning/path-b-api-agent-future.md` | Superseded by this project (that note assumed paid API keys). |
| README / AGENTS.md / CLAUDE.md split | README.md is the main file (the plan, promoted from `agent-outputs/`). AGENTS.md holds the rules, because any harness reads it, OpenCode included. CLAUDE.md just imports it. Kept thin because OpenCode loads it into every local agent call. |

### Concepts

The learning material (Ollama's layers, weights, the inference loop, what fits on 16GB, Claude Code features vs local equivalents, the three orchestration designs) lives in `docs/concepts.md`.
