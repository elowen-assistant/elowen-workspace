# Elowen Workspace

This folder is the umbrella workspace for Elowen, a personal cloud AI-assistant platform composed of multiple service repos and one shared platform repo.

`elowen-workspace` is the meta-repository. Each top-level service directory is tracked as its own Git repository and linked here as a Git submodule.

## Layout

- `elowen-api/` - orchestration API
- `elowen-ui/` - Leptos UI
- `elowen-edge/` - local edge agent for Codex execution
- `elowen-notes/` - versioned notes document service
- `elowen-platform/` - shared deployment assets, contracts, schemas, and docs
- `codex/` - workspace-level Codex instructions, playbooks, and shared skills

`roadmap.md` remains the source of truth for the product and architecture direction.

## Current State

As of 2026-04-23, slices `0` through `39` are complete on this Slice 39 branch, with slices `0` through `38` complete on local `main`.

The shipped baseline includes:

- a VPS-hosted orchestrator deployment over HTTPS
- GHCR prebuilt images for the VPS-hosted API, notes service, and UI
- a standalone Windows laptop edge runtime with env-file startup
- signed edge registration with pinned orchestrator trust
- parent-directory repository discovery on the edge
- real Codex CLI execution in per-job git worktrees
- read-only execution intent for informational repository requests
- approval-backed commit and push execution for mutating jobs
- a chat-first Leptos UI with web-session authentication
- authenticated server-sent events for thread, job, and device updates, with explicit reconnect/backoff recovery and slower polling retained as a fallback
- polished chat transcript/result surfaces and execution-draft handoff cards with local timestamp rendering and keyboard submit affordances
- richer notes retrieval and ranking for promoted memory
- stronger orchestrator-side note context assembly with direct thread/job memory prioritization and top-note detail expansion
- generic-job orchestration where repository execution is one specialization rather than the universal default
- capability-targeted non-repo job drafts, dispatch, routing, and edge execution for the first prompt-first generic-job vertical
- manual Slice 38 UAT proving both capability execution without a repo worktree and repository execution with disposable worktrees plus passing validation in the local Compose stack
- an audited Kubernetes base that reflects the post-Slice-38 stack more truthfully, including the current API auth/config inputs and GHCR-backed image defaults
- manual Slice 39 validation in a local `kind` cluster proving the in-cluster orchestrator stack applies and runs, with private GHCR registry access called out as the first real operator-side deployment prerequisite
- an explicit supported Kubernetes topology where `elowen-edge` remains a separate trusted device runtime and any in-cluster edge manifest is treated as experimental only

The next planned roadmap slice is `Slice 40 - CI Workflow Maintenance`.

## Clone

Clone the workspace with submodules:

```bash
git clone --recurse-submodules https://github.com/elowen-assistant/elowen-workspace.git
```

If already cloned:

```bash
git submodule update --init --recursive
```

## Working Convention

Each top-level service directory is its own repository with independent history and releases. `elowen-platform` is the shared orchestration/deployment repo for cross-cutting assets. `elowen-workspace` pins known-good revisions of the child repositories.

Workspace-level Codex guidance lives under `codex/` and should be updated as new cross-repo conventions emerge.
