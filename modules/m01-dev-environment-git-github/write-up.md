# M01 Write-Up — Dev Environment, Git & GitHub

> My filled-in version of [`write-up-template.md`](./write-up-template.md). My M01 **submission
> deliverables are documentation**: the Nexus EPAV-generated learning materials
> ([`learning-materials.md`](./learning-materials.md)) and this module write-up. The original lab's
> hands-on Git exercises are treated here as **educational concepts and supplementary evidence**, not
> as completion requirements. Where evidence is incomplete, this document says so rather than implying
> completion.

**Documentation deliverables (submission scope):**
- `learning-materials.md` — the Nexus EPAV-generated educational guide for M01.
- `write-up.md` — this reflection.

**Supplementary implementation evidence (outside submission scope):**
- Repository: https://github.com/johnmartin-marasigan/node-js-training-implementation
- Pull request: https://github.com/johnmartin-marasigan/node-js-training-implementation/pull/1
- Branch: `feat/m01-hello-node` · PR #1 state: **Draft — not merged, 0 reviews**.

## What I built

My M01 work produced **two documentation artifacts**, generated through the Nexus EPAV workflow
(Evaluate → Plan → Apply → Validate). The hands-on repository is kept as supplementary evidence.

**Primary deliverables:**

- **`learning-materials.md`** — a comprehensive M01 study guide covering: the toolchain (VS Code,
  Git, Node via a version manager, `.nvmrc`, Bruno, terminal basics); the Git mental model; everyday
  commands; `revert` vs `reset`; branching, merging and conflicts; merge vs rebase (conceptual);
  `.gitignore` and secrets hygiene; GitHub collaboration (remotes, auth, Issues, PRs, review, GitHub
  Flow, Conventional Commits, README basics); a worked branch → PR → conflict → merge example; and a
  best-practices / troubleshooting reference.
- **`write-up.md`** — this reflection on what I learned, the decisions behind the documentation, and
  the EPAV workflow that produced it.

**Supplementary implementation evidence (described, not a completion target):**

The repository `node-js-training-implementation` holds the M01 environment on branch
`feat/m01-hello-node`:

- **`hello-node.js`** — commit `0c13774` (`feat: add hello-node script`).
- **`.nvmrc`** — pins Node.js 24, commit `6f34b4f` (`chore: configure Node.js 24 version`).
- **`.gitignore`** and **`.env.example`** — commit `628bef3`
  (`chore: add Git ignore rules and environment template`).
- **PR #1** — opened from `feat/m01-hello-node` into `main`, carrying four Conventional Commits
  (the three setup/script commits above plus `98eb2e2`, `docs: add M01 training write-up`).

Its current status: PR #1 is a **Draft** with **no reviews**, so it is **not merged**; the branch
history shows **no merge conflict**. These facts are recorded as-is; they are supplementary evidence,
not deliverables of this submission.

## Why it's built this way (key decisions)

**Scope decision — documentation over exercise completion.** The M01 objective for this submission is
to produce comprehensive learning materials and a completed write-up, so I scoped the deliverables to
those two documents. The lab in `brief.md` frames a practical Git exercise (branch → PR → merge with a
planted conflict); rather than treating that exercise as the finish line, I used its subject matter as
the content of the learning materials and kept the real repository/PR as supporting evidence. This
keeps the submission focused on knowledge development while staying honest about what the repository
does and does not contain.

**Why the write-up is the teaching artifact.** Per the program's README, each stage's write-up *is*
the teaching content for the next cohort, and it should be written as the work happens. That is why
the learning materials and this write-up — not a finished exercise — are the deliverables.

**Nexus EPAV process decisions.**
- *Evaluate first:* I oriented on `brief.md`, `tasks.md`, `write-up-template.md`, the README ground
  rules, and the architecture doc before planning, so the documentation matched the module's intent.
- *Plan before Apply:* the plan fixed the filenames, the section outlines, and validation criteria
  before any file was written.
