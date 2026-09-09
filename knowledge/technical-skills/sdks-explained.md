# How SDKs Work

**Purpose:** Conceptual reference for understanding what an SDK is, how it relates to APIs, libraries, and frameworks, and how to read/adapt SDK code with no software-dev background. Uses the Azure AI Foundry Python stack as a worked example.
**Last Updated:** 2026-07-14

---

## 1. What an SDK Is

**SDK = Software Development Kit.** It's a bundle of pre-written code someone else built so you don't have to hand-build the fiddly, repetitive parts of talking to a service.

**Analogy:** Sending a parcel overseas. You *could* research customs forms, weigh the package to the gram, calculate duties, and negotiate with three shipping carriers yourself. Or you drop it at a courier counter and say "this goes to Tokyo" — the courier handles the paperwork, the customs codes, the carrier negotiation, and hands you a tracking number. The SDK is the courier counter. The "shipping the parcel" part (the actual work) still happens — you're just not the one filling out customs forms.

Concretely, an SDK is a **package of code** (functions, classes, helper objects) that:
- handles authentication (proving who you are to the service)
- builds the technical request in the exact format the service expects
- sends it
- unpacks the response into a shape that's easy to use in your code
- handles retries, errors, and edge cases you'd otherwise have to discover the hard way

You call a few functions with plain arguments; the SDK does the plumbing underneath.

---

## 2. SDK vs API vs Library vs Framework

These four words get used loosely and interchangeably, which is where most confusion starts. Here's the crisp version:

| Term | What it is | Analogy |
|---|---|---|
| **API** (Application Programming Interface) | A *contract* — the set of things you're allowed to ask for, and what you get back. Not code you run; a rulebook. | A restaurant menu — a list of what you can order and what arrives |
| **Library / package** | A chunk of reusable code you install and call into. You stay in control of the flow; the library just does jobs for you when asked. | A toolbox — you pick up the tool, use it, put it down |
| **SDK** | A library (or bundle of libraries) built specifically to talk to *one particular service or platform*, including the auth, request-building, and response-parsing for that service. | The courier counter — a toolbox purpose-built for "get things to this destination" |
| **Framework** | Code that's in control of the overall flow — you fill in pieces, and *it* calls *you* (rather than you calling it). | A recipe template — it dictates the steps; you supply the ingredients at each step |

**The important nuance: "API" means two different things, and both are true at once.**

1. **The web API** — the actual rulebook the *service itself* exposes over the internet (usually a "REST API," covered in §3). This exists whether or not an SDK exists for it.
2. **The SDK's own API** — the set of functions, classes, and arguments the SDK lets *you* call in your code. When someone says "check the SDK's API reference," they mean this — the list of callable functions, not the web request format.

So an SDK is best understood as: *a library, purpose-built to be the friendly front door to one service's web API.* It is not a framework — it doesn't take control of your program's overall flow; you still call it, on your terms, whenever you want.

---

## 3. The Key Mental Model — An SDK Is a Wrapper Over a Raw REST API

This is the single idea that makes everything else click.

Most cloud services (OpenAI, Azure, Stripe, GitHub, etc.) expose their functionality as a **REST API** — a set of URLs you send structured web requests to, formatted as **JSON** (a plain-text way of writing structured data, like `{"message": "hello"}`), over **HTTP** (the same protocol your browser uses to load web pages).

**Both the raw REST API and the SDK talk to the exact same service.** The only difference is who does the plumbing.

**Doing it raw (no SDK):**
1. Build the correct URL for the operation you want
2. Attach an authentication token in the right header, formatted exactly right
3. Hand-write the JSON body with the correct field names and structure
4. Send the HTTP request yourself
5. Parse the raw JSON response back into something usable
6. Handle errors, retries, and rate limits yourself

**Doing it with an SDK:**
1. Call a function with plain arguments, e.g. `client.send_message("hello")`
2. Read the result the SDK hands back as a normal object in your programming language

Same destination, same underlying service call — the SDK just did steps 1–6 above for you, invisibly, inside that one function call. This is why SDK documentation and REST API documentation for the same service often describe the *exact same operations* — they're two views of one thing.

---

## 4. How You Actually Use One in Practice

Using an SDK is a modest, repeatable ritual — not a big engineering feat. In Python, it's three steps:

