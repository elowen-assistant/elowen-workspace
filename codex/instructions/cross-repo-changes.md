# Cross-Repo Change Rules

Use these rules whenever one user task touches more than one Elowen repository.

## Ownership

- Put code in the repository that owns the behavior.
- Put interface agreements in the repository that owns the contract surface, or in `elowen-platform` if shared.
- Put process documentation in `elowen-workspace`.

## Ordering

When possible, make cross-repo changes in this order:

1. update shared contracts or platform drafts
2. update the backend or service owner of the behavior
3. update dependent services or UI
4. update workspace docs and submodule pointers

## Commits

- Keep commits scoped to one repository.
- Do not mix child repo code changes into the workspace meta-repo commit.
- The workspace commit should usually only move submodule pointers and update workspace-level docs.

## Validation

- Validate each changed child repository on its own.
- Then validate the integrated workspace state from `elowen-workspace`.
- If Compose or multi-service behavior changes, verify it from the workspace root.
- On Windows, use the MSVC environment when validating Rust services locally.
- Keep Rust Docker base images aligned across service repos when dependency MSRV changes.
- When introducing NATS subjects, record the subject naming convention in workspace docs so later slices reuse the same transport boundary.
- Current shared subjects include `elowen.devices.availability.probe.{device_id}` for request-reply probes, `elowen.jobs.dispatch.{device_id}` for job delivery, and `elowen.jobs.events` for edge lifecycle events.

## Release Intent

- Child repositories should be releasable independently.
- The workspace repository can tag integrated snapshots of known-good child versions.
