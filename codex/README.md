# Codex Workspace Guide

This directory is the workspace-level source of truth for Codex-specific guidance that applies across Elowen repositories.

Use this area for:

- shared instructions that apply to the whole Elowen system
- cross-repo development rules
- playbooks for recurring workflows
- skill definitions or skill drafts that are broader than one service repo

Keep service-specific guidance in the owning repository. Keep cross-repo guidance here.

## Layout

- `instructions/` - shared rules and operating guidance
- `playbooks/` - step-by-step workflows for recurring tasks
- `notes/` - dated session handoff notes and restart context
- `skills/` - workspace-level skills or skill drafts

## Maintenance Rule

When a new cross-repo convention or recurring workflow appears during development, update this directory in the same change or immediately after it. This repository should stay current with how Elowen is actually being built.

## Current Handoff

As of 2026-04-15, slices `0` through `29` are complete and `Slice 30 - UI Browser Automation` is the next planned slice.

Current baseline:

- VPS API/UI/notes deploys pull prebuilt GHCR images instead of compiling on the VPS.
- The web UI is authenticated with API-issued cookie sessions.
- The UI is still client-side rendered Leptos, with local state persistence and authenticated SSE updates plus explicit reconnect/backoff recovery.
- Polling remains only as a slower fallback until browser automation covers realtime behavior.
- The laptop edge uses parent-directory repository discovery and signed registration proof against a pinned orchestrator key.

Next planned step:

- Continue into `Slice 30 - UI Browser Automation`.
