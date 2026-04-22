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

- Slices 0 through 36 are complete on merged `main`.
- Slice 29 delivered SPA-style client state persistence plus authenticated SSE updates with explicit reconnect/backoff recovery.
- Slice 30 delivered browser automation coverage for UI layout, tap, scroll, auth, sticky-composer behavior, and realtime transcript updates.
- Slice 31 closed on 2026-04-15, delivering cleaner execution-draft synthesis, stronger transcript/result differentiation, localized timestamps, and `Ctrl+Enter` / `Cmd+Enter` composer submit on merged `main`.
- Notes are modeled in ArangoDB using document collections, edge collections, and ArangoSearch.
- Notes service contracts should remain portable enough to support a future MongoDB migration if needed.
- Local Windows Rust validation may require loading `vcvars64.bat` before `cargo check`.
- Service Dockerfiles should stay on a Rust base image version that satisfies the current dependency MSRV across the workspace.
- VPS deploys should use GHCR prebuilt images and `docker compose pull && docker compose up -d`, not on-host Rust compilation.
- Device registration is API-backed and can require signed edge proof against a pinned orchestrator key.
- Shared internal Rust API-edge message contracts live in `elowen-platform/contracts/rust/elowen-contracts`.
- Availability probes use NATS request-reply on `elowen.devices.availability.probe.{device_id}`.
- Job dispatch currently uses NATS publish-subscribe on `elowen.jobs.dispatch.{device_id}`.
- Job lifecycle events currently use NATS publish-subscribe on `elowen.jobs.events`.
- The edge runtime discovers nested git repositories under trusted parent roots and creates per-job worktrees under `.elowen/worktrees`.
- The edge runs the real Codex CLI path, captures runner artifacts, runs validation, creates commits for mutating jobs, and waits for approval before push.
- Read-only jobs should not create commits or push approvals when no repository changes are produced.
- The UI is a client-side Leptos app served from a prebuilt image, not `trunk serve`.
- The UI is chat-first, Material-inspired, and uses authenticated SSE plus targeted refreshes, with 30-second polling retained as fallback.
- UI session auth is currently a shared-password gate backed by API-issued cookie sessions; a fuller auth service remains future work.
- Notes creation, revisioning, and retrieval stay inside `elowen-notes`, with `elowen-api` consuming that service over HTTP.
- Kubernetes assets remain migration scaffolding; Compose remains the active deployment model.

## Git Model

- Each child repository has its own history, releases, and tags.
- `elowen-workspace` tracks child repositories as Git submodules.
- A child repo change should normally be committed and pushed in the child repo first.
- After the child repo moves, update the submodule pointer in `elowen-workspace` and commit that change separately.
- Before starting a new slice branch, audit local child-repo branches and worktrees for commits ahead of `main` or `origin/main` so new work does not fork from a stale line while a newer local UI or platform branch is still unmerged.

## Change Discipline

- Prefer vertical slices over broad refactors.
- Keep contracts and storage decisions explicit.
- Avoid leaking database-specific assumptions outside the owning service boundary.
- If a change creates a new workspace-wide convention, update `codex/` as part of the work.
