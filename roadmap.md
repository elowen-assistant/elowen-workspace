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
- post-MVP expansion areas tracked in `Future Enhancements`

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

### Current verified delivery status as of 2026-04-02

- Slice set `0` through `16` is implemented on `main`.
- The completed true-MVP path is now recorded as `Workflow #1`: thread-native request -> dispatched laptop job -> assistant reply back into the same thread.
- The current product now behaves as a thread-native assistant surface backed by jobs, not only as a manual job console.
- `elowen-ui` provides thread chat, chat-dispatch routing fields, global jobs, job detail, approvals, and notes, while keeping the manual job form only as an advanced fallback.
- `elowen-api` persists user, system, and assistant thread messages; exposes `/api/v1/threads/{thread_id}/chat-dispatch`; creates linked jobs from chat requests; and posts assistant lifecycle replies back into the owning thread.
- `elowen-edge` registers the device, creates per-job git worktrees, supports a real `codex exec` runner with startup preflight and captured runner artifacts, runs repo-owned validation from `.assistant/config.toml`, and now enforces a workspace sandbox boundary around runtime working directories and validation command launch policy.
- `elowen-platform` documents both the VPS deployment path and the standalone laptop edge path, including same-origin HTTPS routing for the UI/API split, Windows startup helpers for the laptop runtime, and the edge sandbox expectation.
- `elowen-notes` now preserves note revision ancestry and authorship metadata, while `elowen-api` revises the current promoted job note instead of always creating a parallel note record.
- The VPS-to-laptop flow has been validated as the current user-visible baseline, while `elowen-platform/k8s/base` remains migration scaffolding rather than a production deployment target.
- The true MVP slice set is now complete on `main`.

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
- in progress

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

Remaining gap:
- the current approval-backed push path can still push a branch ref that points at `main` when the worktree changes were never committed, so Slice 20 remains open until commit-before-push is enforced

### Slice 21 - Conversational Execution Drafts
Status:
- planned

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

Follow-up refinements:
- tighten execution-draft title synthesis so generated draft titles are concise task labels rather than just echoing the first clause of the user's request

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

---

## 20. Next Deliverable

Slice set `0` through `26` is complete, including the post-MVP Workflow #2 work for conversational replies, explicit handoff, transcript visibility, approval-backed push execution, conversational execution drafts, read-only request handling, thread-visible final job results, chat-forward UI redesign, a real web UI authentication boundary, and parent-directory repository discovery for edge registration.

Current delivered baseline:
- local Compose stack for the orchestrator topology
- VPS-hosted orchestrator deployment over HTTPS
- standalone laptop edge runtime with env-file based startup and documented Windows launch/install helpers
- real Codex CLI execution path with startup preflight and persisted runner artifacts
- chat-driven job dispatch from a thread without relying on the separate manual job form
- persisted threads, messages, jobs, approvals, notes, and job events
- device registration, probing, dispatch, worktree creation, lifecycle events, summaries, and validation reporting

True MVP critical path from here:
- no remaining slice-level blockers

Post-MVP slice plan from here:
- `Slice 27 - Material 3 System Alignment`

Immediate next deliverable:
- `Slice 27 - Material 3 System Alignment`

Immediate hardening focus inside Slice 27:
- adopt a proper Material 3 token layer instead of the current light visual borrowing
- align adaptive navigation behavior across desktop, tablet, and mobile breakpoints
- standardize component semantics, elevation, and motion so future UI changes stop drifting piecemeal

Important note:
- `Workflow #2` baseline is now live through `Slice 21`
- broader future enhancements still remain outside this slice plan

---

## 21. Future Enhancements

This section tracks post-MVP work that is not on the true-MVP critical path. Some items below are now assigned to planned slices, while others remain backlog only.

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
- The shipped baseline for `Workflow #2` is `Slice 17` through `Slice 19`, and `Slice 21` is the next planned expansion of that conversational path.

### Candidate backlog

- Windows service-grade laptop startup path
  - Current state: Slice 13 ships a working env-file runtime plus Windows wrapper and Startup-folder launcher.
  - Gap: Task Scheduler registration can fail on some machines with local permission or policy issues, so the current auto-start story is session-bound rather than service-grade.
  - Why it matters later: a stronger Windows startup path would improve reliability, reduce dependence on interactive logon, and make the laptop edge easier to operate in stricter environments.
  - Suggested future direction: investigate a more durable Windows service model or a hardened scheduled-task path after the true MVP slices are complete.
- Notes and RAG expansion
  - Current state: notes promotion, revisioned note storage, graph links, and filtered retrieval are in place.
  - Gap: the system does not yet use richer retrieval, ranking, or generated context assembly as a first-class assistant capability.
  - Why it matters later: this is the natural path from durable note storage toward stronger memory and context injection.
- Additional tools and integrations
  - Current state: the coding path is the primary supported execution model.
  - Gap: broader non-coding tool surfaces and integrations are not yet part of the product flow.
  - Why it matters later: expanding the tool surface is part of turning the assistant from a coding orchestrator into a broader personal assistant platform.
- Read-only request handling
  - Assigned slice: `Slice 22 - Read-Only Request Handling`
  - Current state: conversational and job-backed flows can still converge on file edits and push gates when the underlying request was informational rather than change-oriented.
  - Gap: the system does not yet distinguish clearly enough between read-only repository investigation and requests that should result in durable file changes or pushed branches.
  - Why it matters later: users should not need to approve permanent repository mutations just to ask for information, explanation, or review of existing code.
  - Suggested future direction: add explicit read-only execution intent and approval suppression for informational requests, with transcript language that makes the no-change expectation visible before dispatch.
