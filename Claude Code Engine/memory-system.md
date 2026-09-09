# Claude Engine — Memory System

**Location (live):** `~/.claude/auto-memory/`
**Status as of 2026-07-13:** Empty. No `MEMORY.md` and no memory files exist yet — nothing has
been saved in any session so far. This doc captures the *mechanism*, not populated content.

This is a persistent, file-based memory system, separate from agents/ and skills/. It's meant to
build up a picture of the user, working norms, and project context across sessions — not to hold
task-scoped state (that belongs in Plans/Tasks) or anything derivable from the code/git history.

---

## Structure

```
~/.claude/auto-memory/
├── MEMORY.md              ← index only. One line per memory: "- [Title](file.md) — hook"
│                              No frontmatter. Always loaded into context. Truncated after 200 lines.
└── [topic]_[slug].md       ← one memory per file, semantic filename, not chronological
```

## Memory File Format

```markdown
---
name: {{short-kebab-case-slug}}
description: {{one-line summary — used to judge relevance in future conversations}}
metadata:
  type: {{user, feedback, project, reference}}
---

{{content — see per-type structure below. Link related memories with [[other-slug]].}}
```

## The Four Types

| Type | Captures | Body structure |
|---|---|---|
| **user** | Role, goals, responsibilities, domain knowledge — how to tailor future behavior to who they are | Free-form profile notes |
| **feedback** | Corrections *and* confirmed approaches — what to stop doing, what to keep doing | Rule → **Why:** (the reason given) → **How to apply:** (when it kicks in) |
| **project** | Ongoing work, decisions, deadlines not derivable from code/git — convert relative dates to absolute | Fact/decision → **Why:** (motivation/constraint) → **How to apply:** |
| **reference** | Pointers to external systems (Linear project, Slack channel, dashboard URL) — not the content itself | Where to look + what it's for |

## What Never Gets Saved

- Code patterns, architecture, file paths — derivable by reading the repo
- Git history / who-changed-what — `git log`/`git blame` are authoritative
- Debugging fixes — the fix lives in the code, the commit message has the context
- Anything already in a CLAUDE.md
- Ephemeral in-progress task state (use Tasks/Plans instead)

These exclusions hold even if explicitly asked to save — the fix is to ask what was *surprising*
about the request and save only that.

## Save Triggers

- **user**: any detail learned about role/preferences/expertise level
- **feedback**: any correction ("no, don't do X") *or* quiet confirmation of a non-obvious choice
  ("yes exactly, keep doing that") — confirmations are easy to miss; corrections are not
- **project**: who's doing what, why, by when — state changes fast, so absolute-date everything
- **reference**: any external system named as the source of truth for something

## Retrieval Discipline

A memory naming a specific file/function/flag is a claim about state *when it was written* — it
may be stale. Before acting on a recalled memory (not just discussing history), verify the named
thing still exists (file present, symbol greppable). Trust current inspection over old memory
when they conflict, and update/remove the stale memory rather than act on it.
