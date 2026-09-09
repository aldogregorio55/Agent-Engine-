# Build with Claude API — Notes

Part of: Claude Certified Architect — Foundations
Owner: agregorio1@kpmg.com.au · Started: 2026-08-04

Cross-reference: 4D Framework anchor in [ai-fluency-notes.md](ai-fluency-notes.md#the-4d-framework-course-anchor).

---

## Course overview

_The full module list from the course. Sections will be filled in as I work through them._

- **Introduction** — Welcome, Anthropic overview, overview of Claude models.
- **Accessing Claude with the API** — API key, requests, multi-turn conversations, system prompts, temperature, streaming, structured data.
- **Prompt evaluation** — Eval workflow, generating test datasets, running evals, model-based grading, code-based grading.
- **Prompt engineering techniques** — Clear and direct, specific, XML tags, examples.
- **Tool use with Claude** — Tool functions, schemas, message blocks, tool results, multi-turn with tools, multiple tools, fine-grained tool calling, text edit tool, web search tool.
- **RAG and Agentic Search** — Text chunking, embeddings, full RAG flow, BM25 lexical search, multi-index RAG pipeline.
- **Features of Claude** — Extended thinking, image support, PDF support, citations, prompt caching, code execution and Files API.
- **Model Context Protocol (MCP)** — Clients, tools, server inspector, resources, prompts.
- **Anthropic apps** — Claude Code, computer use, MCP-server enhancements.
- **Agents and workflows** — Parallelization, chaining, routing, agents and tools, environment inspection, workflows vs agents.
- **Final assessment.**

---

## Module notes

### 1. Accessing Claude with the API

#### a. Access the API

**The five-step request flow**

Every Claude interaction follows the same pipeline:

1. **Client → your server** — user action in the web/mobile app sends a request to your backend.
2. **Your server → Anthropic API** — backend forwards the request using the stored API key.
3. **Model processing** — Claude tokenises, embeds, contextualises, and generates.
4. **Anthropic API → your server** — structured response returned.
5. **Your server → client** — generated text rendered in the UI.

**Why a server is required**

- API requests need a secret API key for authentication.
- Putting the key in client code exposes it — anyone can extract it and make unauthorised calls.
- The client app must talk to **your** server; only your server talks to Anthropic.

**Making the request**

- Official SDKs: **Python, TypeScript, JavaScript, Go, Ruby**. Plain HTTP also works.
- Every request must include four essential fields:

  | Field | Purpose |
  |---|---|
  | `api_key` | Identifies your account to Anthropic. |
  | `model` | Which model to call (e.g. `claude-3-sonnet`). |
  | `messages` | List containing the user's input. |
  | `max_tokens` | Upper bound on tokens Claude may generate. |

**Inside Claude's processing (4 stages)**

1. **Tokenisation** — input text is split into tokens (whole words, sub-words, spaces, or symbols). Mental model: ~1 word ≈ 1 token.
2. **Embedding** — each token becomes a long vector of numbers representing **all** possible meanings of that token. Think of it as a numerical definition capturing semantic relationships.
   - Example: "quantum" can mean a discrete physical unit, quantum mechanics, something extremely small, or quantum computing — the embedding carries all of these.
3. **Contextualisation** — each embedding is refined based on surrounding tokens so the correct meaning is highlighted for this specific input.
4. **Generation** — contextualised embeddings pass through an output layer that produces a probability distribution over the next token. Claude samples with **controlled randomness** (not pure argmax) to keep responses natural and varied. The chosen token is appended and the whole loop repeats for the next token.

**When Claude stops generating**

After each token, Claude checks three stop conditions:

- **Max tokens reached** — the `max_tokens` limit you set.
- **Natural ending** — an end-of-sequence token was generated.
- **Stop sequence** — a predefined stop phrase was hit.

**The API response**

Structured payload containing:

| Field | Contents |
|---|---|
| `message` | The generated text. |
| `usage` | Input and output token counts. |
| `stop_reason` | Why generation ended. |

**Why this matters**

- Design **secure architectures** that keep API keys off the client.
- Set **appropriate `max_tokens`** for the use case.
- Handle **different stop reasons** in application logic.
- Debug faster by knowing **which pipeline stage** an issue belongs to.

---

#### b. Making a request

**Setting up the environment (Python / Jupyter)**

1. Install dependencies:

   ```bash
   %pip install anthropic python-dotenv
   ```

2. Store the API key in a `.env` file **next to the notebook** (never in code, never in git):

   ```env
   ANTHROPIC_API_KEY="your-api-key-here"
   ```

3. Add `.env` to `.gitignore`.
4. Load the env vars and create the client:

   ```python
   from dotenv import load_dotenv
   load_dotenv()

   from anthropic import Anthropic

   client = Anthropic()
   model = "claude-sonnet-4-0"
   ```

   - `Anthropic()` picks up `ANTHROPIC_API_KEY` from the environment automatically.

**The `client.messages.create()` function**

Three required parameters:

| Parameter | Purpose |
|---|---|
| `model` | Which Claude model to call. |
| `max_tokens` | **Safety cap**, not a target — Claude stops if it hits the cap, but does not try to reach it. |
| `messages` | Conversation history sent to Claude. |

> **Key nuance:** `max_tokens` is a ceiling, not a goal. Claude writes what it thinks is appropriate and stops naturally; the cap only bites if the response would otherwise run longer.

**Messages**

Messages model a chat between you and Claude. Each entry is a dict with a `role` and `content`.

| Role | Meaning |
|---|---|
| `user` | Content sent to Claude (usually written by a human). |
| `assistant` | Content generated by Claude in prior turns. |

**First request — full example**

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "What is quantum computing? Answer in one sentence",
        }
    ],
)
```

The call returns a **response object** containing the generated text plus metadata.

**Extracting the text**

The response carries a lot of metadata; the clean text lives at:

```python
message.content[0].text
```

Example output:

> "Quantum computing is a type of computation that leverages quantum mechanics principles like superposition and entanglement to process information using quantum bits (qubits), potentially solving certain complex problems exponentially faster than classical computers."

**Why this matters**

- `.env` + `python-dotenv` is the standard local-dev pattern for keeping keys out of source.
- `max_tokens` is misunderstood as a length target; treating it as a **safety cap** avoids over-tight limits truncating good responses.
- `content` is a **list of blocks** — `content[0].text` works for a simple text reply, but the same shape scales to tool use, images, etc.

---

#### c. Multi-turn conversations

**The stateless API**

The Anthropic API keeps **no server-side history**. Every `messages.create()` call is independent. If you want Claude to "remember" earlier turns, **you** must replay the whole conversation on every request.

- Ask *"What is quantum computing?"* → good answer.
- Follow up with *"Write another sentence"* in a **new** request with only that message → Claude has no anchor and writes something unrelated.

**Two rules that make it work**

1. Maintain a `messages` list in your code.
2. Send the **entire** list with every request.

**Turn-by-turn flow**

1. Append the user's message (`role="user"`).
2. Send the full list to Claude.
3. Append Claude's reply back as `role="assistant"`.
4. Append the next user turn.
5. Send the full list again. Repeat.

**Helper functions**

```python
def add_user_message(messages, text):
    messages.append({"role": "user", "content": text})

def add_assistant_message(messages, text):
    messages.append({"role": "assistant", "content": text})

def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=messages,
    )
    return message.content[0].text
```

**Usage**

```python
messages = []

add_user_message(messages, "Define quantum computing in one sentence")
answer = chat(messages)
add_assistant_message(messages, answer)

add_user_message(messages, "Write another sentence")
final_answer = chat(messages)
```

Now the follow-up carries the full context and Claude expands on the definition instead of guessing.

**Why this matters**

- Conversation state is an **application concern**, not an API feature.
- Every extra turn re-sends all prior tokens → **input token cost grows with conversation length**. Trim, summarise, or window when it gets long.
- The `assistant` role is how you feed prior model output back in — same shape as user turns.

---

#### d. System prompts

**What a system prompt does**

A **system prompt** is out-of-band guidance you send alongside `messages` telling Claude *who to be* and *how to answer* — role, tone, rules, constraints. It doesn't appear in the `messages` list; it's a separate top-level parameter.

- Shapes tone, style, persona, and behavioural rules for the whole conversation.
- Keeps Claude on-task across many turns without having to repeat instructions in every user message.
- Claude will try to answer the way *someone in that role* would answer.

**Motivating example — math tutor**

Student asks: *"How do I solve 5x + 2 = 3 for x?"*

- **No system prompt** → Claude dumps the full step-by-step solution. Correct, but the student doesn't think.
- **With a tutor system prompt** → Claude replies with a *hint*: "What operation might you do to both sides first to isolate x?"

Same user message, completely different behaviour — driven entirely by the system prompt.

**API shape**

Pass `system` as its own kwarg on `client.messages.create()` — it is **not** an entry in `messages`:

```python
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""

client.messages.create(
    model=model,
    messages=messages,
    max_tokens=1000,
    system=system_prompt,
)
```

**Flexible `chat` helper**

Extend the helper from 1.c so the system prompt is optional. **Important gotcha:** the API rejects `system=None`, so only include the kwarg when a value is provided.

```python
def chat(messages, system=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
    }
    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

Call it either way:

```python
# Generic
answer = chat(messages)

# Role-conditioned
answer = chat(messages, system="You are a patient math tutor. ...")
```

**Why this matters**