- Parent-directory repository discovery
  - Assigned slice: `Slice 26 - Parent-Directory Repository Discovery`
  - Current state: devices can now register trusted parent directories and advertise discovered nested git repositories alongside any explicit per-repo overlay.
  - Remaining gap: there is still no richer policy layer for per-root exclusions, hidden repositories, or UI-native repo picking.
  - Why it matters later: discovery solves the typo-prone manual baseline, but broader operator ergonomics still depend on better policy controls and surfaced repo metadata.
  - Suggested future direction: build on the shipped discovery path with per-root exclusions, device-side policy overrides, and UI controls that present discovered repositories directly.
- Web UI authentication
  - Assigned slice: `Slice 25 - Web UI Authentication`
  - Current state: the shipped VPS stack now supports a real password-gated web session boundary backed by API-issued cookies and server-side session storage.
  - Remaining gap: the current model is still intentionally simple and does not yet distinguish between operators, permission levels, or external identity providers.
  - Why it matters later: the basic session wall is enough for a private operator deployment, but multi-user or internet-exposed operation still needs authorization and stronger identity management.
  - Suggested future direction: build on the current session layer with per-action authorization, better operator identity handling, and eventually external auth if the deployment model expands.
  - Future note: replace the current shared-password session gate with a real authentication service so identity, session lifecycle, and operator management do not stay hand-rolled indefinitely.
- Chat-forward UI redesign
  - Assigned slice: `Slice 24 - Chat-Forward UI Redesign`
  - Current state: the shipped UI now centers the thread/chat view, keeps the composer sticky and primary, moves secondary controls into folded context panels, trims job-completion noise behind a disclosure, and now includes a light Material 3-inspired pass for surfaces, controls, and visual hierarchy.
  - Remaining gap: the redesign still has follow-on polish opportunities around richer message-type differentiation, repository selection ergonomics, a cleaner split between the primary chat surface and deeper operational browsing, and a true design-system-level Material alignment.
  - Why it matters later: Workflow #2 is now the preferred default path, so future UI work should keep making Elowen feel like a high-quality messaging app while preserving deliberate access to operational tools.
  - Suggested future direction: continue iterating on the generic messaging-app shell inspired by ChatGPT, Claude, and Google Messages, with stronger progressive disclosure, clearer completion/update semantics, refined mobile ergonomics, and only a light Material 3 borrowing inside this slice.
  - Future note: repository selection in dispatch controls should become a select or searchable picker fed from orchestrator-known edge repositories, not a free-form text box that risks typos.
  - Future note: the dedicated job browsing surface should likely become a separate screen from the primary chat/thread experience, so users who want to search and inspect jobs can do so without turning the main assistant view into a jobs console.
- Material 3 system alignment
  - Assigned slice: `Slice 27 - Material 3 System Alignment`
  - Current state: Slice 24 now borrows a few Material 3 cues for surface color, control treatment, and hierarchy, but the UI is still fundamentally a custom design.
  - Gap: the app does not yet follow a full Material 3 token system, adaptive navigation model, component semantics, or motion/elevation guidance consistently.
  - Why it matters later: a dedicated alignment pass would make the UI more coherent across desktop/mobile states and reduce ad hoc styling drift as the product grows.
  - Suggested future direction: adopt a proper Material 3 design token layer, component mapping, adaptive navigation behavior, and edge-to-edge/mobile conventions in a dedicated slice instead of continuing to fold those structural changes into general UI cleanup.
- Thread-visible final job result message
  - Assigned slice: `Slice 23 - Thread-Visible Final Job Result Message`
  - Current state: the thread shows lifecycle milestones and summary-oriented assistant replies, but it does not reliably surface the final runner `last_message` as the primary completion artifact in chat.
  - Gap: the most direct textual result from a completed job is still easier to find in job detail than in the main transcript.
  - Why it matters later: for a chat-first product, the thread itself should carry the clearest final answer, especially when the runner's last message is the most useful high-signal explanation of what happened.
  - Suggested future direction: when a job completes, render the final `last_message` directly into the thread transcript with clear visual treatment as the job's outcome message, while keeping deeper execution reports and summaries available as secondary detail.
  - UX note: completion replies currently include too much operational ceremony inline. For example, a read-only result like "The Cargo package name is `elowen-api`..." should be the default chat-visible artifact, while branch/build/test/change-count/no-push detail should be hidden behind a `More Details` disclosure by default.
  - Future note: differentiate thread-visible `Job Update` messages from `Job Complete` messages so interstitial lifecycle chatter can stay visually secondary while final outcome messages get stronger treatment and cleaner defaults.
- Kubernetes deployment path
  - Current state: the architecture keeps a Compose-first approach and an explicit upgrade path, but the working deployment is still Compose-based.
  - Gap: there is no validated Kubernetes deployment story for the current real system.
  - Why it matters later: Kubernetes is a scale and operability enhancement, not part of the true MVP path.
- GitHub Actions runtime maintenance
  - Current state: the new GHCR image-publish workflows are working, but the current third-party action versions still emit GitHub's Node 20 deprecation warning.
  - Gap: the workflow dependencies should be upgraded to versions that explicitly support the newer GitHub-hosted runner JavaScript runtime.
  - Priority: low. The workflows are functional now, and this is routine maintenance rather than a product blocker.
  - Suggested future direction: refresh the checkout/buildx/login/metadata/build-push actions after the current deployment path settles, then rerun the image-publish path to confirm the warning is gone.
