# AI Fluency: Framework & Foundations — Notes

Part of: AI Fluency (Anthropic)
Owner: agregorio1@kpmg.com.au · Started: 2026-07-20

Cross-reference: 4D Framework was introduced in [claude-101-notes.md](claude-101-notes.md#the-4d-framework-for-ai-fluency).

---

## The 4D Framework (course anchor)

This entire course is organised around the **4D Framework for AI Fluency**, developed through a research collaboration between Professor Rick Dakan (Ringling College of Art and Design) and Professor Joseph Feller (University College Cork).

Four core competencies that, when combined, help you make the most of AI interactions:

- **Delegation** — Deciding what work should be done by humans, what by AI, and how to distribute tasks between them. Includes understanding your goals, AI capabilities, and making strategic choices about collaboration.
- **Description** — Effectively communicating with AI systems. Includes clearly defining outputs, guiding AI processes, and specifying desired AI behaviors and interactions.
- **Discernment** — Thoughtfully and critically evaluating AI outputs, processes, behaviors and interactions. Includes assessing quality, accuracy, appropriateness, and determining areas for improvement.
- **Diligence** — Using AI responsibly and ethically. Includes making thoughtful choices about AI systems and interactions, maintaining transparency, and taking accountability for AI-assisted work.

> **Use this section as the reference anchor for the rest of the course.** As new material comes in, it should slot under (or reference) one of the four D's.

---

## Lesson overview

_What this lesson covers, at a glance._

- **The 4D Framework** as the organising spine of the course (Delegation, Description, Discernment, Diligence).
- **The three ways of working with AI** — Automation, Augmentation, Agency — as the core taxonomy under **Delegation**.
- **Generative AI foundations** — the tech journey (algorithms → data → compute), scaling laws, training pipeline (pre-training → fine-tuning → RLHF), prompting, context windows, and core LLM capabilities.
- **Description** — the second D. Communicating with AI to create a productive collaborative environment. Three components: **Product** (what to create), **Process** (how to approach it), **Performance** (how to behave).
- **Foundational prompting techniques** — six practical techniques under Description: give context, show examples, specify constraints, break into steps, ask the AI to think first, define role/tone. Plus troubleshooting and the "ask the AI to improve your prompt" move.
- **Discernment** — the third D. Thoughtfully evaluating what AI produces, how it produced it, and how it behaved. Three components mirror Description: **Product**, **Process**, **Performance** Discernment.
- **Diligence** — the fourth D. Taking responsibility for AI collaborations — the ethical and safety layer. Three components: **Creation** (which AI and how), **Transparency** (disclosing AI's role), **Deployment** (owning what you share).

## Key concepts

_Core ideas, definitions, and mental models._

### Delegation — the four sub-skills

Delegation is the first D of the 4D Framework. It's the meta-skill of deciding *what to hand to the AI, what to keep, and how to shape the handoff.* Four sub-skills compose it:

#### 1. Understand the problem
Before choosing a tool or writing a prompt, get clear on what you're actually trying to solve.
- What is the **desired outcome**? (Deliverable, decision, insight?)
- What are the **constraints**? (Audience, format, deadline, sensitivity, quality bar.)
- What are the **success criteria** — how will you know the output is good?
- What **context** does the problem sit inside? (What has already been tried, what is off-limits, what background info is essential.)

Rule of thumb: if you can't articulate the problem to a smart colleague in two sentences, you can't delegate it to an AI yet.

#### 2. Understand the capabilities of available AI tools
Delegation is only useful if you know what your tools can and can't do. This is a *matching* skill.
- **What the model is strong at** — pattern recognition, summarisation, drafting, code, reasoning over provided context, structured transformation.
- **What the model is weak at** — arithmetic without tools, current events beyond training cut-off, guaranteed factuality, proprietary/private knowledge it was never given, self-verification.
- **What the surrounding *system* adds** — tool use, retrieval, code execution, Projects/Skills, memory, file handling. A model with tools can do things a bare model cannot.
- **Which model / which mode** — bigger frontier model vs faster smaller one, chat vs agentic loop, single-shot vs iterative. Different jobs justify different choices.

Practical form: "For *this* problem, given *these* tools, what's realistic to expect?"

#### 3. Break complex work into smaller parts
LLMs perform dramatically better on well-scoped sub-tasks than on one giant ambiguous ask. Decomposition is a Delegation skill because *how you cut the problem determines what you can delegate.*
- **Sequence** — identify natural steps (research → outline → draft → critique → revise).
- **Isolate dependencies** — which steps need output from earlier steps? Which can run in parallel?
- **Right-size each sub-task** — small enough that the model can do it in one focused pass, big enough that you're not micromanaging tokens.
- **Decide the human/AI split per sub-task** — the same overall project can have human-only steps (judgement calls, stakeholder conversations), AI-only steps (bulk transformation, first-draft generation), and collaborative steps (review, refinement).

This is the bridge between "understanding the problem" and "choosing the mode" — decomposition is what makes mode selection possible.

#### 4. Choose the right mode
Given the problem, the tools, and the decomposition, pick the collaboration mode for each piece: **Automation, Augmentation, or Agency** (defined in the section below).
- **Automation** when the task is well-defined and repeatable — you know exactly what "done" looks like.
- **Augmentation** when the problem is exploratory, judgement-heavy, or benefits from real-time back-and-forth.
- **Agency** when the work is ongoing, patterned, and worth the up-front cost of configuring an actor (knowledge, behaviour, boundaries) that will run repeatedly.

Different sub-tasks within one project can and should use different modes.

> **The four sub-skills work as a pipeline:** understand the problem → understand the tools → break it down → choose the mode(s). Skipping steps is the most common source of bad delegation.

### The three ways of working with AI (Delegation)

Three identified main ways that people work with AI. This is a **Delegation** topic — it's about *how you distribute work between yourself and the model*.

- **Automation** — The AI completes specific tasks based on your instructions. *You define the task; the AI executes it.*
- **Augmentation** — You and AI collaborate as creative thinking and task execution partners. *You and the AI think and work together in real time.*
- **Agency** — You configure AI to work independently on your behalf, establishing its knowledge and behavior patterns rather than just giving it specific tasks. *You set the AI up as an actor; it operates on your behalf.*

Mental model of the spectrum:

| | You do... | The AI does... |
|---|---|---|
| **Automation** | Define the specific task | Execute the specific task |
| **Augmentation** | Think alongside the AI | Think alongside you |
| **Agency** | Configure the actor (knowledge, behaviour, boundaries) | Act independently on your behalf |

The progression moves from **task-level control** → **collaborative co-creation** → **entity-level configuration**. Each mode requires more up-front setup but delivers more leverage.

### Generative AI foundations

A foundational literacy topic — mainly informs **Delegation** (what these systems can/can't do) and **Discernment** (why they fail the way they do).

#### The technology journey

Modern generative AI stands on three converging pillars. Progress happens when all three advance together.

1. **Algorithms** — the model architectures.
   - **Neural networks** — layered computational graphs of "neurons" that learn to map inputs to outputs by adjusting weights during training. The general-purpose substrate.
   - **Transformers** (2017, "Attention is All You Need") — the specific architecture that unlocked modern LLMs. Uses a *self-attention* mechanism that lets every token in a sequence weigh its relationship to every other token in parallel. This replaced older sequential architectures (RNNs, LSTMs) and made it feasible to train on massive corpora.
2. **Data** — the training material.
   - **Articles and websites** — the text web, books, forums, reference material — gives the model language, world knowledge, and reasoning patterns.
   - **Code and multimodal content** — source code (structured reasoning), plus images, audio, and video for multimodal models. Code data in particular is credited with sharpening logical reasoning across the board.
3. **Computation** — the hardware that makes the above tractable.
   - **GPUs, TPUs, and specialised accelerators** — chips designed for the massive parallel matrix multiplications that neural networks require.
   - **Computing clusters** — thousands of accelerators networked together, letting a single training run span weeks or months across petabytes of data.

#### Scaling laws

Empirical observation: **the more compute and data you throw at a well-designed model, the more capability tends to emerge — often in unpredictable, step-change ways.** Scaling laws (Kaplan et al., Chinchilla, etc.) formalised the relationship between model size, dataset size, compute budget, and loss. The practical implication: past a certain threshold, you don't just get incrementally better completion — you get *new* abilities the model wasn't explicitly trained for (see emergent capabilities below).

#### The training pipeline

A modern LLM is produced in stages. Each stage shapes what the model can do and how it behaves.

- **Pre-training** — the foundation. The model is trained on enormous unlabelled text corpora with a simple objective: predict the next token. Doing this at scale, across trillions of tokens, forces the model to internalise grammar, facts, reasoning patterns, and the statistical structure of human knowledge. This is the most compute-expensive stage and produces a *base model* that is capable but not yet useful for conversation — it will happily continue a document rather than answer a question.
- **Fine-tuning** — targeted training on smaller, curated datasets to specialise the base model. Two flavours are common:
  - **Supervised fine-tuning (SFT)** — train on high-quality examples of the desired behaviour (e.g. "here's a good instruction and here's a good response"). Teaches the model to follow instructions and adopt useful formats.
  - **Domain fine-tuning** — adapt a general model to a specific domain (legal, medical, code) by continuing training on domain-specific text.
- **RLHF (Reinforcement Learning from Human Feedback)** — the alignment layer. Humans rank multiple model outputs for the same prompt; a *reward model* is trained to predict those human preferences; the LLM is then optimised (via reinforcement learning) to produce outputs the reward model scores highly. This is what makes a raw fine-tuned model *helpful, harmless, and honest* — it's the difference between "technically-answers-the-question" and "answers the question the way a thoughtful assistant would." Anthropic also pioneered a variant called **Constitutional AI (CAI)**, where an AI critiques and refines its own responses against a written set of principles, reducing the need for human labels at every step.

#### Prompting and generative output

Once trained, LLMs work off the **prompt** you provide. Given the prompt, the model generates output *one token at a time*, where each new token is sampled from a probability distribution shaped by:

- Everything the model learned during training.
- The full text of the prompt (and everything it has already generated so far).

A useful mental model: **the model is doing sophisticated autocomplete over the entire prompt + its own emerging response.** Prompts don't "instruct" the model like code — they *steer* the distribution over what comes next. This is why prompt phrasing, examples, and structure matter so much: they change the statistical landscape the model is sampling from.

#### Context windows

The **context window** is the total amount of text — measured in **tokens** (roughly, word-pieces) — that the model can "see" at once. It's the sum of your prompt, any attached documents, prior turns of the conversation, and the model's own generated response.

Key points:

- Anything **outside** the context window is invisible to the model — it does not remember it, cannot reason about it, and cannot cite it.
- Modern frontier models have context windows ranging from tens of thousands to **millions** of tokens (Claude's larger context windows are a differentiator).
- Bigger contexts unlock new use cases (long documents, entire codebases, multi-hour transcripts) but come with tradeoffs: cost, latency, and "lost in the middle" attention degradation where information buried mid-context gets under-weighted.
- Practically: context window is the model's **working memory** for a single interaction. Projects and Skills (from Claude 101) are ways to make useful context *reload automatically* rather than being carried in a single window forever.

#### Core capabilities of LLMs

Three capabilities that together define what makes modern LLMs different from previous NLP systems:

- **Processing vast amounts of training data and learning complex patterns.** Because pre-training exposes the model to a slice of human knowledge at unprecedented scale, LLMs internalise not just facts but the *shape* of how ideas connect — argument structures, analogies, reasoning chains.
- **Adapting to new tasks through in-context learning.** Give the model a few examples of a task inside the prompt ("few-shot prompting") and it can generalise to new instances of that task — without any retraining. The model is treating the examples as evidence about what pattern to continue.
- **Demonstrating emergent capabilities from scale.** Abilities like multi-step reasoning, code generation, translation, and tool use often appear *suddenly* as models cross size/data thresholds — they weren't explicitly programmed in. Emergence is the flip side of scaling laws: predicted in aggregate, unpredictable in specifics.

### Description — the three components

Description is the second D of the 4D Framework. It's the competency of **communicating with AI systems** — not just "writing prompts," but shaping the *collaborative environment* in which you and the model work together. AI can't read your mind; the quality of the output tracks the quality of your articulation.

Three components, each answering a different question:

#### 1. Product Description — *what* you want created

Clearly define the output itself. This is the component most people already do (often the only one).

- **Output** — the thing itself: an email, a memo, a plan, a diagram, a piece of code.
- **Format** — structure, layout, length, sections, file type.
- **Audience** — who it's for, their level, what they already know.
- **Style / tone** — formal, casual, technical, plain-English, brand voice.
- **Constraints** — what to include, what to avoid, boundaries on scope.
- **Examples of "good"** — showing the model what you're aiming at (few-shot).

#### 2. Process Description — *how* the AI should approach your request

Guide the model's *approach*, not just its target. Often as important as specifying the end goal — and the component most under-used.

- **Steps to follow** — outline first, then draft; research before writing; plan before executing.
- **Order of operations** — sequence and dependencies.
- **Plan-first vs act-first** — should the model propose an approach before doing the work?
- **Clarifying questions** — should it ask before assuming, or make its best guess and flag assumptions?
- **What to research vs assume** — where to look up vs where to rely on training.
- **Checkpoints** — when to stop and check in vs run through end-to-end.

#### 3. Performance Description — *how* the AI should behave during the collaboration

Define the AI's behavioural stance — the "personality" of the interaction.

- **Concise vs detailed** — one-liners vs full explanations.
- **Challenging vs supportive** — should it push back on your reasoning, or run with it?
- **Formal vs casual** — register of the conversation.
- **Deference vs assertion** — when to defer to your judgement, when to hold a position.
- **Proactivity** — surface risks/alternatives unprompted, or only when asked.

#### Mental model

- **Product** = the deliverable.
- **Process** = the method.
- **Performance** = the manner.

All three together create the **collaborative environment**. Product-only prompts get you output that technically matches the ask but often misses the point. Adding Process shapes *how* the model works the problem. Adding Performance shapes *what it's like* to work with.

> **AI systems are interactive partners, not databases or vending machines.** Databases return what you query; vending machines return what you paid for. Partners need to be told what you want, how you want it approached, and how you want them to show up.

### Foundational prompting techniques

**Prompt engineering** = designing effective instructions for AI systems. It's not a separate discipline from Description — it's the *practical craft* of doing Product/Process/Performance description well. Combines ordinary clear-communication principles with a handful of AI-specific moves.

Six foundational techniques:

#### 1. Give context

Be specific about **what** you want, **why** you want it, and the **background** the model needs to do a good job.

- What are you trying to achieve, and why?
- Who is the audience or downstream reader?
- What's already been tried or decided?
- What background — domain, constraints, prior conversations — does the model need?

Rule of thumb: any context you'd give a smart colleague picking up the task cold, the model needs too.

*Maps to: Product (goal, audience) + Process (why, what's already tried).*

#### 2. Show examples

Demonstrate the output style or format you want. "Show, don't tell."

- Paste one or more examples of the kind of output you're after.
- Include both format (structure, length, headings) and voice (tone, register).
- Few-shot examples steer the model far more reliably than describing the style in prose.

*Leverages in-context learning (from the GenAI foundations section) — the model treats examples as evidence of the pattern to continue.*

*Maps to: Product ("examples of good").*

#### 3. Specify constraints

Clearly define format, length, and other output requirements up-front.

- **Format** — bullets vs prose, table vs list, sections, headings, file type.
- **Length** — word/token budget, "one paragraph," "no more than five bullets."
- **Inclusions** — what must be present.
- **Exclusions** — what to avoid (jargon, marketing language, hedging).

Constraints reduce ambiguity and cut down on regeneration cycles.

*Maps to: Product (format, length, constraints).*

#### 4. Break complex tasks into steps

Guide the AI through multi-step reasoning by giving it the steps explicitly rather than asking for the final answer in one shot.

- Spell out the sequence: "First X, then Y, then Z."
- Useful when a single prompt is asking for research + analysis + drafting in one go.
- Related to the Delegation sub-skill of **task decomposition**, but applied *inside a single prompt* rather than across a project.

*Maps to: Process (steps to follow, order of operations).*

#### 5. Ask the AI to think first

Give the model space to work through its process before producing the final output.

- "Think through your approach before answering."
- "Outline your reasoning, then give the final response."
- "Plan the steps first, then execute."
- Especially useful for reasoning-heavy tasks — improves quality noticeably.

Complements extended thinking (from Claude 101) — the same "let the model plan before acting" principle, applied at prompt level.

*Maps to: Process (plan-first vs act-first).*

#### 6. Define the AI's role or tone

Specify how you want the AI to communicate — the "persona" of the interaction.

- Role: "Act as a senior editor / a sceptical reviewer / a plain-English translator."
- Tone: formal, casual, concise, encouraging, blunt.
- Stance: challenging vs supportive, deferential vs assertive.

*Maps to: Performance (behaviour, tone, stance).*

#### The "secret weapon" — ask the AI to improve your prompt

Meta-move: paste your prompt and ask the model to critique or rewrite it.

- "Here's my prompt. What's ambiguous? What's missing? Rewrite it to be clearer."
- Turns the model into a prompt-engineering collaborator.
- Especially useful before running expensive or agentic tasks where re-runs are costly.

#### Troubleshooting and iteration

- **Prompting is iterative** — expect to refine based on what comes back. First-pass prompts rarely produce final-quality output.
- **Prompting is collaborative with the AI** — refinement isn't just you tweaking in isolation; you can hand the failing prompt back and ask the model to diagnose it.
- If output is **too generic** → add context and examples.
- If output is **wrong shape** → tighten constraints (format, length, inclusions/exclusions).
- If output **skips steps or hand-waves** → break the task into explicit steps, or ask it to think first.
- If output has **wrong voice** → define role/tone.

#### Common patterns in successful prompts

- Clear **task overview** — what's being asked and why.
- **Format specification** — how the output should be structured.
- **Explicit constraints** — inclusions, exclusions, length.
- **Relevant background** — the context the model needs to do the job.

### Discernment — the three components

Discernment is the third D of the 4D Framework. It's the competency of **thoughtfully evaluating AI outputs, processes, and behaviours** — the human-judgement layer that keeps AI collaboration grounded.

**Discernment is the flip side of Description.** Description is how you tell the AI what you want; Discernment is how you evaluate what you got. Together they form a continuous feedback loop:

> Describe → generate → discern → re-describe → regenerate.

The three components of Discernment mirror the three components of Description one-for-one.

#### 1. Product Discernment — evaluating the *output*

Assess the quality of what the AI actually produced.

- **Accuracy** — are the facts, figures, citations, and claims correct? Does it hallucinate?
- **Appropriateness** — is it fit for the audience, context, and purpose?
- **Coherence** — does it hang together? Internal contradictions? Logical flow?
- **Relevance** — does it actually answer *your* question, or a nearby but different one?
- **Completeness** — did it cover what was asked, or drop parts silently?

Mirror of Product Description: you specified what you wanted; now check whether that's what you got.

#### 2. Process Discernment — evaluating *how it got there*

Assess the reasoning and approach the AI used to arrive at the output. Even a good-looking answer can be produced badly.

- **Logical errors** — flawed reasoning, invalid inferences, unsupported leaps.
- **Attention gaps** — did it miss part of your prompt, skip a constraint, or ignore attached context?
- **Inappropriate reasoning** — wrong framework applied, wrong assumptions used, motivated reasoning.
- **Shortcut behaviour** — did it skip steps you asked for, hand-wave over the hard part?
- **Traceability** — can you follow how it got from prompt to output?

Mirror of Process Description: you specified how it should approach the task; now check whether it did.

#### 3. Performance Discernment — evaluating *how it behaved*

Assess how the AI *showed up* during the interaction itself — the collaboration dynamics, not just the artefact.

- **Communication style effectiveness** — is its tone, register, and verbosity working for you?
- **Behavioural stance** — is it challenging when it should be, deferring when it should be?
- **Interaction quality** — does it ask when it should ask, act when it should act?
- **Sycophancy vs. calibrated pushback** — is it just agreeing, or engaging critically?
- **Fit to your working style** — over the course of the session, is this collaboration actually productive?

Mirror of Performance Description: you specified how you wanted it to behave; now check whether that behaviour is serving you.

#### Description ↔ Discernment as a feedback loop

| | Description (what you send) | Discernment (what you evaluate) |
|---|---|---|
| **Product** | What to create | Whether the output is good |
| **Process** | How to approach it | Whether the approach was sound |
| **Performance** | How to behave | Whether the behaviour is working |

Every discernment finding is feedback for the next round of description. Bad output → sharpen Product description. Bad reasoning → sharpen Process description. Bad interaction dynamics → sharpen Performance description.

> **Even the most advanced AI systems benefit from human judgement and oversight.** Discernment is the mechanism through which that judgement is applied.

### Diligence — the three components

Diligence is the fourth and final D of the 4D Framework. Where Delegation, Description, and Discernment are mostly about **effectiveness and efficiency**, Diligence is about **ethics and safety** — it's the competency that keeps AI collaboration responsible.

Three components, each covering a different phase of the workflow:

#### 1. Creation Diligence — *before and during* the work

Be thoughtful about **which AI systems you choose** and **how you engage with them**.

- **Model / tool selection** — which system is appropriate for the sensitivity, stakes, and domain of the task?
- **Data handling** — what are you putting into the model? Is any of it confidential, personal, regulated, or protected?
- **Provider policies** — how does the provider use your inputs? Training on your data? Retention? Region?
- **Guardrails and settings** — are you using the right mode, the right access controls, the right memory settings for the task?
- **Engagement quality** — are you working with the AI carefully, or on autopilot?

Mental model: Creation Diligence covers everything *upstream of the output* — the choices that determine whether the collaboration is even set up responsibly.

#### 2. Transparency Diligence — *about* the work

Be **open about AI's role** in your work with everyone who needs to know.

- **Who needs to know?** Collaborators, reviewers, clients, examiners, readers.
- **What needs to be disclosed?** That AI was used, in what capacity (drafted, edited, brainstormed with), and to what extent.
- **When?** At the point of handover, submission, or publication — not after the fact.
- **How much?** Enough that the recipient can exercise their own judgement about what to do with the artefact.

Not the same as attribution or citation — this is about honesty regarding the *process* by which the work was produced.

#### 3. Deployment Diligence — *when you share* the work

Take **ownership for AI-assisted outputs you share with others**. You are still the author of record for anything you put your name on.

- **Verify** — check facts, citations, calculations, and claims before the output leaves your hands.
- **Vouch** — you are accountable for the artefact; "the AI said so" is not a defence.
- **Fit-for-purpose** — is this actually suitable for the audience and use? Would you stand behind it?
- **Downstream effects** — who is affected by this output, and have you considered them?

Deployment Diligence is what turns AI-assisted work into *your* work — with all the responsibility that entails.

#### Context sets the bar

Different contexts have different expectations for **disclosure**, **verification**, and **acceptable use**:

- **Personal** — usually informal; the bar is set by you.
- **Academic** — often strict rules on disclosure and permitted assistance; may be assessed on original reasoning.
- **Professional** — varies widely: firm policies, client expectations, regulatory obligations, jurisdictional rules.
- **Public / published** — highest bar; readers can't verify what they can't see.

> **You have a responsibility to understand and meet the expectations that apply to your context.** Ignorance of the applicable standard is not a defence.

#### Where Diligence sits in the 4D framework

- Delegation, Description, and Discernment optimise for **useful output**.
- Diligence optimises for **responsible output** — output that is *ethical, safe, disclosed, and owned*.
- Effective ≠ responsible. A workflow can be effective and still fail Diligence (e.g. accurate output produced by leaking confidential data into a public model).

## Terminology

| Term | Meaning |
|---|---|
| Delegation (sub-skills) | The four skills that compose the Delegation competency: understanding the problem, understanding tool capabilities, decomposing complex work, choosing the right mode. |
| Task decomposition | Breaking a complex goal into right-sized sub-tasks with clear dependencies, so each piece can be delegated (or kept) deliberately. |
| Mode selection | Choosing Automation, Augmentation, or Agency for a given sub-task based on how well-defined it is and how often it recurs. |
| Automation | Working with AI by giving it specific tasks to complete on your instructions. Task-level delegation. |
| Augmentation | Working with AI as a collaborative partner — co-thinking, co-creating in real time. |
| Agency | Working with AI by configuring it to act independently on your behalf — setting knowledge, behaviour patterns, and boundaries rather than tasks. |
| Neural network | Layered graph of "neurons" whose weights are learned during training. The substrate for modern deep learning. |
| Transformer | 2017 neural architecture built on self-attention. The foundation of all modern LLMs. |
| Self-attention | Mechanism letting every token in a sequence weigh its relationship to every other token in parallel — the key innovation of Transformers. |
| Scaling laws | Empirical relationships showing that model capability grows predictably (and sometimes discontinuously) as compute, data, and parameters scale up. |
| Pre-training | The foundation training stage — predict-the-next-token on massive unlabelled text corpora. Produces a capable but not-yet-helpful base model. |
| Fine-tuning | Continued training on smaller curated datasets to specialise a base model — for instruction-following, domain expertise, or format. |
| SFT (Supervised Fine-Tuning) | Fine-tuning on labelled examples of desired input/output pairs. |
| RLHF | Reinforcement Learning from Human Feedback — training the model to produce outputs humans prefer, using a reward model trained on human rankings. |
| Constitutional AI (CAI) | Anthropic-pioneered alignment method where the AI critiques and refines its own outputs against a written set of principles, reducing the need for constant human labelling. |
| Prompt | The input text (instructions, context, examples) fed to the model. Steers the model's output distribution. |
| Token | The atomic unit of text an LLM processes — roughly a word-piece. Context windows are measured in tokens. |
| Context window | The total token budget for a single interaction — prompt + attachments + prior turns + generated response. Anything outside is invisible to the model. |
| In-context learning | The model's ability to generalise from a few examples provided *in the prompt*, without retraining. Enables few-shot prompting. |
| Emergent capabilities | Abilities that appear suddenly as models scale past certain thresholds — not explicitly trained for, often not predictable in specifics. |
| Description (competency) | The second D of the 4D Framework — communicating with AI to create a productive collaborative environment. Goes beyond prompt-writing; shapes the whole interaction. |
| Product Description | Clearly defining *what* you want the AI to create — output, format, audience, style, constraints, examples. |
| Process Description | Guiding *how* the AI approaches the request — steps, order, plan-first vs act-first, when to ask, when to check in. |
| Performance Description | Defining *how* the AI should behave during collaboration — concise/detailed, challenging/supportive, formal/casual, proactive/reactive. |
| Collaborative environment | The overall interaction context created by Product + Process + Performance description together. The thing Description as a competency is trying to shape. |
| Prompt engineering | The practice of designing effective instructions for AI systems. Under the 4D framework, the practical craft that sits under Description. |
| Context (in a prompt) | The background information — goal, why, audience, prior state — the model needs to do the job well. First foundational prompting technique. |
| Few-shot prompting | Providing worked examples of the desired output inside the prompt. Leverages in-context learning. Second foundational technique. |
| Constraints (in a prompt) | Explicit format, length, inclusion, and exclusion requirements. Third foundational technique. |
| Step decomposition (in a prompt) | Breaking a multi-step task into explicit sequential instructions inside a single prompt. Prompt-level analogue of task decomposition. Fourth foundational technique. |
| "Think first" prompting | Asking the model to plan or outline its reasoning before producing the final output. Prompt-level analogue of extended thinking. Fifth foundational technique. |
| Role / tone prompting | Specifying the persona, register, and stance the model should adopt. Sixth foundational technique. Maps to Performance description. |
| Prompt improvement (secret weapon) | Asking the AI itself to critique or rewrite your prompt for clarity. Meta-move for prompt refinement. |
| Iterative prompting | The practice of refining prompts based on output, rather than expecting first-pass perfection. |
| Discernment (competency) | The third D of the 4D Framework — thoughtfully evaluating AI outputs, processes, and behaviours. The human-judgement layer on AI collaboration. |
| Product Discernment | Evaluating the *quality of the output* — accuracy, appropriateness, coherence, relevance, completeness. Mirror of Product Description. |
| Process Discernment | Evaluating *how the AI arrived at the output* — logical errors, attention gaps, inappropriate reasoning, skipped steps, traceability. Mirror of Process Description. |
| Performance Discernment | Evaluating *how the AI behaved during the interaction* — communication style, stance, sycophancy vs pushback, fit to working style. Mirror of Performance Description. |
| Description–Discernment loop | The continuous feedback cycle: describe → generate → discern → re-describe → regenerate. Each discernment finding refines the next description. |
| Diligence (competency) | The fourth D of the 4D Framework — taking responsibility for AI collaborations. The ethical and safety layer, distinct from the effectiveness-focused Ds. |
| Creation Diligence | Being thoughtful about *which* AI systems you choose and *how* you engage with them — model selection, data handling, provider policies, guardrails. Covers everything upstream of the output. |
| Transparency Diligence | Being open about AI's role in your work with everyone who needs to know — who, what, when, how much. Honesty about the process by which work was produced. |
| Deployment Diligence | Taking ownership for AI-assisted outputs you share — verify, vouch, fit-for-purpose, downstream effects. Turns AI-assisted work into your work. |
| Context expectations | The standards for disclosure and verification set by the context (personal, academic, professional, public). Different contexts, different bars. |

## Examples / demos

_Anything worked through in the lesson._

-

## Questions / to revisit

_Things unclear, or worth drilling before the exam._

-

## Exam-relevant takeaways

_Points flagged as likely to appear on the exam / assessment._

- **Delegation has four sub-skills**: understand the problem, understand AI capabilities, break work into smaller parts, choose the right mode. Expect a question that names or asks you to order these.
- **Decomposition precedes mode selection** — you can't pick Automation/Augmentation/Agency until you've cut the problem into pieces.
- Different sub-tasks in the same project can use **different modes**.
- **Three ways of working with AI**: Automation (task execution), Augmentation (collaboration), Agency (independent action on your behalf). Sits under **Delegation** in the 4D Framework.
- Be able to match a scenario to the right mode — e.g. "Claude drafts a specific email you asked for" → Automation; "Claude and you brainstorm a strategy together" → Augmentation; "you set up a Skill/Project that Claude uses to act autonomously for your team" → Agency.
- **Three pillars of the tech journey**: Algorithms (neural nets → Transformers) + Data (text, code, multimodal) + Compute (GPUs/TPUs + clusters). Progress requires all three.
- **Transformers + self-attention** are *the* architectural unlock behind modern LLMs (2017).
- **Scaling laws**: more compute + more data → more capability, often with step-change emergent abilities.
- **Training pipeline order**: Pre-training → Fine-tuning (SFT + domain) → RLHF (or Constitutional AI). Know what each stage does.
- **Pre-training objective** = predict the next token. Everything else is built on this.
- **Context window** is measured in tokens; anything outside it is invisible to the model. Working memory, not long-term memory.
- **Three core LLM capabilities**: pattern-learning at scale, in-context learning (few-shot), emergent capabilities.
- **Description has three components**: Product (what to create), Process (how to approach it), Performance (how to behave). Expect a question naming these or matching a scenario to one.
- **Product ≠ Process ≠ Performance** — be able to distinguish. Format/audience/style = Product; steps/order/planning = Process; concise/challenging/tone = Performance.
- **Description ≠ just prompting** — it's establishing the collaborative environment. Most people only do Product; Process + Performance are where output quality actually lifts.
- **"AI as partner, not database or vending machine"** — a stated framing worth remembering verbatim.
- Scenario matching: "push back on my reasoning" → Performance; "outline before drafting" → Process; "write it for a non-technical audience" → Product.
- **Six foundational prompting techniques** (memorise the list): give context, show examples, specify constraints, break into steps, ask the AI to think first, define role/tone.
- **Prompt engineering = designing effective instructions for AI.** It sits under Description, not as a separate D.
- Be able to **match a technique to a Description component**: context/examples/constraints → Product; steps/think-first → Process; role/tone → Performance.
- **The "secret weapon"** — ask the AI to improve your own prompt.
- **Prompting is iterative and collaborative** — expect refinement; hand failing prompts back to the model.
- Common successful patterns: clear task overview + format specification + explicit constraints + relevant background.
- **Discernment has three components**: Product (evaluating the output), Process (evaluating the approach), Performance (evaluating the behaviour). Mirrors Description one-for-one.
- **Discernment is the flip side of Description** — they form a continuous feedback loop. Description sets intent; Discernment evaluates result; result feeds back into the next description.
- Product Discernment criteria to remember: **accuracy, appropriateness, coherence, relevance** (course-listed) — add completeness for good measure.
- Process Discernment looks for: **logical errors, attention gaps, inappropriate reasoning** — even good-looking output can be produced badly.
- Performance Discernment: is the AI's *communication style and behaviour* actually working for you? Sycophancy is a discernment failure to catch.
- **Human judgement and oversight are always in the loop** — Discernment is *how* you apply them. Expect a "why does human oversight matter even for advanced AI?" style question.
- Scenario matching: "the citations are fabricated" → Product Discernment; "it skipped the second step of my instructions" → Process Discernment; "it just agrees with everything I say" → Performance Discernment.
- **Diligence has three components**: Creation (which AI, how), Transparency (disclosing AI's role), Deployment (owning what you share). Cover different phases: before/during, about, and at the point of sharing.
- **Diligence is the ethical-and-safety layer** — the other three Ds optimise for useful output; Diligence optimises for *responsible* output.
- Effective ≠ responsible: a workflow can be effective and still fail Diligence (e.g. leaking confidential data into a public model).
- **Different contexts, different bars** for disclosure and verification: personal, academic, professional, public. Know which one applies.
- **"The AI said so" is not a defence** — Deployment Diligence means you own the output you share.
- Scenario matching: "checking which model handles PHI safely" → Creation; "disclosing to the client that AI drafted the memo" → Transparency; "verifying the citations before you send the report" → Deployment.
- The **four Ds together**: Delegation (what work goes where), Description (how you communicate the ask), Discernment (how you evaluate the result), Diligence (how you keep it all responsible). Expect a synthesis question at the end.