- System prompts are the cleanest way to enforce **persona, scope, and refusal rules** without polluting user turns.
- Because it lives outside `messages`, the persona **persists across every turn** without you re-sending it as a fake user message.
- `system=None` is not accepted — build the kwargs dict conditionally.

---

#### e. Temperature

**What temperature controls**

Recall Claude's generation loop: tokenise → embed → contextualise → **sample from a probability distribution over the next token**. `temperature` is a decimal between **0 and 1** that reshapes that distribution before sampling.

- **Low temperature (near 0)** → distribution sharpens onto the top choice. Output becomes **deterministic and repetitive**.
- **High temperature (near 1)** → distribution flattens across plausible tokens. Output becomes **varied and creative**.

Mental model: a "creativity dial" — but it works by widening or narrowing the pool of tokens Claude will consider, not by injecting randomness into the prompt.

**Token-probability example**

For "What do you think ___", the raw distribution might be `about: 30%`, `would: 20%`, `of: 10%`, …

- At `temperature=0.0` → "about" is picked ~100% of the time. Same prompt → same answer.
- At `temperature=1.0` → probabilities stay closer to their raw values; other tokens get real chances.

**Choosing a value**

| Range | Good for |
|---|---|
| **0.0 – 0.3** | Factual answers, coding, data extraction, content moderation, classification. |
| **0.4 – 0.7** | Summarisation, educational content, problem-solving, creative writing with constraints. |
| **0.8 – 1.0** | Brainstorming, open creative writing, marketing copy, joke generation. |

**Adding `temperature` to the `chat` helper**

Extend the helper from 1.d — `temperature` has a valid default (unlike `system`), so it goes straight into `params`:

```python
def chat(messages, system=None, temperature=1.0):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
    }
    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

Usage:

```python
# Predictable
answer = chat(messages, temperature=0.0)

# Creative
answer = chat(messages, temperature=1.0)
```

**Why this matters**

- Temperature is the primary knob for **consistency vs variety**. Pick it based on the task, not by feel.
- **Low temperature ≠ guaranteed identical output**, and **high temperature ≠ guaranteed different output** — it only changes probabilities. Repeats can still happen at `1.0`.
- For evaluations and regressions where determinism matters, pin `temperature=0`.

---

#### f. Streaming

**Why streaming**

A single Claude call can take **10–30 seconds** to return a full response. Blocking the client until the whole message is ready leaves the user staring at a spinner. **Streaming** sends the response in small pieces as tokens are generated so the UI can render text as it appears.

- Same request, same content — only the transport changes.
- Server can forward chunks to the browser/mobile client (SSE, WebSocket, etc.) as they arrive.
- All events belong to a **single API request**.

**Stream event types**

When `stream=True`, the API sends a sequence of typed events:

| Event | Meaning |
|---|---|
| `message_start` | The response has begun; carries top-level message metadata. |
| `content_block_start` | A new block is opening (text, tool use, thinking, etc.). |
| `content_block_delta` | A **chunk of generated content** — for text blocks, the incremental text lives here. |
| `content_block_stop` | The current block is complete. |
| `message_delta` | Top-level message updates (e.g. `stop_reason`, `usage`). |
| `message_stop` | End of the entire response. |

Rule of thumb: to render text, watch `content_block_delta` events.

**Raw streaming (see the events)**

```python
messages = []
add_user_message(messages, "Write a 1 sentence description of a fake database")

stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True,
)

for event in stream:
    print(event)
```

Use this shape when you need full control — tool use, thinking blocks, custom transport, etc.

**Simplified text streaming (SDK helper)**

For "just render text as it arrives," the SDK exposes a context manager that filters everything down to text chunks:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages,
) as stream:
    for text in stream.text_stream:
        print(text, end="")
```

Two things to notice:

- `client.messages.stream(...)` is used as a `with` block — the SDK closes the HTTP connection cleanly on exit.
- `stream.text_stream` yields **only** the incremental text; no event parsing needed.

**Getting the assembled final message**

Streaming is great for UX, but you almost always need the **complete message** afterwards for logging, storage, or feeding back into `messages` as an assistant turn. Call `get_final_message()` after the loop:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages,
) as stream:
    for text in stream.text_stream:
        # forward `text` to the client here
        pass

    final_message = stream.get_final_message()
