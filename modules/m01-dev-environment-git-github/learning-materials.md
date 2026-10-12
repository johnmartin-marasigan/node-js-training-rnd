# M01 Learning Materials — Dev Environment, Git & GitHub

> **Purpose.** A self-contained study guide for the concepts, commands, workflows, and best practices
> named in [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the
> lab in `brief.md` / `tasks.md` against your own public repository, and record what really happened in
> [`write-up.md`](./write-up.md).
>
> **Audience.** The next trainee starting M01 with no Git/GitHub background.
>
> **Scope note.** All command output and repository examples below are **illustrative teaching
> examples**, not records of a completed lab. Replace every example with your own real state before
> treating anything here as evidence.

## Contents

1. [Toolchain setup](#1-toolchain-setup)
2. [The Git mental model](#2-the-git-mental-model)
3. [Everyday Git commands](#3-everyday-git-commands)
4. [Undo safely: `revert` vs `reset`](#4-undo-safely-revert-vs-reset)
5. [Branching, merging, and conflicts](#5-branching-merging-and-conflicts)
6. [Merge vs rebase (conceptual)](#6-merge-vs-rebase-conceptual)
7. [`.gitignore` and secrets hygiene](#7-gitignore-and-secrets-hygiene)
8. [GitHub collaboration](#8-github-collaboration)
9. [Worked example: branch → PR → conflict → merge](#9-worked-example-branch--pr--conflict--merge)
10. [Best practices and troubleshooting](#10-best-practices-and-troubleshooting)
11. [References](#11-references)

---

## 1. Toolchain setup

The M01 toolchain is four things plus a terminal: **VS Code**, **Git**, **Node through a version
manager**, and **Bruno**. Everything is free (program ground rule).

### 1.1 Git

Install Git 2.x for your OS, then confirm it is on your `PATH`:

```bash
git --version
# git version 2.x.x
```

Set the identity that will be attached to every commit. Do this once per machine:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

Check it:

```bash
git config --global --list
```

### 1.2 Node through a version manager

Do **not** install Node from a bare installer. Use a version manager so the whole cohort can pin the
same runtime. Options:

| Manager | Platform | Typical install |
| --- | --- | --- |
| `nvm` | macOS / Linux | shell script from the nvm repo |
| `fnm` | macOS / Linux / Windows | package manager or installer |
| `nvm-windows` | Windows | installer |
| `Volta` | all | installer |

Install Node **24 LTS**, then pin it for the project with a `.nvmrc` file at the repository root:

```bash
# .nvmrc
24
```

Then, inside the repo:

```bash
nvm use        # or: fnm use  /  volta pin node@24
node --version # v24.x.x
npm --version
```

Why pin? "It works on my machine" is usually a Node-version mismatch. `.nvmrc` makes the version a
committed fact instead of a personal default.

> The program trains **plain modern JavaScript in Week 1**; TypeScript arrives in M04. Nothing in M01
> needs TypeScript.

### 1.3 VS Code

Install VS Code and, at minimum, get comfortable with the built-in **Source Control** panel. Useful
extensions for this stage: an ESLint/Prettier pairing (style), a Git history viewer, and the
**GitLens**-style blame tooling. Learn to stage, commit, and view diffs from the terminal **first** —
the GUI is a convenience, not a substitute for understanding the commands.

### 1.4 Bruno

Bruno is the manual API-testing client used from M03 onward. Install it and verify it opens. In M07+
the collections are plain text committed to `bruno/` in your repo. For M01 you only need it installed
and understood as part of the toolchain; there is no API to call yet.

### 1.5 Terminal basics

You will live in the shell. The essentials you need for this module:

| Task | Command |
| --- | --- |
| Print working directory | `pwd` |
| List files | `ls -la` |
| Change directory | `cd path` |
| Make directory | `mkdir name` |
| Create/read file | `touch file`, `cat file` |
| Where is a program | `which git`, `which node` |
| Environment variables | `echo $PATH`, `printenv` |

---

## 2. The Git mental model

Git tracks your project as a sequence of **commits**. Before a change becomes a commit it passes
through three areas — the "three trees":

```
 working tree           staging area (index)         repository (history)
 (your files on disk)   (what will go in the          (commits, saved forever)
                         next commit)
       |                        |                              |
       |      git add           |        git commit            |
       |----------------------->|----------------------------->|
       |                        |                              |
       |<-----------------------|<-----------------------------|
            git restore            git restore --staged / reset
```

- **Working tree** — the files you edit.
- **Staging area (index)** — a curated snapshot of the changes you intend to commit next. `git add`
  moves changes here.
- **Repository** — the committed history. `git commit` takes a snapshot of the index and records it.

Two consequences worth internalising:

1. **Staging is deliberate.** You choose exactly what each commit contains; a commit does not have to
   include every change on disk.
2. **A commit is a snapshot, not a diff.** Git *stores* full snapshots but *shows* you diffs, which is
   why `git diff` has three useful forms (see §3).

A commit is identified by a **SHA** (e.g. `a1b2c3d`) and carries an author, timestamp, message, and a
pointer to its parent commit. That parent pointer is what forms history and what makes branching cheap.

---

## 3. Everyday Git commands

### 3.1 Starting a repository

```bash
git init                 # start a new repo in the current directory
git clone <url>          # copy an existing remote repo locally
```

`git clone` also sets up `origin` as a remote automatically, and checks out the default branch.

### 3.2 Inspecting state

```bash
git status               # what is staged, unstaged, and untracked
git log --oneline --graph --decorate --all   # history as a graph
git diff                 # working tree vs staging area (unstaged changes)
git diff --staged        # staging area vs last commit (what you're about to commit)
git diff HEAD            # working tree vs last commit (everything since HEAD)
git show <sha>           # one commit's metadata and diff
```

Reading `git status` correctly is the single most valuable habit: it tells you which of the three
trees each file is currently sitting in.

### 3.3 Recording changes

```bash
git add <file>           # stage one file
git add .                # stage everything under the current directory
git commit -m "feat: add hello-node script"
git commit               # opens an editor for a multi-line message
```

Prefer small, focused commits. A good test: each commit should be explainable in one sentence, and
the history should read like a list of logical steps.

### 3.4 `restore` and `stash`

`git restore` moves changes **between the trees** without adding history:

```bash
git restore <file>            # discard unstaged changes (reset working tree from index)
git restore --staged <file>   # unstage, keeping the working-tree change
```

`git stash` shelves uncommitted work so you can switch context on a clean tree:

```bash
git stash                 # save tracked modifications, clean the working tree
git stash list            # show the stash stack
git stash pop             # reapply the most recent stash and drop it
git stash apply           # reapply but keep it on the stack
```

Use `stash` when you need to switch branches or pull, but are not ready to commit. It is a shelf,
not a backup — do not leave work there for long.

---

## 4. Undo safely: `revert` vs `reset`

This is a named M01 checkpoint concept. The distinction is about **history**:

| | `git revert` | `git reset` |
| --- | --- | --- |
| What it does | Creates a **new commit** that undoes an existing commit | **Moves the branch pointer** backward to an earlier commit |
| History | Preserved — only ever grows | Rewritten — commits after the target are dropped from the branch |
| Safe on shared branches? | **Yes** — push freely | **No** — others may have the dropped commits |
| Typical use | "We shipped a bad change; undo it publicly" | "Clean up my local, unpushed commits" |

### `git revert`

```bash
git revert <sha>          # makes a new commit that reverses <sha>
git revert HEAD           # reverse the most recent commit
```

Because it adds history, `revert` is the correct tool for anything already pushed.

### `git reset` — three modes

```bash
git reset --soft HEAD~1   # move the branch back 1 commit; KEEP changes staged
git reset HEAD~1          # (default, --mixed) move back 1; UNSTAGE but keep files changed
git reset --hard HEAD~1   # move back 1; DESTROY the changes in index and working tree
```

- `--soft` — undo the commit, keep the work staged (great for re-committing differently).
- `--mixed` (default) — undo the commit and unstage, keep the edits in your files.
- `--hard` — **destructive**: the edits are gone. Use with care.

**Rule of thumb:** if the commit has left your machine, use `revert`; if it is local and unshared,
`reset` is fine. To undo a `reset --hard`, use `git reflog` to find the previous HEAD and
`git reset --hard` back to it — but do not rely on that as a plan.

---

## 5. Branching, merging, and conflicts

### 5.1 Branches

A branch is just a movable pointer to a commit. Creating one is instant and cheap.

```bash
git branch                      # list branches
git switch -c feat/hello-node   # create AND switch to a new branch
git switch main                 # switch back
git switch -                    # switch to the previous branch
```

Modern Git uses `git switch` for branches and `git restore` for files; `git checkout` still works and
does both.

**Never build directly on `main`.** Create a feature branch per unit of work, exactly as the M01 lab
requires.

### 5.2 Merging

```bash
git switch main
git merge feat/hello-node       # bring the branch's commits into main
```

- **Fast-forward** — if `main` has not moved, Git just slides the pointer forward. Linear history, no
  merge commit.
- **Three-way merge** — if both branches moved, Git combines them and creates a **merge commit** with
  two parents.

### 5.3 Conflicts

A conflict happens when both branches changed the same lines and Git cannot choose. Git writes
**conflict markers** into the file:

```text
<<<<<<< HEAD
greeting = "Hello from main"
=======
greeting = "Hello from the feature branch"
>>>>>>> feat/hello-node
```

Resolution procedure:

1. Open each conflicted file; `git status` lists them under "Unmerged paths".
2. Decide the correct final content and **delete the `<<<<<<<`, `=======`, `>>>>>>>` markers**.
3. Stage the resolved files: `git add <file>`.
4. Finish the merge: `git commit` (Git pre-fills the merge message) or `git merge --continue`.
5. To abandon and start over: `git merge --abort`.

Resolving a conflict is an **edit**, not a command. Git only marks where it could not decide; you
supply the decision.

---

## 6. Merge vs rebase (conceptual)

This stage only needs the concept, not the muscle memory.

- **Merge** keeps the true topology: the feature branch's commits and a merge commit joining it back.
  History is honest and non-destructive.
- **Rebase** replays your commits on top of a new base, producing a **linear** history with **new
  commit SHAs**.

```bash
git rebase main        # replay current branch's commits on top of main (conceptual)
```

The trade-off:

| | Merge | Rebase |
| --- | --- | --- |
| History shape | Branchy, truthful | Linear, tidy |
| Commit hashes | Preserved | Rewritten |
| Safe on shared commits? | Yes | **No** — never rebase commits others have |

**Golden rule:** rebase your **local, unpushed** commits for a clean history; merge to combine shared
work. When in doubt in a training context, merge.

---

## 7. `.gitignore` and secrets hygiene

### 7.1 What never belongs in Git

Node projects generate noise and secrets. The M01 `.gitignore` must cover:

| Pattern | Why |
| --- | --- |
| `node_modules/` | Reinstallable from `package.json`; huge; platform-specific |
| `.env` | Real secrets (API keys, DB URLs) — **never commit** |
| `*.db` | Local SQLite databases are machine state, not source |
| `src/generated/prisma/` | The generated Prisma client is built by `prisma generate` |
| `dist/` | Build output, reproduced from source |
| `coverage/` | Test coverage reports, reproduced by running tests |

A starting `.gitignore`:

```gitignore
# dependencies
node_modules/

# secrets and local config
.env
.env.*
!.env.example

# local database
*.db
*.db-journal

# generated code
src/generated/prisma/

# build output
dist/
coverage/

# OS / editor noise
.DS_Store
```

The `.env.*` line ignores variants (`.env.test`) while the `!.env.example` negation keeps the example
template tracked.

### 7.2 `.env.example` — the habit

Because `.env` is ignored, teammates have no idea which variables the app needs. Commit a
**`.env.example`** listing every variable **name** with safe placeholder values:

```dotenv
# Copy to .env and fill in real values. Never commit .env.
NODE_ENV=development
PORT=3000
DATABASE_URL=file:./dev.db
```

Even when there is nothing to configure yet (as in M01), committing `.env.example` builds the habit
that secrets live in `.env` and structure lives in Git.

**A committed secret is compromised the moment it is pushed.** Rotating it is the only fix; deleting
the file does not remove it from history.

---

## 8. GitHub collaboration

### 8.1 Remotes and authentication

A **remote** is a named URL for another copy of the repo. `origin` is the default.

```bash
git remote -v                          # list remotes
git remote add origin <url>            # connect a local repo to GitHub
git push -u origin main                # push and set upstream tracking
git push                               # later pushes on the tracked branch
git pull                               # fetch + merge from upstream
```

Authentication options (free account):

| Method | How |
| --- | --- |
| **GitHub CLI** | `gh auth login` — also gives you `gh pr`, `gh issue` from the terminal |
| **Credential manager** | Git asks on first push; the OS stores a token |
| **SSH** | Generate a key (`ssh-keygen -t ed25519`), add the public key to GitHub |

### 8.2 Issues

Issues are the unit of planned work. Write a clear title, a short description, and acceptance
criteria. Later modules and the capstone require you to plan work as Issues before branching.

### 8.3 Pull requests (PRs)

A PR proposes merging one branch into another and is where review happens.

```bash
git switch -c feat/hello-node
# ... commit work ...
git push -u origin feat/hello-node
gh pr create --title "feat: add hello-node script" --body "Closes #1"
```

A good PR is small, focused, linked to its Issue, and described by *what changed and why* — not just
*what files moved*. Reviewers leave comments; you push more commits to address them. The M01 lab
requires a **peer review** before merge.

### 8.4 GitHub Flow

The branching model the program uses:

```
main  ──●──────────────●────────────────●──►   (always working)
         \            /                   
          ●──●──●────●   feature branch → PR → review → merge
```

1. `main` is always in a working state.
2. Branch off `main` for every change.
3. Commit with meaningful messages (Conventional Commits — §8.5).
4. Open a PR early; get review.
5. Merge when green and reviewed; delete the branch.

### 8.5 Conventional Commits

Commit messages follow a fixed grammar so history is machine-readable:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Common types:

| Type | Use |
| --- | --- |
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code behaviour change |
| `refactor` | Code change that neither fixes nor adds |
| `perf` | Performance improvement |
| `test` | Adding or fixing tests |
| `build` | Build system or dependencies |
| `ci` | CI configuration |
| `chore` | Maintenance that fits nowhere else |
| `revert` | Reverting a previous commit |

Examples:

```text
feat: add hello-node script
fix: correct greeting typo
docs: explain revert vs reset in README
refactor: extract greeting into a function
```

A breaking change is flagged with `!` (`feat!:`) or a `BREAKING CHANGE:` footer. The M01 lab requires
**3 or more** Conventional Commits on the merged PR.

### 8.6 README basics

Every training repo README should answer, at minimum: what the project is, how to install it, how to
run it, and how to work on it (branch/PR conventions). It is the front door for anyone — including
your future self.

---

## 9. Worked example: branch → PR → conflict → merge

> Illustrative only. This walk-through shows the *reasoning* behind the M01 lab sequence; run the real
> version in your own public repo and capture the real links in `write-up.md`.

### Step 1 — A public repo exists

```bash
git clone git@github.com:<you>/hello-node-training.git
cd hello-node-training
printf '24\n' > .nvmrc          # pin the cohort Node version
```

### Step 2 — Work on a feature branch, not `main`

```bash
git switch -c feat/hello-node
printf 'console.log("hello, node");\n' > hello-node.js
git add hello-node.js
git commit -m "feat: add hello-node script"
```

Add the hygiene files as separate, meaningful commits:

```bash
git add .gitignore .env.example
git commit -m "chore: add gitignore and env example"
```

Two commits so far. A third could be `docs: add README usage note`.

### Step 3 — Open the PR

```bash
git push -u origin feat/hello-node
gh pr create --fill
```

At this point the PR carries 3 Conventional Commits (the M01 requirement).

### Step 4 — Deliberately create a conflict

Create a second branch from `main` that edits the same line, and merge both:

```bash
git switch main
git switch -c chore/greeting
# change the greeting text in hello-node.js on this branch
git commit -am "refactor: change greeting wording"

git switch main
git merge feat/hello-node      # brings in the first change
git merge chore/greeting       # conflicts on the same line
```

Git now prints `CONFLICT (content): Merge conflict in hello-node.js` and inserts the markers shown in
§5.3.

### Step 5 — Resolve, with reasoning

Read both sides, decide the one true greeting for the file, delete the markers, then:

```bash
git add hello-node.js
git commit                     # completes the merge commit
```

The **why** matters more than the commands: you resolved the conflict by deciding intent, not by
mechanically keeping "ours" or "theirs". A `rebase` here would have replayed your commit on top of
the other branch instead of joining them — a linear result, but a rewritten SHA.

### Step 6 — Merge

Once the PR has 3+ commits and a peer review:

```bash
gh pr merge --merge --delete-branch
git switch main && git pull
```

**Success condition:** the merged PR's history contains a merge commit (or a resolved-conflict
commit), and `.gitignore` + `.env.example` are present in the repo.

---

## 10. Best practices and troubleshooting

### Best practices

- Commit small and often; one logical change per commit.
- Write messages for the reader (future you): imperative mood, Conventional Commits.
- Never commit secrets; keep `.env` ignored and `.env.example` tracked.
- Branch per change; never push directly to `main`.
- Pull before you push on shared branches; resolve conflicts locally.
- Prefer `revert` over `reset` on anything already pushed.
- Keep the working tree clean before switching branches (commit or stash).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `git push` rejected (non-fast-forward) | Remote moved ahead | `git pull --rebase` then push; or merge then push |
| Accidentally committed a secret | `.gitignore` too late | Rotate the secret, add to `.gitignore`, remove from tracking (`git rm --cached <file>`) |
| Committed to `main` by mistake | Missing branch | Create a branch at the commit, `git reset --hard` `main` back, push branch, PR |
| `node --version` wrong | Version manager not active | `.nvmrc` + `nvm use` |
| Merge markers left in file | Incomplete resolution | Edit, delete markers, `git add`, commit |
| `detached HEAD` | Checked out a commit not a branch | `git switch main` or create a branch here |
| Want to undo last commit, keep work | — | `git reset --soft HEAD~1` |
| Need to discard all local edits | — | `git restore .` (tracked files only) |

---

## 11. References

From the program's source list:

- [Pro Git (book)](https://git-scm.com/book) — the definitive free Git reference.
- [GitHub Docs — Get started](https://docs.github.com/en/get-started) — remotes, PRs, GitHub Flow.
- [Conventional Commits](https://www.conventionalcommits.org/) — the commit grammar.
- [Node.js downloads / version managers](https://nodejs.org/en/download) — Node 24 LTS.
- [Bruno docs](https://docs.usebruno.com) — the API client used from M03 onward.

For the module's deliverable and checkpoints, see [`brief.md`](./brief.md); for an ordered checklist,
see [`tasks.md`](./tasks.md).
