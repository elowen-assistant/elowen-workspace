# Personal Assistant / Local Codex System Delivery Roadmap

## 1. Purpose

Build a personal assistant platform with:

- a cloud orchestrator
- a local Rust edge agent
- local Codex-driven coding execution
- a Leptos UI
- Postgres for operational state
- NATS JetStream for job/event transport
- a separate notes document service
- Docker Compose for initial deployment

The system should support:

- conversational threads
- coding jobs on a primary device
- isolated git worktrees per job
- structured summaries
- controlled approvals
- post-MVP expansion areas tracked in `Planned Post-30 Slice Set`

---

## 2. Core Architectural Decisions

### Runtime / Deployment
- Docker Compose first
- Clear upgrade path to Kubernetes
- Separate repo per service
- Separate orchestration repo for:
  - deployment config
  - contracts
  - docs
  - schemas

### Core Services
- `elowen-api` (orchestrator)
- `elowen-ui` (Leptos)
- `elowen-edge` (edge agent)
- `elowen-notes` (ArangoDB-backed notes service)
- `postgres`
- `arangodb`
- `nats` (JetStream)

### Technology Choices
- Rust (backend + edge agent)
- Leptos (UI)
- TOML (config)
- Protobuf (internal contracts)
- JSON (UI transport)
- ArangoDB for notes, with MongoDB-portable service contracts

### Execution Model
- Local Codex invoked by edge agent
- Git worktree per job
- One active coding job (v1)
- Push requires approval (post-job)
- Local commits allowed

### Context / Memory
- Layered selective context bundle
- Versioned thread and job summaries
- Notes are Markdown-centric documents stored in ArangoDB with explicit graph links
- Notes service is canonical for promoted knowledge

### UX
- Threads are primary interface
- Jobs are first-class entities
- Global jobs view (lightweight)
- Polling in v1
- Approvals inline and visually distinct

---

## 3. Service Responsibilities

### elowen-api
- Thread/message APIs
- Job orchestration
- Device routing + availability probing
- Approval handling
- Summary coordination
- Note retrieval integration
- Job dispatch via JetStream

### elowen-ui
- Thread list/detail
- Message composer
- Approval UI
- Jobs list/detail
- Polling-based updates

### elowen-edge
- Device registration
- Repo allowlist
- Availability responses
- Worktree management
- Codex execution
- Test/build execution
- Event publishing
- Active job lease

### elowen-notes
- Document and edge-backed note creation and revision tracking
- Keyword + metadata retrieval via ArangoSearch
- Tags, backlinks, note types, and source references
- Attachment references and graph traversal

### postgres
Stores:
- threads, messages
- jobs, job events
- approvals
- devices
- summaries
- note references

### nats / jetstream
Carries:
- job dispatch
- availability probes
- job lifecycle events

### arangodb
Stores:
- note documents
- note revisions
- note links / backlinks
- note types
- attachment records
- notes search view

---

## 4. Repository Structure

### Service repos
- elowen-api
- elowen-ui
- elowen-edge
- elowen-notes

### Orchestration repo (`elowen-platform`)
- compose/
- k8s/
- contracts/
- docs/
- env/
- adr/
- scripts/

---

## 5. Contracts (Protobuf)

Define:

- Job
- JobDispatch
- JobAccepted
- JobStarted
- JobEvent
- ApprovalRequest
- ApprovalResolution
- DeviceRegistration
- AvailabilityProbe
- AvailabilityResponse
- SummaryRecord

Suggested files:
- jobs.proto
- devices.proto
- approvals.proto
- events.proto
- summaries.proto

---

## 6. Data Model

### Threads
- id (ULID)
- title
- status
- current_summary_id
- timestamps

### Messages
- id
- thread_id
- role
- content
- status
- timestamps

### Jobs
- id (ULID)
- short_id
- title
- status
- result
- failure_class
- repo_name
- device_id
- branch_name
- base_branch
- parent_job_id
- timestamps

### Job Events
- id (ULID)
- job_id
- event_type
- payload_json
- created_at

### Approvals
- id
- thread_id
- job_id
- action_type
- status
- timestamps

### Devices
- id
- name
- primary_flag
- metadata

### Summaries
- thread_summaries (versioned)
- job_summaries (versioned)

### Notes References
- note_id
- source_type
- source_id

---

## 7. Notes Service Model

### notes
Document collections:
- `notes`
- `note_revisions`
- `note_types`
- `attachments`

### Edge collections
- `note_links`
- `note_sources`

### Search surface
- `notes_search` ArangoSearch view over note titles, metadata, summaries, and Markdown bodies

### Notes identity model
- note_id
- slug
- current_revision_id
- note_type
- timestamps

### Note revision model
- revision_id
- note_id
- previous_revision_id
- title
- summary
- body_markdown
- frontmatter
- source_refs
- created_by
- created_at

### Portability constraints
- Use ULIDs and explicit application IDs instead of storage-generated IDs as the domain contract
- Keep Arango-specific query behavior inside `elowen-notes`
- Model links as explicit records so they can be remapped to MongoDB collections later

---

## 8. Config Model

### Shared config
- skills
- global instructions
- TOML files

### Project config
- `.assistant/config.toml`
- test/build commands
- default branch
- repo metadata

### Edge config
- device identity
- allowed repos
- endpoints
- capabilities

---

## 9. Sub-Project Inventory

### Product-facing services
- `elowen-api` - orchestrator, API, lifecycle state machine
- `elowen-ui` - threads, jobs, approvals, job detail views
- `elowen-edge` - local execution agent, worktrees, Codex runner, test runner
- `elowen-notes` - canonical ArangoDB-backed notes and promoted memory service

### Shared platform repo
- `elowen-platform/contracts` - protobuf contracts and schema ownership
- `elowen-platform/db` - draft SQL and migration planning
- `elowen-platform/compose` - local deployment stack
- `elowen-platform/docs` and `elowen-platform/adr` - shared documentation and decisions
- `elowen-platform/env` and `elowen-platform/scripts` - developer setup and helper scripts

### Infra components used by the service repos
- `postgres` - operational state store
- `arangodb` - notes document, graph, and search store
- `nats` / JetStream - dispatch and lifecycle event transport

---

## 10. Vertical Slice Roadmap