```

`final_message` is the same shape as a non-streaming response (`content`, `stop_reason`, `usage`, …).

**Why this matters**

- Streaming is a **UX concern**, not a correctness one — output is identical to non-streaming.
- For most apps, `client.messages.stream(...)` + `text_stream` + `get_final_message()` is the ergonomic default.
- Reach for raw `stream=True` events only when you need tool calls, thinking blocks, or per-event bookkeeping.

---

#### g. Structured data (assistant prefilling + stop sequences)

**The problem**

Claude is trained to be helpful, so when you ask for JSON / code / a list it often wraps the payload in a markdown fence and adds a sentence of explanation:

````text
```json
{ ... }
```
This rule captures EC2 instance state changes ...
````

For an app that needs to `json.loads()` the reply, or a "copy" button in a UI, that wrapping is friction.

**The technique — two levers together**

Combine **assistant message prefilling** with a **stop sequence**:

1. **Prefill** the assistant turn with the *opening* of the wrapper (e.g. ` ```json `). Claude thinks it has already started the code block and just continues the content.
2. **Stop sequence** on the *closing* wrapper (e.g. ` ``` `). The moment Claude tries to close the fence, generation halts — no trailing prose.

```python
messages = []

add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json")

text = chat(messages, stop_sequences=["```"])
```

Result — raw JSON, nothing else:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": { "state": ["running"] }
}
```

**Post-processing**

You'll often see leading/trailing newlines. Trim and parse:

```python
import json
data = json.loads(text.strip())
```

**Extending the `chat` helper for `stop_sequences`**

`stop_sequences` is optional and defaults to no sequences. Treat it like `system` — only include the kwarg when the caller provides a value:

```python
def chat(messages, system=None, temperature=1.0, stop_sequences=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
    }
    if system:
        params["system"] = system
    if stop_sequences:
        params["stop_sequences"] = stop_sequences

    message = client.messages.create(**params)
    return message.content[0].text
```

**Beyond JSON**

Same recipe works for anything Claude tends to wrap:

| Target | Prefill | Stop |
|---|---|---|
| JSON | ` ```json ` | ` ``` ` |
| Python | ` ```python ` | ` ``` ` |
| CSV | ` ```csv ` | ` ``` ` |
| Bulleted list | `- ` | (usually not needed — end-of-turn suffices) |
| XML block | `<result>` | `</result>` |

Identify **what Claude naturally wraps your content in**, prefill the opening, stop on the closing.

**How `stop_reason` changes**

When a stop sequence fires, the response reports:

- `stop_reason = "stop_sequence"`
- `stop_sequence = "```"` (the actual string that matched)

Watch these when debugging truncated output — it distinguishes "Claude finished naturally" from "your stop sequence bit."

**Why this matters**

- Assistant prefilling is a form of **soft control** — you're not instructing Claude, you're *starting the sentence for it*. Much more reliable than "please output only JSON."
- Stop sequences give you a **hard cut-off**. Combined with a prefill they bracket the output precisely.
- Both work in every mode (single-turn, multi-turn, streaming) and cost nothing extra.

---

### 2. Prompt evaluation

#### a. Prompt engineering vs prompt evaluation

**Two different disciplines**

| | Prompt engineering | Prompt evaluation |
|---|---|---|
| **Focus** | *How* to write a prompt. | *How well* the prompt actually works. |
| **Toolkit** | Multishot examples, XML tags, role/system prompts, prefill, chain-of-thought, etc. | Test datasets, automated grading (code-based or model-based), scoring dashboards, A/B comparisons. |
| **Output** | A better prompt. | Objective evidence a prompt is fit for production. |
| **When** | Every time you write or edit a prompt. | Before shipping, and every time you change the prompt / model / temperature. |

**Three paths after drafting a prompt**

Every engineer faces the same fork after writing a prompt. Only one path scales.

1. **"Ship it after one manual test."**
   Highest risk. Real users produce inputs you didn't imagine and the prompt breaks in production.
2. **"Test a few times, patch a couple of edge cases."**
   Better, but still ad-hoc. You're only ever finding the edge cases *you* can think of, not the ones users will actually send.
3. **"Run the prompt through an evaluation pipeline, score it, iterate on the metrics."**
   More upfront work and API cost, but the only path that produces measurable confidence and lets you compare prompt versions objectively.

Options 1 and 2 are the default traps. The training instinct is to overtrust a prompt that "worked in the notebook."

**The evaluation-first approach**

An eval pipeline gives you four things a manual sanity-check never will:

- **Coverage** — dozens or hundreds of inputs, including adversarial and edge cases, hit the prompt on every run.
- **Objective metrics** — a numeric score per test case and an aggregate, not a vibe.
- **Regression detection** — every prompt / model / temperature change is scored against the same fixed dataset, so improvements and regressions are visible.
- **Faster iteration** — you can try five prompt variants and pick the winner in minutes instead of weeks of production feedback.

**Mental model**

Prompt engineering is *writing the code*. Prompt evaluation is *the test suite*. Shipping an untested prompt to production is the same category of mistake as shipping an untested function.

**Why this matters**

- Reliability of an LLM feature ≈ quality of its eval, not eloquence of its prompt.
- Anything you can't measure, you can't safely change — model swaps and prompt edits become guesswork.
- The rest of Module 2 is about **how** to build this pipeline: test datasets → run evals → grade them (code-based and model-based).

---

#### b. The prompt evaluation workflow

**Five steps, one loop**

Regardless of the tool (homegrown script, Anthropic Console, LangSmith, Braintrust, etc.) the shape is the same:

1. **Draft a prompt.**
2. **Create an eval dataset.**
3. **Feed each input through Claude.**
4. **Feed each response through a grader.**
5. **Change the prompt and repeat.**

Repeat until the aggregate score plateaus or hits your target.

**Step 1 — Draft a prompt**

Write the simplest reasonable version first. It only needs to be a template with a hole for the per-case input.

```python
prompt = f"""
Please answer the user's question:

{question}
"""
```

Start dumb; the eval will tell you exactly where it hurts.

**Step 2 — Create an eval dataset**

A list of representative inputs that will be interpolated into the template. Same shape as unit-test fixtures.

```python
dataset = [
    {"question": "What's 2+2?"},
    {"question": "How do I make oatmeal?"},
    {"question": "How far away is the Moon?"},
]
```

Guidelines:

- **Cover the real distribution.** Include easy cases, tricky cases, and adversarial edge cases.
- **Size scales with stakes.** A dozen for a scratch experiment, hundreds or thousands for a shipping feature.
- **Two ways to build it:** by hand from real user logs, or **generated by Claude** (covered in the next sub-section).
- Optionally include an `expected` field per case so a code-based grader can compare (see 2.d / 2.e).

**Step 3 — Feed through Claude**

Interpolate each case into the template and call the API. One request per case.

```python
outputs = []
for case in dataset:
    filled = prompt.format(question=case["question"])
    messages = [{"role": "user", "content": filled}]
    reply = client.messages.create(model=model, max_tokens=1000, messages=messages)
    outputs.append({"input": case, "output": reply.content[0].text})
```

For speed on larger datasets, run these in parallel (thread pool or async) — order doesn't matter, cases are independent.

**Step 4 — Feed through a grader**

The grader scores each response, giving you a per-case score and an aggregate. Two families (deep-dive later):

- **Code-based grading** — deterministic checks: regex, JSON schema validation, exact match, unit-test assertions. Cheap, fast, no ambiguity — but only works when correctness is mechanically checkable.
- **Model-based grading** — a second Claude call with a rubric that scores the response 1–10 (or pass/fail). Handles subjective quality (helpfulness, tone, thoroughness) but costs more tokens and needs a well-written rubric.

Example scores for the three cases above:

| Question | Score |
|---|---|
| What's 2+2? | 10 |
| How do I make oatmeal? | 4 |
| How far away is the Moon? | 9 |

Aggregate: **(10 + 4 + 9) / 3 = 7.66**.

That single number is your baseline.

**Step 5 — Change prompt and repeat**

Now make one change to the prompt and re-run the same dataset through steps 3–4. Compare scores.

```python
prompt = f"""
Please answer the user's question:

{question}

Answer the question with ample detail.
"""
```

New aggregate: **8.7**. The added instruction helped — objectively, not by feel. Keep this version, form a new hypothesis, repeat.

**Discipline that makes this work**

- **Change one thing at a time.** Prompt *or* model *or* temperature. Otherwise you can't attribute the score change.
- **Freeze the dataset between runs.** Comparing scores across different datasets is meaningless.
- **Keep the grader deterministic.** For model-based grading, pin `temperature=0` on the grader call so the same response scores the same way each time.
- **Log everything.** Save `(prompt_version, case_id, input, output, score, rubric_notes)` so you can diff regressions later.

**Why this matters**

- The workflow is the whole point of prompt evaluation — every eval tool on the market is a UI over these five steps.
- It converts prompt work from "does this look better?" to "did the aggregate score go up?" — a decision you can make in seconds.
- It lets you compare a *new prompt on the current model* vs *the current prompt on a new model* vs *both changed at once*, and know which combination actually wins.

---

#### c. Running the eval pipeline

**Three functions, one flow**

The 5-step workflow (2.b) collapses to three tiny functions in practice. Once you have a dataset — a list of test cases with an input field per case — the pipeline is:

```text
dataset  →  run_prompt (per case)  →  Claude output  →  grader  →  {output, test_case, score}
                                                                        ↓
                                            run_test_case              run_eval loops over all cases
```

**`run_prompt(test_case)` — merge and call Claude**

Interpolates the case into the prompt template and returns Claude's response. Nothing more.

```python
def run_prompt(test_case):
    """Merge the prompt template with the test case, return Claude's raw output."""
    prompt = f"""
Please solve the following task:

{test_case["task"]}
"""
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)
```

Note: the starter prompt is intentionally minimal — no format instructions, no constraints. Claude will over-explain and add prose. That's fine; the eval is what tells you *when* the prompt is too loose.

**`run_test_case(test_case)` — run one case + grade it**

Wraps `run_prompt` with grading and returns a structured result.

```python
def run_test_case(test_case):
    """Run one case, grade the output, return a structured result."""
    output = run_prompt(test_case)

    # TODO: replace with a real grader (code-based and/or model-based).
    score = 10

    return {
        "output": output,
        "test_case": test_case,
        "score": score,
    }
```

The hardcoded `score = 10` is a **deliberate placeholder** so the pipeline can be end-to-end tested before graders exist. Real graders replace this line (see 2.d / 2.e).

**`run_eval(dataset)` — run the whole dataset**

Loops over every case and collects results.

```python
def run_eval(dataset):
    """Run the full dataset through Claude + grader, return a list of results."""
    results = []
    for test_case in dataset:
        results.append(run_test_case(test_case))
    return results
```

**Executing it**

```python
import json

with open("dataset.json", "r") as f:
    dataset = json.load(f)

results = run_eval(dataset)
print(json.dumps(results, indent=2))
```

First run of a full dataset takes 30 s or more even on Haiku because cases run **sequentially** — one API round-trip per case. That's fine at development time; parallelisation is a later optimisation.

**Shape of a result**

Every entry contains three keys:

| Key | Meaning |
|---|---|
| `output` | Claude's raw response text for that case. |
| `test_case` | The original case dict (input, expected, metadata). |
| `score` | Grader output — number (e.g. 1–10) or pass/fail. Placeholder for now. |

Keeping all three in the result object is the point — you can debug regressions later by re-reading the full input + output alongside the score without another API call.

**What this pipeline gives you (and doesn't)**

Have:
- End-to-end automation from dataset → scored results.
- A single structure to serialise, diff, and re-analyse.
- A place to plug in the real grader without touching the rest of the pipeline.

Missing:
- **Grading logic.** Currently a placeholder; next two sub-sections fix that.
- **Parallelism.** One request at a time; fine for tens of cases, painful for thousands.
- **Persistence / dashboards.** Roll your own JSON file, or plug into an eval tool.

**Why this matters**

- Once these three functions exist, adding a new prompt version is a **one-line swap** and re-running the eval is a **one-line call** — the whole point of the workflow.
- Structuring the result as `{output, test_case, score}` from day one saves you from re-running the whole dataset to investigate any single bad case later.
- Sequential is fine to start; don't optimise before you have a working grader.

---

#### d. Graders

**What a grader is**

A **grader** turns a Claude response into a measurable signal — usually a number in `[1, 10]` (10 = high quality, 1 = poor) or a pass/fail. It's the thing that replaces `score = 10` in `run_test_case`.

**Three families**

| Family | Runs on | Best for | Trade-offs |
|---|---|---|---|
| **Code grader** | Your CPU. | Format, length, exact match, syntax (JSON / regex / Python parse), regex hits, word presence/absence, readability scores. | Cheap, fast, deterministic. Only works when correctness is mechanically checkable. |
| **Model grader** | A second Claude call with a rubric. | Response quality, instruction following, completeness, helpfulness, safety, tone. | Handles subjective quality. Slower, costs tokens, needs a good rubric, can be noisy. |
| **Human grader** | People. | Deep subjective judgment: comprehensiveness, depth, conciseness, relevance, taste. | Most flexible, gold-standard signal. Slow, expensive, doesn't scale. Best used to calibrate the other two. |

Real pipelines usually **mix families** — code graders enforce hard constraints (must-be-valid-JSON), model graders assess the softer axes (task following, tone).

**Define the criteria first**

Before writing any grader, list the specific things you care about. Example: a prompt that generates Python code for a task might be judged on:

1. **Format** — output is Python only, no prose or markdown fences.
2. **Valid syntax** — the code parses.
3. **Task following** — the code actually solves the requested task correctly.

Then map each criterion to the cheapest grader that can measure it:

- Criteria 1 and 2 → **code grader** (regex + `ast.parse`).
- Criterion 3 → **model grader** (needs judgment).

That mapping is the whole design of a grading suite.

**Implementing a model grader**

The pattern: build a rubric prompt that names the criteria and asks for a structured JSON response including *strengths*, *weaknesses*, *reasoning*, and *score*. Use prefill + stop sequence (from 1.g) to get raw JSON.

```python
def grade_by_model(test_case, output):
    eval_prompt = f"""
You are an expert code reviewer. Evaluate this AI-generated solution.

Task: {test_case["task"]}
Solution: {output}

Provide your evaluation as a structured JSON object with:
- "strengths":   array of 1-3 key strengths
- "weaknesses":  array of 1-3 key areas for improvement
- "reasoning":   concise explanation of your assessment
- "score":       number between 1 and 10
"""
    messages = []
    add_user_message(messages, eval_prompt)
    add_assistant_message(messages, "```json")

    eval_text = chat(messages, temperature=0.0, stop_sequences=["```"])
    return json.loads(eval_text.strip())
```

Two non-obvious details:

- **Ask for strengths / weaknesses / reasoning, not just a score.** Without a rationale slot, model graders anchor on ~6 for almost everything. Forcing a written justification spreads the distribution and makes the score meaningful.
- **`temperature=0` on the grader.** The same output must score the same way each run, or you can't distinguish "prompt got better" from "grader was in a good mood."

**Wiring the grader into the pipeline**

Only `run_test_case` changes — `run_prompt` and `run_eval` are untouched.

```python
def run_test_case(test_case):
    output = run_prompt(test_case)
    grade = grade_by_model(test_case, output)

    return {
        "output": output,
        "test_case": test_case,
        "score": grade["score"],
        "reasoning": grade["reasoning"],
    }
```

Persist `reasoning` alongside the score — it's what you'll read when a score looks wrong.

**Aggregate reporting**

```python
from statistics import mean

def run_eval(dataset):
    results = [run_test_case(tc) for tc in dataset]
    average_score = mean(r["score"] for r in results)
    print(f"Average score: {average_score:.2f}")
    return results
```

The mean is your headline metric. Track it per prompt version; that's the signal you iterate against.

**Failure modes to watch**

- **Middling-score bias.** Grader keeps returning 6/7 regardless of quality → your rubric is too vague, or you dropped the reasoning field. Add sharper criteria and require justification.
- **Grader temperature drift.** Non-zero temperature on the grader will jitter the aggregate 0.3–0.8 points between runs. Pin to 0.
- **Grader/generator collusion.** Using the same model + no rubric can create optimistic self-grading. For high-stakes evals, grade with a stronger model than you're testing (e.g. Sonnet grading Haiku outputs).
- **JSON parse failures.** Prefill `` ```json `` and stop on `` ``` `` (from 1.g). Wrap `json.loads` in `try/except` and treat parse failures as a zero score plus a logged error.

**Why this matters**

- Graders are what turn the eval loop from "look at outputs" into "read a number." Every remaining lesson in Module 2 is either a specific grader technique or a way to make grader signals more reliable.
- Choosing the right grader family per criterion is a design decision, not a taste one — it directly determines cost, speed, and how much you can trust the score.

---

#### e. Code-based grading

**Why add code-based grading**

Model graders handle subjective quality but they're **slow, expensive, and noisy** for the objective criteria — is this valid JSON? does this Python parse? does this regex compile? Those are `parse()` calls, not API calls. Move every mechanically-checkable criterion off the model grader and onto code.

The 2.d code / model / human split reappears at the *criterion* level:

| Criterion | Grader |
|---|---|
| Format (Python / JSON / Regex only, no prose) | **Code** |
| Valid syntax | **Code** |
| Task following (does it actually solve the task?) | **Model** |

**Three syntax validators**

Each attempts to parse the output as its target language. Success → `10`. Failure → `0`. Deterministic, microsecond-fast, no API cost.

```python
import ast
import json
import re

def validate_json(text):
    try:
        json.loads(text.strip())
        return 10
    except json.JSONDecodeError:
        return 0

def validate_python(text):
    try:
        ast.parse(text.strip())
        return 10
    except SyntaxError:
        return 0

def validate_regex(text):
    try:
        re.compile(text.strip())
        return 10
    except re.error:
        return 0
```

Same shape as `grade_by_model` for compositional purposes: input → number in `[0, 10]`.

**Dataset needs a `format` field**

The code grader has to know which validator to call. Add a `format` field to each case; when generating datasets with Claude (next lesson), include it in the schema.

```json
{
    "task": "Create a Python function to validate an AWS IAM username",
    "format": "python"
}
```

Then dispatch:

```python
VALIDATORS = {
    "python": validate_python,
    "json":   validate_json,
    "regex":  validate_regex,
}

def grade_syntax(output, test_case):
    validator = VALIDATORS[test_case["format"]]
    return validator(output)
```

**Tightening the prompt for code output**

Loose prompts return fenced markdown with prose — every validator will score them `0` because ``` ```python\n...\n``` ``` isn't valid Python. Two changes fix it:

1. **Explicit format instructions** in the user prompt:

   ```text
   * Respond only with Python, JSON, or a plain Regex.
   * Do not add any comments, commentary, or explanation.
   ```

2. **Assistant prefill** (from 1.g) — start the reply *inside* the fence but with a generic language tag so one template covers all three formats:

   ```python
   add_assistant_message(messages, "```code")
   ```

   Combined with `stop_sequences=["```"]`, the response is bracketed to just the payload regardless of whether it's Python, JSON, or regex.

**Composing scores**

Two graders → one number per case. Simplest is an equal-weighted mean:

```python
model_grade = grade_by_model(test_case, output)
model_score = model_grade["score"]
syntax_score = grade_syntax(output, test_case)

score = (model_score + syntax_score) / 2
```

Adjust weights when one axis matters more than the other. For a JSON-schema-critical endpoint you'd weight syntax higher (or make it a hard gate — if `syntax_score == 0`, force `score = 0`).

**Wiring it in**

Only `run_test_case` changes; `run_prompt` / `run_eval` untouched.

```python
def run_test_case(test_case):
    output = run_prompt(test_case)

    model_grade = grade_by_model(test_case, output)
    syntax_score = grade_syntax(output, test_case)
    score = (model_grade["score"] + syntax_score) / 2

    return {
        "output": output,
        "test_case": test_case,
        "score": score,
        "model_score": model_grade["score"],
        "syntax_score": syntax_score,
        "reasoning": model_grade["reasoning"],
    }
```

Persist both sub-scores. When the composite drops, you want to know whether the code broke, the quality dropped, or both.

**Reading the baseline**

The absolute number isn't inherently good or bad. What matters:

- **Baseline first**, then iterate. Is the aggregate going up per prompt change?
- **Per-criterion trace.** If `syntax_score` is a wall of `0`, the prompt is leaking prose or fences — fix the prompt / prefill before touching content quality.
- **A/B on prompt versions**, not on the scale. Compare v0 vs v1 on the same dataset.

**Why this matters**

- Every mechanically-checkable criterion you offload from the model grader is API cost + latency + noise you don't pay.
- Composite scoring makes the eval **multi-axis** — you know *what* dropped, not just that something dropped.
- A dataset `format` field is what turns "run the eval on new tasks" from custom-code into a data change.

---

### 3. Prompt engineering techniques

<!-- TODO: 3.b Specific — lesson not yet received; will be inserted here when available. -->

#### a. Being clear and direct

**Core principle:** The first line of a prompt is the most important sentence you write. It sets the frame for everything that follows.

**Clear = simple language, zero ambiguity**

- State exactly what you want without hedging or circling around the task.
- Vague: `"I need to know about those things people put on their roofs that use sun — those solar panel things, I think they're called."`
- Clear: `"Write three paragraphs about how solar panels work."`

**Direct = instruction-first structure**

- Use imperative action verbs: **Write, Create, Generate, Identify, Summarise, List**.
- Give instructions, not questions.
- Vague: `"I was reading about renewable energy and geothermal sounds neat. What countries use it?"`
- Direct: `"Identify three countries that use geothermal energy. Include generation stats for each."`

**The three-part formula for a strong opening line**

| Part | Purpose | Example |
|---|---|---|
| Action verb | What to do | `Generate` |
| Output object | What to create | `a one-day meal plan` |
| Constraints | Scope / audience / restrictions | `for an athlete that meets their dietary restrictions` |

Full result: `"Generate a one-day meal plan for an athlete that meets their dietary restrictions."`

**Measured impact**

Restructuring the first line alone moved an eval score from **2.32 → 3.92** on the meal-plan example — without changing anything else.

**Mental model:** Treat Claude as a capable assistant who needs clear direction, not one who should guess intent.

---

#### b. XML tags

**Why structure matters**

When a prompt contains large amounts of data or mixes different content types, Claude can struggle to determine which text is instructions vs which is data. XML tags solve this by creating explicit delimiters.

**The pattern**

Wrap each distinct content block in descriptive, custom tags:

```xml
<athlete_information>
- Height: 6'2"
- Weight: 180 lbs
- Goal: Build muscle
- Dietary restrictions: Vegetarian
</athlete_information>

Generate a meal plan based on the athlete information above.
```

**Custom tag names — be descriptive**

| Less useful | More useful |
|---|---|
| `<data>` | `<sales_records>` |
| `<info>` | `<athlete_information>` |
| `<text1>` / `<text2>` | `<my_code>` / `<docs>` |

The more specific the tag name, the clearer the purpose of each section.

**When to use XML tags**

- Including large amounts of context or data
- Mixing different content types (code, documentation, raw data)
- Prompts with multiple interpolated variables
- Any time you want to be explicit about content boundaries

**Key insight:** XML tags are especially valuable as prompt complexity grows. For simple one-liner prompts the benefit is minor; for multi-block prompts it's substantial.

---

#### c. Examples (one-shot / multi-shot prompting)

**Core idea:** Show rather than tell. Instead of describing what you want, demonstrate it with input/output pairs.

**Terminology**

- **Zero-shot** — no examples; Claude relies solely on the instructions.
- **One-shot** — one example provided.
- **Multi-shot** — multiple examples; use when you need to cover different edge cases or output styles.

**Structure: always use XML tags around examples**

```xml
Here is an example input with an ideal response:

<sample_input>
Great game tonight!
</sample_input>
<ideal_output>
Positive
</ideal_output>

Here is another example:

<sample_input>
Oh yeah, I really needed a flight delay tonight! Excellent!
</sample_input>
<ideal_output>
Negative
</ideal_output>
<context>
This tweet uses sarcasm — surface language is positive but intent is negative.
</context>
```

**Annotate why the output is ideal**

Don't just give the pair — explain the reasoning:

```xml
<ideal_output>
[Your example output here]
</ideal_output>

This example is well-structured, provides detailed information on food choices
and quantities, and aligns with the athlete's goals and restrictions.
```

This teaches Claude the reasoning behind the answer, not just the format.

**Mining examples from evals**

Run an eval and look for cases that scored `10` (your highest). Those input/output pairs are ready-made examples. Feed them back into your prompt to raise the floor for future runs.

**When examples are most valuable**

- Handling corner cases (sarcasm, ambiguity, edge formats)
- Defining complex output formats (specific JSON shapes, nested structures)
- Establishing exact style or tone
- Showing how to handle ambiguous inputs

---

### 4. Tool use with Claude

#### a. Introduction — why tools exist

**The problem:** Claude's knowledge is frozen at training time. It cannot access current events, real-time data, or external systems without help.

**The solution:** Tools give Claude a structured way to request and receive fresh information from your application at inference time.

**The three gaps tools solve (reminder system project)**

| Gap | Problem | Tool |
|---|---|---|
| Time awareness | Claude may not know the exact current time | `get_current_datetime` |
| Date arithmetic | Claude handles far-future date addition unreliably | `add_duration_to_datetime` |
| Reminder mechanism | Claude has no built-in way to set a reminder | `set_reminder` |

**The four-step tool use flow**

1. **Initial request** — you send Claude a question plus a description of the available tools.
2. **Tool request** — Claude decides it needs external data and responds with a structured tool call (name + arguments) instead of a final answer.
3. **Data retrieval** — your server executes the requested tool and fetches the result.
4. **Final response** — you send the tool result back to Claude, which generates a complete answer using both the original question and the fresh data.

**Key principle:** When Claude has a limitation, extend its capabilities through tools — don't try to work around the limitation in the prompt.

**Why decompose into separate tools?**

Each tool solves exactly one gap. This keeps tools small, testable, and reusable. Claude chains them together as needed rather than one monolithic tool trying to do everything.

---

#### b. Tool functions

**What is a tool function?**

A plain Python function that gets executed by your server when Claude decides it needs additional data. Claude cannot run the function itself — it requests the call; your code executes it and returns the result.

**Best practices**

| Practice | Why it matters |
|---|---|
| Descriptive function and parameter names | Claude reads the name to decide whether to call the tool |
| Validate all inputs | Claude can see error messages and may retry with corrected arguments |
| Raise clear errors | `"date_format cannot be empty"` teaches Claude what went wrong; a silent failure does not |

**The `get_current_datetime` tool (first implementation)**

```python
from datetime import datetime

def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)
```

- Accepts a `date_format` string so Claude can request the time in different formats.
- Default gives `YYYY-MM-DD HH:MM:SS`.
- The validation check (`if not date_format`) is a pattern — even when the error is unlikely, it demonstrates the input-guard convention that Claude learns from.

**Usage examples**

```python
get_current_datetime()          # "2024-01-15 14:30:25"
get_current_datetime("%H:%M")   # "14:30"
```

**Implementation order for the reminder project**

1. Write the Python function (validate inputs, return a value or raise).
2. Write the JSON schema that describes the function to Claude.
3. Register the schema in the tools list sent with each API request.
4. Add a dispatcher that routes Claude's tool call to the right Python function and returns the result.

Creating the function is step 1 only — the schema and dispatcher come next.

---

#### c. Tool schemas

**What is a tool schema?**

A JSON Schema object that tells Claude what arguments a tool function expects, when to use it, and what it returns. Claude reads the schema at inference time to decide whether and how to call the tool.

**JSON Schema is not AI-specific.** It is a long-standing data-validation specification adopted by the AI community because it cleanly describes function parameters and validates data shapes.

**The three required top-level fields**

| Field | Purpose |
|---|---|
| `name` | Clear, descriptive identifier — matches the Python function name |
| `description` | 3–4 sentences: what it does, when to use it, what it returns, argument details |
| `input_schema` | JSON Schema object describing the function's parameters |

**Writing effective descriptions**

- What the tool does
- When Claude should call it (not just what it returns)
- What the returned data looks like
- Detailed per-argument descriptions inside `input_schema`

**Schema structure**

```python
get_current_datetime_schema = {
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format string. Use this tool whenever the user asks about the current time, today's date, or needs to know when 'now' is. Returns a formatted datetime string.",
    "input_schema": {
        "type": "object",
        "properties": {
            "date_format": {
                "type": "string",
                "description": "A string specifying the output format using Python strftime codes. Default '%Y-%m-%d %H:%M:%S' gives YYYY-MM-DD HH:MM:SS.",
                "default": "%Y-%m-%d %H:%M:%S"
            }
        },
        "required": []
    }
}
```

**Naming convention:** `function_name` + `function_name_schema` — keeps function and schema paired and scannable.

**Generating schemas with Claude**

Instead of writing from scratch:
1. Copy your tool function code.
2. Ask Claude: *"Write a valid JSON schema spec for tool calling for this function. Follow Anthropic's best practices."*
3. Optionally attach the Anthropic tool-use docs as context.
4. Copy the output into your code file and adjust descriptions as needed.

**Adding type safety with `ToolParam`**

```python
from anthropic.types import ToolParam

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    ...
})
```

Wrapping the schema in `ToolParam` is not required for functionality, but it enables type checking and prevents silent errors when the schema is passed to `client.messages.create()`.

---

#### d. Handling message blocks

**New response structure when tools are enabled**

Without tools, `response.content` is a list with a single text block. With tools, Claude returns a **multi-block message** — the list can contain both text and tool-use blocks in the same response.

**Making a tool-enabled API call**

Pass a `tools` list of schemas to `client.messages.create()`:

```python
response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],
)
```

**Anatomy of a multi-block response**

| Block type | `type` field | Contains |
|---|---|---|
| Text block | `"text"` | Human-readable explanation of what Claude is doing |
| Tool-use block | `"tool_use"` | `id`, `name` of function to call, `input` dict of arguments |

**The `tool_use` block fields**

```python
# response.content might look like:
[
    TextBlock(type="text", text="Let me find that for you."),
    ToolUseBlock(
        type="tool_use",
        id="toolu_01XYZ...",   # unique ID for this call
        name="get_current_datetime",
        input={"date_format": "%H:%M:%S"}
    )
]
```

**Preserving multi-block history**

The API is stateless — you must replay history. When an assistant response contains tool blocks, append `response.content` directly (not just the text):

```python
# Correct — preserves all blocks
messages.append({
    "role": "assistant",
    "content": response.content   # the full list of blocks
})
```

Dropping the tool-use block from history breaks Claude's context for the next turn.

**The complete five-step tool flow**

1. Send user message + `tools=[...]` schemas to Claude.
2. Receive assistant message with text block + tool-use block.
3. Extract `name` and `input` from the tool-use block; execute the Python function.
4. Append the assistant message (`response.content`) and a `tool_result` message to history.
5. Send updated history back to Claude; receive the final text response.

**Updating helper functions**

The `add_assistant_message(messages, text)` helper only handles plain strings. For tool responses, append the raw `response.content` list directly rather than routing through the helper — or overload the helper to accept either a string or a content list.

---

#### e. Sending tool results

**Executing the tool**

Extract the input dict from the tool-use block and unpack it into the Python function:

```python
tool_input = response.content[1].input        # dict of arguments
result = get_current_datetime(**tool_input)    # ** unpacks dict as kwargs
```

**The `tool_result` block structure**

After executing the tool, send the output back to Claude inside a user message:

```python
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": response.content[1].id,   # must match the tool_use block id
        "content": str(result),                   # serialised as a string
        "is_error": False
    }]
})
```

| Field | Purpose |
|---|---|
| `tool_use_id` | Links this result to the specific tool-use block that requested it |
| `content` | The function's return value, serialised to a string |
| `is_error` | `True` if execution raised an exception; Claude reads this and may retry or adjust |

**Handling multiple tool calls in one response**

Claude can return several tool-use blocks in a single assistant message. Each needs a matching `tool_result` entry — collect them all into one `content` list before sending:

```python
tool_result_blocks = []
for block in response.content:
    if block.type == "tool_use":
        output = run_tool(block.name, block.input)
        tool_result_blocks.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": str(output),
            "is_error": False,
        })
