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

### Slice 3 - Job Creation and Dispatch
Outcome:
- A coding request from a thread becomes a tracked job
- The orchestrator routes the job to the registered edge device over JetStream

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

### Slice 4 - Local Execution Loop
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

### Slice 5 - Results, Summaries, and Approval Gate
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
- note create/read/query APIs
- note references from threads and jobs
- promotion into an ArangoDB document + graph model with revision tracking
- filtered retrieval for future RAG/context use

### Slice 7 - Hardening and Platform Maturity
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

---

## 11. First End-to-End Slice Definition

Target slice:
- `Slice 3 - Job Creation and Dispatch`, extended through the minimum execution loop from `Slice 4`

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

---

## 19. Next Deliverable

Implement `Slice 2 - Device Presence and Execution Eligibility`.

Primary outputs:

- device registration path in `elowen-edge`
- device persistence and lookup in `elowen-api`
- shared device contracts in `elowen-platform/contracts`
- availability probe and response flow over the platform runtime
- Compose-validated local presence for the primary development machine
