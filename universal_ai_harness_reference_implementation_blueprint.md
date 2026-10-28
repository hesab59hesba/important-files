# Universal AI Application Harness

## Reference Implementation Blueprint

**Document type:** Implementation specification\
**Version:** 1.0\
**Status:** Reference baseline; adapt before production\
**Audience:** Software architects, backend engineers, AI engineers,
security engineers, QA, and platform engineers

------------------------------------------------------------------------

## 1. Purpose

This document translates a reusable AI application harness architecture
into a practical project blueprint. It defines a modular reference
stack, repository layout, domain model, API contracts, runtime flow,
implementation sequence, testing strategy, and deployment baseline.

It is intentionally product-neutral. A chat assistant, research
assistant, coding assistant, internal knowledge bot, or
workflow-oriented AI application can reuse the core and add
product-specific modules.

This is not a claim to reproduce proprietary internals of any commercial
AI system. It is a buildable architecture based on general software
engineering principles.

## 2. Decisions to Make Before Implementation

Complete this profile before writing production code.

  -----------------------------------------------------------------------
  Decision                Example default         Project decision
  ----------------------- ----------------------- -----------------------
  Product type            AI chat plus tools and  TBD
                          files                   

  Client                  Web/mobile/desktop      TBD

  Backend language        TypeScript, C#, Python, TBD
                          Java, or Go             

  API style               REST + SSE streaming    TBD

  Database                PostgreSQL              TBD

  Object storage          S3-compatible object    TBD
                          storage                 

  Cache                   Redis, introduced when  TBD
                          justified               

  Model providers         Two providers behind an TBD
                          adapter                 

  Authentication          OIDC/OAuth-compatible   TBD
                          provider                

  Deployment              Container-based managed TBD
                          platform                

  Tenant model            Single user, teams, or  TBD
                          organizations           

  Data residency          Based on target users   TBD
                          and law                 

  Retention               Explicit product policy TBD

  Tool risk classes       Read-only, reversible   TBD
                          write, consequential    
                          write                   
  -----------------------------------------------------------------------

Do not treat these defaults as mandatory. Pick technologies based on
team skills, operational constraints, provider availability, and total
cost.

## 3. Reference Technology Profile

A practical default for a team comfortable with .NET is:

-   **Backend:** ASP.NET Core on a supported .NET release.
-   **Database:** PostgreSQL.
-   **Data access:** EF Core or a deliberately selected SQL/data-access
    layer.
-   **Streaming:** Server-Sent Events (SSE) for one-way generation
    events; WebSockets only where bidirectional low-latency
    communication is needed.
-   **Object storage:** S3-compatible storage or a managed cloud
    equivalent.
-   **Cache/ephemeral coordination:** Redis only when needed.
-   **Observability:** OpenTelemetry-compatible traces, metrics, and
    structured logs.
-   **API contract:** OpenAPI plus JSON Schema for structured tool
    inputs and outputs.
-   **Testing:** Unit, integration, contract, security, load, and AI
    evaluation tests.
-   **Deployment:** Containers and CI/CD.

Alternative stacks are equally valid. Keep domain contracts independent
from the chosen web framework, database library, and AI provider SDK.

## 4. Architectural Rules

1.  The backend is authoritative for identity, authorization, quotas,
    billing, and persisted state.
2.  The model is an untrusted decision-making component, not a policy
    enforcement authority.
3.  Model providers are accessed only through provider adapters.
4.  Tools are registered with schemas and permissions; the model cannot
    invoke arbitrary code or arbitrary URLs.
5.  Every externally visible operation has an owner, authorization rule,
    and audit strategy.
6.  Long-running work uses durable task state, not only an open HTTP
    request.
7.  Persisted events and API responses have versioned contracts.
8.  Retries are bounded and respect idempotency.
9.  Conversation content is not logged by default.
10. Every expensive or potentially unbounded loop has limits.
11. Tests must verify behavior, not merely that endpoints return HTTP
    200.
12. Start with a modular monolith. Split services only when there is a
    measured reason.

## 5. System Context

