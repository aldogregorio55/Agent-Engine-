````markdown
# Two-Machine Solo Workflow

**Load this file when:** Using GitHub as the sync mediator between two machines you own (e.g., main device + devbox with restricted tooling). Prerequisite: [git-and-github-playbook.md](git-and-github-playbook.md).

---

## The Setup

You are one person acting as two "collaborators" — one identity, two working environments, one shared repo on GitHub. GitHub is the only channel between the two machines.

```
┌──────────────────┐                          ┌──────────────────┐
│  Main device     │                          │   Devbox         │
│  (this machine)  │                          │  (other tools)   │
└────────┬─────────┘                          └────────┬─────────┘
         │                                             │
         │            push / pull only                 │
         └──────────────► GitHub ◄────────────────────┘
                        origin/main
```

The two machines **never talk directly**. Everything flows through GitHub. This introduces exactly one new failure mode compared to solo single-machine work: **you can strand uncommitted work on the machine you're not sitting at.**

---

## The Rhythm

The base playbook's rule — "pull when you sit down, push when you stand up" — is doubly important here. Ritualize it on **both** machines.

### Start of session (on either machine)
```powershell
git status              # anything uncommitted from last time? shouldn't be
git pull                # grab what the other machine pushed
```

### End of session (on either machine)
```powershell
git status              # what did I change?
git add .
git commit -m "..."     # do NOT leave uncommitted
git push                # do NOT walk away without pushing
```

If you break this rhythm once — leave uncommitted work on Machine A and start editing the same file on Machine B — you create a divergence that has to be resolved manually. Fixable, but annoying. The discipline is the whole game.

---

## The `git status -sb` Habit

On both machines, before doing *anything*:

```powershell
git status -sb
```

The `-sb` flag gives a one-line summary with ahead/behind counts vs. `origin/main`:

| Output | Meaning | Action |
|---|---|---|
| `## main...origin/main` | In sync | Safe to edit |
| `## main...origin/main [ahead 2]` | Unpushed local commits | `git push` |
| `## main...origin/main [behind 3]` | Remote has commits you don't | `git pull` |
| `## main...origin/main [ahead 1, behind 2]` | Diverged | `git pull --rebase`, then `git push` |

Make this the muscle-memory first command every time you sit down at either machine.

---

## The Four Failure Modes

| Failure | What it looks like | Fix |
|---|---|---|
| **Forgot to push** on Machine A. Sit down at Machine B, pull shows nothing new, you start editing. | Two divergent local histories. Next push from either side gets rejected. | On whichever side you're on: `git pull --rebase`, resolve any conflicts, `git push`. |
| **Forgot to pull** on Machine B. Start editing on top of stale files. | Same divergence as above. | Same fix. |
| **Uncommitted WIP** on Machine A, need to jump to Machine B urgently. | Machine A has a dirty working tree that isn't on GitHub anywhere. | Commit as `WIP:` and push. Stash is invisible to the other machine — do not use it as a handoff mechanism. |
| **Same lines edited on both sides** without syncing between. | `git pull --rebase` stops with a merge conflict. | Open file, resolve `<<<<<<<` markers, `git add <file>`, `git rebase --continue`. |

---

## The WIP-Commit Trick

Sometimes you'll be mid-thought and need to hand off. Don't try to be clean — **commit ugly and push:**

```powershell
git add .
git commit -m "WIP: mid-edit on doc X, picking up on devbox"
git push
```

You can rewrite it later with `git commit --amend` (before pushing again over the WIP) or squash multiple WIPs into a proper commit once the work is done.

> **A WIP commit that's pushed beats a clean working tree that's stranded.**

`git stash` looks tempting but is local-only — the other machine cannot see stashed changes. Never use stash to hand off between machines.

---

## Do You Need Branches?

**Probably not, at least not yet.** Branches earn their keep when:

- You have a long-running experiment you don't want mixed into `main` until it works
- Two lines of work need to progress in parallel without contaminating each other
- You want to gate changes through review (PRs)

For two-machine solo work on the same project, **staying on `main` and syncing via push/pull is simpler and works fine.**

**When to reach for a branch anyway:** if Machine B is going to run some experimental tool for days and might produce garbage you'll throw away, do that work on a `devbox/experiment-name` branch. Push the branch. When it's ready, merge into `main` locally or via PR on GitHub. When it's not ready, delete the branch — `main` is untouched.

```powershell
# On the devbox
git checkout -b devbox/experiment-name
# ... work, commit, push ...
git push -u origin devbox/experiment-name

# Later, on either machine, when ready to merge:
git checkout main
git pull
git merge devbox/experiment-name
git push
git branch -d devbox/experiment-name              # local delete
git push origin --delete devbox/experiment-name   # remote delete
```

---

## One-Time Setup on the Second Machine

Once, on the devbox (or whichever machine is new):

```powershell
git config --global user.name "<your name>"
git config --global user.email "<same email as your main machine>"
git config --global pull.rebase true
```

**Same email on both machines** keeps commit attribution clean — GitHub shows them as the same author. Different emails will make it look like two people are contributing.

Then clone the repo:

```powershell
git clone https://github.com/<org>/<repo>.git
cd <repo>
```

You now have a fully working local repo tracking `origin/main`. Standard rhythm applies from here.

---

## The Anti-Patterns

Don't do these:

1. **Editing on the GitHub website while work is uncommitted on either machine.** Adds a third source of divergence. If you must fix a typo on the web, ensure both machines are clean and pushed first, then pull immediately on whichever machine you next sit at.
2. **Using `git stash` as a handoff.** Stash is invisible to the other machine. WIP-commit instead.
3. **Force-pushing to `main` because "it's just me."** Force-push rewrites shared history. If Machine B has pulled the old history and you force-push new history, Machine B's next pull will fail confusingly. Fix forward with a new commit instead.
4. **Deferring commits "until it's clean."** A commit that never happens can't be synced. Commit early, commit often, push often. Rewrite before pushing if you need to; never after.
5. **Assuming the other machine is up to date.** Always `git pull` first. Always.

---

## Recovery: "I forgot to push on the other machine"

You're on Machine B. You realize Machine A has unpushed work you need. Options:

1. **Best:** Go back to Machine A, `git push`, then return to Machine B and `git pull`.
2. **If you can't get back to Machine A but can access it remotely** (Remote Desktop, SSH): same thing — push from A, pull on B.
3. **If Machine A is inaccessible right now:** proceed on Machine B, but stay on a separate branch (`git checkout -b devbox/resume-work`) so you don't diverge `main`. When you eventually get back to Machine A, push A's work first, then rebase your devbox branch onto the updated `main` and merge.

The scenario you're avoiding: making conflicting edits on `main` on both machines and having to untangle it under time pressure.

---

## Rules of Thumb (Two-Machine Edition)

1. **Always `git status -sb` first.** Every session, every machine.
2. **Never leave a machine with uncommitted or unpushed work.** WIP-commit is fine.
3. **Same email on both machines.** Clean attribution.
4. **`git pull --rebase` is your friend.** Keeps history linear across machines.
5. **`git stash` is single-machine only.** Never use it to hand off.
6. **When in doubt, WIP-commit and push.** You can always clean up later; you cannot recover work that only exists on the machine you can't reach.

````