### Current verified delivery status as of 2026-04-15

- Slice set `0` through `29` is implemented on `main`.
- `Slice 29 - SPA State Persistence And Realtime Updates` is complete, preserving selected thread, selected job, composer text, panel state, and transcript scroll across background updates while explicitly managing realtime reconnect/backoff recovery.
- The completed true-MVP path remains `Workflow #1`: thread-native request -> dispatched laptop job -> assistant reply back into the same thread.
- The shipped post-MVP baseline now also includes `Workflow #2`: conversational orchestrator replies, explicit handoff into execution, transcript mode visibility, conversational execution drafts, read-only request handling, thread-visible final job results, chat-forward UI updates, web-session authentication, parent-directory repository discovery, Material 3-aligned shell work, and signed edge registration.
- `elowen-ui` now behaves as a chat-first authenticated SPA with local state persistence, authenticated SSE updates, global jobs, job detail, approvals, notes, and the manual create-job form retained only as an advanced fallback.
- `elowen-api` persists user, system, and assistant thread messages; exposes both conversational chat and chat-dispatch routes; creates linked jobs from explicit execution requests; and posts lifecycle and final-result replies back into the owning thread.
- `elowen-edge` registers the device, creates per-job git worktrees, supports a real `codex exec` runner with startup preflight and captured runner artifacts, enforces the workspace sandbox boundary, supports read-only execution intent, and performs signed registration against a pinned orchestrator key.
- `elowen-platform` documents both the VPS deployment path and the standalone laptop edge path, including same-origin HTTPS routing, GHCR-prebuilt VPS images, Windows startup helpers, and trusted edge registration.
- `elowen-notes` preserves note revision ancestry and authorship metadata, while the promoted-note path remains integrated into both job-backed and conversational thread context.
- The VPS-to-laptop flow has been re-validated as the current user-visible baseline, while `elowen-platform/k8s/base` remains migration scaffolding rather than a production deployment target.
- There are no remaining slice-level blockers to the true MVP; the active roadmap focus is now browser automation and post-MVP product maturity.

### True MVP definition

Elowen reaches the true MVP only when the following end-to-end path works reliably:

1. The orchestration layer can be deployed to a VPS.
2. The edge agent can be installed and run as a standalone service on a local laptop.
3. The user can interact with Elowen through the web UI as a chat surface.
4. A chat request can cause the orchestrator to send a coding task to the local laptop.
5. The laptop runs a real Codex-backed execution path, not the simulated default wrapper.
6. The edge reports result and execution state back to the orchestrator.
7. The orchestrator posts a clear assistant response back into the same thread confirming completion or failure.

### Gap between current delivery and the true MVP

- There are no remaining slice-level blockers to the true MVP defined here.
- The edge sandbox is now implemented as a workspace-scoped execution boundary with sandbox-classified failures surfaced through the existing job lifecycle model.
- Notes promotion now preserves revision ancestry, authored-by metadata, and explicit source references for the current revision path.
- Kubernetes migration assets are still intentionally non-MVP scaffolding and are not part of the release bar for the true MVP defined here.

### Slice 0 - Workspace and Runtime Foundation
Status:
- completed on 2026-03-22

Outcome:
- Every Elowen sub-project exists with a minimal runnable scaffold
- Local Compose can boot the shared runtime dependencies
- Contracts and schema drafts exist in one shared place

Sub-projects:
- `elowen-platform`
- `elowen-api`
- `elowen-ui`
- `elowen-edge`
- `elowen-notes`

Deliverables:
- repo skeletons
- Compose stack
- protobuf drafts
- Postgres schema draft
- notes bootstrap draft
- separate git repos initialized for each sub-project
- ArangoDB bootstrap wired into `elowen-notes`

### Slice 1 - Threads and Conversation Surface
Status:
- completed on 2026-03-22

Outcome:
- User can create a thread, post messages, and view thread history in the UI
- Thread state is persisted and polled through the API

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- thread and message APIs
- thread list and detail views
- polling update loop
- persisted thread/message model

Delivered in current state:
- `elowen-api` exposes persisted thread and message endpoints backed by Postgres migrations
- `elowen-ui` provides thread list, thread detail, thread creation, and message posting against the live API
- `elowen-platform/contracts` includes a `threads.proto` draft for thread and message payloads
- local Compose validates the end-to-end Slice 1 path through `elowen-api` and `elowen-ui`

### Slice 2 - Device Presence and Execution Eligibility
Status:
- completed on 2026-03-22

Outcome:
- The primary local machine registers as an edge device
- The orchestrator can determine whether the device is available for work

Sub-projects:
- `elowen-edge`
- `elowen-api`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- device registration
- repo allowlist
- availability probe and response
- device record persistence

Delivered in current state:
- `elowen-edge` self-registers against `elowen-api` and renews its presence on a heartbeat loop
- `elowen-api` persists device metadata and exposes device list/detail endpoints
- availability probes flow over NATS request-reply using a per-device subject
- local Compose validates the primary device registration and probe path end to end

### Slice 3 - Job Creation and Dispatch
Status:
- completed on 2026-03-22

