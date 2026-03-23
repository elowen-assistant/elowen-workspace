# Personal Assistant / Local Codex System Implementation Plan

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
- future expansion to notes/RAG, more tools, and Kubernetes

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
- Edge agent accepts a dispatched job, creates a worktree, runs Codex, and emits lifecycle events
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
- the default execution wrapper is simulated, with an explicit configuration path for a future external Codex command

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
- filtered retrieval for future RAG/context use

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
- pending

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

### Slice 10 - Edge Sandbox Enforcement
Status:
- pending

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

### Slice 11 - Notes Revision Lineage
Status:
- pending

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

---

## 11. First End-to-End Slice Definition

Target slice:
- `Slice 8 - Global Jobs Surface`

Definition of done:

1. Create thread
2. Send coding request
3. Create job
4. Probe device
5. Dispatch job
6. Edge agent accepts
7. Worktree is created
8. Codex runs
9. Tests run
10. Job completes
11. Summary is generated
12. Push approval is proposed

---

## 12. Orchestrator Rules

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

## 13. Context Bundle

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

## 14. Edge Agent Rules

### Must
- validate repo
- create worktree
- run Codex
- publish events
- run tests
- enforce sandbox

### Must not
- run arbitrary commands
- write outside repo
- push without approval

---

## 15. UI Scope (v1)

- thread list
- thread detail
- job cards
- approvals
- job detail page

---

## 16. Observability

- structured logs
- correlation IDs
- job event table
- basic error logging

---

## 17. Codex Build Guidelines

- small vertical slices
- explicit behavior
- typed boundaries
- ULIDs everywhere
- TOML config
- avoid overengineering

---

## 18. Slice Delivery Order

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

---

## 19. Next Deliverable

Slice set `0` through `8` is now complete.

Primary outputs:
- lightweight global jobs list in the UI
- direct job-to-thread navigation from the global jobs surface
- explicit follow-on slices for build/test execution, sandbox enforcement, and notes revision lineage
