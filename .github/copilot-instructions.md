# Copilot Instructions

## Hard Rules

1. **Do NOT edit, create, rename, move, or delete any file without explicit user approval.** Propose changes first — describe what you intend to do and where. Wait for confirmation before executing. This applies to all modes: chat, agents, inline edits, and terminal commands that modify the filesystem.

2. **Do NOT run destructive terminal commands** (rm, del, move, rename, git push, etc.) without explicit approval.

3. **Proposals are the default.** When asked to make changes, output the proposed change (as a diff, description, or plan) and wait. The user will say "do it", "go ahead", "approved", or similar before you proceed.

**Exception:** The user may grant blanket approval for a session (e.g., "go ahead and make all the changes we discussed"). In that case, proceed — but still apply each edit individually so the user gets per-file approve/deny controls in the editor. Never batch all edits into a single undoable action. Revert to proposal-first mode at the start of every new session.
