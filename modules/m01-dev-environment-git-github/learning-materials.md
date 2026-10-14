# M01 Learning Materials — Dev Environment, Git & GitHub

> **Purpose.** A self-contained study guide for the concepts, commands, and workflows named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> against your own public repository, and record what really happened in your own `write-up.md`
> (created from [`write-up-template.md`](./write-up-template.md)).
>
> **Audience.** Developers with foundational programming knowledge and varying experience with Git,
> GitHub, Node.js, and collaborative development.
>
> **Scope note.** All command output and repository examples below are **illustrative teaching
> examples**, not records of a completed lab. Replace every example with your own real state before
> treating anything here as evidence.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**. The guided
> path takes roughly **65–115 minutes**. M01 is allocated **4 hours** overall according to
> [`brief.md`](./brief.md).

## Contents

1. [Toolchain setup](#1-toolchain-setup)
2. [The Git mental model](#2-the-git-mental-model)
3. [Everyday Git commands](#3-everyday-git-commands)
4. [Undo safely: `revert` vs `reset`](#4-undo-safely-revert-vs-reset)
5. [Branching, merging, and conflicts](#5-branching-merging-and-conflicts)
6. [Repository hygiene: `.gitignore` and `.env.example`](#6-repository-hygiene-gitignore-and-envexample)
7. [GitHub collaboration](#7-github-collaboration)
8. [Worked example: branch → PR → conflict → merge](#8-worked-example-branch--pr--conflict--merge)
9. [Best practices and troubleshooting](#9-best-practices-and-troubleshooting)
10. [References](#10-references)

---

## 1. Toolchain setup

_Estimated: 5–10 minutes for verification (plus installation time)._

The M01 toolchain is four things plus a terminal: **VS Code**, **Git**, **Node through a version
manager**, and **Bruno**. Everything is free (program ground rule).

### 1.1 Git

Install Git 2.x for your OS, then confirm it is on your `PATH`:

```bash
git --version
# git version 2.x.x
```

> This module uses `git init -b main`, which requires **Git 2.28 or newer**. If you are on an older
> release, upgrade Git or run `git init` then `git branch -m main`.

Set the identity attached to every commit (once per machine):

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global --list    # verify
```

### 1.2 Node through a version manager

Use a **Node.js version manager** rather than a standalone installer; it lets you install, switch,
and maintain different Node.js versions per project. M01 requires **Node.js 24 LTS**.

Pick the manager for your OS — you only need one:

| Manager       | Platform                | Install / activate Node 24                      |
| ------------- | ----------------------- | ----------------------------------------------- |
| `nvm`         | macOS / Linux           | `nvm install 24`, `nvm use 24` (reads `.nvmrc`) |
| `fnm`         | macOS / Linux / Windows | `fnm install`, `fnm use` (reads `.nvmrc`)       |
| `nvm-windows` | Windows                 | `nvm install 24`, `nvm use 24`                  |
| `Volta`       | macOS / Linux / Windows | `volta install node@24`                         |

**Pin the version with `.nvmrc`.** Create a `.nvmrc` at the repository root containing:

```text
24
```

Then, from the repository directory (using `nvm`):

```bash
nvm install      # installs the version declared in .nvmrc, if needed
nvm use          # activates it in the current shell
node --version   # expect v24.x.x
npm --version
```

Commit `.nvmrc` so the required version is part of the repository's documented configuration.
Version pinning keeps environments consistent, avoids version-related compatibility bugs, and lets
you switch cleanly between projects.

> **Module scope:** M01 uses plain modern JavaScript. TypeScript arrives in M04. §8 uses `.nvmrc` as
> the shared version declaration; Volta users should just ensure their active version matches it — M01
> does not require a `package.json` solely for version pinning.

### 1.3 VS Code

Install VS Code and get comfortable with the built-in **Source Control** panel. Useful extensions: an
ESLint/Prettier pairing, a Git history viewer, and GitLens-style blame. Learn to stage, commit, and
view diffs from the terminal **first** — the GUI is a convenience, not a substitute for the commands.

### 1.4 Bruno

Bruno is a file-based API-testing client used to send HTTP requests and inspect responses. It joins
the workflow in **M05 (Express)**, when collections begin living in a tracked `bruno/` directory. For
M01 the requirement is only to **install Bruno, confirm it launches, and understand its role**. No
requests or collections are required here.

### 1.5 Terminal basics

You will live in the shell. The essentials you need for this module:

| Task                    | Command                   |
| ----------------------- | ------------------------- |
| Print working directory | `pwd`                     |
| List files              | `ls -la`                  |
| Change directory        | `cd path`                 |
| Make directory          | `mkdir name`              |
| Create/read file        | `touch file`, `cat file`  |
| Locate a program        | `which git`, `which node` |
| Environment variables   | `echo $PATH`, `printenv`  |

---

## 2. The Git mental model

_Estimated: 5 minutes._

Git tracks your project as a sequence of **commits**. Before a change becomes a commit, it passes
through three areas — the "three trees":

```
 working tree              staging area (index)          repository (history)
 (your files on disk)      (what goes in the next         (commits; HEAD points
                            commit)                         at the branch tip)
       |                          |                                |
       |       git add            |        git commit              |
       |------------------------->|------------------------------->|
       |                          |                                |
```

Each command moves data in exactly one direction. Keeping these straight is the whole mental model:

| Command                       | Moves HEAD?             | Updates index?    | Modifies working tree?        | Data direction                              |
| ----------------------------- | ----------------------- | ----------------- | ----------------------------- | ------------------------------------------- |
| `git add <file>`              | No                      | Yes               | No                            | working tree → index                        |
| `git commit`                  | Yes (to the new commit) | No                | No                            | index → repository                          |
| `git restore <file>`          | No                      | No                | Yes (discards unstaged edits) | index → working tree                        |
| `git restore --staged <file>` | No                      | Yes (from `HEAD`) | No                            | repository (`HEAD`) → index                 |
| `git reset --soft HEAD~1`     | Yes (back 1)            | No                | No                            | repository; changes stay staged             |
| `git reset --mixed HEAD~1`    | Yes (back 1)            | Yes (to target)   | No                            | repository → index; changes become unstaged |
| `git reset --hard HEAD~1`     | Yes (back 1)            | Yes               | Yes (discards)                | repository → index → working tree           |

Two consequences worth internalising:

1. **Staging is deliberate.** You choose exactly what each commit contains; it need not include
   every change on disk. So `git restore --staged` is _repository → index_ (it unstages), and only
   `git restore <file>` (or `git reset --hard`) touches the working tree.
2. **A commit is a snapshot, not a diff.** Git _stores_ snapshots but _shows_ diffs, which is why
   `git diff` has three useful forms (§3).

A commit is identified by a **SHA** (e.g. `a1b2c3d`) and carries an author, timestamp, message, and a
pointer to its parent. That parent pointer forms history and makes branching cheap.

---

## 3. Everyday Git commands

_Estimated: 5 minutes._

### 3.1 Starting a repository

```bash
git init                 # start a new repo in the current directory
git clone <url>          # copy an existing remote repo locally (sets up origin)
```

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
trees each file is sitting in.

### 3.3 Recording changes

```bash
git add <file>           # stage one file
git add .                # stage everything under the current directory
git commit -m "feat: add hello-node script"
git commit               # opens an editor for a multi-line message
```

Prefer small, focused commits: each should be explainable in one sentence, and the history should
read like a list of logical steps.

### 3.4 `restore` and `stash`

`git restore` moves changes **between the trees** without adding history; `git stash` shelves
uncommitted work so you can switch context on a clean tree:

```bash
git restore <file>            # discard unstaged changes (reset working tree from index)
git restore --staged <file>   # unstage, keeping the working-tree change

git stash                 # save tracked modifications, clean the working tree
git stash list            # show the stash stack
git stash pop             # reapply the most recent stash and drop it
git stash apply           # reapply but keep it on the stack
```

Use `stash` when you must switch branches or pull but are not ready to commit. It is a shelf, not a
backup — do not leave work there for long.

---

## 4. Undo safely: `revert` vs `reset`

_Estimated: 5 minutes._ This is a named M01 checkpoint concept. The distinction is about **history**:

|                          | `git revert`                                            | `git reset`                                                      |
| ------------------------ | ------------------------------------------------------- | ---------------------------------------------------------------- |
| What it does             | Creates a **new commit** that undoes an existing commit | **Moves the branch pointer** backward to an earlier commit       |
| History                  | Preserved — only ever grows                             | Rewritten — commits after the target are dropped from the branch |
| Safe on shared branches? | **Yes** — push freely                                   | **No** — others may have the dropped commits                     |
| Typical use              | "We shipped a bad change; undo it publicly"             | "Clean up my local, unpushed commits"                            |

```bash
git revert <sha>          # makes a new commit that reverses <sha>
git revert HEAD           # reverse the most recent commit
```

Because it adds history, `revert` is the correct tool for anything already pushed. `reset` has three
modes:

```bash
git reset --soft HEAD~1   # move the branch back 1 commit; KEEP changes staged
git reset HEAD~1          # (default, --mixed) move back 1; UNSTAGE but keep files changed
git reset --hard HEAD~1   # move back 1; DESTROY the changes in index and working tree
```

- `--soft` — undo the commit, keep the work staged (great for re-committing differently).
- `--mixed` (default) — undo the commit and unstage, keep the edits in your files.
- `--hard` — **destructive**: the edits are gone. Use with care.

**Rule of thumb:** if the commit has left your machine, use `revert`; if it is local and unshared,
`reset` is fine. To recover from an accidental `reset --hard`, `git reflog` can find the previous
HEAD — but do not rely on that as a plan.

---

## 5. Branching, merging, and conflicts

_Estimated: 10–15 minutes._

### 5.1 Branches

A branch is just a movable pointer to a commit. Creating one is instant and cheap.

```bash
git branch                      # list branches
git switch -c feat/hello-node   # create AND switch to a new branch
git switch main                 # switch back
git switch -                    # switch to the previous branch
```

Modern Git uses `git switch` for branches and `git restore` for files; `git checkout` still does
both. **Never build directly on `main`** — create a feature branch per unit of work, exactly as the
M01 lab requires.

### 5.2 Merging

```bash
git switch main
git merge feat/hello-node       # bring the branch's commits into main
```

- **Fast-forward** — if `main` has not moved, Git slides the pointer forward. Linear, no merge commit.
- **Three-way merge** — if both branches moved, Git combines them into a **merge commit** with two
  parents.

### 5.3 Conflicts

A **merge conflict** occurs when Git cannot automatically reconcile competing changes, such as when
both branches modify the same lines of a file. Git inserts **conflict markers**:

```text
<<<<<<< HEAD
const greeting = "Hello from main";
=======
const greeting = "Hello from the feature branch";
>>>>>>> feat/hello-node
```

- `<<<<<<< HEAD` — start of the current branch's version.
- `=======` — separator between the competing versions.
- `>>>>>>> feat/hello-node` — start of the incoming branch's version.

**Resolution procedure:**

1. Run `git status` and find conflicted files under **Unmerged paths**.
2. Open each file and review both versions.
3. Decide the correct final content — keep either version or combine them.
4. Remove all conflict markers and save the file.
5. Verify the resolved code, then stage it with `git add <file>`.
6. Complete the merge with `git commit` or `git merge --continue`.

```javascript
const greeting = "Hello from the feature branch"; // final resolved content
```

To abandon a merge before completing it, run `git merge --abort` (it tries to restore the pre-merge
state; a clean working tree makes recovery reliable).

> **Key concept:** Resolving a conflict is primarily a decision and an edit, not an automatic Git
> command. Git identifies the competing changes; _you_ choose the correct final content.

### 5.4 Merge vs rebase (conceptual)

You need the concept now, not the muscle memory. **Merge** keeps the true topology (feature commits
plus a merge commit); **rebase** replays your commits on a new base, producing a **linear** history
with **new commit SHAs**.

|                         | Merge             | Rebase                                    |
| ----------------------- | ----------------- | ----------------------------------------- |
| History shape           | Branchy, truthful | Linear, tidy                              |
| Commit hashes           | Preserved         | Rewritten                                 |
| Safe on shared commits? | Yes               | **No** — never rebase commits others have |

**Golden rule:** rebase your **local, unpushed** commits for a clean history; merge to combine shared
work. When in doubt, merge.

---

## 6. Repository hygiene: `.gitignore` and `.env.example`

_Estimated: 5–10 minutes._

### 6.1 What never belongs in Git

Node projects generate noise and secrets. The M01 `.gitignore` must cover:

| Pattern                 | Why                                                        |
| ----------------------- | ---------------------------------------------------------- |
| `node_modules/`         | Reinstallable from `package.json`; huge; platform-specific |
| `.env`                  | Real secrets (API keys, DB URLs) — **never commit**        |
| `*.db`                  | Local SQLite databases are machine state, not source       |
| `src/generated/prisma/` | The generated Prisma client is built by `prisma generate`  |
| `dist/`                 | Build output, reproduced from source                       |
| `coverage/`             | Test coverage reports, reproduced by running tests         |

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

The `.env.*` line ignores variants (e.g. `.env.test`); the `!.env.example` negation keeps the example
tracked.

### 6.2 `.env.example` — the habit

`.env` holds real configuration and secrets and is **never** committed. Because it is ignored,
teammates cannot see which variables the app needs — so you commit a **`.env.example`** listing every
variable **name** with a safe placeholder.

**Create it directly — never copy `.env`** (copying risks carrying real secrets into a committed
file). Type the names and use placeholders only:

```dotenv
# Copy to .env and fill in real values. Never commit .env.
NODE_ENV=development
PORT=3000
DATABASE_URL=file:./dev.db
```

Before staging, scan `.env.example` for keys, tokens, passwords, or real hosts and replace anything
sensitive with a placeholder. Then stage and verify:

```bash
git add .env.example
git status                                  # .env.example under "Changes to be committed"
git check-ignore -v .env                    # pattern test: prints the rule that would ignore .env
git ls-files --error-unmatch .env.example   # exits 0 once the file is staged
```

- `git check-ignore -v .env` is a **pattern test** — it answers "would this path be ignored?" and
  prints the matching rule, **even if `.env` does not exist on disk**.
- `git ls-files` reads the **index**, so a staged file is recognised before it is committed.

Even with nothing to configure yet, committing `.env.example` builds the habit that secrets live in
`.env` and structure lives in Git. **A committed secret is compromised the moment it is pushed** —
rotating it is the only fix; deleting the file does not remove it from history.

---

## 7. GitHub collaboration

_Estimated: 10 minutes._ This section covers the concepts. The complete executable sequence —
repository creation, remote setup, branching, push, pull request, and merge — is in **§8**.

### 7.1 Remotes and authentication

A **remote** is a named reference to another Git repository; `origin` is the conventional name for
the primary one. `git clone` configures `origin` automatically, or you add it once with
`git remote add origin <url>` (never add it twice). Pushing `main` with `git push -u origin main`
establishes upstream tracking, after which plain `git push` / `git pull` suffice.

Signing in to github.com in a browser does **not** authenticate command-line Git. Choose one method:

| Method                                                     | How it works                                                                                                                                                                                                                                   |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **HTTPS + credential manager** (recommended for beginners) | A credential manager authenticates via browser sign-in and stores credentials. If Git asks for a password over HTTPS, use a **personal access token**, never your account password.                                                            |
| **SSH**                                                    | Generate a key (`ssh-keygen -t ed25519 -C "you@example.com"`), add the **public** key under **GitHub → Settings → SSH and GPG keys**, then use the SSH remote URL (`git@github.com:<username>/<repo>.git`). Test with `ssh -T git@github.com`. |

> **Security:** Never place a personal access token or private key in a remote URL, commit, script,
> or shared document. Follow your organization's credential-management practices.
>
> **Optional:** GitHub CLI (`gh auth login`) is another supported way to authenticate. It is not
> required for M01 — the web interface plus standard Git commands are enough.

### 7.2 Issues

Issues are the unit of planned work: a clear title, a short description, and acceptance criteria.
Later modules and the capstone require you to plan work as Issues before branching.

### 7.3 Pull requests (PRs)

A PR proposes merging one branch into another and is where review happens. A good PR is small,
focused, linked to its Issue, and described by _what changed and why_. Reviewers open **Files
changed**, leave comments, and submit a review (Comment / Approve / Request changes); you address
feedback on the same branch and push — the new commits appear in the PR automatically.

The M01 lab does **not** require a peer review, an assigned partner, a reviewer, an approval, or a
collaborator — you complete it on your own. Understand review anyway, because it is how teams use
PRs: note the distinction between _a rule of the program_ and _a setting of GitHub_. In a personal
public repo with no branch protection, GitHub does **not** force an approval before merging; the
owner can merge directly. Before you merge, review your own PR description, **Files changed** diff,
and commit history. The full flow is in §8.

### 7.4 GitHub Flow

```
main  ──●──────────────●────────────────●──►   (always working)
         \            /
          ●──●──●────●   feature branch → PR → review → merge
```

1. `main` is always in a working state.
2. Branch off `main` for every change.
3. Commit with meaningful messages (Conventional Commits).
4. Open a PR early and request review (optional in M01 — self-review is sufficient).
5. Merge when ready; delete the branch.

### 7.5 Conventional Commits

Commit messages follow a fixed grammar so history is machine-readable:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

| Type       | Use                                     |
| ---------- | --------------------------------------- |
| `feat`     | A new feature                           |
| `fix`      | A bug fix                               |
| `docs`     | Documentation only                      |
| `style`    | Formatting, no code behaviour change    |
| `refactor` | Code change that neither fixes nor adds |
| `perf`     | Performance improvement                 |
| `test`     | Adding or fixing tests                  |
| `build`    | Build system or dependencies            |
| `ci`       | CI configuration                        |
| `chore`    | Maintenance that fits nowhere else      |
| `revert`   | Reverting a previous commit             |

```text
feat: add hello-node script
fix: correct greeting typo
docs: explain revert vs reset in README
```

A breaking change is flagged with `!` (`feat!:`) or a `BREAKING CHANGE:` footer. The M01 lab requires
**3 or more** Conventional Commits on the merged PR.

### 7.6 README basics

Every training README should answer, at minimum: what the project is, how to install it, how to run
it, and how to work on it (branch/PR conventions). It is the front door for anyone — including your
future self.

---

## 8. Worked example: branch → PR → conflict → merge

> **Purpose.** The complete GitHub Flow used in M01, from repository creation to a merged pull
> request. It is an illustrative example, **not evidence of a completed lab**. Perform
> the exercises in your own repository and document your results in `write-up.md`, using
> [`write-up-template.md`](./write-up-template.md). This section is self-contained: it repeats the
> commands you need so you do not have to jump elsewhere to finish.

_Estimated hands-on: 30–60 minutes._

### Prerequisites

Before starting, ensure that:

- Git 2.28 or newer is installed and your Git identity is configured (§1.1).
- Node.js 24 LTS is available through a version manager (§1.2).
- You have a GitHub account and can authenticate Git operations over HTTPS or SSH (§7.1).
- You have a terminal and a code editor. You can complete this lab **independently** — no partner,
  reviewer, or collaborator is required.

Use the terminal for Git operations and the GitHub web interface for repository creation, pull
requests, and merging.

### Step 1 — Create a public repository and publish `main`

**1. Create the repository on GitHub.**

1. Open [github.com](https://github.com) and sign in.
2. Click **+ → New repository**.
3. Name it `hello-node-training`.
4. Select **Public**.
5. Leave README, `.gitignore`, and license initialization **disabled** (start empty).
6. Click **Create repository** and copy the HTTPS URL.

**2. Initialize the local repository.**

```bash
mkdir hello-node-training
cd hello-node-training

git init -b main

printf '24\n' > .nvmrc   # declare Node.js 24
nvm install              # if you use another manager, follow its equivalent
nvm use
node --version
```

**3. Create the initial commit.**

```bash
git add .nvmrc
git commit -m "chore: initial commit"
```

**4. Connect and publish the repository.**

Replace `<username>` with your GitHub username:

```bash
git remote add origin https://github.com/<username>/hello-node-training.git
git push -u origin main
```

**Verify:**

```bash
git remote -v
git status
```

Expected: `origin` points to your GitHub repository, the current branch is `main`, the working tree
is clean, and GitHub shows the initial commit with `.nvmrc`.

> **Important:** Add `origin` only **once**. If Git reports that `origin` already exists, inspect it
> with `git remote -v` instead of adding another.

### Step 2 — Create a feature branch and three Conventional Commits

Develop on a feature branch, never directly on `main`.

**1. Create the feature branch.**

```bash
git switch -c feat/hello-node
```

**2. Create the Node.js script.**

```bash
printf 'const greeting = "hello, node";\nconsole.log(greeting);\n' > hello-node.js
node hello-node.js        # expect: hello, node
```

```bash
git add hello-node.js
git commit -m "feat: add hello-node script"
```

**3. Create the repository hygiene files** (patterns introduced in §6):

```bash
cat > .gitignore <<'EOF'
# Dependencies
node_modules/

# Secrets and local configuration
.env
.env.*
!.env.example

# Local databases
*.db
*.db-journal

# Generated files
src/generated/prisma/
dist/
coverage/

# OS files
.DS_Store
EOF

cat > .env.example <<'EOF'
# Copy to .env and fill in real values. Never commit .env.
NODE_ENV=development
PORT=3000
DATABASE_URL=file:./dev.db
EOF
```

Inspect **before** staging — confirm `.env.example` has no passwords, API keys, access tokens, or
other sensitive information:

```bash
cat .gitignore
cat .env.example
git check-ignore -v .env

git add .gitignore .env.example
git commit -m "chore: add gitignore and env example"
```

**4. Add project documentation.**

```bash
printf '# hello-node\n\nRun with: node hello-node.js\n' > README.md
git add README.md
git commit -m "docs: add README usage note"
```

**5. Verify the branch history.**

```bash
git log --oneline
git status
```

The feature branch now carries three Conventional Commits (`feat: add hello-node script`,
`chore: add gitignore and env example`, `docs: add README usage note`). The initial
`chore: initial commit` belongs to the shared history with `main` and does **not** count.

### Step 3 — Push the feature branch and open a pull request

**1. Publish the feature branch.**

```bash
git push -u origin feat/hello-node
```

**2. Create the pull request** (GitHub web UI):

1. Open your repository on GitHub.
2. Click **Compare & pull request** (or **Pull requests → New pull request**).
3. Set **base: `main`**, **compare: `feat/hello-node`**.
4. Title it `feat: add hello-node script`; describe what changed and how you verified it.
5. Click **Create pull request**.

**3. Review the PR yourself (optional: request a peer).** Before merging, re-read the PR description,
the **Files changed** diff, and the commit list; confirm the branch carries at least three
Conventional Commits and no accidental files. Requesting a peer reviewer (right sidebar →
**Reviewers**) is good professional practice but is **not required for M01** — complete the lab solo
if no reviewer is available.

The PR is created only once; later commits pushed to `feat/hello-node` appear in the same PR.

### Step 4 — Deliberately create a merge conflict

This exercise creates a **reproducible** conflict **without modifying `main` directly**. A merge
conflict occurs when Git cannot automatically reconcile competing changes (the markers are explained
in §5.3).

**1. Modify the greeting on the feature branch.**

```bash
git switch feat/hello-node

printf 'const greeting = "Hello from the feature branch";\nconsole.log(greeting);\n' > hello-node.js
git add hello-node.js
git commit -m "feat: personalize greeting"
```

Call this commit **X**. It will become the common ancestor of the two branches used in the conflict exercise.

**2. Create a second branch from commit X.**

```bash
git switch -c chore/greeting

printf 'const greeting = "Hello, world";\nconsole.log(greeting);\n' > hello-node.js
git add hello-node.js
git commit -m "refactor: reword greeting"
```

**3. Return to the feature branch and make a competing change.**

```bash
git switch feat/hello-node

printf 'const greeting = "Hello from the feature branch (v2)";\nconsole.log(greeting);\n' > hello-node.js
git add hello-node.js
git commit -m "feat: refine greeting wording"
```

Both branches now change the same line, each starting from ancestor X.

**4. Attempt the merge** (while on `feat/hello-node`):

```bash
git merge chore/greeting
```

Git reports a conflict similar to:

```text
CONFLICT (content): Merge conflict in hello-node.js
Automatic merge failed; fix conflicts and then commit the result.
```

Inspect the repository:

```bash
git status
```

Open `hello-node.js`. It contains conflict markers similar to:

```text
<<<<<<< HEAD
const greeting = "Hello from the feature branch (v2)";
=======
const greeting = "Hello, world";
>>>>>>> chore/greeting
console.log(greeting);
```

The merge cannot finish until the conflict is resolved.

### Step 5 — Resolve the conflict and complete the local merge

Resolving requires understanding the intended result, not blindly picking a side. For this exercise,
keep the feature branch's greeting **without** the `(v2)` suffix.

**1. Replace the conflicted file with the resolved content.**

```bash
printf 'const greeting = "Hello from the feature branch";\nconsole.log(greeting);\n' > hello-node.js
```

**2. Verify the resolved file and the merge state.**

```bash
# The resolved file must contain no conflict markers:
grep -nE '^(<<<<<<<|=======|>>>>>>>)' hello-node.js   # expect: no output

node hello-node.js        # expect: Hello from the feature branch
git status                # expect: no files listed under "Unmerged paths"
```

**3. Stage the resolution and complete the merge.**

```bash
git add hello-node.js
git status                # confirm nothing is still listed as unmerged
git diff --cached --check # reports whitespace errors in the staged diff
git commit -m "merge: resolve greeting conflict"
```

> `git diff --cached --check` is a **whitespace check** on the staged diff — it does not by itself
> prove the merge is resolved. Confirm resolution with `git status` (no **Unmerged paths**), by
> inspecting the file for leftover markers, and by running `node hello-node.js`.

This completes the pending merge. Unlike an ordinary commit, the resulting **merge commit has two
parents**: the previous tip of `feat/hello-node` and the tip of `chore/greeting`. It records the
resolved result while preserving both branches' histories.

**4. Inspect the commit graph.**

```bash
git log --oneline --graph --decorate --all
```

The graph should show the two development paths joining at the local merge commit.

> **Key concept:** Git identifies competing changes; the developer determines the correct final
> implementation.

### Step 6 — Push and merge the pull request

**1. Push the updated feature branch.**

```bash
git push
```

Because the branch already tracks `origin/feat/hello-node`, no arguments are needed. Open the
existing pull request — the new commits appear automatically.

**2. Address any review comments (optional).** If you requested a peer reviewer and they left
comments, address them on the same branch, then push — the PR updates automatically:

```bash
# Edit the relevant files, then stage them.
git add <files>
git commit -m "fix: address review feedback"
git push
```

Reply to comments and resolve conversations where appropriate. Peer review is a valuable habit, but
M01 does **not** require a reviewer or an approval before merging.

**3. Merge the pull request using Create a merge commit.** On GitHub choose the **Create a merge
commit** method, then **Merge pull request → Confirm merge**, then **Delete branch**.

- **Create a merge commit** preserves the individual feature commits and the local conflict merge
  commit — use this.
- **Squash and merge** collapses everything into one commit on `main`, losing that history.
- **Rebase and merge** produces linear history and drops the two-parent merge commit.

**4. Understand the two merge commits.**

- **Local merge commit (Step 5):** created resolving the conflict between `feat/hello-node` and
  `chore/greeting`; parents are the previous feature-branch tip and the `chore/greeting` tip.
- **GitHub PR merge commit (Step 6):** created when GitHub merges `feat/hello-node` into `main`;
  parents are the previous `main` tip and the feature-branch tip — which already contains the local
  merge commit. The PR merge therefore brings both into `main`'s history.

**5. Synchronize local `main` and verify history.**

```bash
git switch main
git pull
git log --oneline --graph --decorate --all
git status
```

History should show the feature branch's individual commits, its local conflict-resolution merge
commit, and the separate GitHub PR merge commit.

### Completion checklist

- [ ] The public GitHub repository exists with `main` as its default branch.
- [ ] `.nvmrc` declares Node.js 24.
- [ ] `hello-node.js` runs successfully.
- [ ] `.gitignore` covers the required generated files and local configuration.
- [ ] `.env.example` contains only safe example values.
- [ ] The feature branch includes at least three Conventional Commits.
- [ ] A pull request was opened from `feat/hello-node` into `main`.
- [ ] A deliberate merge conflict was created and resolved.
- [ ] The conflict resolution is recorded in a two-parent merge commit.
- [ ] The pull request was self-reviewed before merging (peer review is optional, not required).
- [ ] The pull request was merged using **Create a merge commit**.
- [ ] Local `main` is synchronized with the merged changes.
- [ ] You can explain, in your own words, the difference between `git revert` and `git reset` (§4).
- [ ] Actual results and evidence are recorded in your own `write-up.md`, following
      [`write-up-template.md`](./write-up-template.md).

> **Remember:** This walkthrough teaches the workflow. Your submitted evidence must come from the
> actual repository, commits, pull request, and conflict resolution you completed — not from the
> illustrative examples here.

---

## 9. Best practices and troubleshooting

_Estimated: 5–7 minutes._

### Best practices

- **Commit small, logical changes** with clear [Conventional Commit](#75-conventional-commits)
  messages, and inspect with `git status` / `git diff --staged` before committing.
- **Never commit secrets.** Keep `.env` ignored; commit `.env.example` with placeholders (§6).
- **Develop on feature branches** and integrate through pull requests.
- **Preserve shared history.** Prefer `git revert` for published commits; reserve `git reset` for
  local, unshared history (§4).
- **Synchronize carefully.** Fetch and understand remote changes before merging or rebasing.

### Troubleshooting

| Symptom                                             | Likely cause                                           | Recommended fix                                                                                                                                                                |
| --------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `git push` rejected (non-fast-forward)              | The remote branch has commits you lack locally.        | `git fetch`, inspect, then merge the remote changes. Rebase only **your own unpushed** commits when appropriate, then push.                                                    |
| Accidentally committed a secret                     | A sensitive file was staged before being ignored.      | If pushed, treat it as **compromised** and rotate/revoke immediately. `git rm --cached <file>`, update `.gitignore`, commit. Removing the file does not erase it from history. |
| Committed to `main` by mistake                      | Changes were committed without a feature branch.       | If the commit is **local and unpushed**, see the recovery note below. If already pushed, do **not** rewrite shared history — use `git revert` (or coordinate with your team).  |
| `node --version` shows the wrong version            | The version manager is inactive or Node 24 is missing. | Check `.nvmrc`, run `nvm install`, `nvm use`, then re-check `node --version`.                                                                                                  |
| Merge conflict markers remain in a file             | The conflict resolution is incomplete.                 | Choose the intended content, remove the markers, `git add` the file, and complete the merge (§5.3, §8 Step 5).                                                                 |
| `detached HEAD`                                     | A specific commit was checked out instead of a branch. | `git switch main`. If you committed while detached, create a branch at the current commit first to preserve the work.                                                          |
| Need to undo the last local commit but keep changes | The latest **unpushed** commit needs correcting.       | `git reset --soft HEAD~1` keeps its changes staged. Confirm the commit is unpushed first.                                                                                      |
| Need to discard local edits                         | Uncommitted changes are no longer wanted.              | `git status` first. `git restore <file>` discards unstaged changes to a tracked file; it does not touch untracked or staged changes.                                           |

### Recovery note — committed to `main` by mistake

Only act if the accidental commit is **local and has not been pushed**. Verify first:

```bash
git status
git log --oneline -3     # confirm the commit is the latest and your working tree is clean
```

If it is unpushed, preserve the work on a branch before moving `main`:

```bash
git switch -c feat/recover-work   # new branch points at the accidental commit
git switch main
git reset --keep HEAD~1           # moves main back one commit, protecting working-tree changes
git switch feat/recover-work
git push -u origin feat/recover-work
```

`git reset` here **rewrites local history**, so use it only after confirming the commit is unpushed
and `main` should move back exactly one commit. It is not a general-purpose fix. If the commit was
**already pushed to a shared `main`**, do not reset and force-push — preserve shared history and use
`git revert` (or coordinate with your team).

**Before running any command that rewrites history or discards changes, inspect the repository
first:**

```bash
git status
git log --oneline --graph --decorate -5
git diff
git diff --staged
```

---

## 10. References

- [Pro Git (book)](https://git-scm.com/book) — the definitive free Git reference.
- [GitHub Docs — Get started](https://docs.github.com/en/get-started) — remotes, PRs, GitHub Flow.
- [Conventional Commits](https://www.conventionalcommits.org/) — the commit grammar.
- [Node.js downloads / version managers](https://nodejs.org/en/download) — Node 24 LTS.
- [Bruno docs](https://docs.usebruno.com) — the API client used from M05 onward.

For the module's deliverable and checkpoints, see [`brief.md`](./brief.md); for an ordered checklist,
see [`tasks.md`](./tasks.md).
