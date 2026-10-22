# Research: Local Engine OSS Scan (Phase 0)
**Date:** 2026-10-22
**Confidence:** Moderate overall. Repo facts (stars, license, activity) are High. Small-model behavior inside any harness is Low: no controlled, independent test found.
**Sources:** 24 (see table)

Labels: **Confirmed** = read in primary source today (GitHub API, official docs, repo issues). **Reported** = secondary source or search summary. **Anecdotal** = single user report. **Unverified** = from model memory, not checked today. Each claim tagged **[P]** practitioner or **[V]** vendor/official docs. GitHub API figures are neutral facts, tagged **[API]**.

## Executive Summary
OpenCode is the best harness fit: MIT, 220k stars, shipping weekly, reads `SKILL.md` straight from `~/.claude/skills/`, and runs markdown agents headlessly with `opencode run --agent`. Agent Skills is a real open standard (agentskills.io, Apache-2.0 repo) adopted by OpenCode and Goose, among others. promptfoo is the best eval pick (YAML, native Ollama provider, MIT). The main risk is not tooling: it is that 8-14B models fail at tool calls in every harness, and the existing agent frontmatter will not map 1:1 to OpenCode.

---

## Fit with the existing system (defined before searching)
Read: `agents/research-analyst.md`, `agents/senior-engineer.md`, `skills/synthesize-research/SKILL.md`, `skills/chain-prompts/SKILL.md`. Confirmed.

| # | Fit requirement | Why it matters |
|---|---|---|
| 1 | Reads `skills/<name>/SKILL.md` with `name` + `description` frontmatter unchanged | Skills already use this. Extra fields seen: `allowed-tools`, `context`, `agent`. |
| 2 | Loads `agents/*.md`, body = system prompt | Agent frontmatter has `name, description, model: sonnet, color, skills: [15 names], context: fork`. |
| 3 | `model` is overridable per run or per agent | `model: sonnet` is meaningless locally. |
| 4 | Headless, scriptable single-agent call | Design decision: code orchestrates, model does one specialist job per call. |
| 5 | `/v1/chat/completions` via Ollama | Principle 3. |
| 6 | Works with small tool-call-weak models | The model must not need to run a long tool loop to be useful. |

Disqualifiers: archived or stale repo; no OpenAI-compatible/Ollama path; non-OSS license; skill or agent format that requires a hand-maintained copy (violates Principle 5); needs a hosted service.

---

## 1. Agent harness
**Confidence:** Moderate on tool facts, Low on small-model behavior.

