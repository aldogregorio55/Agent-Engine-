# Firm Skills Library

Skills authored for use across the firm — written to be **read, reviewed, and transferred** as
work outputs, independent of any one person's agent setup.

**Goal:** hold the readable, portable copy of each skill so a colleague can open any `SKILL.md`
here and understand it in any editor or previewer, and carry it between machines without anything
rendering oddly or getting stripped. Anything tied to a private setup (e.g. `context: fork`,
`agent:` routing) is left out here; the private-engine copy keeps those.

**Metadata format — read as documents, not machine-parsed.** These files intentionally do **not**
use YAML frontmatter (the `--- … ---` fenced block). Frontmatter is a machine format for a skill
that will actually be installed/uploaded — many editors hide it or render it as raw text, so it
reads and transfers inconsistently. Here the file opens with the skill's own `# H1` header, then
the metadata (name, allowed tools, description) as bold-labelled `**Name:**`, `**Allowed tools:**`,
and `**Description:**` fields, then the body directly. No `---` separator under the fields — a `---`
sitting there just reads as frontmatter again, the thing being avoided. This is an interim,
review-friendly layout; when a skill from here is genuinely being installed, a separate
install-ready copy is rewritten to carry real YAML frontmatter.

## Contents

- `detect-injection/` — an on-demand ingress shield that screens ingested content (web/tool results,
  fetched URLs, pasted documents, upstream agent handoffs) for prompt-injection before an agent acts
  on it, then neutralizes, describes, and escalates to the user. Aligned with the Safe AI Agents
  safety-block terminology.
- `threat-detection/` — the broader sibling, versioned (`v1/`–`v4/` — see
  [Versioning](#versioning) below). Judges content by intent rather than by whether it reads like
  instructions, then neutralizes or redacts. The versions differ in scope **and output target**,
  not just wording: `v1`/`v2` are general-purpose screens for any external data an agent ingests
  (web/tool results, fetched URLs, documents, files), returning a human-facing message; `v3`
  narrows to the specific use case of vetting a **new skill the user is adding** — from a
  marketplace, a GitHub repo, or any other source — before it's trusted and allowed to run with
  real tool access, still handing the decision back to the user in prose. `v4` is the same
  skill-vetting screen re-pointed for **automated use: it runs automatically on install and returns
  a machine-readable structured verdict to the calling system, not a message to the user** — so it
  drops the human-facing prose reply, and instead of quoting a flagged payload back it returns a
  redacted excerpt with a typed marker (e.g. `<REDACTED:api-key>`). Reach for `detect-injection`
  when the question is narrowly "is this an injection attempt," `threat-detection` `v1`/`v2` for "is
  anything wrong with this content," `v3` for "is this skill safe to install" with a human in the
  loop, and `v4` for the automatic-on-install gate whose verdict is consumed by another system.

## Versioning

Skills in this folder are outputs written for work, not the personal engine's live skill roster —
they accumulate revisions over time as requirements change. Each skill that has more than one
revision lives in versioned subfolders: `<skill-name>/v1/SKILL.md`, `<skill-name>/v2/SKILL.md`,
and so on, rather than being overwritten in place. A skill with only one version can stay flat
(`<skill-name>/SKILL.md`) until a second version exists, at which point both move into `v1/`/`v2/`
subfolders.
