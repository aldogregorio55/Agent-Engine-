# Git and GitHub Playbook

**Load this file when:** Setting up a new repo, pushing an existing project to GitHub for the first time, or troubleshooting sync issues between local and remote.

---

## The Mental Model

Git keeps your project in **three places** that can drift apart:

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│   Your laptop   │      │  Local git repo │      │  GitHub (remote)│
│   (working      │◄────►│  (.git folder)  │◄────►│  origin/main    │
│    files)       │      │  main branch    │      │                 │
└─────────────────┘      └─────────────────┘      └─────────────────┘
        │                        │                         │
   edit files              git commit                  git push
                                                       git pull
```

1. **Working files** — what you see in your editor
2. **Local git repo** — commits saved locally in the hidden `.git/` folder
3. **GitHub (remote)** — what everyone else sees online

Every problem you'll hit with Git comes from these three drifting out of sync.

---

## The Four Commands That Move Code

| Command | Moves what, where |
|---|---|
| `git add` | Working files → staged for local commit |
| `git commit` | Staged changes → local git repo (new commit) |
| `git push` | Local commits → GitHub |
| `git pull` | GitHub commits → local repo AND working files |

**Push and pull are the only commands that talk to GitHub.** Everything else is local.

---

## Day-to-Day Workflow

```powershell
# Start of session
git pull                    # grab any remote changes first

# Do work in your editor...

# When ready to save a chunk of progress
git status                  # see what changed
git add .                   # stage everything
git commit -m "message"     # save locally

# When ready to publish
git push                    # send to GitHub
```

That's 90% of what you'll ever need.

---

## The One Habit That Prevents Most Problems

**"Pull when you sit down, push when you stand up."**

- **Pull when you sit down** — before making any local changes, run `git pull`. This grabs any remote-side edits (from you on the website, or from a collaborator) and syncs your local repo. Now you're editing the *current* version, not a stale one.
- **Push when you stand up** — after finishing a chunk of work and committing it locally, push it to GitHub so it's backed up and visible.

If you follow this rhythm, sync problems basically never happen.

---

## The Divergence Problem (and Why Git Refuses to Push)

When you edit a file **directly on the GitHub website** and save, GitHub creates a commit on the remote. That commit exists on GitHub but **not on your laptop**. Your local repo has no idea it exists.

If you then make a local edit and try to push, histories diverge:

```
Before your GitHub edit:
   Local:   A ── B ── C (main)
   GitHub:  A ── B ── C (main)
   ✅ In sync

After your GitHub edit:
   Local:   A ── B ── C           (main)
   GitHub:  A ── B ── C ── D      (main)
                          ↑
                    your web edit
   ⚠️ Diverged

Then you commit locally:
   Local:   A ── B ── C ── E      (main)
   GitHub:  A ── B ── C ── D      (main)
   ⚠️ Both sides have work the other doesn't have
```

Git refuses to push with this error:

```
! [rejected] main -> main (fetch first)
Updates were rejected because the remote contains work that you do not have locally.
```

**Git is protecting you.** If it let you push, commit D on GitHub would just... vanish. Your web edit would be lost forever.

---

## The Fix: `git pull --rebase` then `git push`

`git pull --rebase` is a two-step operation:

1. **Fetch** the remote commit (D) down to your local repo
2. **Rebase** — take your local commit (E) and replay it *on top of* D

```
After git pull --rebase:
   Local:   A ── B ── C ── D ── E'     (main)
   GitHub:  A ── B ── C ── D           (main)
   Now local is ahead — safe to push
```

E' is your commit E, re-anchored to sit on top of D instead of C. Same content, new position in history. Then `git push` sends E' up and both sides match.

**When it works cleanly:** Different parts of the file were touched on each side. Git merges automatically.

**When it doesn't:** Same lines were edited on both sides. Git stops and asks you to resolve the conflict manually (edit the file, remove the `<<<<<<<` / `=======` / `>>>>>>>` markers, save, `git add`, `git rebase --continue`).

---

## Recommended Config for Solo Work

Set rebase as the default pull behavior — keeps history linear and clean, no cluttered "Merge branch 'main'" commits:

```powershell
git config --global pull.rebase true
```

After this, plain `git pull` behaves like `git pull --rebase`.

---

## Publishing an Existing Project to GitHub for the First Time

This is the sequence when you already have a folder of work and want to put it on GitHub.

### 1. Create the empty repo on GitHub first

Web UI → New repository → give it a name → **do NOT** initialize with README, .gitignore, or license (you already have those locally or will add them). Copy the HTTPS URL.

### 2. Prepare your local folder

Create a `.gitignore` before your first `git add` — otherwise junk files get committed and you have to clean up later. Minimum contents for most projects:

```
# Python
.venv/
__pycache__/
*.pyc

