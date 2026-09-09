# Python Projects: The Standard Setup

**Purpose:** A reusable mental model + recipe for spinning up any small Python project — especially LLM-API-based ones — with clean isolation, safe secret handling, and reproducibility across machines. Written for a reader with no software-dev background.
**Last Updated:** 2026-08-05
**Written from:** [claude-api-lab](../../../claude-api-lab/) (Build with Claude API course)
**Related:** [sdks-explained.md](sdks-explained.md) — the layer above (how the API client itself works).

---

## 1. The one-sentence template

> **An isolated Python project that talks to a hosted API, keeps its secrets safe, and can be rebuilt on any machine from a recipe.**

Every word in that sentence is a concept:

| Word | What it forces you to do |
|---|---|
| **Isolated** | Use a virtual environment so this project's packages can't collide with another project's. |
| **Talks to a hosted API** | Install an SDK (or use raw HTTP) to send authenticated requests. |
| **Keeps its secrets safe** | Store the API key in `.env`, never in code, never in git. |
| **Rebuilt on any machine** | Commit a dependency list (`requirements.txt` or `pyproject.toml`). |

Everything below is how to satisfy each of those in the standard way.

---

## 2. The standard folder shape

Any small Python project of this kind will look like this. Learn the shape once — it transfers.

```
my-project/
├── .venv/                  ← private Python interpreter + packages (gitignored)
├── .env                    ← real secrets (gitignored)
├── .env.example            ← template showing which secrets are needed (committed)
├── .gitignore              ← tells git what to skip
├── README.md               ← what this is + how to run it
├── requirements.txt        ← the dependency list (committed)
└── main.py  or  01-*.ipynb ← your actual code
```

Two categories of files:

- **Committed** → `.env.example`, `.gitignore`, `README.md`, `requirements.txt`, your code. These are the "recipe" — small, portable, safe to share.
- **Gitignored** → `.venv/`, `.env`, `__pycache__/`, `.ipynb_checkpoints/`. These are large, machine-specific, or secret. Each machine builds its own.

**Rule of thumb:** if a file was *generated* (a venv, a cache) or is *secret*, it's gitignored. If it's *source* or *recipe*, it's committed.

---

## 3. The three-layer dependency hierarchy

Every Python project sits on three layers. Understanding this stops most beginner confusion.

| Layer | Thing | Example |
|---|---|---|
| 1. **Interpreter** | The `python.exe` that runs code. | System Python 3.14 at `C:\Program Files\Python314\python.exe` |
| 2. **Environment** | An isolated copy of the interpreter + a private `site-packages/` folder for installed packages. | `.venv\` inside your project |
| 3. **Packages** | Libraries installed via `pip` into that environment's `site-packages/`. | `anthropic`, `python-dotenv`, `httpx`, ... |

**Concepts to name:**

- **Isolation.** Every project gets its own environment. When you `pip install` inside project A's venv, project B is untouched. Version conflicts between projects become impossible.
- **Reproducibility.** The environment can be **rebuilt** from a text file. That's what `requirements.txt` is for. "Works on my machine" becomes "works on any machine that reads the recipe."
- **Transitive dependencies.** You install `anthropic`; pip pulls in `httpx`, `pydantic`, `certifi`, and a dozen others because `anthropic` needs them. `pip freeze` captures the whole tree so a rebuild matches exactly.

**Windows PowerShell recipe:**

```powershell
# Create the venv (one-time, inside the project folder)
py -3.14 -m venv .venv

# Activate it (per terminal session)
.\.venv\Scripts\activate

# Install packages
pip install anthropic python-dotenv
```

Once activated, `python` and `pip` in that terminal refer to the venv's copies, not the system's.

---

## 4. The secrets pattern (`.env`)

**Rule:** secrets never live in source code. They live in **environment variables**, and a small library reads them at runtime.

### Why

- If a secret is in code, it's in git history, forever, even if you delete it.
- Bots scan public GitHub for leaked keys within minutes and drain accounts.
- Even in private repos, secrets in code make code reviews and screen-shares dangerous.

### The pattern

Two files, same variable names, one committed and one not:

```env
# .env.example  (committed — placeholder values, safe to share)
ANTHROPIC_API_KEY="your-api-key-here"
```

```env
# .env  (gitignored — real values, private to your machine)
ANTHROPIC_API_KEY="sk-ant-api03-…"
```

### How the code reads it

```python
from dotenv import load_dotenv
load_dotenv()          # reads .env → puts each KEY=value into os.environ

from anthropic import Anthropic
client = Anthropic()   # SDK reads ANTHROPIC_API_KEY from os.environ automatically
```

The SDK never sees your `.env` file directly. The bridge is `os.environ`. `load_dotenv()` fills it; the SDK reads from it.

### Rules

- **Same variable names in both files.** If you add a new secret, add a placeholder line to `.env.example` too.
- **Never commit `.env`.** Always in `.gitignore`.
- **Never `print()` a secret.** Even by accident. Notebook output cells are especially dangerous — they save the output into the `.ipynb` file, which can end up in git.
- **Rotate keys** if they leak. Every provider has a revoke-and-recreate flow.

### The bigger principle

This is a Python implementation of a well-known architecture principle: **config in the environment**, not the codebase. It comes from the [12-Factor App](https://12factor.net/config) methodology and applies to any language, not just Python. Once you internalise it, secrets-in-code will feel wrong forever.

---

## 5. Reproducibility across machines

Two levels of tooling. Start with the simple one; graduate when the project grows.

### Level 1 — `requirements.txt` (basic, universal)

After you've installed everything you need in the venv:

```powershell
pip freeze > requirements.txt
```

That produces a pinned list of every installed package:

```
anthropic==0.120.2
httpx==0.28.1
pydantic==2.13.4
python-dotenv==1.2.2
…
```

**On another machine** (clone the repo, or copy the folder without `.venv` / `.env`):

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
Copy-Item .env.example .env
# open .env, paste your key, save
```