- *Apply without fabricating:* the write-up uses real repository/PR facts where known and explicit
  pending markers otherwise. No merged PR, review, or conflict resolution is claimed.
- *Preserve the originals:* `brief.md`, `tasks.md`, and `write-up-template.md` were left untouched;
  the deliverables are separate files in the M01 directory.

**Git concepts kept as teaching material.** Branching, pull requests, and merge conflicts remain in
the learning materials as *concepts* — the modules after M01 assume them, and the capstone's rubric
assesses a PR workflow. They are explained for understanding, not assigned as additional tasks here.

**Public repo convention (context).** The brief fixes public training repos by program convention:
branch protection on private repositories depends on the GitHub plan, and a public repo avoids that
dependency. Had the repo been private with a no-direct-push rule, the only change would be the
*guardrail* (enforced by a repository setting rather than convention); the branch → PR → review →
merge workflow itself is unchanged.

## How to build it (teach it to the next trainee)

A guide to producing the M01 documentation deliverables with the Nexus EPAV workflow.

1. **EVALUATE — orient before anything else.** Read all of `brief.md`, `tasks.md`, and
   `write-up-template.md`, plus the program README ground rules (free tools, public repos, Node 24,
   Conventional Commits) and the architecture doc's environment/stack notes. Write a short summary of
   the task, what it touches, constraints, and risks. *Reason:* the documentation must match the
   module's stated intent, not a guess about it.
2. **PLAN — blueprint before writing.** Propose the files to create, the section outline of each, and
   the validation criteria; do not touch any file yet. *Reason:* it makes scope and acceptance
   criteria explicit and reviewable before effort is spent.
3. **APPLY — create the two documents.** Write `learning-materials.md` (concepts, commands, workflows,
   examples, best practices) and `write-up.md` (reflection). Keep the originals untouched, and never
   invent completed work: use real facts where available and clearly marked pending items otherwise.
   *Reason:* this keeps the submission both comprehensive and truthful.
4. **VALIDATE — check against the criteria.** Confirm every scope topic is covered, the write-up keeps
   the template's headings, and no fabricated evidence slipped in. *Reason:* the checkpoint question
   is "could someone else learn from this?", which coverage and honesty answer directly.

**Git concepts a next trainee will meet (explained, not assigned).** The module's subject matter is
the everyday Git workflow, so the learning materials explain it as background:

- A **branch** is a movable pointer to a commit; work happens on a feature branch so `main` stays
  stable. In my repository this is `feat/m01-hello-node`.
- A **pull request** proposes merging one branch into another and is where review occurs; mine is
  PR #1.
- A **merge conflict** happens when two branches change the same lines; Git inserts
  `<<<<<<<` / `=======` / `>>>>>>>` markers that a human resolves by deciding the intended result, not
  by mechanically picking a side.
- **Merge vs rebase:** a merge preserves branch topology with a merge commit; a rebase replays commits
  onto a new base for linear history but rewrites commit hashes, so it is unsafe on shared commits.

**The key mindset:** commands are mechanics; the value is knowing *which* of the three trees
(working / staging / history) a change is in, and choosing the operation that moves it where you want.
See `learning-materials.md` §2–§6 for the model and §9 for the flow with commands.

## Concepts worth explaining

**1. The working tree → staging → commit model.** Git moves changes through three areas. The
**working tree** is my files on disk. `git add` copies changes into the **staging area (index)**, a
curated snapshot of what the next commit will contain. `git commit` records that snapshot permanently
in the **repository** as a commit with its own SHA. The reason staging exists is *deliberateness*: a
commit need not include everything I have changed, so I can shape exactly what story each commit
tells. `git status` is what tells me which area each change currently sits in.

**2. `git revert` vs `git reset`.** Both undo work, but in opposite directions with respect to
history.
- `git revert <sha>` creates a **new commit that reverses** the selected commit. History only grows,
  which is why it is the safe choice for anything already pushed or shared.