``` text
Client
  |
  | HTTPS / SSE
  v
API and Authentication
  |
  v
Application Use Cases
  |
  +--> Conversation and Turn Manager
  +--> Context Builder
  +--> Policy / Permission Service
  +--> Model Gateway
  |      +--> Provider Adapter A
  |      +--> Provider Adapter B
  |
  +--> Tool Registry and Executor
  +--> Retrieval Service
  +--> Memory Service
  +--> Artifact Service
  +--> Durable Task Service
  |
  v
PostgreSQL / Object Storage / Optional Cache
```

### 5.1 Request path

1.  Authenticate the caller.
2.  Authorize access to the conversation or project.
3.  Validate request shape, limits, and idempotency key.
4.  Create or load the conversation turn.
5.  Build a bounded context package.
6.  Choose a model using explicit routing policy.
7.  Execute the model request.
8.  If the model requests a tool, validate the tool and arguments,
    authorize it, and execute it under limits.
9.  Add tool results to the context and continue the model loop within a
    bounded budget.
10. Validate the final output.
11. Persist the final message and usage metadata.
12. Emit completion events and return the result.

## 6. Repository Structure

Suggested modular-monolith layout:

``` text
src/
  AiHarness.Api/
    Endpoints/
    Middleware/
    Authentication/
    Streaming/
    OpenApi/
  AiHarness.Application/
    Conversations/
    Turns/
    Context/
    Models/
    Tools/
    Memory/
    Retrieval/
    Artifacts/
    Tasks/
    Policies/
    Common/
  AiHarness.Domain/
    Conversations/
    Messages/
    Turns/
    ToolCalls/
    Tasks/
    Artifacts/
    Policies/
    Events/
    Common/
  AiHarness.Infrastructure/
    Persistence/
      Migrations/
      Repositories/
    ModelProviders/
    ObjectStorage/
    VectorSearch/
    Cache/
    Observability/
    BackgroundJobs/
  AiHarness.Worker/
    Handlers/
    Scheduling/
  AiHarness.Contracts/
    Requests/
    Responses/
    Events/
    Schemas/
tests/
  AiHarness.UnitTests/
  AiHarness.IntegrationTests/
  AiHarness.ContractTests/
  AiHarness.SecurityTests/
  AiHarness.EvaluationTests/
deploy/
  docker/
  environments/
  observability/
docs/
  adr/
  api/
  runbooks/
  threat-model/
scripts/
```

Keep business rules in application/domain layers. Infrastructure
implements interfaces; it should not define product policy.

## 7. Core Domain Model

### 7.1 Principal and tenant

-   `User`: authenticated human or service identity.
-   `Organization`: optional tenant boundary.
-   `Membership`: links users to organizations and roles.
-   `Project`: optional workspace grouping conversations, files, and
    tasks.
-   `Conversation`: ordered interaction container.
-   `Message`: user, assistant, tool, or system-generated content.
-   `Turn`: one user request and all processing required to complete it.
-   `ModelCall`: one provider request and its outcome.
-   `ToolInvocation`: one attempted tool execution.
-   `Task`: durable long-running operation.
-   `Artifact`: generated or uploaded file with version history.
-   `Memory`: user-approved or policy-permitted retained information.
-   `UsageRecord`: normalized usage and cost estimate.
-   `AuditEvent`: security- and action-relevant event.

### 7.2 Suggested identifiers

Use opaque IDs such as UUIDs/ULIDs. Do not expose sequential database
IDs where they could reveal volume or enable enumeration.

### 7.3 Message roles

Use a controlled enum, for example:

-   `user`
-   `assistant`
-   `tool`
-   `system_internal` (never expose internal instructions as ordinary
    user-visible content)

Provider-specific role names belong in provider adapters, not in the
core domain.

### 7.4 Turn states

``` text
accepted -> assembling_context -> generating
generating -> awaiting_tool -> executing_tool -> generating
generating -> validating_output -> completed
any_active_state -> cancelling -> cancelled
any_active_state -> failed
```

State transitions must be validated in application code and persisted.
Define recovery behavior for a worker restart at every nonterminal
state.

## 8. Data Model Baseline

The following is a conceptual relational model. Adjust types, indexes,
partitioning, and tenancy to the selected database and expected scale.