| Rank | Repo | License | Activity | Ollama support | Small-model fitness | Fit with markdown agents/skills | Notes |
|---|---|---|---|---|---|---|---|
| 1 | anomalyco/opencode (was sst/opencode) | MIT | 220k stars, last push 2026-10-22, releases v1.19.1 to v1.19.4 in 8 days, 3.7k open issues [API] | Yes, via `@ai-sdk/openai-compatible`, baseURL `localhost:11434/v1` [V] | Mixed. Open tool-call failures with Ollama models (see Risks) [P] | **Skills: excellent.** Scans `.claude/skills/`, `~/.claude/skills/`, `.agents/skills/`, `.opencode/skills/` [V]. **Agents: partial.** Reads `.opencode/agents/*.md` or `~/.config/opencode/agents/`; filename = agent name; `description`, `mode`, `model` (`provider/model-id`), `permission`, `temperature`, `steps`, `color` [V]. Does not read `.claude/agents/` (not mentioned in docs). | `opencode run --agent X -m ollama/qwen3.5:9b -f file --format json` is non-interactive. `opencode serve` plus `--attach` gives a headless HTTP API [V]. Unrecognized agent options "pass through to the provider" [V], so `skills:` and `context:` could leak into requests. Unverified: test it. |
| 2 | aaif-goose/goose (was block/goose; moved to Linux Foundation AAIF) | Apache-2.0 | 55.5k stars, last push 2026-10-22, v1.52 to v1.55 in 3 weeks [API] | Yes, provider doc says full OpenAI-compatible API with tool calling; default model qwen3 [Reported, mintlify mirror]. Tool shim (`GOOSE_TOOLSHIM=true`) for models that print tools as text [Reported, bestllmfor guide] | Better small-model plumbing than OpenCode: a Qwen3-coder XML tool-call fallback was merged (PR #6882) [P/API] | **Skills: good.** Scans `~/.agents/skills/`, `.agents/skills/`, plus `.claude/skills/`, `~/.claude/skills/` [V]. **Agents:** docs do not describe a markdown agent format (recipes are separate YAML) [V]. | Goose Rust/Electron-class app; CLI exists. Weaker fit for "one agent.md per call". |
| 3 | Aider-AI/aider | Apache-2.0 | 49.7k stars, **last push 2026-05-22, last release v0.86.0 on 2025-08-09** [API] | Yes (long-standing; not re-verified today) [Unverified] | Good for edit-format on weak models historically [Unverified] | No skills or markdown-agent concept found [Unverified] | **Stale.** 5 months without a push, 14 months without a release. Drop. |
| 4 | cline/cline | Apache-2.0 | 70.7k stars, active [API] | Reported | Not assessed | Not assessed | VS Code extension, not a headless runner. Wrong shape. |
| 5 | QwenLM/qwen-code | Apache-2.0 | 28.6k stars, active [API] | Not assessed | Tuned for Qwen models (from name; not read) | Not assessed | Candidate for a follow-up look. |
| - | openai/codex, google-gemini/gemini-cli | Apache-2.0 | 130k / 108k stars, active [API] | Not assessed | Not assessed | Both listed as skills-compatible in secondary sources [Reported] | Vendor-owned. Fail the "no single vendor" goal on principle; not tested. |
| - | continuedev/continue | Apache-2.0 | 36k stars [API] | Not assessed | - | - | IDE-centric. Skipped. |

**Practitioner evidence (OpenCode + local models), all [P] from the repo's own issue tracker, Anecdotal-to-Reported:**
- #1034 "Local Ollama tool calling either not calling or failing outright" (2025-07, 32 comments, closed). Reported.
- #4428 "Why is opencode not working with local llms via Ollama?" (2025-11, 37 comments, closed). Reported.
- #20995 "Gemma 4 (e4b) tool calling fails via Ollama OpenAI-compatible API" (2026-04). Closed **not_planned** by stale-bot on 2026-09-09; a commenter on 2026-10-04 says devs were inactive on it. Confirmed (read issue and comments).
- #6918 "qwen3-coder fails to call edit tool" (invalid args: `oldString` received as object). A commenter on 2026-04-21 reports it still happens with qwen3.6-35b-a3b and says the same file edits work in Claude Code. Closed not_planned by stale-bot. Confirmed that the issues exist; cause not established.
- OpenCode docs themselves say: "If tool calls aren't working, try increasing `num_ctx`... around 16k-32k" [V]. Confirmed.

Read: OpenCode's headline strength (skills + headless agent runs + activity) is real. Its weakness for this build is that small-model tool-call bugs get auto-closed. Inference: because your design makes code the orchestrator and keeps tool use minimal, you may avoid most of these failures. Not tested.

**Agent-file mismatch (Confirmed from files vs docs):** your `model: sonnet` is invalid in OpenCode (needs `provider/model-id`); `skills:` list and `context: fork` are not documented OpenCode fields; `name` comes from the filename (matches yours). Principle 5 says generate, do not hand-edit: a ~20-line script that rewrites frontmatter into `.opencode/agents/` is the only glue needed.

---

## 2. Agent Skills / SKILL.md portability
**Confidence:** High that it is an open standard; Moderate on exact adopter list and depth of support.

- Open standard: agentskills.io states "originally developed by Anthropic, released as an open standard, and has been adopted by a growing number of agent products"; spec page, GitHub org `agentskills/agentskills` (Apache-2.0, 26.4k stars, last push 2026-08-09) [V/API]. Confirmed.
- Required: `name` and `description` only; optional `scripts/`, `references/`, `assets/`. Loading is progressive (discovery, activation, execution) [V]. Confirmed. Your skills already conform; extra keys (`allowed-tools`, `context`, `agent`) are harmless to OpenCode, which lists `license`, `compatibility`, `metadata` as optional [V]; behavior on unknown keys not tested.
- Timeline: Anthropic launched 2025-10-16, open standard 2025-12-18 [Reported, secondary blogs]. 
- Confirmed adopters from official docs: **OpenCode** (paths above), **Goose** (`~/.agents/skills/` recommended, also reads `.claude/skills/`). Reported adopters (secondary sources only): Codex, Gemini CLI, GitHub Copilot, VS Code, Cursor, JetBrains Junie, Kiro. Count claims range from 27 to 40+ [Reported].
- Neither smolagents (README has no mention) nor Pydantic AI (docs describe "on-demand capability, like an Agent Skill" and "loading skills from markdown files" but never name SKILL.md) are confirmed adopters [V].
- Key limit: in all cases the skill is only a prompt. Small models decide whether to activate a skill from its description, and the existing skills tell the model to use the Skill tool, fork, etc. (Plan gotcha 3). Confirmed from your skills (`context: fork`, "Return file path").
- Direction: the engine's skills folder can be the single source. Point OpenCode at it with a symlink or `~/.claude/skills`; no conversion.

---

## 3. Eval tooling
**Confidence:** Moderate-High.

| Rank | Repo | License | Activity | Ollama support | Small-model fitness | Fit with markdown agents/skills | Notes |
|---|---|---|---|---|---|---|---|
| 1 | promptfoo/promptfoo | MIT | 26.3k stars, release 0.124.1 on 2026-10-08, push 2026-10-22 [API] | Native: `ollama:chat:<model>`, `OLLAMA_BASE_URL`, `num_ctx`, `num_predict`. Also `openai:chat:<model>` with `apiBaseUrl` for any OpenAI-compatible server [V] | Documents local-model traps: reasoning tokens eat the `num_predict` budget; set `think: false` for JSON on qwen3/R1; unknown config keys silently dropped; `keep_alive: 0` to unload between serial runs [V] | YAML config in git: fits "plain text is the system". Side-by-side matrix of N models x M prompts, `llm-rubric` and similarity assertions, reference outputs as vars. No SKILL.md awareness (not needed). | Node/npx. Grader gotcha: `llm-rubric` graders must be pointed at a provider separately (`defaultTest.options.provider`) [V]. |
| 2 | UKGovernmentBEIS/inspect_ai | MIT | 3.1k stars (low, but UK AISI-backed), 0.3.283 on 2026-10-21, near-daily releases [API] | `ollama/<model>`, `OLLAMA_BASE_URL`, default `localhost:11434/v1`; generic `openai-api/<provider>/<model>`; `emulate_tools=true` for models without native tools [V] | Explicit tool-emulation switch for weaker models | Python tasks, not YAML; stronger for multi-step agent/tool-use evals and sandboxing | Better ceiling, higher authoring cost. Good phase 3+ addition. |
| 3 | confident-ai/deepeval | Apache-2.0 | 19k stars, push 2026-10-19 [API] | Ollama support not verified today [Unverified] | - | Pytest-style Python | Metric-rich (G-Eval etc.). Pulls toward its hosted platform [Unverified]. |
| 4 | EleutherAI/lm-evaluation-harness | MIT | 14.3k stars, last push 2026-09-14 [API] | Not verified | - | Academic benchmarks, not custom task sets | Wrong tool for "compare against Claude reference outputs". |
| - | explodinggradients/ragas | (moved to vibrantlabsai/ragas) | 16.1k stars, **last push 2026-02-24** [API] | - | - | RAG-specific | Skip: wrong domain, quiet. |
| - | openai/evals | NOASSERTION | 19.6k stars, last push 2026-04-14 [API] | - | - | - | Stale, skip. |

Eval design flag (analysis): reference = Claude output, scoring by `llm-rubric`. A local 8-14B judge is unreliable for rubric grading [Unverified, general knowledge]. Options: use Claude as judge for the baseline run only (breaks offline independence, acceptable for Phase 2), or deterministic assertions plus similarity. Decide in Phase 0 with ai-evaluator.

---

## 4. Lightweight orchestration / chaining
**Confidence:** Moderate. The harness covers most of what you need.

| Rank | Repo | License | Activity | Ollama support | Small-model fitness | Fit with markdown agents/skills | Notes |
|---|---|---|---|---|---|---|---|
| 1 | **No library: shell/Python loop over `opencode run`** (or the OpenAI SDK against `:11434/v1`) | n/a | n/a | Direct | Best: you control every prompt | Chain = stage N writes a file, stage N+1 gets the path via `-f`. Matches your `chain-prompts` handoff-contract design | Zero dependencies to maintain. Adopt-before-build satisfied because OpenCode is the adopted piece; the glue is ~30 lines. |
| 2 | pydantic/pydantic-ai | MIT | 20.8k stars, v2.59.0 on 2026-10-21 [API] | Ollama listed as provider [V] | Typed structured output is the useful part | No SKILL.md; markdown loading is described only as a capability pattern [V] | Pydantic Graph for typed control flow [V]. Pick if you want validated JSON handoffs. |
| 3 | huggingface/smolagents | Apache-2.0 | 30k stars, v1.26.0 on 2026-05-29, push 2026-10-06 [API] | `ollama`/`transformers`/LiteLLM models [V] | CodeAgent is an agent loop; claim "30% fewer steps" is the vendor's, with no model sizes given [V] | No SKILL.md | Contradicts your decision that the model does not run the loop. Not recommended. |
| - | langchain-ai/langgraph | MIT | 43.7k stars, active [API] | - | - | - | Heavy framework. Excluded per brief. |

---

## 5. Model shortlist (16GB M2)
**Confidence:** Low-Moderate. Sizes and capabilities are Confirmed from ollama.com. Quality notes are Reported or Unverified: independent agentic benchmarks for these sizes were not found; the ranking pages found are SEO-style blogs with some unconfirmable model names.

Platform facts (Confirmed, Ollama docs): default context is **4k tokens under 24GiB VRAM**; Ollama recommends **at least 64k** for agents/tools; set with `OLLAMA_CONTEXT_LENGTH`. OpenCode docs advise 16-32k. Inference: 64k KV cache on a 14B model will not fit comfortably in the ~10-11GB GPU budget stated in your plan.

| Rank | Model (Ollama tag) | Size on disk | Context | Tools / thinking | Quality notes |
|---|---|---|---|---|---|
| 1 | qwen3.5:9b | 6.6-7.6GB | 256K | tools, thinking, vision [V] | Newer than qwen3; fits with headroom for context. No independent agentic score found. Test first. |
| 2 | gemma4:12b | 7.7-8.0GB | 256K | native function calling claimed at family level; per-variant not listed [V]; thinking mode configurable | Documented OpenCode failure with **gemma4:e4b** over Ollama OpenAI-compat (#20995) [P, Confirmed]. 12b untested. |
| 3 | qwen3:14b | not read today (about 9GB by memory) [Unverified] | 40K by memory [Unverified] | tools + thinking by memory [Unverified] | Secondary guide calls it the smallest Qwen3 reliable at tool calling [Reported]. 8b disputed: one guide says chat-only, another cites 0.933 F1 [Reported, conflicting]. |
| 4 | gpt-oss:20b | 14GB | 128K | tools, configurable reasoning effort, Apache-2.0 [V] | Most-recommended 16GB model in secondary guides [Reported]. **Tight:** 14GB exceeds the ~10-11GB GPU budget in your plan, so expect CPU offload or swap (inference). Test with `ollama ps`. |
| 5 | qwen3:8b | ~5GB by memory [Unverified] | - | - | Your fallback fast model. |
| 6 | deepseek-r1:8b (distill) | not read | - | Goose issue #877 shows R1 tool calling was a problem early [P] | Plan already notes it is a Qwen/Llama distill. Whether Ollama's tag supports tools is Unverified. Use for reasoning-only steps, not agent calls. |
| - | ministral-3 (3b/8b/14b), phi4 14b, granite4.2 (3b/8b) | listed on library page | - | - | Exist; not assessed. |
| Out | qwen3.6 (27b/35b only), gemma4:26b/31b, devstral 24b | 16-22GB | - | - | Do not fit. qwen3.6:35b-a3b appears in an OpenCode issue [P]. |

---

## 6. Nice-to-have
**Confidence:** Moderate.

| Repo | License | Activity | Ollama | Notes |
|---|---|---|---|---|
| open-webui/open-webui | NOASSERTION per GitHub API [API]; license text not read. Believed to be a custom license with a branding clause [Unverified] | 156k stars, active, 205 open issues [API] | Yes, native (not re-verified today) [Unverified] | Check the license before relying on it for "any vendor" independence. |
| BerriAI/litellm | NOASSERTION per GitHub API [API]; believed MIT core plus separate enterprise directory [Unverified] | 61.8k stars, active, 5.6k open issues [API] | Yes (smolagents docs use it as an Ollama path) [V] | Only needed if a tool cannot speak `/v1`. Skip until then; every tool here already does. |

---

## Recommended stack

| Category | Pick | Reason | Runner-up |
|---|---|---|---|
| Harness | OpenCode | Reads existing `SKILL.md` folders with no conversion; headless `run --agent`; MIT; fastest release cadence; 220k stars | Goose (better small-model tool-call fallbacks, but no markdown agent format) |
| Skills portability | agentskills.io standard (no new tool) | Open spec, Apache-2.0 repo, confirmed in OpenCode and Goose docs | n/a |
| Eval | promptfoo | YAML in git, native Ollama provider, side-by-side matrix, MIT, release 2 weeks old | Inspect AI (add for agentic evals later) |
| Orchestration | Plain script over `opencode run` | Handoff = file paths; no dependency | Pydantic AI (typed handoffs) |
| Models | qwen3.5:9b primary; gemma4:12b and qwen3:14b as comparators; gpt-oss:20b as a stretch test | Fit the memory budget with room for context | deepseek-r1:8b for reasoning-only steps |
| Chat UI / proxy | Open WebUI later; skip LiteLLM | Both optional; check licenses first | n/a |

---

## Known Risks
1. **Tool calls from 8-14B models over Ollama's OpenAI-compat layer are the dominant failure mode** (OpenCode #1034, #4428, #6918, #20995; Goose #3748). Several were closed by stale-bot, not fixed. [P, Confirmed that issues exist]
2. **Silent context truncation:** Ollama default 4k under 24GiB; OpenCode's own docs say tool calls break when `num_ctx` is too low. Set explicitly in Phase 1. [V, Confirmed]
3. **Agent frontmatter does not map:** `model: sonnet`, `skills:`, `context: fork`. Pass-through of unknown options is a possible leak. Needs a generator, tested in Phase 2. [Confirmed/Inference]
4. **Judge problem** for promptfoo `llm-rubric` with a local model. [Analysis]
5. **Naming churn:** sst/opencode and block/goose both redirect to new orgs (anomalyco, aaif-goose). Pin versions, expect more renames. [API, Confirmed]
6. **OpenCode growth is partly sentiment-driven** (Jan 2026 surge tied to Anthropic blocking third-party auth) [Reported, one source]. Stars measure attention, not local-model quality.

## Knowledge Gaps
- No controlled test of any harness with qwen3.5:9b / gemma4:12b on 16GB. Needs Phase 2 evidence.
- No independent agentic benchmark for the 8-14B tier; ranking blogs are SEO-style.
- Not read today: Open WebUI and LiteLLM license text; DeepEval Ollama docs; Aider docs; qwen3:14b/8b and deepseek-r1 Ollama pages; Codex/Gemini CLI/Qwen Code local-model support.
- Whether OpenCode ignores or forwards unknown agent frontmatter keys. Test directly.
- Whether OpenCode reads `CLAUDE.md`/`AGENTS.md` automatically (config page silent; /docs/rules not read).
- No r/LocalLLaMA or Hacker News threads retrieved; search returned none. Practitioner evidence here is GitHub issues and guides only.
- Foundry Local (backend swap target) not assessed; outside this brief.
- Agent Skills adopter count: 27 vs 40+ unreconciled; client showcase page did not render.

## Time Sensitivity
This space moves monthly: OpenCode ships weekly, Ollama's library updated qwen3.5/gemma4/gpt-oss within the last week, and repo names are changing. Stars and versions are as of 2026-10-22. Re-run the model and harness rows before Phase 2 starts (target: within 4 weeks).

## Sources
| # | Source | Type | Credibility |
|---|--------|------|-------------|
| 1 | GitHub REST API: repo, releases, commits, issues for opencode, goose, aider, promptfoo, inspect_ai, deepeval, smolagents, pydantic-ai, ollama, open-webui, litellm, agentskills, cline, continue, codex, gemini-cli, qwen-code, ragas, evals, lm-eval (fetched 2026-10-22) | Primary | High |
| 2 | https://opencode.ai/docs/skills/ | Official | High |
| 3 | https://opencode.ai/docs/agents/ | Official | High |
| 4 | https://opencode.ai/docs/cli/ | Official | High |
| 5 | https://opencode.ai/docs/providers/ | Official | High |
| 6 | https://opencode.ai/docs/config/ | Official | High |
| 7 | https://agentskills.io/home | Official | High |
| 8 | https://goose-docs.ai/docs/guides/context-engineering/using-skills | Official | High |
| 9 | https://www.promptfoo.dev/docs/providers/ollama/ | Official | High |
| 10 | https://www.promptfoo.dev/docs/providers/openai/ | Official | High |
| 11 | https://inspect.aisi.org.uk/providers.html | Official | High |
| 12 | https://docs.ollama.com/context-length | Official | High |
| 13 | https://ollama.com/library (list), /library/qwen3.5, /library/gemma4, /library/gpt-oss | Official | High (sizes), n/a (quality) |
| 14 | https://github.com/huggingface/smolagents | Official | High |
| 15 | https://pydantic.dev/docs/ai/overview/ | Official | High |
| 16 | opencode issues #1034, #4428, #6918, #20995 (github.com/anomalyco/opencode) | Practitioner | Moderate (anecdotal, unreproduced) |
| 17 | goose issues #877, #3748; PR #6882 (github.com/aaif-goose/goose) | Practitioner | Moderate |
| 18 | https://www.mintlify.com/block/goose/api/providers/ollama (via search) | Secondary mirror | Moderate |
| 19 | https://bestllmfor.com/guide/goose-agent-local/ (via search) | Blog | Low-Moderate |
| 20 | https://www.dash0.com/guides/observing-goose-on-ollama-with-opentelemetry (via search) | Blog | Moderate |
| 21 | https://aiwiki.ai/wiki/opencode; https://saiyampathak.substack.com/p/opencode-just-overtook-claude-code (via search) | Secondary | Low-Moderate |
| 22 | https://www.agentpatterns.ai/standards/agent-skills-standard/; strapi.io, atlan.com blogs (via search) | Secondary | Low-Moderate |
| 23 | https://www.morphllm.com/best-ollama-models; haimaker.ai guide (via search) | SEO blog | Low |
| 24 | Local files: `Claude Code Engine/agents/{research-analyst,senior-engineer}.md`, `skills/{synthesize-research,chain-prompts}/SKILL.md`, `local-engine/agent-outputs/local-model-plan.md` (since moved to `local-engine/README.md`) | Primary | High |