Outcome:
- A coding request from a thread becomes a tracked job
- The orchestrator routes the job to the registered edge device over NATS

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-edge`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- job creation from thread context
- job status model
- dispatch messages
- job cards in the UI

Delivered in current state:
- `elowen-api` creates jobs from thread context, persists `jobs` and `job_events`, probes the selected device, and dispatches the job over NATS
- `elowen-edge` subscribes to per-device job dispatch subjects and acknowledges the current dispatch path by logging received work
- `elowen-ui` shows thread-linked job cards and exposes a create-job form from the thread detail view
- `elowen-platform/contracts` includes expanded job payloads for create, list, detail, and event surfaces
- local Compose validates the end-to-end path from thread to persisted dispatched job on the primary device

### Slice 4 - Local Execution Loop
Status:
- completed on 2026-03-22

Outcome:
- Edge agent accepts a dispatched job, creates a worktree, runs the Codex wrapper, and emits lifecycle events
- The orchestrator and UI show real execution progress

Sub-projects:
- `elowen-edge`
- `elowen-api`
- `elowen-ui`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- active job lease
- worktree manager
- Codex wrapper
- event publishing and persistence
- job detail view

Delivered in current state:
- `elowen-edge` claims one active job at a time, creates a mounted git worktree, and writes request metadata into the worktree
- `elowen-edge` publishes lifecycle events over NATS for acceptance, worktree creation, start, completion, and failure
- `elowen-api` consumes job lifecycle events, persists them to `job_events`, and advances the `jobs` status model beyond dispatch
- `elowen-ui` now polls and displays a job detail pane with the persisted event stream for the selected thread job
- the default execution wrapper is simulated, with an explicit configuration path for an external Codex command

### Slice 5 - Results, Summaries, and Approval Gate
Status:
- completed on 2026-03-22

Outcome:
- Completed jobs include execution result, test outcome, and summary
- Push remains manual and must pass through an explicit approval step

Sub-projects:
- `elowen-api`
- `elowen-edge`
- `elowen-ui`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- diff and result summary generation
- test/build execution reporting
- approval request and resolution flow
- inline approval UI

### Slice 6 - Notes Promotion and Retrieval
Status:
- completed on 2026-03-22

Outcome:
- Threads and jobs can retrieve relevant notes
- Selected knowledge can be promoted into the notes system with versioning

Sub-projects:
- `elowen-notes`
- `elowen-api`
- `elowen-ui`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- responsive and interactive workspace shell correction for thread/job flows
- note create/read/query APIs
- note references from threads and jobs
- promotion into an ArangoDB document + graph model with revision tracking
- filtered retrieval primitives for note and context lookup

Delivered in current state:
- `elowen-notes` exposes note search, note detail, and note promotion APIs backed by ArangoDB collections and revision documents
- `elowen-api` enriches thread and job detail with related notes, and can promote job summaries into notes while keeping the notes boundary inside `elowen-notes`
- `elowen-ui` restores a responsive interactive shell, shows related notes in thread and job detail, and exposes a job-summary promotion action
- promoted job notes are now visible from both the owning job and the parent thread
- local Compose validates note promotion and retrieval without reintroducing the old Trunk overlay failure mode

### Slice 7 - Hardening and Platform Maturity
Status:
- completed on 2026-03-23

Outcome:
- The v1 stack is observable, supportable, and ready for broader use
- The path from Compose to Kubernetes is explicit

Sub-projects:
- `elowen-platform`
- `elowen-api`
- `elowen-edge`
- `elowen-ui`
- `elowen-notes`

Primary capabilities:
- structured logging
- correlation IDs
- job event auditability
- operational docs
- Kubernetes migration assets

Delivered in current state:
- `elowen-api`, `elowen-edge`, and `elowen-notes` support structured JSON logging through `ELOWEN_LOG_FORMAT=json`
- `elowen-api` now creates a durable `correlation_id` for each job and persists it on both `jobs` and `job_events`
- `elowen-edge` propagates the same `correlation_id` through dispatch handling and lifecycle event publishing
- `elowen-ui` surfaces the `correlation_id` in job detail and event history for audit/debug work
- `elowen-platform/docs/operations.md` documents runtime health, log shape, troubleshooting, and audit lookup flow
- `elowen-platform/k8s/base` provides a concrete Kustomize base mirroring the current Compose topology

### Slice 8 - Global Jobs Surface
Status:
- completed on 2026-03-23

Outcome:
- Jobs are visible outside the thread-local context
- Users can navigate directly from the global jobs surface into the owning thread and job detail

Sub-projects:
- `elowen-ui`
- `elowen-api`
- `elowen-workspace`

Primary capabilities:
- lightweight global jobs list in the UI
- cross-thread job selection
- job-driven navigation back into thread detail

Delivered in current state:
- `elowen-ui` now renders a lightweight global jobs list alongside the thread list
- selecting a job from the global jobs list opens the owning thread and the selected job detail
- the UI continues to poll the global jobs list from `GET /api/v1/jobs`

### Slice 9 - Build and Test Execution
Status:
- completed on 2026-03-23

Outcome:
- Execution reports reflect real project build and test results instead of scaffold placeholders

Sub-projects:
- `elowen-edge`
- `elowen-api`
- `elowen-ui`
- `elowen-platform`

Primary capabilities:
- repo-aware build/test command execution
- structured build/test result capture
- surfaced build/test output in job detail

Delivered in current state:
- `elowen-edge` now loads repo-owned validation commands from `.assistant/config.toml`, with a Cargo fallback when no explicit config is present
- build and test commands now run after Codex execution and are captured in the persisted execution report
- validation failures now mark jobs as failed and suppress push approval requests
- service repositories now include explicit validation config for local edge execution

### Slice 10 - Edge Sandbox Enforcement
Status:
- complete

Outcome:
- The edge runtime enforces explicit execution boundaries instead of relying only on worktree isolation

Sub-projects:
- `elowen-edge`
- `elowen-platform`
- `elowen-workspace`

Primary capabilities:
- sandbox policy definition
- runtime command and filesystem boundary enforcement
- auditable sandbox failures in job events and logs

Delivered notes:
- `elowen-edge` now writes a per-job sandbox policy artifact under `.elowen-sandbox/` inside each worktree and redirects temp/cache state into that boundary
- validation working directories and repo-relative command paths must resolve inside the job worktree, which closes the earlier `working_dir` escape path
- shell-style validation launches such as `powershell`, `cmd`, `sh`, and `bash` are blocked, and sandbox-blocked runs now surface as failure class `sandbox`
- `elowen-platform/contracts/proto/jobs.proto` now models `FAILURE_CLASS_SANDBOX`, and the laptop-edge docs describe the default workspace sandbox mode

### Slice 11 - Notes Revision Lineage
Status:
- complete

Outcome:
- Promoted knowledge preserves richer authorship and revision ancestry

Sub-projects:
- `elowen-notes`
- `elowen-api`
- `elowen-platform/contracts`
- `elowen-platform/db`

Primary capabilities:
- revision ancestry with `previous_revision_id`
- explicit source references
- authored-by metadata on note revisions

Delivered notes:
- `elowen-notes` now stores `previous_revision_id`, `authored_by`, and explicit `source_references` on each note revision and returns them from note detail
- note promotion supports revising an existing note by `note_id`, incrementing the revision version instead of always creating a new note document
- `elowen-api` now reuses the latest promoted note for a job when present and records explicit source references for the job, owning thread, and current job summary
- `elowen-platform/contracts/proto/notes.proto` and the notes bootstrap indexes now reflect the richer revision lineage model

### Slice 12 - VPS Orchestrator Deployment
Status:
- complete

Outcome:
- The orchestrator stack can run on a VPS with documented environment setup, persistence, and externally reachable endpoints

Sub-projects:
- `elowen-platform`
- `elowen-api`
- `elowen-ui`
- `elowen-notes`

Primary capabilities:
- remote environment and secrets configuration
- deployable Compose or equivalent VPS topology
- persistent data volumes and restart behavior
- reverse proxy / public endpoint documentation
- remote health check and operational validation flow

Delivered notes:
- the orchestrator stack now runs on a public VPS over HTTPS with documented env templates, reverse proxying, and persistent service topology
- `elowen-ui` uses same-origin API routing for VPS deployment instead of assuming a local `:8080` backend
- a laptop-hosted edge has been validated end to end against the remote orchestrator, including remote job dispatch from the UI and lifecycle reporting back to the VPS

### Slice 13 - Standalone Laptop Edge Install
Status:
- complete

Outcome:
- The edge agent can be installed on a laptop and connected to the remote orchestrator without relying on the shared local Compose topology

Sub-projects:
- `elowen-edge`
- `elowen-platform`
- `elowen-workspace`

Primary capabilities:
- standalone edge configuration flow
- laptop workspace path configuration
- service or foreground run instructions
- remote API and NATS connectivity
- device registration validation against the remote orchestrator

Delivered notes:
- `elowen-edge` now supports `--env-file` and `ELOWEN_EDGE_ENV_FILE` so laptop runtime configuration no longer depends on reconstructing a long shell command by hand
- the Windows host path now has checked-in startup helpers for foreground launch, detached launch, and Startup-folder installation
- the standalone wrapper flow has been validated end to end against the deployed VPS orchestrator with remote device registration and successful job execution

### Slice 14 - Real Codex Execution Path
Status:
- complete

Outcome:
- A dispatched job runs through a supported external Codex command as the primary validated execution path

Sub-projects:
- `elowen-edge`
- `elowen-platform`
- `codex`

Primary capabilities:
- external Codex command configuration and validation
- preflight checks for Codex availability
- execution log capture for the real runner path
- documented local setup for the laptop agent host

Delivered notes:
- `elowen-edge` now treats the Codex CLI as a supported execution path rather than a generic opaque wrapper command
- the edge performs a startup preflight against the configured Codex CLI and validates the extra argument contract
- dispatched jobs pass the generated request file into `codex exec`, capture JSONL runner output plus the final assistant message, and persist those artifacts in the worktree
- the real runner path has been validated end to end against the deployed VPS orchestrator with a successful remote Codex-executed job

### Slice 15 - Chat-To-Job Bridging
Status:
- complete

Outcome:
- A user can ask for work in a thread and have Elowen turn that request into a dispatched coding job without a separate manual job form as the primary interaction model

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-edge`