### 8.1 Tables

**users** - `id` - `external_subject` (unique within identity issuer) -
`created_at` - `status`

**organizations** (optional) - `id` - `name` - `created_at`

**organization_memberships** (optional) - `organization_id` -
`user_id` - `role` - `created_at`

**projects** (optional) - `id` - `owner_id` - `organization_id`
nullable - `name` - `created_at` - `updated_at`

**conversations** - `id` - `owner_id` - `organization_id` nullable -
`project_id` nullable - `title` - `status` - `created_at` -
`updated_at` - `deleted_at` nullable

**messages** - `id` - `conversation_id` - `turn_id` -
`sequence_number` - `role` - `content_json` - `status` - `created_at` -
`completed_at` nullable - `metadata_json` (allowlisted, non-sensitive
metadata only)

**turns** - `id` - `conversation_id` - `user_message_id` -
`assistant_message_id` nullable - `state` - `idempotency_key` -
`model_route` - `started_at` - `completed_at` nullable - `error_code`
nullable - `cancel_requested_at` nullable

**model_calls** - `id` - `turn_id` - `provider` - `model` - `purpose` -
`status` - `started_at` - `completed_at` - `input_tokens` nullable -
`output_tokens` nullable - `estimated_cost` nullable -
`provider_request_id` nullable - `error_category` nullable

**tool_invocations** - `id` - `turn_id` - `tool_name` - `tool_version` -
`status` - `arguments_redacted_json` - `result_reference` nullable -
`started_at` - `completed_at` nullable - `approval_id` nullable -
`error_code` nullable

**tasks** - `id` - `owner_id` - `organization_id` nullable - `type` -
`state` - `progress_json` - `input_reference` - `result_reference`
nullable - `created_at` - `updated_at` - `deadline_at` nullable -
`idempotency_key`

**artifacts** - `id` - `owner_id` - `organization_id` nullable -
`project_id` nullable - `name` - `media_type` - `object_key` -
`size_bytes` - `sha256` - `version` - `created_at` - `deleted_at`
nullable

**usage_records** - `id` - `owner_id` - `organization_id` nullable -
`turn_id` - `provider` - `model` - `input_tokens` - `output_tokens` -
`estimated_cost` - `created_at`

**audit_events** - `id` - `actor_id` nullable - `organization_id`
nullable - `action` - `resource_type` - `resource_id` - `outcome` -
`correlation_id` - `created_at` - `safe_metadata_json`

### 8.2 Data integrity and indexes

-   Unique `(conversation_id, sequence_number)` for messages.
-   Unique `(conversation_id, idempotency_key)` for turn creation where
    appropriate.
-   Index `messages(conversation_id, sequence_number)`.
-   Index `conversations(owner_id, updated_at DESC)`.
-   Index `tasks(state, updated_at)` for worker polling if a queue is
    not used.
-   Index audit events by resource and timestamp.
-   Enforce ownership and tenant filters in repository queries, not only
    in client logic.
-   Use transactions when creating a turn and its user message.
-   Use optimistic concurrency or row locking for state transitions.
-   Do not store large file bytes in relational message rows; store
    object references.

## 9. Application Interfaces

Define interfaces in the application layer and implement them in
infrastructure.

Example C#-style contracts:

``` csharp
public interface IModelGateway
{
    IAsyncEnumerable<ModelEvent> ExecuteAsync(
        ModelRequest request,
        CancellationToken cancellationToken);
}

public interface IToolExecutor
{
    Task<ToolResult> ExecuteAsync(
        AuthorizedToolCall call,
        CancellationToken cancellationToken);
}

public interface IContextBuilder
{
    Task<ContextPackage> BuildAsync(
        TurnContext turn,
        CancellationToken cancellationToken);
}

public interface IConversationRepository
{
    Task<Conversation?> GetAuthorizedAsync(
        ConversationId id,
        Principal principal,
        CancellationToken cancellationToken);

    Task AppendMessageAsync(
        ConversationId id,
        Message message,
        CancellationToken cancellationToken);
}

public interface ITaskStore
{
    Task<TaskRecord> CreateAsync(
        NewTask task,
        CancellationToken cancellationToken);

    Task<TaskRecord?> GetAsync(
        TaskId id,
        Principal principal,
        CancellationToken cancellationToken);

    Task<bool> TryTransitionAsync(
        TaskId id,
        TaskState expected,
        TaskState next,
        CancellationToken cancellationToken);
}
```

