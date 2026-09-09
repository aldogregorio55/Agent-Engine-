# Slide Deck Production Pipeline

**The Problem** — A good deck needs two things people usually attempt in one pass: *true content* and *good design*. Done together, you get confident-looking slides with wrong numbers. This is the pipeline that separates them — using specialist AI agents so each concern is done properly, and the facts are verified before anything is made pretty.

Companion to [`slide-deck-creation.md`](./slide-deck-creation.md): that file is the **craft** (what makes a good slide). This file is the **process** (how to produce one with AI).

---

## The pipeline — four stages, four specialists

```
RESEARCH  →  DRAFT  →  VERIFY  →  BUILD
 dossier     slides    audit      artifact
```

Each stage is a **separate agent in its own isolated context**. They never see each other's conversations — they receive a brief and return a *file path + a short summary*. The orchestrator stays clean and steers; it does not do the specialist work itself.

**1. Research → a source-traced dossier, not slides.**
Ask for a dossier first: every number traced to a `file:line`, caveats flagged, nothing polished. Separating *gather the facts* from *make the artifact* is what keeps the whole thing honest. A deck built straight from a repo invents confident numbers; a dossier forces the facts to stand on their own first.

**2. Draft → the copy and structure.**
Turn the dossier into slides + speaker notes against the target template. The drafter's only source of truth is the dossier — it can shape, not invent.

**3. Verify → independently, against the original source.**
The stage most people skip, and where the quality actually comes from. Tell the evaluator: *do not trust the dossier — go to the primary files directly.* An independent checker with a different source of truth catches the errors a self-review rubber-stamps. In the reference run, this caught 4 number errors — 3 of which the dossier had introduced and the draft had faithfully copied.

**4. Build → the artifact, from verified facts.**
Only after the numbers are trustworthy, build the final artifact (e.g. the HTML deck), folding the corrections into the build brief. Use a design skill (e.g. `dataviz`) so the visuals form one coherent system instead of ad-hoc charts.

---

## Principles

**1. Separate finding the truth from making it pretty.** Facts first (dossier), artifact last. Never let the design stage also be the stage that decides what's true.

**2. Verify with an independent source, not a re-read.** The audit works *because* it ignores the intermediate summary and returns to raw source. Self-review confirms its own mistakes.

**3. One specialist per job.** Research, writing, evaluation, and engineering are different skills. One agent doing all four does none of them well.

**4. Pass file paths, not content.** Each stage writes a file and hands the *path* forward (`dossier → deck → audit → html`). The orchestrator's context stays clean enough to steer the entire chain.

**5. Brief, don't command.** Every agent gets: who it is, tight relevant context, *one* task, an exact output path, and a capped return ("path + N-sentence summary"). The cap is what turns floods into signal.

**6. Skills carry the craft.** A design skill (`dataviz`) or a writing skill is codified expertise. Invoking it is how a generalist produces disciplined output instead of guessing at color, layout, or structure.

**7. Sequence by dependency; parallelize when independent.** These four ran sequentially because each needed the last (you can't verify a deck that doesn't exist). Independent tasks would launch in parallel instead.

---

## Be loud about what wasn't verified

The pipeline's job is to make the *remaining* human review small and targeted — not to pretend it's zero. Always report what couldn't be checked in-process. In the reference run that meant: no browser to render the HTML, no runtime to run the design skill's palette validator, and a set of claims that traced to files outside the evidence set. Naming these is part of the deliverable, not an admission of failure.

---

## The recipe, condensed

1. **Research agent** → dossier (every fact traced to `file:line`).
2. **Draft agent** → slides from the dossier only.
3. **Eval agent** → verify every number against the *original* source, independently.
4. Fold the corrections into the build brief.
5. **Build agent (+ design skill)** → final artifact.
6. Report what still needs a human eye.