Primary capabilities:
- thread message interpretation into job intent
- explicit job creation policy from chat requests
- in-thread visibility into the linked job lifecycle
- reduced dependence on the separate manual create-job workflow

Delivered notes:
- `elowen-api` now exposes an explicit chat-dispatch endpoint that stores the user message, creates the linked job, and writes a thread-visible system acknowledgement
- `elowen-ui` now routes the primary chat composer through that bridge, with explicit repo and optional title/base-branch routing fields instead of relying on the separate manual job form
- the manual job form remains available only as an advanced fallback
- the chat-driven dispatch path has been validated end to end against the deployed VPS orchestrator, with the linked job executed on the laptop through the real Codex runner

### Slice 16 - In-Thread Assistant Completion Replies
Status:
- complete

Outcome:
- The thread itself reflects execution progress and completion so the UI behaves like an interactive assistant, not just a job console

Sub-projects:
- `elowen-api`
- `elowen-ui`

Primary capabilities:
- assistant message persistence for execution milestones
- completion and failure summaries posted back into the owning thread
- clear thread-level acknowledgement when work has been accomplished
- durable linkage between assistant replies and the underlying job record

Delivered notes:
- `elowen-api` now persists assistant-role thread messages for job milestones, keyed by durable lifecycle status markers so replies are not duplicated on re-consume
- threads now show assistant progress when work starts and an assistant completion or failure summary when execution finishes
- the live VPS flow has been validated end to end with thread-visible assistant replies around a real Codex-executed laptop job

### Slice 17 - Conversational Orchestrator Replies
Status:
- complete

Outcome:
- A thread can receive direct assistant replies from the orchestrator without creating a laptop job by default

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `codex`

Primary capabilities:
- orchestrator-side assistant reply path for normal chat
- persisted conversational assistant messages without linked job records
- reply context assembled from thread history, notes, and orchestrator state
- model and runtime configuration for non-job assistant turns

Scope notes:
- add a thread chat endpoint distinct from chat-dispatch so `Workflow #2` does not reuse the job-creation path
- preserve the current chat-dispatch route as the explicit `Workflow #1` execution entry point
- conversational failures should surface in-thread without creating job or approval records

Delivered notes:
- normal thread chat now routes through an orchestrator-side assistant reply path instead of creating a laptop job by default
- OpenAI-backed assistant configuration is wired into the VPS stack with a deterministic fallback path when the assistant runtime is unavailable
- the UI now treats explicit job creation as a secondary path rather than the default composer action

### Slice 18 - Explicit Workflow Handoff
Status:
- complete

Outcome:
- Conversational chat can escalate visibly and intentionally into the existing job-backed execution workflow

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `codex`

Primary capabilities:
- explicit handoff from conversation to execution
- assistant-generated execution intent summary before dispatch
- visible linkage from conversational turns to the created job
- no silent creation of laptop jobs from ordinary chat

Scope notes:
- `Workflow #2` remains the default for normal conversation
- `Workflow #1` remains the only path that creates real jobs, approvals, and push gates
- handoff records should make it obvious which message or assistant plan caused the dispatch