The contracts are illustrative. Prefer strongly typed domain identifiers
and result types in a real implementation.

## 10. Model Provider Adapter Contract

Provider adapters translate the common internal request to
provider-specific API calls and normalize responses.

### 10.1 Normalized request

``` json
{
  "purpose": "conversation",
  "model_preference": "balanced",
  "messages": [],
  "tools": [],
  "response_format": "text",
  "limits": {
    "max_output_tokens": 2048,
    "timeout_ms": 60000
  },
  "metadata": {
    "trace_id": "opaque-trace-id"
  }
}
```

Do not allow client-supplied fields to override server-owned safety
instructions, provider credentials, authorization, or spending limits.

### 10.2 Normalized events

Use an internal event union such as:

-   `response_started`
-   `text_delta`
-   `tool_call_delta`
-   `tool_call_ready`
-   `usage`
-   `response_completed`
-   `response_failed`

Each event should have a schema version, turn ID, sequence number, and
timestamp where useful.

### 10.3 Adapter requirements

-   Explicit timeout and cancellation.
-   Bounded retries for transient failures.
-   No blind retries for non-idempotent actions.
-   Normalize provider errors.
-   Track provider request IDs.
-   Respect provider-specific rate limits.
-   Never log API keys or full sensitive prompts.
-   Test streaming interruptions and malformed responses.
-   Treat tool-call arguments as untrusted input.

## 11. Model Routing

Routing is a server-owned policy. A client may express a preference, but
the backend determines the allowed model and budget.

Inputs can include: - Task category. - Required modalities. - Context
length. - Latency target. - Quality tier. - Provider health. - User
quota. - Cost budget. - Data residency restrictions.

Routing flow:

1.  Validate requested capability.
2.  Filter models by policy and availability.
3.  Apply budget and quota limits.
4.  Choose a model using configurable policy.
5.  Record the decision and reason code.
6.  Apply fallback only to an approved compatible model.
7.  Expose safe, understandable status to the client.

Do not route sensitive data to a provider unless the provider and
configuration are approved for that data class.

## 12. Context Assembly

The context builder should return a structured package, not an arbitrary
string.

Suggested package:

``` text
ContextPackage
  instruction_version
  conversation_window
  conversation_summary (optional)
  retrieved_passages[]
  approved_memories[]
  tool_definitions[]
  task_state
  output_constraints
  token_budget
  provenance[]
```

### 12.1 Assembly sequence

1.  Load the authorized conversation and relevant project state.
2.  Load stable application instructions by version.
3.  Apply the task's output and modality requirements.
4.  Select recent messages within a token budget.
5.  Include a summary only if it is trustworthy and relevant.
6.  Retrieve documents with access control applied before model context
    construction.
7.  Retrieve only relevant memories.
8.  Include only tools the user and current workflow are permitted to
    use.
9.  Estimate token use.
10. Trim lower-priority context using a documented policy.
11. Record provenance metadata without logging sensitive content.

Never allow retrieved content to silently replace trusted system policy.

## 13. Tool Registry and Execution

Every tool definition should include:

-   Stable name and version.
-   Description.
-   JSON Schema input.
-   Output contract.
-   Permission scope.
-   Risk classification.
-   Timeout.
-   Resource limits.
-   Idempotency behavior.
-   Network policy.
-   Audit policy.
-   Approval policy.

### 13.1 Risk classes

  -----------------------------------------------------------------------
  Class                   Example                 Default policy
  ----------------------- ----------------------- -----------------------
  Read-only               Search an authorized    Execute after
                          document                authorization

  Reversible write        Create a draft artifact Execute within user
                                                  scope

  External side effect    Send a message or       Require explicit
                          publish content         approval or narrowly
                                                  scoped prior consent

  High impact             Financial, legal,       Strong authorization
                          account, or destructive and explicit
                          action                  confirmation; often
                                                  exclude from early
                                                  releases
  -----------------------------------------------------------------------

