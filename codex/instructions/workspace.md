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
- Slice 2 is complete.
- Slice 3 is complete.
- Slice 4 is complete.
- Slice 5 is complete.
- Slice 6 is complete.
- Slice 7 is complete.
- Slice 8 is complete.
- Slice 9 is the next planned slice.
- Slice 10 is planned after Slice 9.
- Slice 11 is planned after Slice 10.
- Notes are modeled in ArangoDB using document collections, edge collections, and ArangoSearch.
- Notes service contracts should remain portable enough to support a future MongoDB migration if needed.
- Local Windows Rust validation may require loading `vcvars64.bat` before `cargo check`.
- Service Dockerfiles should stay on a Rust base image version that satisfies the current dependency MSRV across the workspace.
- Device registration is API-backed, while availability probes use NATS request-reply on `elowen.devices.availability.probe.{device_id}`.
- Job dispatch currently uses NATS publish-subscribe on `elowen.jobs.dispatch.{device_id}`.
- Job lifecycle events currently use NATS publish-subscribe on `elowen.jobs.events`.
- The edge runtime now creates mounted git worktrees under `/workspace/.elowen/worktrees` in Compose.
- The Slice 4 execution wrapper defaults to a simulated runner until an external Codex command is configured.
- The UI is served as a static production build behind nginx in Compose, not `trunk serve`.
- Slice 5 persists execution reports, generated job summaries, and push approvals in the API database.
- Slice 5 approval resolution is API-driven and surfaced inline in the UI job detail view.
- Slice 6 keeps note creation, revisioning, and retrieval inside `elowen-notes`, with `elowen-api` consuming that service over HTTP.
- Slice 6 related-note retrieval for a thread includes notes promoted from jobs that belong to that thread.
- Slice 6 includes an explicit responsive/interactivity correction for the UI shell so thread/job workflows remain usable in the containerized deployment.
- Slice 7 adds durable `correlation_id` propagation across jobs, job events, and edge lifecycle messages.
- Slice 7 enables structured JSON logs through `ELOWEN_LOG_FORMAT=json` in the Rust services.
- Slice 7 platform docs live in `elowen-platform/docs/operations.md`, and the Kubernetes migration base lives in `elowen-platform/k8s/base`.
- Slice 8 adds a lightweight global jobs list in the UI that can select the owning thread and job detail from any thread context.

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