messages.append({"role": "user", "content": tool_result_blocks})
```

**Include the tool schema on the follow-up request**

Even when you don't expect Claude to call another tool, the schema must still be passed to `client.messages.create()` so Claude can interpret the tool references already in the conversation history.

---

#### f. Multi-turn tool conversations — refactoring helpers

When Claude needs to chain multiple tools to answer a single question (e.g. "What day is 103 days from today?" → `get_current_datetime` → `add_duration_to_datetime` → final answer), the plain string-based helpers break. Three refactors are needed.

**Update `add_user_message` and `add_assistant_message` to accept blocks**

```python
from anthropic.types import Message

def add_user_message(messages, message):
    content = message.content if isinstance(message, Message) else message
    messages.append({"role": "user", "content": content})

def add_assistant_message(messages, message):
    content = message.content if isinstance(message, Message) else message
    messages.append({"role": "assistant", "content": content})
```

Both helpers now accept a plain string, a list of blocks, or a full `Message` object.

**Update `chat` to accept tools and return the full message**

```python
def chat(messages, system=None, temperature=1.0, stop_sequences=[], tools=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
        "stop_sequences": stop_sequences,
    }
    if tools:
        params["tools"] = tools
    if system:
        params["system"] = system
    return client.messages.create(**params)   # return full Message, not just text