### 13.2 Tool execution sequence

1.  Parse the model's requested tool call.
2.  Validate the tool name and version.
3.  Validate arguments against the schema.
4.  Check user, tenant, resource, and action permissions.
5.  Apply risk and approval policy.
6.  Enforce timeout, size, rate, and concurrency limits.
7.  Execute within a constrained environment.
8.  Validate and truncate or externalize the result.
9.  Persist a safe audit record.
10. Return a normalized result to the turn manager.

The model must never be allowed to select arbitrary shell commands,
credentials, internal network destinations, or unrestricted database
queries.

## 14. API Baseline

Prefix versioned endpoints with `/v1`.

### 14.1 Conversations

-   `POST /v1/conversations`
-   `GET /v1/conversations`
-   `GET /v1/conversations/{conversationId}`
-   `PATCH /v1/conversations/{conversationId}`
-   `DELETE /v1/conversations/{conversationId}`

### 14.2 Messages and turns

-   `POST /v1/conversations/{conversationId}/turns`
-   `GET /v1/turns/{turnId}`
-   `POST /v1/turns/{turnId}/cancel`
-   `GET /v1/turns/{turnId}/events` (SSE)

### 14.3 Files and artifacts

-   `POST /v1/files/upload-sessions`
-   `POST /v1/files/{fileId}/complete`
-   `GET /v1/files/{fileId}`
-   `DELETE /v1/files/{fileId}`
-   `GET /v1/artifacts/{artifactId}`
-   `GET /v1/artifacts/{artifactId}/versions`

### 14.4 Tasks

-   `POST /v1/tasks`
-   `GET /v1/tasks/{taskId}`
-   `POST /v1/tasks/{taskId}/cancel`

### 14.5 Common requirements

-   Authenticate every non-public endpoint.
-   Authorize every resource access.
-   Use pagination with stable cursors.
-   Validate request sizes and content types.
-   Use idempotency keys for retryable create operations.
-   Return consistent error envelopes.
-   Add request correlation IDs.
-   Version breaking changes.
-   Generate OpenAPI from actual endpoint contracts where possible.

## 15. Request and Error Contracts

Example request:

``` json
{
  "message": {
    "text": "Explain this document"
  },
  "model_preference": "balanced",
  "attachments": ["file_opaque_id"],
  "response_format": "text"
}
```

Example accepted response:

``` json
{
  "turn_id": "turn_opaque_id",
  "conversation_id": "conversation_opaque_id",
  "state": "accepted",
  "events_url": "/v1/turns/turn_opaque_id/events"
}
```

Example error:

``` json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "The request limit has been reached.",
    "retryable": true,
    "correlation_id": "opaque-correlation-id"
  }
}
```

Do not expose stack traces, secrets, raw provider errors, internal
hostnames, or private policy details in public error messages.

## 16. Streaming Event Contract

SSE is a good default for server-to-client generation updates.

Example:

``` text
event: turn.started
data: {"schema_version":1,"turn_id":"turn_123"}

event: message.delta
data: {"schema_version":1,"message_id":"msg_123","text":"Hello"}

event: tool.started
data: {"schema_version":1,"tool_call_id":"tool_123","tool_name":"document_search"}

event: turn.completed
data: {"schema_version":1,"turn_id":"turn_123","message_id":"msg_456"}
```

Requirements: - Each event has a defined schema. - Clients tolerate
unknown optional fields and reconnects. - Events are ordered within a
turn. - Reconnection uses an event ID or durable cursor if replay is
supported. - Partial output is not treated as a completed answer. -
Cancellation and failure have distinct events. - Do not include secrets
or hidden instructions in events.

## 17. Idempotency and Concurrency

For a turn creation request: 1. Require or accept an idempotency key. 2.
Scope the key to the authenticated principal and relevant resource. 3.
Store the request fingerprint and created turn. 4. Return the existing
turn when the same key and same request are repeated. 5. Reject reuse of
the same key with a materially different request.