# Office temp/lock files
~$*

# OS metadata
.DS_Store
Thumbs.db
desktop.ini

# Secrets
.env
.env.local
```

Delete any test/scratch files you don't want in history.

### 3. Initialize and push

```powershell
git init -b main                                    # init on main branch
git add .                                           # stage everything
git status --short | Measure-Object -Line           # sanity check: file count
git status --short | Select-Object -First 30        # sanity check: sample
# PAUSE — verify no .venv/, no secrets, no junk
git commit -m "Initial commit"
git remote add origin https://github.com/<org>/<repo>.git
git push -u origin main
```

The `-u` flag on the first push tells your local `main` to track `origin/main`. After this, plain `git push` and `git pull` work without arguments.

### 4. Sanity check before every first push

Before committing hundreds of files, always inspect what's staged:

```powershell
# Count staged files
git diff --cached --name-only | Measure-Object -Line

# Verify no junk leaked past .gitignore
git diff --cached --name-only | Where-Object { $_ -match '\.venv|__pycache__|\.pyc$|\.env$' }
# Should return nothing

# See top-level folders being committed
git diff --cached --name-only | ForEach-Object { ($_ -split '/')[0] } | Group-Object | Sort-Object Count -Descending
```

---

## Common Warnings You Can Ignore

**"LF will be replaced by CRLF"** — cosmetic Windows line-ending warning. Git will normalize line endings when it stores the file. Nothing is broken.

**"warning: adding embedded git repository"** — you have a nested `.git/` folder somewhere (usually inside a submodule or an accidentally-cloned dependency). Investigate before committing — usually means you need to delete the inner `.git/` folder or handle it as a proper submodule.

---

## Common Problems and Fixes

| Symptom | Cause | Fix |
|---|---|---|
| `! [rejected] main -> main (fetch first)` | Remote has commits you don't have | `git pull --rebase` then `git push` |
| Huge staging list including `.venv/` or `node_modules/` | No `.gitignore` at time of first `git add` | Add gitignore entries, then `git rm --cached -r .venv/` and re-commit |
| Committed a secret to GitHub | Push happened before review | **Rotate the secret immediately.** Removing from history is possible but the secret is already public on GitHub's servers |
| "Please tell me who you are" on first commit | Global git identity not set | `git config --global user.name "Your Name"` and `git config --global user.email "you@example.com"` |
| Push hangs asking for credentials | Auth not configured | Git Credential Manager should open a browser. If not, use a Personal Access Token: GitHub → Settings → Developer settings → Personal access tokens |
| Accidentally committed the wrong thing | Not pushed yet | `git reset --soft HEAD~1` un-does the last commit but keeps the changes staged |
| Accidentally committed the wrong thing | Already pushed | Fix it forward with a new commit that reverts or corrects. Do NOT rewrite history on a shared branch |

---

## Rules of Thumb

1. **Never edit on the GitHub website unless it's a one-line typo fix.** Every web edit creates the divergence problem. Prefer: edit locally → commit → push.
2. **If you do edit on the website, `git pull` immediately before your next local edit.** Otherwise the next commit diverges.
3. **When in doubt, `git status`.** It tells you what's changed locally and whether you're ahead of, behind, or in sync with GitHub.
4. **Commit messages matter later.** Six months from now you'll thank yourself for "Fix workflow diagram: match yaml (add review loop, HITL gate)" over "update readme."
5. **Small, focused commits beat one huge commit.** Easier to review, easier to revert if something breaks.
6. **`git push` is the point of no return** for anything sensitive. Once a secret or client name is on GitHub, assume it's public even in a private repo.

---

## What This Playbook Does NOT Cover

- **Branches and pull requests** — this covers solo work on `main`. Team workflows with feature branches and PRs are a separate topic.
- **Merge conflicts in depth** — the "when it doesn't work cleanly" note above is the summary; resolving complex conflicts is its own skill.
- **Rewriting history** (`git rebase -i`, `git reset --hard`, force-push) — powerful and dangerous. Learn when you need it, not before.
- **Submodules, LFS, worktrees** — advanced features. Ignore until you have a specific reason to need them.