Delivered notes:
- a conversational message or assistant plan can now be explicitly promoted into a dispatched job from the thread UI
- handoff acknowledgements are persisted in-thread so the transcript shows which conversational turn created the job
- live verification confirmed the `Workflow #2` to `Workflow #1` transition with a real laptop execution

### Slice 19 - Conversational Context and Mode Visibility
Status:
- complete

Outcome:
- Conversational replies are grounded, and the UI makes conversational versus job-backed messages easy to distinguish

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-notes`
- `elowen-platform`

Primary capabilities:
- conversational context bundle from thread history, promoted notes, related jobs, and summaries
- UI indicators for conversational replies versus execution milestones
- source references or lightweight citations for grounded assistant replies where appropriate
- conversation-first composer and thread affordances that coexist with explicit dispatch controls

Scope notes:
- `Workflow #2` should not depend on a laptop worktree
- note retrieval for conversational grounding should reuse the existing notes service rather than introduce a separate memory system
- thread transcripts should remain legible when a conversation escalates into a dispatched job

Delivered notes:
- conversational replies are now grounded with labeled thread, job, summary, approval, and note context from the orchestrator state
- thread messages render with distinct conversational, handoff, dispatch, and job-update badges so the transcript stays readable
- explicit handoff controls are now limited to messages that make sense to promote into execution

### Slice 20 - Approval-Backed Push Execution
Status:
- complete

Outcome:
- approving a push gate triggers a real post-approval push on the laptop instead of only recording the approval in the API

Sub-projects:
- `elowen-api`
- `elowen-edge`
- `elowen-ui`
- `elowen-platform`

Primary capabilities:
- approval-resolution command path from API to edge
- explicit post-approval push lifecycle states and events
- thread and job detail visibility for push start, completion, and failure
- approval UI that reflects the real push action rather than a passive status change

Scope notes:
- approval should no longer mark a job complete before the gated push actually runs
- the laptop edge should push from the existing worktree and publish its own success or failure outcome
- job and thread surfaces should make the post-approval phase visible without obscuring the original execution summary
- approval-backed push should only complete when the branch contains a meaningful commit to publish; pushing an uncommitted worktree state is not sufficient

Delivered notes:
- approval resolution now triggers a real edge-side post-approval push command instead of only recording approval in the API
- the edge now publishes explicit `job.push_started` and `job.push_completed` lifecycle events, and the UI/thread surfaces show that phase clearly
- live verification confirmed the branch ref is pushed to `origin` after approval
- the current handoff treats approval-backed push as a shipped product behavior rather than an open roadmap blocker

### Slice 21 - Conversational Execution Drafts
Status:
- complete

Outcome:
- `Workflow #2` can refine a structured execution draft before the user explicitly escalates into `Workflow #1`

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `codex`

Primary capabilities:
- assistant-generated execution draft with normalized title, repo, branch, and request text
- inline draft review and refinement during normal conversation
- one-click promotion from a reviewed draft into an actual laptop job
- clearer separation between exploratory chat and execution-ready intent

Scope notes:
- draft generation should improve the handoff quality without silently dispatching a job
- the assistant should stay conversational and editable until the user explicitly promotes the draft
- this slice builds on the shipped `Workflow #2` baseline rather than replacing it

Delivered notes:
- conversational turns can now produce a structured execution draft with normalized title, repository, branch, and request text before the user explicitly escalates into `Workflow #1`
- reviewed drafts can be promoted into real laptop jobs from the thread UI without collapsing normal conversation back into silent auto-dispatch
- remaining draft-quality polish is now tracked in a later planned slice instead of leaving Slice 21 open

### Slice 22 - Read-Only Request Handling
Status:
- complete

Outcome:
- informational repository requests can run through Elowen without implying file edits, approvals, or pushes

Sub-projects:
- `elowen-api`
- `elowen-edge`
- `elowen-ui`
- `codex`

Primary capabilities:
- explicit read-only execution intent
- approval suppression for informational requests
- transcript language that makes the no-change expectation visible
- result handling that keeps repository investigation distinct from mutating work

Delivered notes:
- the shipped baseline now supports read-only execution intent for informational repository requests
- read-only requests can complete without surfacing unnecessary push approval flow
- transcript and result handling make the no-change expectation visible before and after execution

### Slice 23 - Thread-Visible Final Job Result Message
Status:
- complete