Byte-for-byte the same package set. Works everywhere Python does.

### Level 2 — `pyproject.toml` + `uv` or `poetry` (modern)

For anything bigger than a couple of files, prefer:

- **`pyproject.toml`** — a standard file that declares your *direct* dependencies (not the transitives).
- **A lock file** (`uv.lock` or `poetry.lock`) — auto-generated, pins the entire resolved tree.
- **A CLI tool** (`uv` or `poetry`) — reads both, sets up the venv, installs everything with one command.

This is what most professional Python projects look like in 2026. `requirements.txt` is fine while you're learning; graduate when you feel the pain of managing dependencies by hand.

---

## 6. Talking to APIs (with or without an SDK)

Almost every project of this kind ends up calling a **hosted HTTP API**. The shape is always the same:

```
your code → build request → auth → send → receive → parse → use
```

An **SDK** (like `anthropic`) is a Python library that wraps that whole flow so you can write:

```python
client.messages.create(model=…, messages=…)
```

instead of manually building JSON, setting `x-api-key` headers, and parsing the response.

Without an SDK, the same call is a raw HTTP request (`httpx` or `requests`) with the auth header set by hand. Every SDK is optional convenience — the API underneath is what actually exists.

**For deeper detail, cross-reference:** [sdks-explained.md](sdks-explained.md) — SDK vs API vs library vs framework, plus how to read SDK code.

---

## 7. Notebook vs script — pick your workflow

Same Python, two workflows.

| `.ipynb` (notebook) | `.py` (script) |
|---|---|
| Run one cell at a time; state persists in memory between cells. | Run top-to-bottom, exit when done. |
| Output rendered inline (text, tables, plots). | Output to stdout, files, or the terminal. |
| Great for **exploration**, teaching, one-off analysis. | Great for **automation**, deployment, anything scheduled. |
| Hard to code-review (JSON under the hood, cells can be re-run out of order). | Easy to code-review, diff, and test. |

**Rule of thumb:** start in a notebook while you're figuring out *what* the code should do. When you know, port the useful bits into a `.py` script or module.

---

## 8. The LLM-project variant

Everything above is the *base*. For projects that call an LLM API (Anthropic, OpenAI, etc.), you'll want to layer on a few more disciplines as the project grows:

| Concern | Why it matters | Typical shape |
|---|---|---|
| **Prompt management** | Prompts are code. Editing them in the middle of an app file leads to lost history and copy-paste drift. | Prompts stored as separate files (`.txt`, `.md`, or `.yaml`), versioned in git. |
| **Evaluation ("evals")** | You can't tell if a prompt change is *better* without measuring. | A dataset of test inputs + expected behaviour, scored by code or by another LLM. |
| **Cost + usage tracking** | Every call costs money. `usage` metadata (input/output tokens) is right there in the response. | Log `usage` per call; roll it up per prompt / user / day. |
| **Guardrails / safety** | Untrusted user input can hijack prompts (prompt injection); model output can leak or misbehave. | Input validation before the call; output filtering after; system prompts that constrain behaviour. |
| **Retries + rate limits** | LLM APIs return 429s under load and 5xxs occasionally. | SDK usually handles some; wrap with `tenacity` or similar for the rest. |
| **Caching** | Repeated identical calls waste money and time. | Anthropic offers prompt caching; you can also cache locally by prompt hash. |

You don't need any of this on day one. But the **base pattern in §2–§6 is the ground floor** every LLM project stands on. Once the base is clean, adding these concerns is straightforward.

---

## 9. Starter checklist (Windows PowerShell)

Copy-paste-adapt to spin up any new Python-plus-API project:

```powershell
# 1. Create the folder and enter it
mkdir my-new-project
cd my-new-project

# 2. Create the isolated environment
py -3.14 -m venv .venv
.\.venv\Scripts\activate

# 3. Install the packages you need
pip install <package-a> <package-b> python-dotenv

# 4. Freeze the dependency list
pip freeze > requirements.txt

# 5. Create the safety files
@"
.venv/
.env
__pycache__/
*.py[cod]
.ipynb_checkpoints/
"@ | Out-File .gitignore -Encoding utf8

@'
API_KEY="your-api-key-here"
'@ | Out-File .env.example -Encoding utf8

Copy-Item .env.example .env
# open .env in editor, paste real key, save

# 6. Initialise git
git init
git add .
git commit -m "Initial project scaffold"
```

Six steps, five minutes, works for essentially any small Python project.

---

## 10. Reading someone else's Python project in 30 seconds

When you land on any Python repo — a course exercise, a GitHub project, a colleague's code — scan for these in order:

1. **`README`** — what is this thing? Should tell you in the first two paragraphs.
2. **`requirements.txt` or `pyproject.toml`** — what does it depend on? Reveals the tech stack instantly.
3. **`.env.example`** — what secrets does it need? Tells you which external services it calls.
4. **Entry point** — `main.py`, `app.py`, `cli.py`, or a notebook. Where execution starts.

If all four exist and are readable, the project is well-run. If any are missing, you're about to have a longer afternoon than you planned.

---

## The one takeaway

**Python programming is maybe 30% writing Python. The other 70% is managing environments, dependencies, secrets, and version control.** The pattern in this document is a small, complete example of the 70%. Every future project you build will reuse this same shape with different code inside.

Get comfortable with the shape first. The code inside gets easier once the scaffolding is second nature.