- `git reset <sha>` **moves the branch pointer backward**, dropping the commits after the target from
  the branch. `--soft` keeps the changes staged, `--mixed` (default) unstages but keeps the edits,
  and `--hard` destroys the edits. It rewrites history, so it is only safe on my own local,
  unpushed commits.

In one line: **`revert` undoes a change publicly and safely; `reset` rewinds my local branch.**

**3. Branches, pull requests, and merge conflicts (conceptual).** A **branch** isolates work as a
movable pointer to a commit. A **pull request** is a review-and-merge request between branches. A
**merge conflict** arises when two branches touch the same lines; Git cannot choose, so it marks the
region and a human resolves it by editing to the intended outcome. These are the mechanics behind the
module's Git topic and the workflow later modules and the capstone assume.

## What tripped me up

**Factual observations from producing the deliverables:**
- The primary effort was documentation quality, not command execution: the harder problem was
  deciding what belonged in the learning materials versus the write-up.
- The repository carries four Conventional Commits, one more than the brief's "3 or more" because a
  `docs:` commit (`98eb2e2`) records the write-up alongside the three setup/script commits.
- The write-up template's reflection sections ("What tripped me up", "What I'd do differently") are
  inherently personal; mechanical details are documented, but genuine personal reflection still needs
  my own input (see below).

**Personal input needed (to fill in with my real experience):**
- What part of the Git and GitHub material was genuinely new to me, and what did I already know?
- Which concept took the longest to explain clearly enough for the next trainee?

## Checkpoint evidence

Two distinct sets: my **documentation deliverables** (this submission's scope) and the **original lab
checkpoints** from `brief.md` (retained as supplementary evidence, not completion requirements).

**A. Documentation deliverables (submission scope):**

| Deliverable | Status | Evidence |
| --- | --- | --- |
| Comprehensive learning materials | **Complete** | [`learning-materials.md`](./learning-materials.md) |
| Module write-up (template headings preserved) | **Complete** | This file |
| Original module files preserved | **Complete** | `brief.md`, `tasks.md`, `write-up-template.md` unchanged |
| No fabricated exercises/PR/verification | **Complete** | Pending items marked explicitly |

**B. Original M01 lab checkpoints (supplementary evidence — not claimed as complete):**

| Lab checkpoint | Status | Evidence |
| --- | --- | --- |
| Merged PR with 3+ Conventional Commits | **Not met — PR #1 is a Draft, not merged** | PR #1: four typed commits (`6f34b4f`, `628bef3`, `0c13774`, `98eb2e2`) |
| Resolved merge conflict in history | **Not evidenced** | No merge or conflict-resolution commit on `feat/m01-hello-node` |
| `.gitignore` covering the Node list | **Present** | Commit `628bef3` |
| `.env.example` present | **Present** | Commit `628bef3` |
| `revert` vs `reset` explained | **Written** | Concepts section, item 2 |

**Revert vs reset, in my own words:** `revert` is additive — it adds a new commit that undoes a
previous one, so it is safe on shared history. `reset` is subtractive — it moves my branch pointer
back and drops commits, so it rewrites history and is safe only locally before pushing.

## What I'd do differently

**Reflection on the documentation process:**
- I would capture learning notes *while* drafting `learning-materials.md`, so the write-up's personal
  sections came from live notes rather than being reconstructed afterwards.
- I would decide the learning-materials/write-up split earlier in the Plan step, since the boundary
  between "teaching content" and "reflection" took the most rework.
- I would include a short coverage checklist in the plan so each `brief.md` scope bullet maps to a
  learning-materials section before Apply, making Validate mechanical.

**Personal input needed (to fill in with my real reflection):**
- Would I have written the learning materials in a different order (toolchain first vs concepts
  first)?
- Which section would I expand for the next cohort?

**Note on the supplementary repository:** its PR #1 remains a Draft and is not needed to satisfy this
submission's deliverables; it is kept as supporting evidence of the underlying M01 environment.