```

**Add `text_from_message` utility**

```python
def text_from_message(message):
    return "\n".join(
        block.text for block in message.content if block.type == "text"
    )
```

Use this whenever you need a printable string from a multi-block response.

---

#### g. Implementing multiple turns — the conversation loop

**Detecting when Claude is done**

`response.stop_reason == "tool_use"` means Claude wants another tool call. Any other value (typically `"end_turn"`) means it has a final answer.

**The conversation loop**

```python
def run_conversation(messages):
    while True:
        response = chat(messages, tools=[get_current_datetime_schema])
        add_assistant_message(messages, response)
        print(text_from_message(response))

        if response.stop_reason != "tool_use":
            break

        tool_results = run_tools(response)
        add_user_message(messages, tool_results)

    return messages
```

**The `run_tools` dispatcher**

```python
import json

def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    # add elif branches for each additional tool

def run_tools(message):
    tool_requests = [b for b in message.content if b.type == "tool_use"]
    tool_result_blocks = []
    for req in tool_requests:
        try:
            output = run_tool(req.name, req.input)
            tool_result_blocks.append({
                "type": "tool_result",
                "tool_use_id": req.id,
                "content": json.dumps(output),
                "is_error": False,
            })
        except Exception as e:
            tool_result_blocks.append({
                "type": "tool_result",
                "tool_use_id": req.id,
                "content": f"Error: {e}",
                "is_error": True,
            })
    return tool_result_blocks