Outcome:
- the main thread transcript carries the clearest final job outcome instead of forcing the user into job detail for the primary answer

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-edge`

Primary capabilities:
- thread-visible final result message derived from the job outcome
- clearer separation between interim lifecycle chatter and the final answer
- supporting execution detail kept available without overwhelming the chat surface

Delivered notes:
- final job outcomes are now surfaced back into the owning thread as a primary chat artifact rather than only as supporting job metadata
- the thread-visible completion path now favors the high-signal execution result over raw operational ceremony
- remaining message presentation polish is tracked as later chat-surface follow-on work instead of as an open blocker for Slice 23

### Slice 24 - Chat-Forward UI Redesign
Status:
- complete

Outcome:
- the primary product surface is chat-first, with operational detail available through deliberate progressive disclosure

Sub-projects:
- `elowen-ui`
- `elowen-api`

Primary capabilities:
- centered thread/chat-first layout
- sticky primary composer with folded secondary controls
- reduced inline job-noise in the main transcript
- clearer navigation from chat into supporting operational detail

Delivered notes:
- the shipped UI now centers the thread/chat view and keeps the composer sticky and primary
- secondary controls moved into folded context panels, and job-completion noise is trimmed behind disclosure instead of dominating the transcript
- remaining shell and ergonomics polish is now tracked in later planned UI slices rather than keeping Slice 24 open

### Slice 25 - Web UI Authentication
Status:
- complete

Outcome:
- the VPS-hosted web UI is protected by a real authenticated session boundary instead of relying on network location alone

Sub-projects:
- `elowen-api`
- `elowen-ui`
- `elowen-platform`

Primary capabilities:
- password-gated web session flow
- API-issued cookie sessions
- server-side session validation for UI/API access
- authenticated browser behavior for the deployed product surface

Delivered notes:
- the shipped VPS stack now supports a real password-gated web session boundary backed by API-issued cookies and server-side session storage
- authenticated session behavior has been validated against the deployed public UI/API surface
- richer operator identity and authorization work is now tracked as later planned security hardening rather than as an open Slice 25 blocker

### Slice 26 - Parent-Directory Repository Discovery
Status:
- complete

Outcome:
- the edge can discover and advertise nested repositories from trusted parent roots instead of relying only on manually typed repository paths

Sub-projects:
- `elowen-edge`
- `elowen-api`
- `elowen-ui`

Primary capabilities:
- trusted parent-directory registration
- nested repository discovery during device registration
- orchestrator-visible discovered repository inventory
- reduced typo-prone manual repository configuration

Delivered notes:
- devices can now register trusted parent directories and advertise discovered nested git repositories alongside any explicit per-repo overlay
- the shipped edge baseline uses parent-directory repository discovery during laptop registration
- richer repository policy controls and UI-native selection are now tracked in a later planned slice rather than as an open Slice 26 blocker

### Slice 27 - Material 3 System Alignment
Status:
- complete

Outcome:
- the UI has a coherent Material 3-inspired shell instead of a purely utilitarian scaffold

Sub-projects:
- `elowen-ui`

Primary capabilities:
- chat-dominant viewport behavior
- Material-style navigation rail and send affordances
- clearer mobile/desktop chrome and details behavior
- stronger visual hierarchy across the chat-first shell

Delivered notes:
- Slice 27 shipped the first Material 3-aligned UI shell pass, including chat-dominant viewport behavior, explicit Jobs and Details rail destinations, Material Symbols for the send control, compact mobile details sheet behavior, and tighter desktop/mobile chrome
- the shipped pass provides the current design baseline rather than leaving the UI on the earlier utilitarian shell
- deeper token, component, and motion cleanup is now tracked as later planned UI polish rather than as an open Slice 27 blocker

### Slice 28 - Mutual Orchestrator And Edge Trust
Status:
- complete

Outcome:
- edge registration is backed by mutual proof instead of only by reachability and a claimed device identifier

Sub-projects:
- `elowen-api`
- `elowen-edge`
- `elowen-platform`

Primary capabilities:
- orchestrator-signed registration challenge
- pinned orchestrator key verification on the edge
- edge-signed registration proof verification on the API
- rejection of unsigned registration when trusted mode is required

Delivered notes:
- signed edge registration is now live, with the orchestrator signing registration challenges, the edge verifying a pinned orchestrator public key, and the API verifying the edge proof
- unsigned registration is rejected when `ELOWEN_REQUIRE_TRUSTED_EDGE_REGISTRATION=true`
- key rotation, revocation, and multi-edge enrollment hardening are now tracked in a later planned slice rather than as an open Slice 28 blocker

### Slice 29 - SPA State Persistence And Realtime Updates
Status:
- complete

Outcome:
- the UI behaves like a long-lived client application instead of a page that keeps replacing its own state

Sub-projects:
- `elowen-ui`
- `elowen-api`

Primary capabilities:
- local persistence for selected thread, job, panel, and composer state
- authenticated SSE for thread, job, and device updates
- targeted refresh logic that cooperates with the chat-first shell
- reduced dependence on page-style full-state replacement

Scope notes:
- keep the current client-side rendered model instead of switching to SSR
- focus on preserving interaction state across background updates, reconnects, and navigation
- make realtime behavior trustworthy enough that polling becomes only a fallback path

Planned validation:
- authenticated UI sessions should survive the normal realtime update loop
- transcript position, selected job, and active composer text should stay stable while background updates arrive
- polling should remain available only as a slower fallback until browser automation covers realtime behavior

Delivered baseline so far:
- the current Slice 29 baseline already preserves selected thread, selected job, composer text, panel state, and transcript scroll across background updates
- authenticated SSE is now the normal update path for thread, job, and device change notifications
- the UI now explicitly closes, retries, and catch-up refreshes SSE connections with capped reconnect/backoff instead of relying on implicit browser retry behavior
- polling remains as a slower fallback until browser automation covers the critical realtime UI flows

Remaining gap:
- none inside Slice 29; the next remaining validation step moves to Slice 30 browser automation

### Slice 30 - UI Browser Automation
Status:
- planned

Outcome:
- the chat-first product surface has browser-level regression coverage for layout, navigation, auth, and realtime behavior

Sub-projects:
- `elowen-ui`
- `elowen-api`
- `elowen-platform`

Primary capabilities:
- dev/CI Playwright coverage for the deployed UI shell
- stable `data-testid` hooks for critical user flows
- browser-level checks for login, mobile layout, sticky composer behavior, and realtime updates
- confidence to remove the remaining polling fallback after realtime behavior is proven

Scope notes:
- keep the suite focused on product-critical UI behavior rather than snapshot churn
- use stable selectors and deterministic local/dev deployment assumptions
- use the suite to unlock removal of the remaining polling fallback in later UI hardening

Planned first-pass coverage:
- login flow against the authenticated web UI
- mobile details sheet interactions and backdrop stacking
- sticky composer placement and message-pane scroll containment
- realtime UI updates such as `Job Update` versus `Job Complete` presentation
- browser-driven confidence for eventually retiring the remaining polling fallback

---

## 11. Workflow #1 - Job Dispatch MVP

Target slice:
- `Slice 16 - In-Thread Assistant Completion Replies`

Definition of done for `Workflow #1`:

1. Deploy the orchestrator stack to a VPS.
2. Install and start the edge agent on the local laptop.
3. Open the web UI and start an interactive thread.
4. Post a coding request in the thread.
5. Elowen converts that request into a dispatched job for the laptop.
6. The laptop accepts the job, creates a worktree, and runs real Codex.
7. Validation runs and the edge reports lifecycle events back to the orchestrator.
8. The orchestrator persists the result and posts assistant replies into the same thread for both start and completion.
9. The UI shows the task as accomplished in the chat, with job detail and push approval available as supporting context.

Current gap relative to the full true MVP bar:
- none at the slice level; the true MVP bar described here is now implemented on `main`

---

## 12. True MVP End-to-End Definition

Definition of done:

1. Deploy the orchestrator stack to a VPS.
2. Install and start the edge agent on the local laptop.
3. Open the web UI and start an interactive thread.
4. Post a coding request in the thread.
5. Elowen converts that request into a dispatched job for the laptop.
6. The laptop accepts the job, creates a worktree, and runs real Codex.
7. Validation runs and the edge reports lifecycle events back to the orchestrator.
8. The orchestrator persists the result and posts an assistant reply into the same thread.
9. The user can see in the chat that the task was accomplished, with job detail available as supporting context.

