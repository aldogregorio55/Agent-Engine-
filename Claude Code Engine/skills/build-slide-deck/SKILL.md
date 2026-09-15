---
name: build-slide-deck
description: >-
  Generates a real, editable PowerPoint (.pptx) slide deck from a slide-by-slide markdown spec, using
  python-pptx to build native text boxes, tables, and shapes — not images, not HTML. Output stays fully
  editable in PowerPoint, Keynote, and Google Slides. Use when the user wants a slide deck, presentation,
  or lecture deck built as an actual file meant for hand-editing afterward — "build a deck from this
  outline", "turn this into slides I can edit in Google Slides", "generate a pptx", or any request for a
  presentation deliverable meant to be edited by hand after generation, not just viewed.
  Do NOT use when: an HTML/web artifact is what's actually wanted (use artifact-design), or the ask is a
  one-off static visual rather than an editable multi-slide presentation.
allowed-tools: Read, Write, Bash
context: inline
agent: content-strategist
---

# Build Slide Deck

## Your Task
Turn a slide-by-slide markdown spec into a real, editable `.pptx` file — native shapes and tables a
person can open in PowerPoint, Keynote, or Google Slides and keep editing by hand. This is a **build**
step, not a content-writing step: the spec should already exist (or get written first, separately)
before any code runs.

## When to reach for this vs. other options
- Use this when the ask is a real, multi-slide, editable presentation file.
- For a fact-heavy deck (numbers, external claims) where accuracy matters, get the content verified
  before building — whether that's a `produce-deliverable` pass or any other fact-check step is a
  separate call, not something this skill assumes or requires.
- Skip to a plain HTML/artifact-design build when the target is a webpage, not a file meant for
  Slides/PowerPoint/Keynote editing.

## Steps

1. **Source spec first.** Expect (or write) a slide-by-slide markdown map: headline, body, and visual
   direction per slide, plus a locked color/theme convention up top. Never invent slide content inline
   in the build script — content decisions belong in the spec, not buried in code.

2. **Prefer a real template over a blank Presentation().** If the user has a company/brand template
   file, open it directly — `Presentation("company-template.pptx")` — instead of `Presentation()`.
   This inherits fonts, colors, and placeholder positions for free, lets you copy a logo or footer
   verbatim by duplicating an existing slide and only swapping its text, and — as a real bonus — starts
   from Microsoft's own OOXML conventions instead of `python-pptx`'s default template, which sidesteps
   the Google Slides compatibility issue in point 4 below entirely. Default to a blank Presentation()
   only when no template exists.

3. **Fixed layout system.** Categorize slides into a small set of reusable types (title, section header,
   bullets, table, diagram, mock-UI, code-card) so the deck reads as one system, not one-off slides per
   page. Build one helper function per type.

4. **Build with `python-pptx` only. Save only via `prs.save()`. Never re-zip the output afterward.**
   This is the load-bearing rule — breaking it is what causes real, hard-to-diagnose failures:
   - **Never hand-edit or re-zip a finished `.pptx` with an external tool** (e.g. macOS `zip -r`).
     `python-pptx`'s writer produces a flat archive with no directory entries; re-zipping with a shell
     zip utility adds explicit directory entries that **Google Slides' importer rejects outright** —
     even though PowerPoint and Keynote open the same file without complaint. If a fix is needed after
     the fact, regenerate from the script. Don't patch the archive by hand.
   - **Set slide dimensions before adding any slides**, and don't leave a stale `sldSz type` attribute
     behind from resizing the default 4:3 template to widescreen. A declared type that doesn't match
     the actual cx/cy dimensions is a second real, independently-confirmed cause of "can't open"
     failures in Slides specifically.
   - Prefer plain autoshapes (`MSO_SHAPE`) and native tables (`shapes.add_table`). Avoid custom
     geometry (`custGeom`), gradients, and connectors beyond simple decorative lines — these are the
     shape types most likely to trip stricter importers.
   - **The workaround, if Slides still rejects a file that opens fine elsewhere:** open it in
     PowerPoint (desktop or web) and re-export/re-save as `.pptx`, then upload that copy. The
     round-trip normalizes the XML into conventions Google's importer accepts. Confirmed to work; keep
     it in your back pocket rather than debugging the raw XML further once dimensions/zip-structure are
     already clean.

5. **Verify before handoff.** Zip integrity (`unzip -t` or a fresh `python-pptx` reload), slide count
   matches the spec, no stray directory entries, no unused leftover parts (e.g. the default template's
   `printerSettings1.bin` if nothing needs it).

6. **Output is a strong first draft, not an iteration target.** Hand off the `.pptx` (plus the generator
   script, so it can be regenerated or tweaked programmatically later) for the user to edit by hand from
   there. This tool is for getting to a good, brand-consistent base fast — it is not built for many
   further generation round-trips once a human starts editing. If the user says the file is a good base
   and they're taking it from here, that's success, not a stopping point to push past.

## Output Format
```
Deck:   [path to the .pptx]
Script: [path to the generator script]
Notes:  [any content deviations from the spec, and why; anything still needing a manual asset/graphic]
```