```

**Error handling principle:** always return a `tool_result` block even on failure — Claude reads `is_error: True` and the error message to decide what to do next.

**Complete five-step flow**

1. Send user message + `tools=[...]` to Claude.
2. Claude responds: text block(s) + tool-use block(s); `stop_reason = "tool_use"`.
3. Execute all requested tools; build `tool_result` blocks.
4. Append assistant message and tool results to history; loop back to step 1.
5. When `stop_reason != "tool_use"`, Claude's final answer is in the last assistant message.

---

#### h. Using multiple tools

**Pattern for adding a new tool**

1. Write the Python function (validate inputs, return value or raise).
2. Define `function_name_schema` with `name`, `description`, `input_schema`.
3. Add the schema to the `tools=[...]` list in `run_conversation`.
4. Add an `elif` branch in `run_tool`.

**Updated `run_conversation` with all three reminder tools**

```python
response = chat(messages, tools=[
    get_current_datetime_schema,
    add_duration_to_datetime_schema,
    set_reminder_schema,
])
```

**Updated `run_tool` router**

```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)
    elif tool_name == "set_reminder":
        return set_reminder(**tool_input)
```

**Example conversation flow** ("Set a reminder for my doctor's appointment — 177 days after Jan 1st 2050"):

1. Claude calls `add_duration_to_datetime("2050-01-01", 177, "days")` → `"Monday, June 27, 2050 ..."`
2. Claude calls `set_reminder("Doctor's appointment", "2050-06-27T00:00:00")` → prints confirmation.
3. Claude produces a final text response confirming the reminder.

**Key insight:** once the core loop and dispatcher exist, adding a new tool is a data + routing change only — the conversation loop itself never changes.

---

## Key concepts

_Core ideas, definitions, and mental models. Populated as I go._

- **Never call the Anthropic API from client code.** Route through your own server so the API key stays secret.
- **Claude generates one token at a time**, sampling from a probability distribution with controlled randomness — not always the top-probability token.
- **Every response tells you why it ended** via `stop_reason` (`max_tokens`, natural end-of-sequence, or a matched stop sequence).
- **`max_tokens` is a safety cap, not a target.** Claude does not try to fill it.
- **Messages are a list of `{role, content}` dicts** with `role` in `{"user", "assistant"}`. The whole conversation is re-sent each request — the API is stateless.
- **Response `content` is a list of blocks**, not a single string. Access text via `message.content[0].text`.
- **The API is stateless.** No history is stored server-side; you must replay the full `messages` list each call.
- **Assistant turns are appended by you.** Take `message.content[0].text` from a response and push it back as `{"role": "assistant", "content": ...}` before the next user turn.
- **Every turn re-bills prior tokens.** Multi-turn cost scales with conversation length — plan for trimming.
- **System prompts steer behaviour, not just tone.** They set persona, rules, and constraints for the whole conversation and live outside `messages`.
- **`system` is a top-level kwarg**, not a message with `role="system"`. Never send it inside the `messages` list.
- **`system=None` is invalid.** Only include the kwarg when you actually have a value — build the kwargs dict conditionally.
- **Temperature reshapes the next-token distribution.** Low = sharper / deterministic, high = flatter / varied — it does not add randomness, it changes the pool Claude samples from.
- **Temperature is a probability change, not a guarantee.** High temperature can still repeat; low temperature can still vary. Pin `temperature=0` when you want repeatability for evals.
- **Streaming is UX-only.** Output is identical to a blocking call — chunks just arrive as they're generated so the UI can render early.
- **Text lives in `content_block_delta`.** All other event types are structural — start / stop / metadata.
- **Use the `with client.messages.stream(...)` context manager** for text rendering. Fall back to `stream=True` on `messages.create` only when you need raw event access.
- **Always call `get_final_message()`** if you need the assembled response for storage or to append as an assistant turn.
- **Assistant prefilling shapes output more reliably than instructions.** Start the assistant turn with the format prefix (e.g. ` ```json `) and Claude will continue *inside* it.
- **Stop sequences are a hard cut-off, not a suggestion.** The moment Claude generates the string, generation ends — no trailing prose.
- **Prefill + stop sequence brackets the payload.** Use them together for structured output; use either alone when only one edge needs constraining.
- **When a stop sequence fires, `stop_reason == "stop_sequence"`** and the matched string is in `stop_sequence`. This is how you distinguish a natural end from a stop-triggered end.
- **Prompt engineering ≠ prompt evaluation.** Engineering is *how to write* a prompt; evaluation is *how well* it works, measured with a fixed test set.
- **Manual "spot testing" is the default trap.** Users always produce inputs you didn't imagine — the only defence is an automated eval pipeline.
- **Evals turn prompt work into engineering.** Fixed datasets + objective scoring make prompt changes, model swaps, and temperature changes measurable rather than vibes-based.
- **Every prompt / model / temperature change should be re-scored** against the same dataset — otherwise regressions ship silently.
- **Five-step eval loop:** draft → dataset → run through Claude → run through grader → change and repeat.
- **The aggregate score is the decision-maker.** Individual case scores are for debugging; the mean (or pass rate) tells you which prompt version wins.
- **Change one variable per run** — prompt *or* model *or* temperature. Otherwise you can't attribute score movement.
- **The dataset is the contract.** Freezing it between runs is what makes score comparisons meaningful; expanding it is a separate, deliberate action.
- **Two grader families:** code-based (cheap, deterministic, only for mechanical correctness) and model-based (handles subjective quality, costs tokens, needs a rubric).
- **The eval pipeline is three functions:** `run_prompt` (merge + call Claude), `run_test_case` (call + grade), `run_eval` (loop over dataset). Every eval tool is a UI over this shape.
- **Result objects carry `output`, `test_case`, and `score`.** Keeping all three lets you debug any bad case later without re-running the API.
- **Start with a hardcoded score.** Getting the pipeline end-to-end runnable before graders exist proves the plumbing works; drop the real grader in as a one-line change.
- **Three grader families:** code (deterministic, cheap), model (subjective quality, needs rubric), human (gold-standard, doesn't scale). Real pipelines mix them.
- **Design starts from criteria, not code.** List what you care about; map each criterion to the cheapest grader family that can measure it.
- **Model graders need a rationale slot.** Asking only for a score anchors the model at ~6; requiring strengths / weaknesses / reasoning spreads the distribution and makes the score meaningful.
- **Pin the grader to `temperature=0`.** Non-zero temperature jitters aggregates and destroys run-to-run comparability.
- **Persist `reasoning` next to `score`.** When a score looks wrong, the rationale is what you read — no re-running the API.
- **Move every mechanically-checkable criterion to a code grader.** Syntax, format, schema — API-grading these is wasted cost and noise.
- **Composite scores are per-axis.** Persist `model_score` and `syntax_score` separately alongside the composite so you can tell *what* dropped.
- **Dataset `format` field is the dispatcher.** Adding a per-case `format` lets one `grade_syntax` function pick the right validator for JSON / Python / regex.
- **Tighten the prompt before you trust the syntax score.** Format instructions + assistant prefill (` ```code `) + stop sequence keep the payload clean; otherwise every case scores 0 on syntax.
- **First line = most important line.** Lead with a direct action verb + output object + constraints; everything else follows from that frame.
- **Instructions beat questions.** Rewrite questions as imperative commands; they're shorter and clearer for Claude.
- **XML tags solve the boundary problem.** When data + instructions mix in one prompt, wrap each block in descriptive custom tags.
- **Show, don't tell — use examples.** One or more input/output pairs demonstrate requirements more precisely than prose descriptions.
- **One-shot vs multi-shot:** one example sets the pattern; multiple examples cover corner cases and edge formats.
- **Mine best eval results for examples.** Top-scoring cases from your eval pipeline become your prompt's example bank.
- **Annotate why, not just what.** Explain why your example output is ideal; Claude learns reasoning, not just format.
- **Tool use = extending Claude at inference time.** When Claude has a limitation, give it a tool; don't prompt-engineer around the gap.
- **Tool functions are plain Python functions.** They validate inputs, perform work, and return a value (or raise a clear error).
- **Claude requests tool calls; your server executes them.** Claude never runs code directly.
- **Validate tool inputs and raise descriptive errors.** Claude reads error messages and may retry with corrected arguments.

## Terminology

| Term | Meaning |
|---|---|
| Token | Chunk of text (word, sub-word, space, or symbol) Claude reads and generates. |
| Embedding | Numerical vector representing all possible meanings of a token. |
| Contextualisation | Refining a token's embedding using surrounding tokens to resolve its meaning in context. |
| `max_tokens` | Caller-set upper bound on tokens Claude may generate in one response. |
| Stop sequence | Caller-defined string that, if generated, halts output. |
| `stop_reason` | Response field explaining why generation ended (e.g. `end_turn`, `max_tokens`, `stop_sequence`). |
| `usage` | Response field reporting input and output token counts. |
| SDK | Official Anthropic client library — available for Python, TypeScript, JavaScript, Go, and Ruby. |
| `Anthropic()` | Python SDK client class; reads `ANTHROPIC_API_KEY` from env by default. |
| `client.messages.create()` | Primary Python SDK call for sending a request to Claude. |
| `role` | Field on each message: `"user"` (input) or `"assistant"` (Claude's prior output). |
| `content` | Field on each message; on the response it's a **list of blocks** (e.g. `content[0].text`). |
| `.env` file | Local file storing secrets (e.g. `ANTHROPIC_API_KEY`) loaded via `python-dotenv`; must be gitignored. |
| Stateless API | Anthropic stores no conversation history; each request is independent. |
| Conversation history | Client-side `messages` list replayed on every request to give Claude context. |
| `add_user_message` / `add_assistant_message` / `chat` | Convenience helpers that append to the messages list and call `client.messages.create()`. |
| System prompt | Out-of-band instructions passed via the `system` kwarg that set Claude's role, tone, and rules for every turn. |
| Persona / role conditioning | Using a system prompt to make Claude respond as a specific kind of expert (tutor, reviewer, translator, …). |
| `temperature` | Float in `[0, 1]` that reshapes the next-token probability distribution — low = deterministic, high = varied. |
| Streaming | Server-side event stream that emits response chunks as Claude generates them. |
| `stream=True` | Kwarg on `client.messages.create()` that switches the call to raw event streaming. |
| `client.messages.stream(...)` | SDK context manager giving a `text_stream` iterator plus `get_final_message()`. |
| `text_stream` | Iterator over incremental text chunks — filters out non-text events for you. |
| `content_block_delta` | Stream event carrying incremental content (text token slice, tool-input JSON slice, etc.). |
| `get_final_message()` | Method on the streaming context that returns the fully assembled response after the stream closes. |
| Assistant prefilling | Seeding the next assistant turn with an opening fragment so Claude continues from there instead of starting fresh. |
| `stop_sequences` | Optional list-of-strings kwarg on `client.messages.create()` carrying the stop sequences. |
| `stop_sequence` (response) | Response field naming the specific stop string that fired (paired with `stop_reason="stop_sequence"`). |
| Prompt engineering | The craft of writing effective prompts (structure, examples, role setting, prefill, etc.). |
| Prompt evaluation | Systematic measurement of a prompt's quality via automated tests against a dataset. |
| Evaluation pipeline | End-to-end setup that runs a prompt over a test dataset and produces per-case and aggregate scores. |
| Test dataset | Collection of representative inputs (and often expected outputs) used to score a prompt. |
| Regression (LLM) | Silent quality drop when a prompt / model / config change breaks previously working cases. |
| Prompt template | A prompt string with placeholder(s) (e.g. `{question}`) that get filled from the dataset per case. |
| Eval dataset | List of input cases (optionally with expected outputs) used to score prompt versions. |
| Case / record | A single entry in the eval dataset — one input the prompt will be run against. |
| Grader | Function or model that scores a Claude response for one case; returns a numeric score or pass/fail. |
| Code-based grader | Deterministic grader using rules (regex, schema, exact match, assertions). |
| Model-based grader | Grader that uses a second Claude call with a rubric to score the response. |
| Aggregate score | Mean (or pass rate) across all cases in the dataset for a given prompt version. |
| Baseline score | The aggregate score of the initial prompt version; the number every subsequent iteration is compared against. |
| `run_prompt` | Pipeline function that merges the prompt template with a test case and calls Claude. |
| `run_test_case` | Pipeline function that calls `run_prompt` then grades the result into a `{output, test_case, score}` dict. |
| `run_eval` | Pipeline function that loops `run_test_case` over the whole dataset and collects results. |
| Result object | The `{output, test_case, score}` dict emitted per case by the eval pipeline. |
| Human grader | A human reviewer scoring model outputs; gold-standard for subjective quality but doesn't scale. |
| Rubric | Structured instructions to a model grader listing the criteria and score range. |
| `grade_by_model` | Convention name for a model-grader function that returns strengths / weaknesses / reasoning / score. |
| Reasoning field | Free-text rationale a model grader returns alongside the score; makes scores auditable. |
| Middling-score bias | Model graders' tendency to converge on ~6 when no rationale is required; fixed by adding strengths / weaknesses / reasoning. |
| Grader/generator collusion | Optimistic self-grading when the same model both generates and grades without a rubric; mitigate by using a stronger grader model. |
| Syntax validator | Code function that attempts to parse output as a target language (`json.loads`, `ast.parse`, `re.compile`) and returns a numeric score. |
| `grade_syntax` | Dispatcher function that picks the right validator based on the test case's `format` field. |
| `format` field | Per-case dataset field (e.g. `"python"`, `"json"`, `"regex"`) telling the code grader which validator to run. |
| Composite score | Combined score across multiple graders (e.g. mean of `model_score` and `syntax_score`) that becomes the case's `score`. |
| Hard gate | Grader rule that forces score to 0 (or fail) when a critical sub-check fails, regardless of other axes. |
| Clear and direct | Prompt principle: use simple language + imperative verbs + explicit constraints in the opening line. |
| Action verb (prompt) | Opening verb that tells Claude what operation to perform (Write, Create, Generate, Identify, Summarise). |
| Three-part opening | Prompt formula: action verb + output object + constraints. |
| XML tags (prompting) | Custom delimiters (e.g. `<my_code>`, `<docs>`) that separate distinct content blocks in a prompt. |
| Zero-shot | Prompting without any examples; relies entirely on instructions. |
| One-shot | Prompting with a single example input/output pair. |
| Multi-shot | Prompting with multiple example pairs; preferred when edge cases or varied formats must be covered. |
| Example annotation | Explanation of *why* an example output is ideal, placed alongside the pair to teach reasoning not just format. |
| Corner case (prompting) | An edge scenario (e.g. sarcasm, ambiguity) that a simple instruction would handle poorly; best addressed by an example. |
| Tool function | A plain Python function registered with Claude that it can request be called to retrieve external data or perform actions. |
| Tool call | Claude's structured request to invoke a named tool with specific arguments, returned instead of a final answer. |
| Tool result | The value returned by executing a tool function, sent back to Claude so it can complete its response. |
| Tool dispatcher | Server-side code that routes Claude's tool call to the correct Python function and returns the result. |
| `get_current_datetime` | Tool function that returns the current date/time formatted by a caller-supplied `date_format` string. |
| `add_duration_to_datetime` | Tool function that adds a duration to a datetime, handling the unreliable date arithmetic Claude does natively. |
| `set_reminder` | Tool function that persists a reminder in the system; bridges Claude's lack of a built-in reminder mechanism. |
| Input validation (tools) | Checking tool parameters for empty or invalid values and raising descriptive errors before doing any work. |
| Tool schema | JSON Schema object with `name`, `description`, and `input_schema` that tells Claude when and how to call a tool. |
| `input_schema` | The JSON Schema sub-object inside a tool schema that describes the function's parameters, types, and which are required. |
| `ToolParam` | Anthropic SDK type that wraps a tool schema dict to enable static type checking. |
| `required` (schema) | JSON Schema field listing parameter names Claude must always supply; empty list `[]` means all params are optional. |
| Schema naming convention | Pair `function_name` with `function_name_schema` in the same file to keep function and schema co-located. |
| Multi-block message | An assistant response whose `content` list contains more than one block (e.g. a text block + a tool-use block). |
| Text block | Content block with `type="text"` carrying human-readable explanation from Claude. |
| Tool-use block | Content block with `type="tool_use"` carrying `id`, `name`, and `input` dict telling your server which function to call and with what arguments. |
| `tool_use` block `id` | Unique string identifying this specific tool call; must be echoed back in the `tool_result` message. |
| Tool result message | A `{"role": "user", "content": [{"type": "tool_result", "tool_use_id": ..., "content": ...}]}` message that returns the function output to Claude. |
| `tool_result` block | Individual entry in a tool result message with `type`, `tool_use_id`, `content` (string), and `is_error` fields. |
| `is_error` | Boolean field on a `tool_result` block; `True` signals execution failed so Claude can adapt. |
| `**tool_input` unpacking | Python syntax for calling a function with a dict of keyword arguments matching its parameter names. |
| `stop_reason = "tool_use"` | Signal that Claude wants to call a tool rather than give a final answer; the loop should continue. |
| `run_conversation` | Convention name for the while-loop function that drives multi-turn tool interactions until `stop_reason != "tool_use"`. |
| `run_tools` | Convention name for the function that iterates over all tool-use blocks in a response and returns a list of `tool_result` blocks. |
| `run_tool` | Convention name for the dispatcher that maps a `tool_name` string to the correct Python function call. |
| `text_from_message` | Utility function that extracts and joins all `text`-typed blocks from a multi-block message into a plain string. |

## Examples / demos

_Code snippets and worked examples from the course._

### First request (Python SDK)

```python
from dotenv import load_dotenv
load_dotenv()

from anthropic import Anthropic

client = Anthropic()
model = "claude-sonnet-4-0"

message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "What is quantum computing? Answer in one sentence"}
    ],
)