1. **Install the package** — a one-line terminal command that downloads the SDK's code onto your machine, e.g.:
   ```
   pip install azure-ai-projects
   ```
   (`pip` is Python's package installer — think "app store," but for code libraries.)

2. **Import it** — tell your script "make this package's functions available here":
   ```python
   from azure.ai.projects import AIProjectClient
   ```

3. **Call its functions** — use the objects and functions the SDK gives you, following its documented API (§2's second meaning of "API"):
   ```python
   client = AIProjectClient(...)
   response = client.some_function(...)
   ```

Then you **run** the program (execute the script), and the SDK's code fires the real network requests behind the scenes.

Every mainstream language has its own SDKs for a given service — Python, JavaScript/TypeScript, Java, C#, Go. **The concepts are identical across all of them**; only the syntax (how you write `install`, `import`, and `call`) differs by language. Once you understand one language's SDK pattern, reading another language's SDK code is a translation exercise, not a new concept to learn.

---

## 5. Worked Example — The Azure AI Foundry Stack

This is the situation you're actually in day-to-day, so it's worth walking through concretely. Sending one message to an AI model through Azure AI Foundry uses **three Python SDKs stacked on top of each other**, each responsible for one layer of the problem.

| SDK package | What it's for | Key object/call |
|---|---|---|
| `azure-identity` | Proves who you are to Azure — handles login and token refresh so you never touch raw credentials | `DefaultAzureCredential()` |
| `azure-ai-projects` | Connects to your specific AI Foundry *project* (the workspace containing your models, data, and config) | `AIProjectClient` |
| OpenAI-compatible client | Sends the actual chat message and streams back the model's reply | `responses.create(...)` |

**Why three, not one?** Each SDK owns a distinct job: *proving who you are* (identity) is a different concern from *finding which project you're working in* (projects) which is different again from *actually chatting with a model* (the OpenAI-compatible client). Chaining them means: authenticate → locate the project → talk to the model.

**The interesting fact:** the third layer speaks "OpenAI's language" — the same function names and message format OpenAI's own API uses (`responses.create`, chat-style message objects) — *even when the model answering you is not an OpenAI model at all* (e.g., a Claude model hosted in Foundry). Microsoft deliberately made Foundry's chat interface compatible with the OpenAI SDK's shape, so the same client code can point at many different underlying models without you rewriting your call pattern each time. The SDK's shape and the model's identity are two separate things.

**The stack, visually:**

```
Your code
   |  client.responses.create(...)
   v
+--------------------------------------------------+
|  OpenAI-compatible client   (send message, get reply)  |
+--------------------------------------------------+
|  azure-ai-projects: AIProjectClient (find the project)  |
+--------------------------------------------------+
|  azure-identity: DefaultAzureCredential (prove who you are) |
+--------------------------------------------------+
   |  hidden raw HTTP/JSON request, built + sent for you
   v
Azure AI Foundry service  (routes to the actual model, e.g. Claude)
```

Everything below the dashed boxes — the raw REST request, the JSON body, the auth header — is exactly what §3 described. You never touch it; the three SDKs jointly build and send it, then unpack the response into a plain object your code can read.

**One clarifying contrast:** a *web framework* like FastAPI or Flask is a different kind of toolkit — it's for **receiving** requests (e.g., building your own API that other people or systems call *into*), not for calling out to someone else's service. It fits the "framework" row in §2's table — it takes control of the incoming-request flow and calls your code when a request arrives. Don't confuse it with an SDK; they solve opposite directions of the same problem (serving vs. consuming).

---

## 6. How to Learn This — A Realistic Path

You do not need to become a software engineer to use SDKs competently. You need two things:

1. **Enough of the host language to read and adapt a working example.** For Python, that's a small, learnable set: variables, function calls, `import` statements, and basic control flow (if/for). You're not writing new logic from scratch — you're recognizing the pattern in an SDK's example code and swapping in your own values (a message string, a project name, a model name).

2. **The SDK's documented function names and arguments** — its "API" in the second sense from §2. Every well-maintained SDK ships a reference (a page listing each callable function, what arguments it takes, and what it returns) and a "quickstart" example. Learning an SDK is mostly: find the quickstart for the exact task you want, run it as-is to confirm it works, then change one thing at a time until it does what you need.

**Practical order:**
- Get a minimal working example running unmodified first (proves your install/auth is correct)
- Change one small thing (the message text, a parameter) and re-run
- When something breaks, read the error message literally — it usually names the exact function and argument at fault
- Keep a personal "known-good" snippet once something works, so you're never starting from zero next time

This mirrors how the Azure Foundry stack above should be approached: get all three SDKs authenticating and returning a reply with the sample code untouched, *then* start adapting the message content and model choice to your actual task.
