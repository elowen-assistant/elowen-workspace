# Slice Planning Checklist

Use this checklist before implementing a roadmap slice. Prefer planning mode when it is available; otherwise, still complete the same planning output before changing files.

## Required Inputs

- slice number
- exact roadmap title
- assigned scope from `roadmap.md`
- touched child repos
- workspace repo changes, if any

## Planning Output

Produce a plan that locks down:

- goal and acceptance criteria
- explicit non-goals for the slice
- per-repo branch names using `slice/<number>-<slug>`
- implementation order across repos
- first automated validation phase commands
- Rust refactor scope for each touched Rust repo
- second automated validation phase commands
- manual UAT checklist
- required `roadmap.md` and `codex/notes/YYYY-MM-DD-handoff.md` updates

## Branch Naming

Derive the slug from the roadmap title.

Examples:

- `Slice 30 - UI Browser Automation` -> `slice/30-ui-browser-automation`
- `Slice 36 - Shared Rust Message Contracts` -> `slice/36-shared-rust-message-contracts`

## Multi-Repo Ordering

When multiple child repos are touched:

1. plan contract and dependency order first
2. implement and validate in child repos
3. merge child repos into `main`
4. update workspace submodule pointers
5. run any required workspace-level integrated checks
6. update roadmap and handoff docs
