# Path B — API-Native Agent Automation

**Status:** Parked. Build after API access is available.
**Last noted:** 2026-04-06

---

## What This Would Be

A Python-based orchestration layer that replaces the manual file-loading workflow with actual automated agent execution. Real prompt chaining. Outputs written to files. Inputs fed forward automatically. Claude Code-style behavior, self-built.

---

## Architecture Sketch

```
orchestrator.py
  → reads task input
  → selects agent + loads context module
  → sends to API (Anthropic or OpenAI)
  → writes output to engagements/[name]/outputs/
  → optionally feeds output as input to next agent in chain
```

**Key components needed:**

| Component | Description |
|---|---|
| `orchestrator.py` | Task router — reads intent, selects agent, calls API |
| `agents/` | Agent context modules loaded as system prompts |
| `chains/` | Defined prompt chains for multi-step workflows |
| `engagements/` | Per-engagement input/output/eval directories |
| `eval/runner.py` | LLM-as-judge evaluation loop |
| `cache_layer` | Prompt caching for static system prompts (Anthropic API) |

---

## API Access Needed

- Anthropic API key → Claude Sonnet, Claude Opus
- OpenAI API key → GPT-4o, GPT-4.1, GPT-4.5

---

## Why This Is Worth Building Eventually

- True subagent spawning — not simulated by context loading
- Prompt chaining without manual copy-paste between steps
- Evaluation loops that run automatically and write results
- Full version control on prompts + outputs + evals
- Cost tracking per engagement
- Reproducible: same inputs always produce auditable outputs

---

## Reference Patterns

- Claude Code agent loop (subagent spawning, tool use)
- LangChain / LangGraph for orchestration patterns (optional dependency)
- Anthropic prompt caching API for cost reduction on stable system prompts

---

## Starting Point When Ready

1. Get API keys (Anthropic + OpenAI)
2. Scaffold `orchestrator.py` with a simple single-agent call
3. Port agent context modules from markdown → system prompt strings
4. Add prompt chaining for one workflow (e.g., research → synthesis → output)
5. Add eval runner as a separate module
6. Add cost tracking

