# Concepts

How local models work, written for learning. Project status and decisions live in `../README.md`.

Add lasting lessons from `notes/` here.

---

## The Runtime

**Ollama's layers:** an inference engine (llama.cpp, running on the M2 GPU via Metal) → a model manager (`~/.ollama/models/`) → a local server on `localhost:11434` → the CLI, which is just a client of that server. Nothing talks to the model directly; everything goes through the server.

**OpenCode vs Ollama:** OpenCode is the frontend *and* the agent logic. It decides what to send: it builds the prompt, runs the loop and executes tools. Ollama is the backend and only turns prompts into tokens. A bad prompt or a mishandled tool call fails even with a good model.

**Weights:** a GGUF file. The header holds metadata (architecture, layer count, context length, vocabulary, chat template), followed by named tensors: matrices of numbers, e.g. `blk.0.attn_q.weight`. It's data only, nothing executable. Q4 quantization shrinks an 8B model from about 16GB to about 5GB.

**What happens on each prompt:** load the weights into memory → apply the chat template → tokenize → generate one token at a time through every layer → stop token → turn tokens back into text. The KV cache holds the conversation so far, so a longer context means more RAM. That's why `num_ctx` matters.

**What fits on 16GB:** Qwen 8–14B ✅ · DeepSeek R1 distills 8B/14B ⚠️ (Qwen/Llama models trained on R1 outputs, not DeepSeek's own model) · DeepSeek V3 and the full R1 (671B) ❌ · Kimi K2 (about 1T) ❌ locally, ✅ via API (Tier 1).

## Claude Code Features vs Local Equivalents

| Claude Code feature | Local equivalent | On 16GB |
|---|---|---|
| Agent `.md` identities | System prompt | Easy |
| Skills | Appended to the system prompt | Easy, but limited by context size |
| `context: fork` | A fresh message list per call | Comes for free |
| Side-channel output to file | Harness/code writes the file | Easy |
| Routing | You name the agent, or code picks it by rule | Easy |
| Sequential chains | Harness/code passes file paths | Moderate |
| Parallel agents | Requests queue (one model at a time) | Effectively sequential |
| Tool use | The model must output tool calls in the right format | Unreliable at 8–14B |
| Model as orchestrator | An agent loop | Small models break here |

**Three designs:** (1) model as orchestrator, like Claude Code, which doesn't work on 16GB; (2) code/harness as orchestrator, **the chosen design**; (3) hybrid, where Claude Code stays the orchestrator and hands chosen tasks to the local harness (Phase 5).