---

## 13. Orchestrator Rules

### Owns
- state transitions
- routing
- approvals
- job lifecycle

### Model assists with
- interpreting requests
- selecting context
- generating summaries

### Constraints
- one active job
- no scheduling
- no automatic push
- no automatic note promotion

---

## 14. Context Bundle

### Always
- instructions
- repo context
- branch
- config

### Sometimes
- summaries
- notes
- skills

### Never
- full history dumps

---

## 15. Edge Agent Rules

### Must
- validate repo
- create worktree
- run the Codex wrapper and, for the true MVP path, a configured external Codex command
- publish events
- run tests
- enforce sandbox once Slice 10 lands

### Must not
- run arbitrary commands
- write outside repo
- push without approval

---

## 16. UI Scope (v1)

- thread list
- thread detail
- job cards
- approvals
- job detail page
- MVP path: thread-native execution replies

---

## 17. Observability

- structured logs
- correlation IDs
- job event table
- basic error logging

---

## 18. Codex Build Guidelines

- small vertical slices
- explicit behavior
- typed boundaries
- ULIDs everywhere
- TOML config
- avoid overengineering

---

## 19. Slice Delivery Order

1. `Slice 0 - Workspace and Runtime Foundation`
2. `Slice 1 - Threads and Conversation Surface`
3. `Slice 2 - Device Presence and Execution Eligibility`
4. `Slice 3 - Job Creation and Dispatch`
5. `Slice 4 - Local Execution Loop`
6. `Slice 5 - Results, Summaries, and Approval Gate`
7. `Slice 6 - Notes Promotion and Retrieval`
8. `Slice 7 - Hardening and Platform Maturity`
9. `Slice 8 - Global Jobs Surface`
10. `Slice 9 - Build and Test Execution`
11. `Slice 10 - Edge Sandbox Enforcement`
12. `Slice 11 - Notes Revision Lineage`
13. `Slice 12 - VPS Orchestrator Deployment`
14. `Slice 13 - Standalone Laptop Edge Install`
15. `Slice 14 - Real Codex Execution Path`
16. `Slice 15 - Chat-To-Job Bridging`
17. `Slice 16 - In-Thread Assistant Completion Replies`
18. `Slice 17 - Conversational Orchestrator Replies`
19. `Slice 18 - Explicit Workflow Handoff`
20. `Slice 19 - Conversational Context and Mode Visibility`
21. `Slice 20 - Approval-Backed Push Execution`
22. `Slice 21 - Conversational Execution Drafts`
23. `Slice 22 - Read-Only Request Handling`
24. `Slice 23 - Thread-Visible Final Job Result Message`
25. `Slice 24 - Chat-Forward UI Redesign`
26. `Slice 25 - Web UI Authentication`
27. `Slice 26 - Parent-Directory Repository Discovery`
28. `Slice 27 - Material 3 System Alignment`
29. `Slice 28 - Mutual Orchestrator And Edge Trust`
30. `Slice 29 - SPA State Persistence And Realtime Updates`
31. `Slice 30 - UI Browser Automation`
32. `Slice 31 - Chat Surface And Draft UX Polish`
33. `Slice 32 - Identity And Authorization Hardening`
34. `Slice 33 - Repository Policy And Selection UX`
35. `Slice 34 - Trust Lifecycle Management`
36. `Slice 35 - Service-Grade Laptop Edge Runtime`
37. `Slice 36 - Shared Rust Message Contracts`
38. `Slice 37 - Notes Retrieval And Context Expansion`
39. `Slice 38 - Tools And Integration Expansion`
40. `Slice 39 - Kubernetes Deployment Validation`
41. `Slice 40 - CI Workflow Maintenance`

---

## 20. Next Deliverable

Slice set `0` through `28` is complete, including the post-MVP Workflow #2 work for conversational replies, explicit handoff, transcript visibility, approval-backed push execution, conversational execution drafts, read-only request handling, thread-visible final job results, chat-forward UI redesign, a real web UI authentication boundary, parent-directory repository discovery for edge registration, the first Material 3-aligned UI shell pass, and mutual orchestrator/edge trust for signed edge registration.

Current delivered baseline:
- local Compose stack for the orchestrator topology
- VPS-hosted orchestrator deployment over HTTPS
- standalone laptop edge runtime with env-file based startup and documented Windows launch/install helpers
- GHCR prebuilt images for VPS-hosted Rust services so the small VPS does not compile on deploy
- real Codex CLI execution path with startup preflight and persisted runner artifacts
- chat-driven job dispatch from a thread without relying on the separate manual job form
- persisted threads, messages, jobs, approvals, notes, and job events
- signed edge registration, parent-directory repo discovery, probing, dispatch, worktree creation, lifecycle events, summaries, and validation reporting
- authenticated web UI sessions
- authenticated SSE notifications for thread, job, and device changes, with polling retained as a fallback

True MVP critical path from here:
- no remaining slice-level blockers

Post-MVP slice plan from here:
- `Slice 30 - UI Browser Automation`
- `Slice 31 - Chat Surface And Draft UX Polish`
- `Slice 32 - Identity And Authorization Hardening`
- `Slice 33 - Repository Policy And Selection UX`
- `Slice 34 - Trust Lifecycle Management`
- `Slice 35 - Service-Grade Laptop Edge Runtime`
- `Slice 36 - Shared Rust Message Contracts`
- `Slice 37 - Notes Retrieval And Context Expansion`
- `Slice 38 - Tools And Integration Expansion`
- `Slice 39 - Kubernetes Deployment Validation`
- `Slice 40 - CI Workflow Maintenance`

Immediate next deliverable:
- `Slice 30 - UI Browser Automation`

Slice 29 closeout:
- selected thread, selected job, composer text, panel state, and transcript scroll now persist across background updates
- authenticated SSE remains the normal update path for thread, job, and device change notifications
- the UI explicitly manages capped reconnect/backoff recovery and catch-up refresh after SSE interruptions
- polling remains as a slower fallback until browser automation proves the critical realtime UI flows