Use concurrency control for: - Message sequence allocation. - Turn state
transitions. - Cancellation races. - Task lease acquisition. - Artifact
version creation.

Retries should be bounded and classified by failure type.

## 18. Durable Tasks

Task state should be persisted and recoverable.

Suggested states:

``` text
queued -> running -> waiting_for_approval -> running
running -> succeeded
running -> failed
queued/running -> cancelling -> cancelled
```

Store: - Task input reference. - Owner and tenant. - Current state. -
Progress metadata. - Attempt count. - Lease/heartbeat. - Deadline. -
Cancellation flag. - Checkpoint reference. - Result reference. - Safe
failure code.

A worker should acquire a lease, periodically heartbeat, and use
idempotent step handlers. On restart, stale leases are recovered
according to a documented policy.

## 19. Files and Artifacts

### 19.1 Upload flow

1.  Authenticate and authorize.
2.  Create a limited upload session.
3.  Upload to quarantine storage.
4.  Validate size, media type, and file signature.
5.  Scan or process according to risk and available controls.
6.  Extract content in a sandbox.
7.  Store metadata and a content hash.
8.  Apply access-control metadata.
9.  Make the file available only after validation succeeds.

### 19.2 Artifact lifecycle

-   Store content in object storage.
-   Keep metadata and versions in the relational database.
-   Generate short-lived download URLs only after authorization.
-   Track provenance and generation metadata.
-   Support retention and deletion.
-   Prevent untrusted HTML or active content from executing in the
    application origin.
-   Set upload, extraction, and output limits.

## 20. Memory and Retrieval

Start with explicit, user-visible memory before automatic long-term
extraction.

Memory records should have: - Owner and optional tenant. - Content or
structured value. - Category. - Source/provenance. - Confidence or
review status where relevant. - Created and updated timestamps. -
Expiration or retention rule. - User-editable and deletable status.

Retrieval must enforce access control before passages enter a prompt.
Deleting a document should trigger deletion or invalidation of its
extracted text, embeddings, caches, and derived references according to
the documented lifecycle.

## 21. Authentication, Authorization, and Tenancy

-   Use a trusted identity provider where practical.
-   Validate issuer, audience, expiry, and signature.
-   Resolve a server-side principal.
-   Authorize every object by ownership or tenant membership.
-   Deny by default.
-   Avoid trusting owner IDs supplied in request bodies.
-   Keep service credentials separate from user credentials.
-   Use least privilege for database and object-storage identities.
-   Audit administrative and consequential actions.
-   Test cross-tenant access explicitly.

## 22. Configuration and Secrets

Separate configuration into: - Non-secret application settings. -
Environment-specific settings. - Feature flags. - Secret references. -
Model/provider policy. - Tool policy. - Budget and quota policy.

Use a managed secret store or deployment platform secret mechanism.
Never commit secrets or place them in client applications. Rotate keys
and ensure logs redact secret-shaped values.

## 23. Observability

Use structured logs, metrics, and distributed traces.

Recommended metrics: - Request rate, error rate, and latency. - Time to
first token. - Total turn duration. - Provider failures and fallback
rate. - Tool success/failure rate. - Queue age and task duration. -
Token use and estimated cost. - Context assembly duration. - Retrieval
hit rate and evaluation scores. - Cancellation rate.

Keep high-cardinality identifiers in traces/logs rather than metric
labels. Avoid logging raw prompts or full document contents by default.

## 24. Testing Strategy

### 24.1 Unit tests

-   State transitions.
-   Authorization decisions.
-   Context prioritization.
-   Routing rules.
-   Schema validation.
-   Retry classification.
-   Cost/quota calculations.

### 24.2 Integration tests

-   Database transactions.
-   Provider adapter behavior with mocked responses.
-   Streaming cancellation.
-   Tool execution policy.
-   File lifecycle.
-   Durable task recovery.
-   Object storage permissions.

### 24.3 Contract tests

-   API schemas.
-   Provider adapter normalized events.
-   Tool input/output schemas.
-   Event schema compatibility.

### 24.4 AI evaluations

Maintain versioned cases with: - Input. - Expected properties, not
necessarily exact wording. - Required citations or evidence. - Forbidden
outcomes. - Tool-use expectations. - Scoring rubric.

