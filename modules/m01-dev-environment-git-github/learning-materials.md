# M01 Learning Materials — Dev Environment, Git & GitHub

> **Purpose.** A self-contained study guide for the concepts, commands, workflows, and best practices
> named in [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the
> lab in `brief.md` / `tasks.md` against your own public repository, and record what really happened in
> your own `write-up.md` (created from [`write-up-template.md`](./write-up-template.md)).
>
> **Audience.** Developers with foundational programming knowledge and varying levels of experience with Git, GitHub, Node.js, and collaborative software development. The materials reinforce essential concepts while emphasizing practical workflows, engineering conventions, and hands-on application.
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

> This module uses `git init -b main` to create a repository directly on the `main` branch. The `-b`
> flag requires **Git 2.28 or newer**. Check your version above; if you are on an older 2.x release,
> either upgrade Git or run `git init` and then `git branch -m main`.

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

Use a **Node.js version manager** instead of installing Node directly through a standalone installer. A version manager allows developers to install, switch between, and maintain different Node.js versions across projects.

For M01, the required runtime is **Node.js 24 LTS**.

Common version managers include:

| Manager       | Platform                | Typical installation                                 |
| ------------- | ----------------------- | ---------------------------------------------------- |
| `nvm`         | macOS / Linux           | Installation script from the official nvm repository |
| `fnm`         | macOS / Linux / Windows | Package manager or installer                         |
| `nvm-windows` | Windows                 | Windows installer                                    |
| `Volta`       | macOS / Linux / Windows | Official installer                                   |

Choose one version manager appropriate for your operating system. You do not need to install all of them.

**Pin the Node.js version with `.nvmrc`.**

Create a `.nvmrc` file at the repository root containing:

```text
24
```

This declares the Node.js major version expected by the project.

**Using `nvm` (macOS / Linux)**

From the repository directory:

```bash
nvm install
nvm use
node --version
npm --version
```

- `nvm install` installs the Node.js version declared in `.nvmrc`, if necessary.
- `nvm use` activates that version in the current shell.
- `node --version` verifies the active Node.js runtime.
- `npm --version` verifies that npm is available.

Expected Node.js version:

```text
v24.x.x
```

**Using another version manager**

Other managers provide similar functionality, but their version-pinning mechanisms differ:

| Manager       | Version selection                                                                                                                                                                   |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fnm`         | Supports `.nvmrc`; use `fnm install` and `fnm use`.                                                                                                                                 |
| `nvm-windows` | Install Node.js 24 with `nvm install 24`, then activate it using `nvm use 24`.                                                                                                      |
| `Volta`       | Install and select Node.js 24 using `volta install node@24`. For projects with a `package.json`, `volta pin node@24` records the project version in that file rather than `.nvmrc`. |

> **Note:** Section 9 uses `.nvmrc` as the shared version declaration. Developers using Volta should ensure their active Node.js version matches it. M01 does not require creating a `package.json` solely for Volta version pinning.

**Why version pinning matters**

Different Node.js versions may introduce differences in runtime behavior, supported features, and dependency compatibility.

Declaring the expected version helps developers:

- Maintain consistent development environments.
- Reduce version-related compatibility issues.
- Switch between projects that require different Node.js versions.
- Reproduce the environment used by other contributors.

Commit `.nvmrc` to Git so the required version becomes part of the repository's documented configuration rather than an individual developer's machine settings.

> **Module scope:** M01 uses plain modern JavaScript. TypeScript is introduced in M04 and is not required for this module.

### 1.3 VS Code

Install VS Code and, at minimum, get comfortable with the built-in **Source Control** panel. Useful
extensions for this stage: an ESLint/Prettier pairing (style), a Git history viewer, and the
**GitLens**-style blame tooling. Learn to stage, commit, and view diffs from the terminal **first** —
the GUI is a convenience, not a substitute for understanding the commands.

### 1.4 Bruno

Bruno is an API-testing client used to send HTTP requests, inspect responses, and manually verify API behavior during development.

It becomes part of the training workflow starting in **M05 (Express)**, when developers begin working with HTTP APIs.

Unlike tools that primarily store collections in a hosted workspace, Bruno supports file-based API collections that can be maintained alongside application source code. From M05 onward, these collections are stored in the repository's `bruno/` directory and tracked with Git.

For M01, the requirement is limited to:

- Installing Bruno.
- Verifying that the application launches successfully.
- Understanding its role in the development toolchain.

**No API requests or Bruno collections are required in M01.** Practical API testing begins in later modules.

### 1.5 Terminal basics

You will live in the shell. The essentials you need for this module:

| Task                    | Command                   |
| ----------------------- | ------------------------- |
| Print working directory | `pwd`                     |
| List files              | `ls -la`                  |
| Change directory        | `cd path`                 |
| Make directory          | `mkdir name`              |
| Create/read file        | `touch file`, `cat file`  |
| Where is a program      | `which git`, `which node` |
| Environment variables   | `echo $PATH`, `printenv`  |

---

## 2. The Git mental model

Git tracks your project as a sequence of **commits**. Before a change becomes a commit it passes
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

The forward arrows are the only paths shown, because they are unambiguous: `git add` copies from the
working tree into the index, and `git commit` records the index as a new commit. The reverse
operations are described in the table below — each moves data in exactly one direction, and it is
important to keep them straight.

| Command                       | Moves HEAD?             | Updates index?    | Modifies working tree?        | Data direction                              |
| ----------------------------- | ----------------------- | ----------------- | ----------------------------- | ------------------------------------------- |
| `git add <file>`              | No                      | Yes               | No                            | working tree → index                        |
| `git commit`                  | Yes (to the new commit) | No                | No                            | index → repository                          |
| `git restore <file>`          | No                      | No                | Yes (discards unstaged edits) | index → working tree                        |
| `git restore --staged <file>` | No                      | Yes (from `HEAD`) | No                            | **repository (`HEAD`) → index**             |
| `git reset --soft HEAD~1`     | Yes (back 1)            | No                | No                            | repository; changes stay staged             |
| `git reset --mixed HEAD~1`    | Yes (back 1)            | Yes (to target)   | No                            | repository → index; changes become unstaged |
| `git reset --hard HEAD~1`     | Yes (back 1)            | Yes               | Yes (discards)                | repository → index → working tree           |

Three things to keep distinct:

- **Moving HEAD** — `git reset --soft/--mixed/--hard HEAD~1` moves the branch pointer (which is what
  `HEAD` follows) back one commit.
- **Updating the index** — `git restore --staged <file>` copies the file's content from `HEAD` into
  the index (it _unstages_); it never writes the working tree. `git reset --mixed` also rewrites the
  index, restoring it to the target commit.
- **Modifying the working tree** — `git restore <file>` writes the working tree from the index, and
  `git reset --hard` overwrites the working tree to match the target commit (losing local edits).

So `git restore --staged` is **repository (`HEAD`) → index**, not index → working tree. Only
`git restore <file>` (and `git reset --hard`) touch the working tree.

- **Working tree** — the files you edit.
- **Staging area (index)** — a curated snapshot of the changes you intend to commit next. `git add`
  moves changes here.
- **Repository** — the committed history. `git commit` takes a snapshot of the index and records it.

Two consequences worth internalising:

1. **Staging is deliberate.** You choose exactly what each commit contains; a commit does not have to
   include every change on disk.
2. **A commit is a snapshot, not a diff.** Git _stores_ full snapshots but _shows_ you diffs, which is
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

|                          | `git revert`                                            | `git reset`                                                      |
| ------------------------ | ------------------------------------------------------- | ---------------------------------------------------------------- |
| What it does             | Creates a **new commit** that undoes an existing commit | **Moves the branch pointer** backward to an earlier commit       |
| History                  | Preserved — only ever grows                             | Rewritten — commits after the target are dropped from the branch |
| Safe on shared branches? | **Yes** — push freely                                   | **No** — others may have the dropped commits                     |
| Typical use              | "We shipped a bad change; undo it publicly"             | "Clean up my local, unpushed commits"                            |

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

A **merge conflict** occurs when Git cannot automatically reconcile changes between branches, such as when both branches modify the same lines of a file.

When this happens, Git inserts **conflict markers** to identify the competing changes.

For example, suppose `main` and `feat/hello-node` modify the same JavaScript declaration:

```text
<<<<<<< HEAD
const greeting = "Hello from main";
=======
const greeting = "Hello from the feature branch";
>>>>>>> feat/hello-node
```

The markers indicate:

- `<<<<<<< HEAD` — The beginning of the current branch's version.
- `=======` — The separator between the conflicting versions.
- `>>>>>>> feat/hello-node` — The end of the incoming branch's version.

**Resolution procedure:**

1. Run `git status` to identify conflicted files listed under **Unmerged paths**.
2. Open each conflicted file and review both versions.
3. Decide the correct final content. You may keep either version or combine the changes, depending on the intended behavior.
4. Remove all conflict markers (`<<<<<<<`, `=======`, and `>>>>>>>`) and save the file.
5. Verify the resolved code, then stage the file using `git add <file>`.
6. Complete the merge using `git commit` or `git merge --continue`.

For example, the final resolved JavaScript code might be:

```javascript
const greeting = "Hello from the feature branch";
```

If you need to abandon the merge before completing it, use:

```bash
git merge --abort
```

This attempts to restore the repository to its pre-merge state. Uncommitted changes present before the merge may complicate recovery, so keep the working tree clean before starting a merge.

> **Key concept:** Resolving a conflict is primarily a decision and an edit, not an automatic Git command. Git identifies the conflicting changes, but the developer determines the correct final content.

Section 9 demonstrates this process through a deliberate conflict between `main` and `feat/hello-node`, followed by resolution, verification, and merge completion.

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

|                         | Merge             | Rebase                                    |
| ----------------------- | ----------------- | ----------------------------------------- |
| History shape           | Branchy, truthful | Linear, tidy                              |
| Commit hashes           | Preserved         | Rewritten                                 |
| Safe on shared commits? | Yes               | **No** — never rebase commits others have |

**Golden rule:** rebase your **local, unpushed** commits for a clean history; merge to combine shared
work. When in doubt in a training context, merge.

---

## 7. `.gitignore` and secrets hygiene

### 7.1 What never belongs in Git

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

The `.env.*` line ignores variants (`.env.test`) while the `!.env.example` negation keeps the example
template tracked.

### 7.2 `.env.example` — the habit

Because `.env` is ignored, teammates have no idea which variables the app needs. The fix is a
**`.env.example`** file that lists every variable **name** with a safe placeholder value. This section
shows how to create it, what belongs in it, and how to prove it is tracked while `.env` is not.

**1. Create it directly — never copy `.env`.** Copying `.env` risks carrying real secrets into a file
you intend to commit. Create a fresh file and type the variable names yourself:

```bash
touch .env.example
# open .env.example in your editor and add the names below
```

**2. Safe contents — placeholders and non-sensitive configuration only:**

```dotenv
# Copy to .env and fill in real values. Never commit .env.
NODE_ENV=development
PORT=3000
DATABASE_URL=file:./dev.db
```

Every value above is a placeholder or local-only, non-secret configuration. There are no API keys,
tokens, passwords, or credentialed connection strings here.

**3. `.env` vs `.env.example`.** `.env` is your local configuration file. It may contain secrets (API
keys, database URLs with credentials) and is **never** committed. `.env.example` documents the _names_
of the variables the app requires, with safe placeholders, so teammates know what to set — without any
credential being exposed.

**4. Inspect before staging.** Open `.env.example` and scan every line for secrets (keys, tokens,
passwords, real hosts). Replace anything sensitive with a placeholder **before** you run `git add`.
This step is the habit the whole module is building: `.env.example` should be safe to publish.

**5. The `.gitignore` rules that make this work** (matching §7.1):

```gitignore
.env
.env.*
!.env.example
```

`.env` and its variants are ignored, and the `!.env.example` negation re-includes the example template.

**6. Stage the example:**

```bash
git add .env.example
```

**7. Verify — and understand what each command actually proves:**

```bash
git check-ignore -v .env                    # prints the ignore rule that would match the path
git status                                  # .env.example shows under "Changes to be committed"
git ls-files --error-unmatch .env.example   # exits 0 when the file is in the index
git status --ignored                        # lists actual ignored files; .env only appears if it exists
```

- `git check-ignore -v .env` is a **pattern test**: it answers "would this path be ignored?" and
  prints the matching rule. It works **even if `.env` does not exist on disk** — it is not looking for
  a file, only matching the path against the ignore rules. (It exits non-zero when nothing matches;
  `.env.example` is covered by the `!` negation, so it is _not_ ignored.)
- `git status --ignored` lists **real ignored files present in the working tree**. `.env` shows up
  here **only if a real `.env` file exists**. If you never created one, it will not appear, even though
  `git check-ignore` still confirms the rule. To see it in this output, create a local `.env`
  (`touch .env`) first.
- `git ls-files` reads the **index**, so a file staged with `git add` is recognized **before** it is
  committed. `git ls-files --error-unmatch .env.example` exits 0 as soon as `.env.example` is staged.

Even when there is nothing to configure yet (as in M01), creating and committing `.env.example` builds
the habit that secrets live in `.env` and structure lives in Git.

**A committed secret is compromised the moment it is pushed.** Rotating it is the only fix; deleting
the file does not remove it from history.

---

## 8. GitHub collaboration

### 8.1 Creating a repo, remotes and authentication

A **remote** is a named reference to another Git repository, usually hosted on a service such as GitHub. `origin` is the conventional name for the primary remote.

This section introduces repository creation, remote configuration, and authentication. The complete hands-on workflow is demonstrated in §9.

**Step 1 — Create a public repository on GitHub.**

1. Sign in at [github.com](https://github.com) and select **+ → New repository**.
2. Enter `hello-node-training` as the repository name.
3. Select **Public**.
4. Do not initialize the repository with a README, `.gitignore`, or license. Leave these options disabled to start with an empty remote repository.
5. Click **Create repository** and copy its HTTPS URL, for example `https://github.com/<username>/hello-node-training.git`.

**Step 2 — Initialize the local repository and publish `main`.**

Open your terminal, then create and enter the project directory:

```bash
mkdir hello-node-training
cd hello-node-training
git init -b main
```

Create `.nvmrc` to declare the project's Node.js version, then make the initial commit:

```bash
printf '24\n' > .nvmrc
git add .nvmrc
git commit -m "chore: initial commit"
```

Connect the local repository to GitHub and publish `main`:

```bash
git remote add origin https://github.com/<username>/hello-node-training.git
git push -u origin main
```

Replace `<username>` with your GitHub username.

The commands serve different purposes:

- `git remote add origin <url>` registers the remote repository under the name `origin`. Run it only once for a given remote name.
- `git push -u origin main` publishes `main` and establishes its upstream tracking relationship.
- Subsequent `git push` and `git pull` commands can use the configured upstream branch without repeating the remote and branch names.

Verify the configuration:

```bash
git remote -v
git branch -vv
git status
```

`git remote -v` should display `origin` with fetch and push URLs. `git branch -vv` should show that `main` tracks `origin/main`, and `git status` should report a clean working tree.

**Cloning an existing repository**

If a repository already exists and contains project history, use `git clone` instead of initializing another local repository:

```bash
git clone https://github.com/<username>/hello-node-training.git
cd hello-node-training
```

Cloning downloads the repository, creates the local working directory, and automatically configures `origin`.

Do not combine `git clone` with the `git init` and `git remote add` workflow above. They are alternative starting points.

**Step 3 — Authenticate Git operations with GitHub.**

Signing in to github.com through a browser does not automatically guarantee that command-line Git operations are authenticated.

GitHub supports two common approaches for this module:

| Method                                                     | How it works                                                                                                                                                                                                                                                                |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **HTTPS + credential manager** (recommended for beginners) | A configured credential manager can authenticate through a browser-based sign-in flow and securely store credentials for future Git operations. If Git requests a password directly over HTTPS, use a GitHub personal access token (PAT), not your GitHub account password. |
| **SSH**                                                    | Authenticate using an SSH key pair. Register the public key with your GitHub account, then use the repository's SSH remote URL instead of HTTPS.                                                                                                                            |

**Option A — HTTPS authentication**

The remote URL uses HTTPS:

```text
https://github.com/<username>/hello-node-training.git
```

When you run `git push`, your configured Git credential helper may open a browser for authentication or prompt for credentials.

Depending on your operating system and Git installation, you may use Git Credential Manager or an operating-system credential helper.

If Git explicitly asks for a username and password:

- **Username:** Your GitHub username.
- **Password:** A personal access token with the required repository permissions.

GitHub does not accept account passwords for Git operations over HTTPS.

> **Security:** Never place a personal access token directly in a Git remote URL, commit, script, or shared documentation. Follow your organization's approved credential-management practices.

**Option B — SSH authentication**

Generate an SSH key if you do not already have a suitable one:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Follow the prompts to save the key and configure a passphrase.

Add the contents of the **public key** (`~/.ssh/id_ed25519.pub`, if you used the default filename) to **GitHub → Settings → SSH and GPG keys**.

Never upload or share the private key.

Test authentication:

```bash
ssh -T git@github.com
```

A successful authentication should produce a message confirming that GitHub recognizes your account, although GitHub does not provide shell access.

For SSH, use this remote URL format:

```text
git@github.com:<username>/hello-node-training.git
```

If you initially configured HTTPS and want to switch to SSH, update the existing remote rather than adding a second `origin`:

```bash
git remote set-url origin git@github.com:<username>/hello-node-training.git
```

**Optional — GitHub CLI (`gh`)**

GitHub CLI is not required for M01. Developers who already use it can authenticate with:

```bash
gh auth login
```

Alternatively, when working inside an initialized local repository that has at least one commit and no `origin` remote, GitHub CLI can create and publish the repository:

```bash
gh repo create hello-node-training --public --source=. --remote=origin --push
```

This is an alternative to creating the repository through GitHub's web interface. Do not run both workflows for the same repository.

For M01, **use the GitHub web interface and standard Git commands** so that repository initialization, remote configuration, authentication, and publishing are practiced explicitly.

The complete sequence continues in §9, including feature branching, Conventional Commits, pull requests, conflict resolution, peer review, and merging.

### 8.2 Issues

Issues are the unit of planned work. Write a clear title, a short description, and acceptance
criteria. Later modules and the capstone require you to plan work as Issues before branching.

### 8.3 Pull requests (PRs)

A PR proposes merging one branch into another and is where review happens. For M01 we open and manage
PRs through the **GitHub web UI** (the full click-by-click walkthrough is in §9). In outline:

1. Push your feature branch: `git push -u origin feat/hello-node`.
2. On github.com, click the **"Compare & pull request"** banner for the branch (or **Pull requests →
   New pull request**), set **base: `main`** and **compare: `feat/hello-node`**, then click **Create
   pull request**.
3. **Request a reviewer:** right sidebar → **Reviewers** → **gear icon** → pick a peer. The list is
   limited to people with access to the repo, so add your peer as a **collaborator** first
   (**Settings → Collaborators and teams**).
4. **Respond to feedback:** the reviewer opens **Files changed**, leaves comments, and submits a
   review (Comment / Approve / Request changes). Address the comments, commit, and `git push` to the
   same branch — the new commits appear in the PR automatically. Reply and click **Resolve
   conversation**.
5. **Merge:** once the PR carries 3+ Conventional Commits and a peer review, click **Merge pull
   request** → **Confirm merge**, then **Delete branch**.

A good PR is small, focused, linked to its Issue, and described by _what changed and why_ — not just
_what files moved_.

The M01 lab requires a **peer review** before merge, but note the distinction between a _rule of the
program_ and a _setting of GitHub_: in a personal public repository with no branch protection, GitHub
does **not** force an approval before merging — the owner can merge regardless. The peer review is a
**training requirement**, enforced by the program, not necessarily by GitHub. Whether approval is
_required_ to merge, who can be _requested_ as a reviewer, and who can _submit_ an approving review all
depend on repository permissions and settings (branch protection / rulesets). M01 does not ask you to
configure branch protection, so read the review state yourself and merge only after your peer has
reviewed. See §9 for the approval-caveat when new commits are pushed.

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

> **Purpose.** This walkthrough demonstrates the complete GitHub Flow used in M01, from repository creation to a reviewed and merged pull request. It is an illustrative example, not evidence of a completed lab. Perform the actual exercises in your own repository and document your results in `write-up.md`, using `write-up-template.md`.

### Prerequisites

Before starting, ensure that:

- Git 2.28 or newer is installed and your Git identity is configured (§1.1).
- Node.js 24 LTS is available through a version manager (§1.2).
- You have a GitHub account and can authenticate Git operations through HTTPS or SSH (§8.1).
- You have access to a terminal and a code editor.
- A peer is available to review your pull request.

The example uses the terminal for Git operations and the GitHub web interface for repository creation, pull requests, reviews, and merging.

### Step 1 — Create a public repository and publish `main`

**1. Create the repository on GitHub.**

1. Open [github.com](https://github.com) and sign in.
2. Click **+ → New repository**.
3. Enter `hello-node-training` as the repository name.
4. Select **Public**.
5. Leave README, `.gitignore`, and license initialization disabled.
6. Click **Create repository**.
7. Copy the repository's HTTPS URL.

**2. Initialize the local repository.**

Open your terminal and run:

```bash
mkdir hello-node-training
cd hello-node-training

git init -b main
```

Create `.nvmrc` to declare the required Node.js version:

```bash
printf '24\n' > .nvmrc
```

If you use `nvm`, activate the declared version:

```bash
nvm install
nvm use
node --version
```

If you use another version manager, follow its equivalent commands.

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

Verify the connection:

```bash
git remote -v
git status
```

Expected results:

- `origin` points to your GitHub repository.
- The current branch is `main`.
- The working tree is clean.
- GitHub displays the initial commit containing `.nvmrc`.

> **Important:** Add `origin` only once. If Git reports that `origin` already exists, inspect the current remote with `git remote -v` instead of adding another.

### Step 2 — Create a feature branch and three Conventional Commits

Development should happen on a feature branch rather than directly on `main`.

**1. Create the feature branch.**

```bash
git switch -c feat/hello-node
```

**2. Create the Node.js script.**

```bash
printf 'const greeting = "hello, node";\nconsole.log(greeting);\n' > hello-node.js
```

The script contains:

```javascript
const greeting = "hello, node";
console.log(greeting);
```

Verify it:

```bash
node hello-node.js
```

Expected output:

```text
hello, node
```

Create the first Conventional Commit:

```bash
git add hello-node.js
git commit -m "feat: add hello-node script"
```

**3. Create the repository hygiene files.**

Create `.gitignore` using the patterns introduced in §7:

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
```

Create `.env.example` directly, using only safe placeholders:

```bash
cat > .env.example <<'EOF'
# Copy to .env and fill in real values. Never commit .env.
NODE_ENV=development
PORT=3000
DATABASE_URL=file:./dev.db
EOF
```

Inspect both files before staging:

```bash
cat .gitignore
cat .env.example
git check-ignore -v .env
```

Confirm that `.env.example` contains no passwords, API keys, access tokens, or other sensitive information.

Create the second Conventional Commit:

```bash
git add .gitignore .env.example
git commit -m "chore: add gitignore and env example"
```

**4. Add project documentation.**

Create a basic README:

```bash
printf '# hello-node\n\nRun with: node hello-node.js\n' > README.md
```

Create the third Conventional Commit:

```bash
git add README.md
git commit -m "docs: add README usage note"
```

**5. Verify the branch history.**

```bash
git log --oneline
git status
```

At this point, the feature branch contains three new Conventional Commits:

- `feat: add hello-node script`
- `chore: add gitignore and env example`
- `docs: add README usage note`

The initial `chore: initial commit` belongs to the shared history with `main` and is not counted as one of the three feature-branch commits.

### Step 3 — Push the feature branch and open a pull request

**1. Publish the feature branch.**

```bash
git push -u origin feat/hello-node
```

**2. Create a pull request using GitHub Web UI.**

1. Open your repository on GitHub.
2. Click **Compare & pull request**, or navigate to **Pull requests → New pull request**.
3. Set the base branch to `main`.
4. Set the compare branch to `feat/hello-node`.
5. Enter the title `feat: add hello-node script`.
6. Describe what changed and how you verified it.
7. Click **Create pull request**.

The pull request proposes merging:

```text
feat/hello-node → main
```

**3. Request peer review.**

In the pull request's right sidebar:

1. Find **Reviewers**.
2. Select a peer reviewer.
3. If the peer is not selectable, arrange appropriate repository access, such as inviting them as a collaborator.
4. Ask the reviewer to inspect the changes and provide feedback.

> **Training requirement:** Peer review must be completed before merging. GitHub may not enforce this automatically in a personal public repository without branch protection.

The PR is created only once. Subsequent commits pushed to `feat/hello-node` will appear in the same pull request.

### Step 4 — Deliberately create a merge conflict

A merge conflict occurs when Git cannot automatically reconcile competing changes.

This exercise creates a reproducible conflict without modifying `main` directly.

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
```

Modify the greeting:

```bash
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

Both branches now have different modifications to the same line, starting from their shared ancestor X.

**4. Attempt the merge.**

While on `feat/hello-node`, run:

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

The markers identify competing versions:

- `HEAD` represents the current `feat/hello-node` branch.
- The other section represents the incoming `chore/greeting` branch.
- `=======` separates the competing changes.

The merge cannot finish until the conflict is resolved.

### Step 5 — Resolve the conflict and complete the local merge

Resolving a conflict requires understanding the intended result rather than automatically choosing one branch's version.

For this exercise, retain the feature branch's greeting without the `(v2)` suffix.

**1. Replace the conflicted file with the resolved content.**

```bash
printf 'const greeting = "Hello from the feature branch";\nconsole.log(greeting);\n' > hello-node.js
```

The resolved file is:

```javascript
const greeting = "Hello from the feature branch";
console.log(greeting);
```

**2. Verify the result.**

```bash
node hello-node.js
git status
```

Expected Node.js output:

```text
Hello from the feature branch
```

**3. Stage the resolution and complete the merge.**

```bash
git add hello-node.js
git diff --cached --check
git commit -m "merge: resolve greeting conflict"
```

This commit completes the pending merge.

Unlike an ordinary commit, the resulting merge commit has **two parents**:

1. The previous tip of `feat/hello-node`.
2. The tip of `chore/greeting`.

The merge commit records the resolved result while preserving both branches' histories.

**4. Inspect the commit graph.**

```bash
git log --oneline --graph --decorate --all
```

The graph should show the two development paths joining at the local merge commit.

> **Key concept:** Resolving a conflict does not mean Git has chosen the correct business logic. Git identifies competing changes; the developer determines the correct final implementation.

### Step 6 — Push, complete peer review, and merge

**1. Push the updated feature branch.**

```bash
git push
```

Because the branch already tracks `origin/feat/hello-node`, no additional remote or branch argument is required.

Open the existing pull request on GitHub. The additional commits should appear automatically.

**2. Complete the review process.**

The reviewer should:

1. Open the pull request's **Files changed** tab.
2. Inspect the implementation and commit history.
3. Leave comments or request changes where appropriate.
4. Submit a review.

If changes are requested, address them on the same feature branch:

```bash
# Edit the relevant files, then stage them.
git add <files>
git commit -m "fix: address review feedback"
git push
```

Replace `<files>` with the actual changed file paths.

Respond to the review comments and resolve conversations where appropriate.

If a previous approval was dismissed after new commits were pushed, request another review.

**Do not merge until the required peer review is complete.**

**3. Merge the pull request through GitHub.**

On GitHub, select the **Create a merge commit** merge method, then click **Merge pull request → Confirm merge**.

For this exercise:

- **Create a merge commit** preserves the individual feature-branch commits and the local conflict-resolution merge commit.
- **Squash and merge** creates a single combined commit on `main`, so the individual feature-branch commits and local merge commit are not preserved in `main`'s history.
- **Rebase and merge** replays commits into a linear history and does not preserve the local merge commit's two-parent topology.

Use **Create a merge commit** to retain the history needed for the M01 exercise.

After merging, click **Delete branch** if GitHub offers it.

**4. Understand the two merge commits.**

The exercise produces two distinct merge commits.

**Local merge commit — Step 5**

Created when resolving the conflict between `feat/hello-node` and `chore/greeting`.

Its two parents are the previous feature-branch tip and the `chore/greeting` tip.

**GitHub PR merge commit — Step 6**

Created when GitHub merges `feat/hello-node` into `main`.

Its two parents are the previous `main` tip and the feature-branch tip.

Because the feature branch already contains the local merge commit, the completed PR merge brings both into `main`'s history.

**5. Synchronize the local `main` branch.**

```bash
git switch main
git pull
```

Verify the resulting history:

```bash
git log --oneline --graph --decorate --all
git status
```

The history should show the feature branch's individual commits, its local conflict-resolution merge commit, and the separate GitHub PR merge commit.

### Completion checklist

Before considering the exercise complete, verify:

- [ ] The public GitHub repository exists and has `main` as its default branch.
- [ ] `.nvmrc` declares Node.js 24.
- [ ] `hello-node.js` runs successfully.
- [ ] `.gitignore` covers the required generated files and local configuration.
- [ ] `.env.example` contains only safe example values.
- [ ] The feature branch includes at least three Conventional Commits.
- [ ] A pull request was opened from `feat/hello-node` into `main`.
- [ ] A deliberate merge conflict was created and resolved.
- [ ] The conflict resolution is recorded in a two-parent merge commit.
- [ ] Peer review was completed before the pull request was merged.
- [ ] The pull request was merged using **Create a merge commit**.
- [ ] The local `main` branch is synchronized with the merged changes.
- [ ] The actual results and supporting evidence are recorded in your own `write-up.md`, following `write-up-template.md`.

> **Remember:** This walkthrough teaches the workflow. Your submitted evidence must come from the actual repository, commits, pull request, review, and conflict resolution you completed—not from the illustrative examples in this document.

---

## 10. Best practices and troubleshooting

### Best practices

- **Commit small, logical changes.** Each commit should represent one meaningful unit of work.
- **Use Conventional Commits.** Write clear, imperative commit messages such as `feat: add login endpoint` or `fix: handle invalid input`.
- **Never commit secrets.** Keep `.env` and other sensitive configuration files ignored. Commit `.env.example` with safe placeholder values instead.
- **Develop on feature branches.** Create a branch for each change and use pull requests to integrate changes into `main`.
- **Inspect before committing.** Use `git status` and `git diff --staged` to review what will be included in a commit.
- **Synchronize shared branches carefully.** Fetch remote changes and understand any differences before merging or rebasing.
- **Preserve shared history.** Prefer `git revert` for undoing published commits. Reserve `git reset` for local, unshared history.
- **Keep the working tree organized.** Commit or stash unfinished changes before switching contexts when necessary.
- **Review before merging.** Ensure the pull request has the required commits, passes applicable checks, and receives peer review.

### Troubleshooting

| Symptom                                             | Likely cause                                                                | Recommended fix                                                                                                                                                                                                                                                            |
| --------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `git push` rejected (non-fast-forward)              | The remote branch contains commits missing locally.                         | Run `git fetch`, inspect the differences, then merge the remote changes. Rebase only your own unshared commits when appropriate. Push again after resolving any conflicts.                                                                                                 |
| Accidentally committed a secret                     | A sensitive file was staged and committed before being ignored.             | If pushed, treat the secret as compromised and rotate or revoke it immediately. Remove the file from tracking using `git rm --cached <file>`, update `.gitignore`, and commit the correction. Removing the file does not erase it from Git history.                        |
| Committed to `main` by mistake                      | Changes were committed without creating a feature branch.                   | If the commit is **local and unpushed**, create a feature branch at the current commit, return to `main`, and safely restore it to the previous commit. If already pushed, do not rewrite shared history; coordinate with your team and use `git revert` when appropriate. |
| `node --version` shows the wrong version            | The version manager is not active or the required version is not installed. | Check `.nvmrc`, install the required version if necessary (`nvm install`), then run `nvm use` and verify with `node --version`.                                                                                                                                            |
| Merge conflict markers remain in a file             | The conflict resolution is incomplete.                                      | Inspect the conflicting changes, choose the intended final content, remove the markers, stage the resolved files with `git add`, and complete the merge.                                                                                                                   |
| `detached HEAD`                                     | A specific commit was checked out instead of a branch.                      | Use `git switch main` to return to a branch. If you created commits while detached, create a branch at the current commit first to preserve them.                                                                                                                          |
| Need to undo the last local commit but keep changes | The latest unpushed commit needs correction.                                | Run `git reset --soft HEAD~1` to undo the commit while keeping its changes staged.                                                                                                                                                                                         |
| Need to discard local edits                         | Uncommitted changes are no longer needed.                                   | Inspect `git status` first. Use `git restore <file>` to discard unstaged changes in a specific tracked file. This does not remove untracked files or discard staged changes.                                                                                               |

### Recovery example — committed to `main` by mistake

Suppose you accidentally committed a feature directly to `main` instead of creating a feature branch.

**Scenario:** The commit exists only locally and has **not been pushed**.

First, verify your current state:

```bash
git status
git log --oneline -3
```

Ensure your working tree is clean and confirm that the accidental commit is the latest commit.

**Step 1 — Preserve the commit on a feature branch.**

```bash
git switch -c feat/recover-work
```

The new branch now points to the accidental commit, preserving your work.

**Step 2 — Restore the local `main` branch.**

```bash
git switch main
git reset --keep HEAD~1
```

This moves `main` back one commit while protecting against overwriting local working-tree changes. It still rewrites local branch history, so use it only after verifying that the commit is unpushed and `main` should move back exactly one commit.

**Step 3 — Continue using the feature branch.**

```bash
git switch feat/recover-work
git push -u origin feat/recover-work
```

Open a pull request from `feat/recover-work` into `main` and follow the normal review process.

> **Important:** If the accidental commit has already been pushed to a shared `main` branch, do not use `git reset` followed by a force push. Preserve shared history and coordinate an appropriate correction, such as `git revert`, with your team.

### Final reminder

Before running a Git command that modifies history or discards changes, understand which of the three Git areas it affects: the working tree, staging area, or committed history (§2).

When uncertain, inspect the repository first:

```bash
git status
git log --oneline --graph --decorate -5
git diff
git diff --staged
```

These commands help you understand the current state before taking corrective action.

---

## 11. References

From the program's source list:

- [Pro Git (book)](https://git-scm.com/book) — the definitive free Git reference.
- [GitHub Docs — Get started](https://docs.github.com/en/get-started) — remotes, PRs, GitHub Flow.
- [Conventional Commits](https://www.conventionalcommits.org/) — the commit grammar.
- [Node.js downloads / version managers](https://nodejs.org/en/download) — Node 24 LTS.
- [Bruno docs](https://docs.usebruno.com) — the API client used from M05 onward.

For the module's deliverable and checkpoints, see [`brief.md`](./brief.md); for an ordered checklist,
see [`tasks.md`](./tasks.md).
