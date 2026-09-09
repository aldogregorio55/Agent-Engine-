---
title: Dissecting Anthropic's skill-creator — a craft breakdown
source: [skill-creator.md](skill-creator.md)
captured: 2026-07-20
purpose: Reverse-engineer the authorship techniques Anthropic uses to write world-class skills, so you can write your own to the same standard.
---

# Dissecting Anthropic's `skill-creator`

This document breaks down **how** the skill is written — the choices, patterns, and philosophy behind the prose — not what it does. Each section pulls a technique out of the source and explains why it works and how to reuse it.

If you internalise the ten patterns below, you can author skills at the same level of depth, precision, and comprehensiveness.

---

## 0. The mental model: a skill is a system prompt for a task

Before dissecting technique, absorb the framing implicit in this skill:

- A skill is **not** a prompt template. It's a **conditionally-loaded system prompt** the model consults when it thinks a task fits.
- Every word costs context. Every instruction competes with the user's actual task for the model's attention.
- The author is writing for **another intelligent agent**, not a function call. Explanation beats coercion.

Everything below flows from those three premises.

---

## 1. Frontmatter: description is the router, not a summary

```yaml
---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---
```

**What's happening:**

- Two sentences. First is **what it does** (verb-led, present-tense capabilities). Second is a **"Use when..." trigger list** — a chain of concrete user intents joined with "or".
- The description is written to be **skimmed by a router model** deciding whether to load the skill. It packs synonyms and situations rather than prose.
- The skill explicitly names its own **anti-pattern** later in the body — "Claude has a tendency to *undertrigger* skills" — and prescribes a "pushy" description style to compensate.

**The trigger-list formula:** `Use when users want to [X], [Y], [Z], or [W].` Cover paraphrases, adjacent verbs, and edge cases in one flat list.

**Reusable template:**

```yaml
name: <kebab-case-verb-noun>
description: <One sentence: what it does, verb-led.> Use when users want to <intent 1>, <intent 2>, <intent 3>, or <edge case>.
```

**Why it works:** the routing model has ~100 words of context to decide. Concrete intents beat abstract descriptions because they match the shape of what users actually type.

---

## 2. Open with the "at a high level" summary — always

```markdown
# Skill Creator

A skill for creating new skills and iteratively improving them.

At a high level, the process of creating a skill goes like this:
- Decide what you want the skill to do...
- Write a draft of the skill...
- Create a few test prompts...
```

**What's happening:**

- One-line identity statement.
- Immediate **procedural TL;DR** as a bullet list — the whole loop compressed to 7 lines.
- **Then** the sections drill into each step.

**Why it works:** the executing model gets the shape of the whole task before any detail. This lets it plan, decide where the user is in the loop, and skip forward if needed. Contrast: a skill that opens with "## Prerequisites" forces the model to read linearly.

**Reusable pattern:** every skill should have a "here's the whole loop in 5-10 bullets" block within the first 30 lines.

---

## 3. Explicit stance on flexibility — pre-empt rigidity

```markdown
Your job when using this skill is to figure out where the user is in this process...

On the other hand, maybe they already have a draft of the skill. In this case you can go straight to the eval/iterate part of the loop.

Of course, you should always be flexible and if the user is like "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.
```

**What's happening:** the author explicitly tells the model **the process is a menu, not a script**. It gives permission to skip steps, jump to the middle, or drop the framework entirely if the user signals they want to.

**Why it works:** without this, models tend to march through numbered lists literally. This paragraph gives the model **theory-of-mind license** — it can read the user's intent and respond accordingly.

**Reusable pattern:** whenever your skill has a sequence, add a short "read the user's cue and pick up where they are" paragraph. Give explicit escape hatches ("if the user says X, just do Y instead").

---

## 4. Meta-instruction: teach the model about its audience

```markdown
## Communicating with the user

The skill creator is liable to be used by people across a wide range of familiarity with coding jargon...

- "evaluation" and "benchmark" are borderline, but OK
- for "JSON" and "assertion" you want to see serious cues from the user...
```

**What's happening:** the author uses skill context space to describe **the user population** and give **specific vocabulary tiers**. This isn't about the task — it's about how to talk while doing the task.

**Why it works:** communication style is a common failure mode that no amount of task-specific instruction fixes. Calling it out once, with examples, transfers a whole social skill in ~5 lines.

**Reusable pattern:** if your skill's outputs go to a human (not another system), reserve a "Communicating with the user" section covering:

- Who the user likely is
- Words to prefer / avoid
- When to over-explain vs. when to be terse
- Whether to ask clarifying questions or just proceed

---

## 5. Progressive disclosure — the anatomy pattern

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

Followed by:

> Skills use a three-level loading system:
> 1. **Metadata** — Always in context (~100 words)
> 2. **SKILL.md body** — In context whenever skill triggers (<500 lines ideal)
> 3. **Bundled resources** — As needed (unlimited)

**What's happening:** the skill teaches a **context budget hierarchy**. Every skill has three tiers of loading, with a soft budget for each. Large content goes into `references/` and is only read when relevant. Repetitive deterministic work goes into `scripts/` and is *executed*, not loaded.

**Why it works:** it turns "how big can my skill be?" from a vibes question into an engineering one. The 500-line rule for SKILL.md forces you to move detail into references — which forces you to organise by topic.

**Reusable pattern for your own skills:**

- SKILL.md = the workflow, decisions, principles
- `references/*.md` = anything you'd otherwise put in a footnote or appendix
- `scripts/*.py` (or `.ps1`) = anything the model would otherwise write from scratch on each invocation

The critical move: **read the transcripts of past runs.** If you see the model re-inventing the same helper across three test cases, that helper belongs in `scripts/`.

---

## 6. The "explain the why" doctrine — replacing MUSTs with reasons

This is arguably the most important pattern in the whole document:

```markdown
### Writing Style

Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs. Use theory of mind and try to make the skill general and not super-narrow to specific examples.
```

And later, applied recursively to itself:

```markdown
3. **Explain the why.** Try hard to explain the **why** behind everything you're asking the model to do... If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag — if possible, reframe and explain the reasoning...
```

**What's happening:** the skill treats ALL-CAPS MUSTs as **code smell**. Every constraint should be paired with the reason it exists, because:

- A model that understands *why* can generalise to unforeseen cases.
- A model given only *what* will follow rules literally and miss the point.
- Rigid rules calcify skills against evolving models.

**Concrete rewrite examples from the source:**

- ❌ "ALWAYS write output to `feedback.json`"
- ✅ "Save results to `feedback.json` — the next iteration's reviewer script reads it from that path"

- ❌ "You MUST spawn baseline runs"
- ✅ "For each test case, spawn two subagents in the same turn — one with the skill, one without. This is important: don't spawn the with-skill runs first and then come back for baselines later. Launch everything at once so it all finishes around the same time."

Notice the second one: the "why" is *load timing*. The instruction is still directive, but the rationale is transparent.

**Reusable pattern:** after writing any instruction, ask "*would a smart new hire understand why this matters?*" If not, add the reason on the same line.

---

## 7. Voice: conversational, first-person, occasionally weird

```markdown
- **the rest of the skill :)**

...

Cool? Cool.

...

Sorry in advance but I'm gonna go all caps here: GENERATE THE EVAL VIEWER *BEFORE* evaluating inputs yourself.
```

**What's happening:** the author writes to the model **like a colleague**. Emojis, meta-jokes ("Cool? Cool."), apologising before an emphatic instruction. The all-caps rule from §6 is broken **once**, and the author flags it as a deliberate exception.

**Why it works:**

- **Salience.** Distinctive tone stands out in a context window full of formal instructions.
- **Modelling.** By being casual with the model, the author gives it license to be casual back — useful when the user is casual.
- **Signalling exceptions.** When the author says "sorry, going caps here", the caps carry more weight *because* they're rare.

**Reusable pattern:** don't be robotic. Occasional first-person, mild humour, and framing yourself as a collaborator (not an authority) all improve compliance and reasoning quality. But keep it *rare* so it stays load-bearing.

---

## 8. Structural motifs — how the sections are shaped

Look at the recurring shapes:

| Motif | Where it appears | Purpose |
|---|---|---|
| **Numbered procedure** with headings `### Step 1:`, `### Step 2:`... | "Running and evaluating test cases" | Sequential work — the model can literally check off steps |
| **Bulleted principle list** | "How to think about improvements" (4 items, each bolded lead) | Non-sequential mindset shifts — order doesn't matter, but each item stands alone |
| **Fenced code block with the schema** | `evals.json`, `feedback.json`, `timing.json` | Contracts. The exact JSON shape the next tool expects |
| **"What the user sees" section** | Viewer description | Transfers UX context so the model can talk to the user about it |
| **Environment carve-out** | "Claude.ai-specific instructions", "Cowork-Specific Instructions" | Same skill, different runtimes — one authoritative doc, environment deltas at the bottom |
| **Repeat-the-loop coda** | "Repeating one more time the core loop here for emphasis" at the end | Two exposures to the top-level loop, front and back, in case the model forgot |

