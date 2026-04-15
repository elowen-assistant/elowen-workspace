---
name: slice-delivery
description: Standardize Elowen roadmap slice planning, implementation, validation, refactoring, and closeout across child repositories and the workspace repo. Use when Codex needs to plan a slice, create or use `slice/N-slug` branches, implement slice work, run the two required automated validation phases, perform the mandatory behavior-preserving Rust refactor across touched Rust repos, capture manual UAT, update `roadmap.md`, refresh `codex/notes/YYYY-MM-DD-handoff.md`, or merge completed slice work back to `main`.
---

# Slice Delivery

## Overview

Use this skill for any Elowen roadmap slice that spans one or more child repositories and may also require workspace documentation or submodule updates. Keep the workflow planning-first, enforce the branch and closeout conventions, and treat slice completion as unfinished until validation, manual UAT, documentation refresh, and merge-to-`main` are all complete.

## Start With The Workspace Sources Of Truth

Read these before proposing or changing slice work:

1. `roadmap.md`
2. `codex/instructions/workspace.md`
3. relevant repo-local instructions and validation config in each touched child repo
4. `codex/notes/` only when recent handoff context affects the current slice

Treat the roadmap title and assigned scope as the authority for branch naming, acceptance criteria, and closeout wording.

## Plan The Slice Before Editing Code

When the user asks to start or continue a slice, first use planning mode when available and produce a decision-complete plan before editing code. Lock down:

- slice number and exact roadmap title
- touched repositories and whether the workspace repo will change
- acceptance criteria and out-of-scope items
- branch names for every touched repo using `slice/<number>-<slug>`
- automated checks for the first and second test phases
- manual UAT checklist and expected evidence
- roadmap and handoff-note updates required at closeout

If the slice spans multiple child repos, plan the dependency order explicitly. Use child-repo-first delivery, then update workspace submodule pointers after the child repos are validated and merged.

See [planning.md](references/planning.md) for the planning checklist and deliverable shape.

## Execute The Slice In Order

Follow this sequence without collapsing stages:

1. Create or switch to the `slice/<number>-<slug>` branch in each touched child repo.
2. Create the matching workspace branch when submodule pointers or workspace docs will change.
3. Implement the feature in small vertical increments.
4. Run the first automated test/build phase after feature completion.
5. Perform the mandatory Rust refactoring phase across all modules in every touched Rust repo.
6. Add or improve unit tests created during the current slice as needed during refactoring.
7. Add Rust doc comments for new or changed public APIs and any obvious missing public API docs within the refactor scope.
8. Run the second automated test/build phase after refactoring.
9. Run manual UAT and capture a short pass/fail summary plus known gaps.
10. Close out the slice only after commits, merges, documentation updates, and workspace pointer updates are complete.

Do not treat refactoring as optional polish. It is a required post-feature hardening stage for touched Rust repos and must preserve behavior.

See [validation-and-refactor.md](references/validation-and-refactor.md) for the phase gates and refactor rules.

## Apply Git And Test Boundaries

Use these repository rules consistently:

- Use descriptive slice branch names derived from the roadmap title: `slice/N-slug`.
- Commit in each touched child repo before changing workspace submodule pointers.
- Squash-merge each slice branch into `main` after validation and UAT pass.
- Leave unrelated repositories untouched.
- Keep refactors behavior-preserving; if behavior must change, move that change back into the feature scope and revalidate.

Use these test-editing rules during the slice:

- Update tests created during the current slice without extra approval when needed.
- Do not modify tests that existed before the current slice without explicit human approval.
- Stop and ask for approval before editing pre-slice tests, even if the change looks mechanical.
- Prefer adding new coverage over rewriting existing tests when approval has not been granted.

## Capture Manual UAT And Closeout Evidence

Manual UAT is mandatory even after both automated validation phases pass. Record:

- the exact flows exercised
- pass/fail status
- any known gaps, flaky areas, or deferred follow-up

Closeout is not complete until all of the following are true:

- child repo work is committed and merged into `main`
- workspace submodule pointers match the merged child repo commits
- `roadmap.md` reflects the slice outcome and next active slice
- `codex/notes/YYYY-MM-DD-handoff.md` is created or refreshed with resume context
- the workspace tree is clean except for intentional post-merge state

See [manual-uat.md](references/manual-uat.md) for the checklist structure and [closeout.md](references/closeout.md) for the closeout checklist and handoff note contents.