print(message.content[0].text)
```

## Questions / to revisit

_Things unclear, or worth drilling before the exam._

-

## Exam-relevant takeaways

_Points flagged as likely to appear on the final assessment._

- **Five-step request flow:** client → your server → Anthropic API → model processing → response back down the chain.
- **Four processing stages:** tokenise → embed → contextualise → generate.
- **Three stop conditions:** `max_tokens` hit, end-of-sequence token, matched stop sequence.
- **Four required request fields:** `api_key`, `model`, `messages`, `max_tokens`.
- **Security rule:** API key lives on the server, never in client code.
- **Three required `create()` parameters:** `model`, `max_tokens`, `messages`.
- **`max_tokens` is a cap, not a target.**
- **Message shape:** `{"role": "user" | "assistant", "content": ...}`.
- **Response text access:** `message.content[0].text` (content is a list of blocks).
- **Multi-turn = you replay the whole history.** Append assistant replies back into `messages` before the next user turn; the API remembers nothing on its own.
- **System prompt = separate `system` kwarg**, not a `messages` entry, not `role="system"`.
- **Conditionally include `system`** in the request; the API rejects `system=None`.
- **Same user message + different system prompt → different behaviour.** The system prompt is the primary lever for controlling persona and answer style.
- **Temperature ranges:** `0.0–0.3` factual / code / extraction; `0.4–0.7` summarisation / education / constrained creative; `0.8–1.0` brainstorming / open creative.
- **Temperature default in your helper is a design choice.** `temperature=1.0` is Anthropic's default when unset; pin lower for repeatability.
- **Six stream event types:** `message_start` → `content_block_start` → `content_block_delta` (text!) → `content_block_stop` → `message_delta` → `message_stop`.
- **Two ways to stream:** raw (`create(..., stream=True)`) or simplified (`with client.messages.stream(...) as stream: stream.text_stream`).
- **After streaming, get the final message** with `stream.get_final_message()` to persist or extend the conversation.
- **Same request, one call.** All stream events belong to a single API request, not multiple calls.
- **Structured-output recipe:** prefill the opener as an `assistant` message + set `stop_sequences=[closer]`.
- **`stop_sequences` is a list**, not a string — even for a single sequence, wrap it: `["```"]`.
- **Prefill is a real assistant turn.** Include it in your `messages` list before calling; Claude sees it as the start of its own reply.
- **Detect stop-sequence termination** via `stop_reason == "stop_sequence"` + inspect `stop_sequence` on the response.
- **Prefill != system prompt.** Both steer behaviour, but prefill controls the *literal opening tokens* of the reply; the system prompt sets the persona.
- **Prompt engineering vs prompt evaluation is the split.** Engineering = how to write; evaluation = how well it works.
- **Three post-draft paths:** ship after one test (risky), ad-hoc patching (still risky), or evaluation pipeline (production-grade).
- **Evals give coverage + objective metrics + regression detection + fast iteration.** Manual testing gives none of those.
- **Rule of thumb:** treat a prompt like code — no untested prompt goes to production.
- **Memorise the five-step workflow:** draft → dataset → Claude → grader → iterate.
- **Two grader families:** code-based (deterministic, cheap) and model-based (subjective quality, needs rubric).
- **Freeze the dataset**, change one variable per run, compare aggregate scores.
- **Grader temperature = 0** so the same output scores the same each time.
- **Baseline first, then iterate.** Every prompt change is judged against the baseline aggregate.
- **Three-function pipeline:** `run_prompt` → `run_test_case` → `run_eval`. Know what each one owns.
- **Result shape:** `{output, test_case, score}` — all three fields, every case, every run.
- **Sequential is fine first.** Parallelism is a later optimisation; get graders working before you speed the loop up.
- **Three grader families:** code, model, human. Know when each fits.
- **Model-grader rubric must request strengths / weaknesses / reasoning + score.** Score-only rubrics converge on 6.
- **Grader `temperature=0`.** Non-negotiable for run-to-run comparability.
- **Map criteria to graders first, code second.** Format/syntax → code grader; task following / quality → model grader.
- **Log the reasoning.** A score without justification is unfalsifiable and useless for debugging regressions.
- **Three syntax validators to know:** `json.loads` → JSON, `ast.parse` → Python, `re.compile` → regex. Wrap each in try/except; success = 10, failure = 0.
- **First line is the most important line in any prompt.** Get it right before tuning anything else.
- **Three-part opening formula:** action verb + output object + constraints (e.g. `Generate / a one-day meal plan / for an athlete meeting their dietary restrictions`).
- **Instructions, not questions.** Rewrite interrogative prompts as direct commands.
- **XML tag names should be descriptive.** `<athlete_information>` beats `<data>`; `<my_code>` / `<docs>` beats `<text1>` / `<text2>`.
- **XML tags are most valuable when:** prompts are long, content types are mixed, or multiple variables are interpolated.
- **Zero-shot / one-shot / multi-shot.** Know the names; choose multi-shot when corner cases or varied edge formats matter.
- **Always wrap examples in XML tags** — `<sample_input>` / `<ideal_output>` — so Claude can parse them unambiguously.
- **Annotate ideal outputs.** Add a sentence explaining *why* the output is ideal; Claude learns reasoning not just format.
- **Best source for examples: your eval results.** Top-scoring outputs (score = 10) are ready-made few-shot examples.
- **Tool use four-step flow:** initial request → tool request → data retrieval → final response.
- **Each tool bridges one specific gap.** Decompose limitations into discrete, single-purpose tools.
- **Tool functions are defined and executed by you.** Claude only requests the call with name + arguments.
- **Validate inputs; raise descriptive errors.** Claude reads the error message and may self-correct on retry.
- **Four implementation steps per tool:** (1) write the Python function, (2) write its JSON schema, (3) register the schema in the `tools` list, (4) add a dispatcher that routes the call and returns the result.
- **Tool schema has three required fields:** `name`, `description`, `input_schema`. All three must be present.
- **Description does the heavy lifting.** Claude reads it to decide *when* to call the tool — write 3–4 sentences covering what it does, when to use it, and what it returns.
- **`ToolParam` adds type safety.** Not required for function but prevents silent type errors when passing the schema to the API.
- **Generate schemas with Claude.** Paste your function + ask for a JSON schema following Anthropic tool-use best practices; edit descriptions as needed.
- **Naming convention:** `function_name` + `function_name_schema` keeps function and schema paired.
- **Tools enabled = multi-block responses.** `response.content` is a list; expect `TextBlock` + `ToolUseBlock` when Claude decides to call a tool.
- **Append `response.content`, not just the text.** Dropping the tool-use block from history breaks Claude's context on the next turn.
- **Five-step tool flow:** send message + schemas → receive text + tool-use block → execute function → send tool result → receive final response.
- **Tool-use block carries three key fields:** `id` (for the result message), `name` (function to call), `input` (arguments dict).
- **`add_assistant_message` must be updated** to accept a content list, not just a plain string, when tool responses are in play.
- **`tool_result` block requires `tool_use_id`** matching the requesting block's `id` — mismatched IDs break the conversation.
- **Serialise tool output to string** before putting it in `content`; use `json.dumps` for structured data.
- **Always send a `tool_result` even on error.** Set `is_error: True` and put the error message in `content`; never leave a tool call unanswered.
- **Include tool schemas on every follow-up request.** Claude needs them to interpret tool references already in history.
- **`stop_reason == "tool_use"` → loop; anything else → break.** This is the single condition controlling the conversation loop.
- **`run_tool` is a name → function dispatcher.** One `if/elif` chain; add a branch for each new tool.
- **Four steps to add a new tool:** write function → write schema → add to `tools=[]` list → add `elif` in `run_tool`.
- **`chat` should return the full `Message`, not just `content[0].text`**, once tools are in play.
- **`text_from_message` extracts readable text** from a multi-block response without losing the block structure.
- **Dataset needs a `format` field** for code grading to know which validator to run.
- **Composite score = mean of model + syntax scores** by default; weight axes when one matters more. Persist sub-scores separately.
- **Prompt tightening for code output:** explicit format instructions in the user turn + assistant prefill of ` ```code ` + `stop_sequences=["```"]`.