**Reusable pattern:** pick the right motif for the content type. Numbered steps for pipelines; bulleted principles for judgement calls; fenced schemas for anything another script consumes; carve-outs for environment differences.

---

## 9. Failure-mode anticipation — write against known bad behaviour

Throughout the document, the author names specific ways the model tends to go wrong and pre-empts them:

- **Undertriggering** → "make the skill descriptions a little bit 'pushy'"
- **Overfitting to test cases** → "if the skill only works for those examples, it's useless... branch out, use different metaphors"
- **Reinventing helpers** → "if all 3 test cases resulted in the subagent writing a `create_docx.py`, that's a strong signal the skill should bundle that script"
- **Skipping the viewer** → "GENERATE THE EVAL VIEWER *BEFORE* evaluating inputs yourself"
- **Rigid rule-following** → "If you find yourself writing ALWAYS or NEVER in all caps, that's a yellow flag"
- **Prematurely optimising the description** → "please save it until you've fully finished making the skill"

**Reusable pattern:** for every skill you write, list the **5 most likely ways it can go wrong**, and address each one explicitly with a named callout. This is worth more than any amount of positive instruction.

---

## 10. The self-referential proof

The final proof that this skill works: **it is itself a skill written to its own standards.** The document you just read demonstrates every principle it teaches:

- ✅ Verb-led, trigger-listy description
- ✅ Opens with an at-a-glance loop summary
- ✅ Gives the model flexibility to skip steps
- ✅ Explains the audience ("plumbers to grandparents")
- ✅ Uses progressive disclosure (`agents/`, `references/`, `assets/`)
- ✅ Explains "why" rather than commanding
- ✅ Conversational voice with a deliberate all-caps exception
- ✅ Structural motifs matched to content type
- ✅ Anticipates and pre-empts failure modes
- ✅ Repeats the core loop at the end

**Takeaway:** when you finish writing your own skill, run through the checklist below and confirm you've hit each item. If you haven't, you're leaving quality on the table.

---

## Authoring checklist

Use this every time you write a new skill.

**Frontmatter**
- [ ] `name` is kebab-case, verb-noun form
- [ ] `description` is 2 sentences: what it does, then `Use when users want to X, Y, or Z`
- [ ] Description includes synonyms, casual phrasings, and adjacent intents
- [ ] Description is *slightly pushy* to counter undertriggering

**Opening**
- [ ] One-line identity statement immediately after the `#` title
- [ ] "At a high level..." bulleted loop within the first 30 lines
- [ ] Explicit permission to skip steps or adapt to user cues

**Body structure**
- [ ] Under ~500 lines (move detail to `references/*.md`)
- [ ] Numbered steps for sequential procedures
- [ ] Bulleted principles for mindset/judgement content
- [ ] Fenced schemas for any structured contract
- [ ] Environment carve-outs at the bottom, not scattered inline

**Voice and reasoning**
- [ ] Every non-obvious instruction paired with a reason
- [ ] No ALL-CAPS MUSTs unless there's a deliberate load-bearing exception
- [ ] Imperative form for actions
- [ ] Occasional first-person / conversational asides

**Bundled resources**
- [ ] Large reference material moved to `references/*.md`
- [ ] Repetitive deterministic work moved to `scripts/`
- [ ] Templates, icons, fonts, HTML shells moved to `assets/`
- [ ] Each bundle referenced from SKILL.md with clear "read this when..." guidance

**Failure-mode anticipation**
- [ ] Named the 3-5 most likely ways this skill will fail in practice
- [ ] Each failure mode has a specific counter-instruction

**Coda**
- [ ] Core loop repeated at the end (2× exposure — front and back)
- [ ] TodoList / tracking reminder if the workflow is long

---

## Quick-reference: the four highest-leverage rules

If you only remember four things:

1. **Description is the router.** Trigger-list format, slightly pushy, packed with synonyms.
2. **Explain why, don't command what.** MUSTs are a smell; reasons generalise, rules don't.
3. **Progressive disclosure.** SKILL.md < 500 lines. Everything else in `references/`, `scripts/`, `assets/`.
4. **Anticipate failure modes by name.** Every skill needs a "here's how you'll mess this up" section.

Everything else is decoration.
