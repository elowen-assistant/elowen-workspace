# Workspace Instructions

These instructions apply across the Elowen workspace unless a service repository has a more specific rule.

## Repository Boundaries

- `elowen-workspace` is the meta-repository and coordination layer.
- `elowen-api`, `elowen-ui`, `elowen-edge`, `elowen-notes`, and `elowen-platform` are independently versioned child repositories.
- Cross-repo process, shared instructions, and development playbooks belong in `elowen-workspace/codex/`.
- Service-owned implementation details belong in the corresponding child repository.

## Current Architecture Snapshot

- `elowen-api` owns orchestration state and thread/job lifecycle logic.
- `elowen-ui` owns the user-facing thread, job, and approval experience.
- `elowen-edge` owns local execution, worktrees, Codex invocation, and build/test execution.
- `elowen-notes` owns notes and promoted memory, backed by ArangoDB.
- `elowen-platform` owns shared Compose, contracts, schema drafts, ADRs, and deployment assets.

## Current Platform Decisions

- Slice 0 is complete.
- Slice 1 is complete.
- Slice 2 is the active next slice.
- Notes are modeled in ArangoDB using document collections, edge collections, and ArangoSearch.
- Notes service contracts should remain portable enough to support a future MongoDB migration if needed.
- Local Windows Rust validation may require loading `vcvars64.bat` before `cargo check`.
- Service Dockerfiles should stay on a Rust base image version that satisfies the current dependency MSRV across the workspace.

## Git Model

- Each child repository has its own history, releases, and tags.
- `elowen-workspace` tracks child repositories as Git submodules.
- A child repo change should normally be committed and pushed in the child repo first.
- After the child repo moves, update the submodule pointer in `elowen-workspace` and commit that change separately.

## Change Discipline

- Prefer vertical slices over broad refactors.
- Keep contracts and storage decisions explicit.
- Avoid leaking database-specific assumptions outside the owning service boundary.
- If a change creates a new workspace-wide convention, update `codex/` as part of the work.