Track regression rates and compare candidate model/prompt versions
against a baseline.

### 24.5 Security tests

-   Cross-user and cross-tenant access.
-   Prompt injection in retrieved documents.
-   Malformed tool arguments.
-   SSRF attempts.
-   Upload abuse.
-   Rate-limit bypass.
-   Secret leakage.
-   Unauthorized side effects.

## 25. CI/CD and Environments

Suggested environments: - Local development. - Automated test. -
Staging. - Production.

CI pipeline: 1. Formatting and linting. 2. Static analysis. 3. Unit
tests. 4. Integration/contract tests. 5. Dependency and secret scanning.
6. Build immutable artifacts. 7. Database migration validation. 8.
Staging deployment. 9. Smoke tests. 10. Controlled production rollout.
11. Post-deployment monitoring.

Use backward-compatible database migrations where possible. Have a
rollback or forward-fix plan for each release.

## 26. Deployment Baseline

For an initial production deployment, consider: - API container. -
Worker container if durable tasks exist. - Managed PostgreSQL with
backups. - Object storage. - Optional cache/queue. - Centralized logs
and metrics. - Secret manager. - TLS and managed ingress. - Automated
database migrations. - Health/readiness endpoints.

Do not introduce Kubernetes, a service mesh, or many microservices
unless operational requirements justify the added complexity.

## 27. SLOs and Operational Targets

Set initial service-level objectives based on measured user needs.
Example starting targets, not universal guarantees:

-   API availability: define a monthly target appropriate to the
    product.
-   Non-streaming API latency: track p50, p95, and p99 separately.
-   Time to first token: measure by provider and task class.
-   Turn completion: measure by task category and output length.
-   Task recovery: define maximum tolerated time for stale task
    detection.
-   Data durability: define backup frequency and recovery-point
    objective.
-   Recovery time: define recovery-time objective.
-   Cost: establish per-user and per-task budgets.

Do not promise latency targets before measuring real traffic and
provider behavior.

## 28. Implementation Milestones

### Milestone 1 --- Runnable foundation

-   Create solution and module boundaries.
-   Configure local database.
-   Add migrations.
-   Implement authentication boundary.
-   Add health checks and structured logs.
-   Add CI.

**Acceptance:** App starts locally; migrations run; tests pass; secrets
are not committed.

### Milestone 2 --- Reliable text chat

-   Conversation and message repositories.
-   Turn manager.
-   One provider adapter.
-   Streaming events.
-   Cancellation and timeout.
-   Idempotent turn creation.
-   Usage tracking.

**Acceptance:** Multi-turn chat works; duplicate requests do not create
duplicate turns; cancellation works; provider errors are normalized.

### Milestone 3 --- Provider abstraction and routing

-   Second adapter or provider simulator.
-   Model capability registry.
-   Routing policy.
-   Fallback policy.
-   Provider contract tests.

**Acceptance:** Product code does not depend on provider-specific
response types.

### Milestone 4 --- Tools

-   Tool registry.
-   Schema validation.
-   Authorization and risk classification.
-   Bounded execution.
-   Audit events.
-   Approval flow for side effects.

**Acceptance:** Unauthorized and malformed tool calls are rejected
before execution.

### Milestone 5 --- Files and retrieval

-   Upload sessions.
-   File validation.
-   Text extraction.
-   Retrieval pipeline.
-   Source provenance.
-   Deletion workflow.

**Acceptance:** Users cannot retrieve other users' documents; citations
map to stored source passages.

### Milestone 6 --- Durable tasks and artifacts

-   Task state machine.
-   Worker leases/checkpoints.
-   Cancellation.
-   Artifact versions.
-   Result notifications.

**Acceptance:** A worker restart does not silently lose task state.

### Milestone 7 --- Production hardening

-   Threat model.
-   Security tests.
-   Evaluation suite.
-   Load tests.
-   Backup/restore rehearsal.
-   Runbooks and alerts.
-   Cost limits.

**Acceptance:** Release checklist is reviewed and signed off by
responsible owners.

## 29. Definition of Done

