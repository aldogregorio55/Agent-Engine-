---
name: produce-deliverable
description: >-
  Produces a high-stakes deliverable (slide deck, report, spec, eval set) by orchestrating
  specialist agents through a research → draft → verify → build pipeline, gating on an independent
  fact-verification pass before the artifact is built.
  Use when the user says "build a deck/report/spec from this", "turn this repo/project into a
  presentation", "produce a polished deliverable where the numbers have to be right", or any request
  to create a fact-heavy artifact for an external or high-scrutiny audience.
  Do NOT use when: the task is a single-pass write with no facts to verify (use the content-strategist
  directly), or when designing the agent system itself rather than running it (use design-agent-system).
allowed-tools: Agent, Read, Write, Edit
context: inline
agent: content-strategist
---

# Produce Deliverable

## Your Task
Orchestrate a fact-safe production pipeline that turns source material into a finished deliverable.
Separate *finding the truth* from *making it pretty*: gather source-traced facts first, draft from
those facts only, verify every claim against the original source independently, then build the
artifact last from verified facts. **Never let the build stage decide what is true.**

Run the four stages as separate agents in isolated contexts. Pass file paths between stages — never
raw content. Return the final artifact path plus an explicit list of what still needs a human eye.

## When to reach for this vs. a single agent
Use the full pipeline when the deliverable is **fact-heavy and high-scrutiny** — numbers in front of
managers, external audiences, anything expensive to walk back. For a low-stakes or fact-light artifact,
skip to a single content-strategist pass; the pipeline's overhead only pays off when a wrong figure is
costly.

## Steps

1. **Research → a source-traced dossier (not the artifact)**
   - Spawn a research agent (`research-analyst`, or `content-strategist` for lighter work).
   - Brief: produce a dossier where every fact and number is traced to a `file:line` or source URL,
     caveats and ambiguities flagged, nothing polished.
   - Output: `agent-outputs/[project]-dossier.md`.
   - Gate: any claim the dossier cannot source is a known gap — it does not enter the draft as fact.

2. **Draft → copy and structure**
   - Spawn the drafting agent (`content-strategist`).
   - Its ONLY source of truth is the dossier — it may shape and structure, never invent.
   - Match the target template / house style. Put depth in speaker notes or appendices; keep the
     on-slide / on-page text tight.
   - Output: `[project]/[deliverable]-draft.md`.

3. **Verify → an independent fact audit (this is the gate)**
   - Spawn an evaluation agent (`ai-evaluator`).
   - Instruct it explicitly: **do NOT trust the dossier or the draft — verify every quantitative and
     factual claim against the ORIGINAL primary sources directly.** A different source of truth is
     what catches errors propagated from the dossier into the draft.
   - Output: a claim-by-claim table (claim → `source:line` → MATCH / MISMATCH / UNVERIFIABLE), a
     corrections list, and a bottom-line pass/fail.
   - Gate: MISMATCHes must be corrected before build. UNVERIFIABLE claims are flagged for human
     review, not silently shipped.

4. **Build → the artifact, from verified facts**
   - Spawn the build agent (`senior-engineer` for HTML/code artifacts; `content-strategist` or
     `document-writer` for prose).
   - Fold the verified corrections into the build brief so it builds from clean data.
   - Invoke the relevant craft skill (e.g. `dataviz` for charts; the Slide Decks knowledge for decks).
   - Output: the final artifact, and apply the corrections back to the draft source so the two stay
     in sync.

## Briefing rules (apply to every stage)
- Identity + skill activation, tight relevant context (not everything), one task, an exact output path.
- Cap the return: "file path + N-sentence summary." Substantive output goes to the file, not the reply.
- Sequential by dependency (each stage needs the last). Parallelize only genuinely independent work.

## Output Format

```
Deliverable: [final artifact path]
Pipeline:    research → draft → verify (N claims, M mismatches fixed) → build
Needs a human eye: [what couldn't be verified in-process — rendering, tools not run, out-of-scope claims]
```

## Edge Cases

- **Fact-light deliverable:** skip stages 1 and 3; a single content-strategist pass is enough — say so
  rather than running the full pipeline for show.
- **No independent source to verify against:** the verify stage cannot do its job — flag that the numbers
  rest on a single unverified source, and make that limitation explicit in the deliverable.
- **Verification finds the dossier is the error source:** correct the dossier too, not just the draft, or
  the next build re-poisons from it.
- **Build environment can't render/validate the artifact (no browser, no runtime):** hand-verify what you
  can and name the unrun checks as required human follow-up. Never imply zero remaining review.
- **Slide decks specifically:** pair with the `knowledge/Slide Decks/` reference — `slide-deck-creation.md`
  (craft) and `slide-deck-production-pipeline.md` (this pattern, worked example).