Stop point as of 2026-04-06:
- workspace pins documentation-refreshed child repos: `elowen-api` at `1200014`, `elowen-edge` at `80cd15e`, `elowen-notes` at `5f3c4ec`, `elowen-platform` at `af9752a`, and `elowen-ui` at `96dc194`
- VPS API image is `ghcr.io/elowen-assistant/elowen-api:sha-d0e31259aab80b3b1897a97b589372aabb3271ff`
- VPS UI image is `ghcr.io/elowen-assistant/elowen-ui:sha-a480c8b52ee51177aef2c4ebc58a96e0adb48477`
- latest deployed verification covered public UI/API reachability, authenticated session behavior, and authenticated SSE delivery for `device.changed`
- recommended next project step is `Slice 30 - UI Browser Automation`

Important note:
- `Workflow #2` baseline and the first post-MVP execution/chat/security/discovery expansion set are now live through `Slice 28`
- all currently tracked remaining work is now assigned to planned slices `29` through `40`

---

## 21. Planned Post-30 Slice Set

This section tracks all remaining outstanding work after the current `Slice 30` focus. Every currently tracked follow-on item is assigned to a planned slice.

### Workflow #2 - Conversational Orchestrator Reply Path

Definition of done:

1. The user can post a normal conversational message in a thread without that message automatically becoming a dispatched laptop job.
2. The orchestrator-side assistant can reply directly in the thread in a chatbot-style manner.
3. The assistant can answer questions, clarify intent, discuss plans, and help the user decide whether execution is needed.
4. The assistant can explicitly transition from `Workflow #2` into `Workflow #1` when the conversation turns into a real coding task.
5. The thread makes it clear whether a reply came from conversational orchestrator mode or from a job-backed execution path.
6. Existing approval and push gates remain tied to real jobs, not to conversational replies.

Design constraints:

- `Workflow #1` remains the execution workflow.
- `Workflow #2` is conversational first and should not silently create jobs from ordinary chat.
- Context for `Workflow #2` should default to thread history, notes, and orchestrator state rather than a laptop worktree.
- When `Workflow #2` escalates into `Workflow #1`, the transition should be visible in the thread and result in a normal dispatched job record.
- The shipped baseline for `Workflow #2` is now live through `Slice 21`, with later polish and expansion work assigned to the planned slices below.

### Slice 31 - Chat Surface And Draft UX Polish

Status:
- planned

Assigned scope:
- stronger differentiation between `Job Update` and `Job Complete` messages
- cleaner final-result presentation with operational detail behind disclosure by default
- tighter execution-draft synthesis for title, repository, branch, and request text
- chat-shell polish such as `Ctrl+Enter`, timestamp formatting, clearer operational navigation, viewport height discipline, and follow-on Material-style cleanup

Why this slice exists:
- Slice 21, Slice 23, Slice 24, and Slice 27 shipped the baseline behavior, but the remaining gaps are now mostly about transcript quality, ergonomics, and UI polish

### Slice 32 - Identity And Authorization Hardening

Status:
- planned

Assigned scope:
- per-action authorization on top of the current authenticated session wall
- clearer operator identity handling and session lifecycle management
- eventual replacement of the current shared-password gate with a real authentication service

Why this slice exists:
- Slice 25 is good enough for a private operator deployment, but broader or more exposed operation needs stronger identity semantics

### Slice 33 - Repository Policy And Selection UX

Status:
- planned

Assigned scope:
- per-root exclusions and hidden-repository policy
- device-side repository policy overrides
- UI-native repository picker fed from orchestrator-known discovered repositories
- less reliance on free-form repository text in execution handoff and dispatch controls

Why this slice exists:
- Slice 26 solved the discovery baseline, but operator ergonomics still depend on better policy controls and safer repository selection UX

### Slice 34 - Trust Lifecycle Management

Status:
- planned

Assigned scope:
- orchestrator and edge key rotation
- revocation handling
- visible trust state in the UI
- safer multi-edge enrollment workflow

Why this slice exists:
- Slice 28 shipped the first real trust boundary, but long-term secure operation needs lifecycle tooling rather than static key setup

### Slice 35 - Service-Grade Laptop Edge Runtime

Status:
- planned

Assigned scope:
- stronger Windows service or hardened scheduled-task model
- more durable startup and recovery behavior
- reduced dependence on Startup-folder and session-bound launch flows

Why this slice exists:
- Slice 13 shipped a workable standalone runtime, but the laptop startup story is not yet service-grade on every host

### Slice 36 - Shared Rust Message Contracts

Status:
- planned

Assigned scope:
- shared internal Rust message crate
- reduced DTO drift across Rust services
- clearer ownership for common contract evolution while keeping external wire compatibility stable

Why this slice exists:
- the current cross-crate message duplication is now a meaningful maintenance risk

### Slice 37 - Notes Retrieval And Context Expansion

Status:
- planned

Assigned scope:
- richer retrieval and ranking
- stronger generated context assembly
- clearer use of promoted notes as assistant memory rather than only archival storage

Why this slice exists:
- the notes system already supports promotion, revision history, and filtered retrieval, so the next useful step is deeper grounding and memory quality

### Slice 38 - Tools And Integration Expansion

Status:
- planned

Assigned scope:
- broader non-coding tool surfaces
- additional integration paths
- product flows that do not assume repository work as the only meaningful task type

Why this slice exists:
- the coding path is strong enough to serve as the foundation, but the broader assistant vision depends on more than repository execution

### Slice 39 - Kubernetes Deployment Validation

Status:
- planned

Assigned scope:
- validated Kubernetes deployment path for the current real system
- operational guidance for the Kubernetes topology
- confirmation that the Compose-first architecture still has a real migration path

Why this slice exists:
- Kubernetes remains intentionally non-MVP, but the migration path should eventually become real rather than remaining only scaffolding

### Slice 40 - CI Workflow Maintenance

Status:
- planned

Assigned scope:
- refresh third-party GitHub Action versions in image-publish and related workflows
- rerun and verify the image-publish path on the current GitHub-hosted runner platform
- remove avoidable runtime deprecation noise from the release pipeline

Why this slice exists:
- this work is low priority compared with product slices, but it is still real outstanding maintenance work and now has an explicit place in the plan