A feature is not done until: - Requirements and failure modes are
documented. - Authorization is tested. - Inputs and outputs are
validated. - Timeouts and cancellation are handled. - Logs and metrics
are added without leaking sensitive content. - Unit and integration
tests pass. - API/schema changes are documented. - Cost and resource
limits are defined. - Rollback and operational behavior are
understood. - Relevant AI evaluation cases pass.

## 30. Architecture Decision Records

Create an ADR for meaningful decisions. Each ADR should contain: - Title
and date. - Context. - Decision. - Alternatives considered. -
Consequences. - Security/privacy impact. - Revisit conditions.

Suggested first ADRs: 1. Modular monolith versus distributed services.
2. Backend language and framework. 3. Database and migration strategy.
4. Streaming protocol. 5. Provider adapter contract. 6. Tenant and
authorization model. 7. File/object storage design. 8. Task durability
strategy. 9. Memory retention policy. 10. Model routing and spending
limits.

## 31. Risks and Mitigations

  -----------------------------------------------------------------------
  Risk                                Mitigation
  ----------------------------------- -----------------------------------
  Provider outage                     Timeouts, bounded retries,
                                      compatible fallback

  Provider behavior changes           Adapter contract tests and
                                      evaluation suite

  Prompt injection                    Treat retrieved content as
                                      untrusted; enforce policy outside
                                      the model

  Cost spikes                         Per-user quotas, task budgets, loop
                                      limits, alerts

  Data leakage                        Authorization at every boundary,
                                      tenant-isolation tests

  Duplicate turns                     Idempotency keys and transactional
                                      persistence

  Lost long-running tasks             Durable task state, leases,
                                      checkpoints

  Tool misuse                         Registry, schemas, permissions,
                                      approvals, audit logs

  Context overflow                    Token budgets and deterministic
                                      prioritization

  Silent quality regression           Versioned evaluation suite and
                                      release gates
  -----------------------------------------------------------------------

## 32. Production Readiness Checklist

### Product

-   [ ] Application profile completed.
-   [ ] User journeys and failure states documented.
-   [ ] Data retention and deletion behavior documented.

### Architecture

-   [ ] Module boundaries and contracts are documented.
-   [ ] Provider SDKs are isolated behind adapters.
-   [ ] Durable tasks have recovery behavior.
-   [ ] API and event schemas are versioned.

### Security

-   [ ] Authentication and authorization are enforced server-side.
-   [ ] Cross-tenant tests pass.
-   [ ] Secrets are managed securely.
-   [ ] Uploads and tools are constrained.
-   [ ] Consequential actions have approval policy.
-   [ ] Threat model is reviewed.

### Reliability

-   [ ] Timeouts and cancellation work.
-   [ ] Retries are bounded and idempotency-aware.
-   [ ] Backups and restoration are tested.
-   [ ] Alerts and runbooks exist.

### AI quality

-   [ ] Evaluation baseline exists.
-   [ ] Tool-use tests exist.
-   [ ] Retrieval/citation tests exist where applicable.
-   [ ] Prompt/model changes are evaluated before rollout.

### Operations

-   [ ] Cost and quota limits are active.
-   [ ] Metrics and traces are available.
-   [ ] Deployment and rollback are documented.
-   [ ] Incident ownership is assigned.

## 33. Further Reading

Use official documentation for current API details and supported
versions:

-   ASP.NET Core: https://learn.microsoft.com/en-us/aspnet/core/
-   PostgreSQL: https://www.postgresql.org/docs/
-   OpenTelemetry: https://opentelemetry.io/docs/
-   OpenAPI: https://spec.openapis.org/oas/latest.html
-   JSON Schema: https://json-schema.org/
-   Temporal: https://docs.temporal.io/
-   OWASP: https://owasp.org/
-   pgvector: https://github.com/pgvector/pgvector

## 34. Final Guidance

Build a small vertical slice first: authenticate a user, create a
conversation, stream a model response, persist the result, handle
cancellation, and measure usage. Then add tools, retrieval, memory, and
durable workflows one at a time.

The objective is not to maximize the number of components. It is to make
every component understandable, testable, secure, observable, and
replaceable.
