# Universal AI Application Harness

## Detailed Technical Design and Interface Contracts

**Document type:** Detailed engineering design\
**Version:** 1.0\
**Status:** Reference specification; adapt and validate before
production\
**Audience:** Backend, AI platform, client, infrastructure, QA, and
security engineers

------------------------------------------------------------------------

## 1. Purpose

This document defines detailed contracts for a reusable AI application
harness. It builds on the architecture, implementation blueprint, and
security/reliability specification by defining module responsibilities,
domain types, state transitions, model execution, context assembly, tool
invocation, API schemas, streaming events, persistence boundaries,
errors, testing, and acceptance criteria.

It is technology-aware but not bound to one language or provider. Code
examples use language-neutral pseudocode with occasional C#-style
notation.

This is a reference design, not a claim to reproduce proprietary
internals of any commercial AI product.

## 2. Design Goals

The implementation should be:

-   **Modular:** components can be replaced without rewriting unrelated
    features.
-   **Explicit:** state, permissions, limits, and errors are represented
    in contracts.
-   **Durable:** accepted turns and long-running tasks survive process
    restarts.
-   **Observable:** each request can be traced through model and tool
    execution.
-   **Secure:** model output and external content are untrusted.
-   **Testable:** core decisions can be tested without live provider
    calls.
-   **Cost-aware:** tokens, tools, retries, and elapsed time are
    bounded.
-   **Versioned:** API, event, tool, and persisted contracts evolve
    deliberately.

## 3. Non-Goals

The harness does not automatically provide: - Guaranteed factual
correctness. - A universal defense against every prompt injection. -
Unlimited autonomous execution. - A single best model for every task. -
Compliance with every jurisdiction without product-specific review. - A
requirement to split every module into a separate service.

## 4. Component Boundaries

### 4.1 API layer

Responsibilities: - Authenticate requests. - Parse and validate
transport contracts. - Apply request-size limits and rate limits. - Call
application use cases. - Return response contracts and stream events.

Must not: - Contain provider-specific model logic. - Make model output
the authorization authority. - Execute arbitrary tool code directly in
endpoint handlers.

### 4.2 Application layer

Responsibilities: - Coordinate use cases and transactions. - Orchestrate
turns and tasks. - Apply policy decisions. - Build context and call
abstractions. - Manage cancellation, retries, and state transitions.

### 4.3 Domain layer

Responsibilities: - Define entities, value objects, state transitions,
invariants, and domain events. - Remain independent of web frameworks,
provider SDKs, and database implementations.

### 4.4 Infrastructure layer

Responsibilities: - Implement repositories, provider adapters, storage,
queues, retrieval backends, and telemetry exporters. - Translate
external errors into normalized internal errors.

### 4.5 Worker layer

Responsibilities: - Execute durable background tasks. - Acquire leases,
checkpoint progress, and recover interrupted work. - Apply the same
authorization and policy rules as synchronous execution.

## 5. Core Type Catalogue

Use strongly typed identifiers where supported.

``` text
UserId
OrganizationId
ProjectId
ConversationId
MessageId
TurnId
ModelCallId
ToolInvocationId
TaskId
ArtifactId
ApprovalId
TraceId
```

### 5.1 Principal

``` text
Principal
  subject_id: UserId or ServicePrincipalId
  organization_ids: Set<OrganizationId>
  roles: Set<Role>
  scopes: Set<PermissionScope>
  authentication_method: String
  issued_at: Instant
  expires_at: Instant
```

The principal is constructed by trusted authentication middleware. Do
not accept a client-supplied principal as authoritative.

### 5.2 Result types

Prefer explicit success/failure results for expected errors.

``` text
Result<T>
  success: Boolean
  value: T? 
  error: AppError?
```

Unexpected exceptions should be captured at a boundary, mapped to safe
errors, and recorded with a correlation ID. Do not turn every exception
into a successful empty result.

### 5.3 AppError

``` text
AppError
  code: String
  category: ErrorCategory
  retryable: Boolean
  safe_message: String
  correlation_id: TraceId
  details: Map<String, SafeValue>
```

Never place credentials, raw prompts, full provider payloads, or
sensitive arguments in `details`.

## 6. Domain Entities and Invariants

### 6.1 Conversation

Fields: - `id` - `owner_id` - optional `organization_id` - optional
`project_id` - `title` - `status` - `created_at` - `updated_at` -
optional `deleted_at` - `version`

Invariants: - Every conversation has an authorized owner or tenant. - A
deleted conversation is not returned through ordinary reads. - Mutations
require authorization and concurrency control.

### 6.2 Message

Fields: - `id` - `conversation_id` - `turn_id` - `sequence_number` -
`role` - `content_parts` - `status` - `created_at` - optional
`completed_at` - safe metadata

Content parts may represent text, image references, file references,
structured data, or other supported modalities. The core domain should
not assume every message is a single text string.

Invariants: - Sequence numbers are unique within a conversation. -
Partial assistant output is not marked completed. - Tool messages
reference a known tool invocation. - Provider-specific message formats
do not leak into the domain model.

### 6.3 Turn

Fields: - `id` - `conversation_id` - `user_message_id` - optional
`assistant_message_id` - `state` - `idempotency_key` - `model_route` -
`created_at` - `started_at` - optional `completed_at` - optional
`cancel_requested_at` - `attempt_count` - `version`

Invariants: - State transitions follow the state machine. - One accepted
idempotency key maps to one logical turn for its scope. - A terminal
turn cannot return to an active state. - A turn cannot execute a tool
without a valid policy decision. - Budget limits apply across the whole
turn, not separately to each retry.

### 6.4 ToolInvocation

Fields: - `id` - `turn_id` - `tool_name` - `tool_version` -
`arguments` - `risk_class` - `status` - optional `approval_id` -
optional `result_reference` - timestamps - safe error code

Invariants: - The tool must be registered and enabled. - Arguments must
pass schema validation. - Permissions must be checked at execution
time. - Approval must match the exact material action. - An invocation
has a bounded timeout and result size.

### 6.5 Task

Fields: - `id` - `owner_id` - optional `organization_id` - `type` -
`state` - `progress` - `attempt` - optional `lease_owner` - optional
`lease_expires_at` - optional `checkpoint_reference` - optional
`result_reference` - `deadline_at` - `idempotency_key` - timestamps

Invariants: - A durable task is persisted before acceptance is
acknowledged. - Only a worker holding a valid lease may advance a leased
task. - Stale leases can be recovered. - Terminal states are immutable
except through explicit administrative repair procedures.

## 7. State Machines

### 7.1 Turn state machine

``` text
accepted
   |
   v
assembling_context
   |
   v
generating <---------------------------+
   |                                  |
   | model requests tool              | tool result added to context
   v                                  |
awaiting_tool                         |
   |                                  |
   v                                  |
authorizing_tool                      |
   |                                  |
   +-- denied --> generating or failed
   |
   v
awaiting_approval (if required)
   |
   v
executing_tool -----------------------+
   |
   v
generating
   |
   v
validating_output
   |
   v
completed
```

Any active state may transition to `cancelling`, then `cancelled`,
subject to in-flight side-effect semantics. An unrecoverable error
transitions to `failed`.

### 7.2 Transition rules

-   `accepted -> assembling_context`: persisted turn exists and
    execution budget is initialized.
-   `assembling_context -> generating`: context package passes
    validation and budget checks.
-   `generating -> awaiting_tool`: provider returned a complete,
    parseable tool request.
-   `awaiting_tool -> authorizing_tool`: tool is known and arguments can
    be validated.
-   `authorizing_tool -> executing_tool`: authorization passes and
    approval requirements are satisfied.
-   `authorizing_tool -> awaiting_approval`: policy requires user
    approval.
-   `executing_tool -> generating`: normalized tool result is recorded
    and safely added to context.
-   `generating -> validating_output`: model indicates final output.
-   `validating_output -> completed`: output passes required
    schema/policy checks and is persisted.
-   `active -> failed`: unrecoverable failure is recorded.
-   `active -> cancelling -> cancelled`: cancellation is acknowledged
    and work is stopped or safely reconciled.

Invalid transitions must be rejected. Use compare-and-swap/version
checks or database locking to prevent concurrent workers from advancing
the same turn.

### 7.3 Terminal state rules

Terminal states: - `completed` - `failed` - `cancelled`

A terminal turn cannot be silently restarted. A retry should create a
new attempt or a new turn according to the product contract, preserving
auditability.

## 8. Turn Manager Contract

The turn manager coordinates a single user turn. It should not contain
provider-specific SDK logic.

``` text
ITurnManager
  StartTurn(request, principal, cancellation) -> AcceptedTurn
  GetTurn(turn_id, principal) -> TurnStatus
  CancelTurn(turn_id, principal) -> CancellationResult
```

### 8.1 StartTurn algorithm

1.  Validate request shape and size.
2.  Resolve authenticated principal.
3.  Load conversation using an authorization-aware repository method.
4.  Validate idempotency key and request fingerprint.
5.  In one transaction:
    -   allocate turn ID,
    -   append user message,
    -   create turn in `accepted`,
    -   store idempotency record.
6.  If asynchronous execution is used, enqueue a durable work item after
    the transaction using an outbox or equivalent reliable handoff.
7.  Return accepted turn metadata.

Do not call the model before the accepted turn and idempotency state are
safely persisted.

### 8.2 Transactional outbox

If a database transaction must trigger a queue message, use a
transactional outbox or an equivalent durable pattern. A naive sequence
of "commit database, then publish queue message" can lose work if the
process crashes between those operations.

An outbox record is written in the same transaction as the task/turn. A
dispatcher publishes it and marks it delivered. Consumers must tolerate
duplicate delivery.

## 9. Model Execution Loop

### 9.1 Budgets

Create a turn budget at the beginning:

``` text
TurnBudget
  deadline
  max_model_calls
  max_tool_calls
  max_agent_steps
  max_input_tokens
  max_output_tokens
  max_parallel_tools
  max_estimated_cost
  max_context_bytes
```

Every model call and tool call consumes the shared budget. Stop before
exceeding a limit. Budget values are server-controlled and may be
adjusted by plan, task type, or risk class.

### 9.2 Pseudocode

``` text
function ExecuteTurn(turnId, principal, cancellation):
    turn = store.LoadAuthorizedTurn(turnId, principal)
    budget = budgetPolicy.CreateFor(turn, principal)

    transition(turn, accepted, assembling_context)

    context = contextBuilder.Build(turn, principal, budget)
    validateContext(context, budget)
    transition(turn, assembling_context, generating)

    while true:
        cancellation.throwIfRequested()
        budget.assertTimeRemaining()
        budget.assertModelCallAvailable()

        route = modelRouter.Select(context, turn, principal, budget)
        modelRequest = requestBuilder.Build(context, route, budget)

        response = modelGateway.Execute(modelRequest, cancellation)
        usageRecorder.Record(response.usage)

        if response.isToolCall:
            budget.consumeToolStep()
            invocation = toolParser.Parse(response.toolCall)
            validateToolSchema(invocation)
            decision = policy.Authorize(invocation, principal, turn)

            if decision.denied:
                toolResult = safeDeniedResult(decision.publicCode)
                context = contextBuilder.AddToolResult(context, toolResult)
                continue

            if decision.requiresApproval:
                approval = approvalService.Request(invocation, principal)
                persistWaitingForApproval(turn, invocation, approval)
                return AwaitingApproval(approval.id)

            result = toolExecutor.Execute(invocation, decision, cancellation)
            persistToolInvocation(invocation, result)
            context = contextBuilder.AddToolResult(context, result)
            transitionIfNeeded(turn, executing_tool, generating)
            continue

        candidate = outputValidator.Validate(response.finalOutput, turn.outputContract)
        persistAssistantMessage(candidate, response.usage)
        transition(turn, validating_output, completed)
        return Completed(candidate)
```

This is illustrative. Production code must handle streaming, partial
provider output, tool-call deltas, worker recovery, provider errors,
cancellation races, and persistence failures.

### 9.3 Critical rules

-   Never execute a tool from an incomplete streaming fragment.
-   Validate the complete tool call before authorization and execution.
-   Do not interpret a model's explanation as proof of permission.
-   Do not exceed turn-level budgets across retries.
-   Persist state before emitting events that imply durable progress.
-   Record partial output distinctly from completed output.
-   Apply output validation before exposing machine-consumed structured
    results.
-   A failed optional tool may be represented as a safe result only if
    the task can continue honestly.

## 10. Model Gateway Contracts

### 10.1 Request

``` text
ModelRequest
  purpose: ModelPurpose
  model_id: ApprovedModelId
  messages: List<NormalizedMessage>
  tools: List<ToolDefinition>
  response_format: ResponseFormat
  max_output_tokens: Integer
  timeout: Duration
  metadata: SafeMetadata
```

### 10.2 Response events

``` text
ModelEvent =
    ResponseStarted(provider_request_id?)
  | TextDelta(text)
  | ToolCallDelta(index, fragment)
  | ToolCallReady(call_id, name, arguments_json)
  | Usage(input_tokens?, output_tokens?)
  | ResponseCompleted(finish_reason)
  | ResponseFailed(NormalizedProviderError)
```

Each event must be associated with a model-call ID and turn ID in the
orchestration layer. Provider adapter events should be normalized before
reaching application logic.

### 10.3 Provider adapter

``` text
IModelProvider
  Capabilities() -> ProviderCapabilities
  Stream(request, cancellation) -> AsyncStream<ModelEvent>
```

Provider capabilities should include supported modalities, tool calling,
structured output, context limit, and relevant operational constraints.
Capability metadata should be maintained and tested rather than guessed.

### 10.4 Error normalization

Suggested categories: - `provider_rate_limited` -
`provider_unavailable` - `provider_timeout` -
`provider_invalid_request` - `provider_authentication_failed` -
`provider_content_rejected` - `provider_response_invalid` -
`provider_unknown_error`

Expose safe error codes to clients. Keep raw provider payloads out of
ordinary logs and user responses.

## 11. Model Routing Contract

``` text
ModelRoutingRequest
  purpose
  required_capabilities
  data_classification
  quality_tier
  latency_target
  context_estimate
  remaining_budget
  allowed_regions
  allowed_providers
```

``` text
ModelRoute
  provider_id
  model_id
  policy_version
  reason_code
  fallback_candidates[]
```

Routing policy: 1. Filter models by capability. 2. Filter by data policy
and allowed region. 3. Filter by health and availability. 4. Filter by
budget and quota. 5. Rank remaining candidates according to configured
task policy. 6. Record the chosen route and a safe reason code. 7. Apply
fallback only to candidates that satisfy the same policy constraints.

Do not expose internal provider credentials or sensitive routing logic
to clients.

## 12. Context Builder Contract

``` text
IContextBuilder
  Build(turn, principal, budget, cancellation) -> ContextPackage
```

### 12.1 ContextPackage

``` text
ContextPackage
  instruction_version
  messages[]
  retrieved_passages[]
  approved_memories[]
  tool_definitions[]
  output_contract
  token_estimate
  provenance[]
```

### 12.2 Priority policy

A typical order, subject to product requirements: 1. Non-overridable
system and security policy. 2. Current user request and required output
contract. 3. Application-specific instructions. 4. Tool definitions
required for the task. 5. Relevant recent conversation. 6. Relevant
retrieved evidence. 7. Relevant approved memories. 8. Older conversation
summaries.

This ordering is a context budgeting policy, not permission to discard
mandatory safety constraints.

### 12.3 Retrieval constraints

-   Authorize documents before retrieving their content.
-   Apply tenant and user filters inside retrieval queries.
-   Keep source identifiers and chunk offsets for citations.
-   Limit retrieved passages by relevance and token budget.
-   Treat retrieved text as untrusted evidence.
-   Do not allow retrieved content to change tool permissions or system
    policy.

### 12.4 Summarization

Summaries are lossy and may contain errors. Store provenance, summary
version, and source message range. When accuracy matters, retrieve
original messages or source passages instead of relying solely on a
summary.

## 13. Tool Contract and Lifecycle

### 13.1 Tool definition

``` text
ToolDefinition
  name
  version
  description
  input_schema
  output_schema
  required_scopes[]
  risk_class
  timeout
  max_input_bytes
  max_output_bytes
  network_policy
  idempotency_mode
  approval_policy
```

### 13.2 Tool executor interface

``` text
IToolExecutor
  Execute(AuthorizedToolCall, CancellationToken) -> ToolExecutionResult
```

`AuthorizedToolCall` must be created by trusted application code only
after schema validation and policy evaluation. It must not be
constructible from raw client input without those checks.

### 13.3 Tool result

``` text
ToolExecutionResult
  status: succeeded | failed | denied | timed_out | cancelled
  output: StructuredValue?
  artifact_reference?
  safe_error_code?
  started_at
  completed_at
  resource_usage
```

Large outputs should be stored as artifacts and referenced by ID. Do not
blindly place huge tool outputs in model context.

### 13.4 Approval flow

1.  Tool policy determines approval is required.
2.  System records the exact proposed action and material parameters.
3.  User sees a human-readable summary of the action.
4.  Approval is bound to the principal, action hash, expiry, and policy
    version.
5.  On approval, backend revalidates authorization and current resource
    state.
6.  If material parameters or target resource changed, require new
    approval.
7.  Record the result and audit event.

## 14. Structured Output Validation

When the product requires JSON or another machine-consumed format:

1.  Request a supported structured format when available.
2.  Parse with a strict parser.
3.  Validate against the expected schema.
4.  Reject unknown or invalid fields where appropriate.
5.  Apply semantic validation beyond JSON Schema when necessary.
6.  Retry repair only within a small, bounded budget.
7.  Never execute actions solely because a generated object parses
    successfully.

Schema-valid output can still be unauthorized or semantically wrong.

## 15. API Contracts

All API routes are versioned under `/v1`. Use OpenAPI as the source of
truth for HTTP contracts.

### 15.1 Create conversation

`POST /v1/conversations`

Request:

``` json
{
  "title": "Optional title",
  "project_id": null
}
```

Response `201 Created`:

``` json
{
  "id": "conv_opaque",
  "title": "Optional title",
  "created_at": "2026-01-01T12:00:00Z"
}
```

The server determines ownership and tenant membership from the
authenticated principal.

### 15.2 Start turn

`POST /v1/conversations/{conversationId}/turns`

Headers:

``` text
Authorization: Bearer <access-token>
Idempotency-Key: <opaque-key>
Content-Type: application/json
```

Request:

``` json
{
  "message": {
    "parts": [
      {
        "type": "text",
        "text": "Summarize the attached document."
      }
    ]
  },
  "attachment_ids": ["file_opaque"],
  "model_preference": "balanced",
  "response_format": {
    "type": "text"
  }
}
```

Response `202 Accepted`:

``` json
{
  "turn_id": "turn_opaque",
  "conversation_id": "conv_opaque",
  "state": "accepted",
  "events_url": "/v1/turns/turn_opaque/events"
}
```

The server validates that each attachment belongs to a resource the
principal can access.

### 15.3 Get turn

`GET /v1/turns/{turnId}`

Response:

``` json
{
  "id": "turn_opaque",
  "state": "generating",
  "created_at": "2026-01-01T12:00:00Z",
  "updated_at": "2026-01-01T12:00:02Z",
  "error": null
}
```

Do not expose internal prompt content, credentials, raw provider
payloads, or private policy traces.

### 15.4 Cancel turn

`POST /v1/turns/{turnId}/cancel`

Response:

``` json
{
  "turn_id": "turn_opaque",
  "state": "cancelling",
  "cancellation_requested": true
}
```

Cancellation acknowledgement means the request was recorded, not
necessarily that every external operation has already stopped.

### 15.5 Error envelope

``` json
{
  "error": {
    "code": "turn_not_found",
    "message": "The requested turn was not found.",
    "retryable": false,
    "correlation_id": "trace_opaque"
  }
}
```

For unauthorized resources, consider returning a non-enumerating
response that does not disclose whether another user's resource exists.

## 16. Streaming Event Contracts

Use SSE for one-way server-to-client streaming unless product
requirements justify WebSockets.

Every event should include a schema version and relevant turn/message
identifier.

### 16.1 Event types

-   `turn.accepted`
-   `turn.started`
-   `message.started`
-   `message.delta`
-   `tool.started`
-   `tool.completed`
-   `approval.required`
-   `task.progress`
-   `message.completed`
-   `turn.completed`
-   `turn.failed`
-   `turn.cancelling`
-   `turn.cancelled`

### 16.2 Examples

``` text
id: 1
event: turn.accepted
data: {"schema_version":1,"turn_id":"turn_opaque","state":"accepted"}

id: 2
event: message.delta
data: {"schema_version":1,"turn_id":"turn_opaque","message_id":"msg_opaque","text":"The document"}

id: 3
event: turn.completed
data: {"schema_version":1,"turn_id":"turn_opaque","message_id":"msg_opaque"}
```

### 16.3 Streaming rules

-   Do not emit a completed event before persistence succeeds.
-   Distinguish partial text from final persisted content.
-   Support heartbeat events where needed.
-   Define reconnect and replay behavior.
-   If replay is supported, use stable event IDs and a durable cursor.
-   Clients must tolerate duplicate delivery and unknown optional
    fields.
-   Do not stream internal instructions or hidden reasoning.
-   Redact sensitive tool results before event emission.
-   On disconnect, decide explicitly whether server-side execution
    continues or is cancelled.

## 17. Error Taxonomy

Use stable machine-readable error codes.

  ------------------------------------------------------------------------
  Code                     Category                Typical retry behavior
  ------------------------ ----------------------- -----------------------
  `invalid_request`        Validation              Do not retry unchanged

  `unauthenticated`        Identity                Refresh or authenticate

  `forbidden`              Authorization           Do not retry without
                                                   permission change

  `resource_not_found`     Resource                Do not retry unchanged

  `idempotency_conflict`   Request consistency     Use a new key for a
                                                   different request

  `rate_limit_exceeded`    Quota                   Retry after supplied
                                                   delay if safe

  `budget_exceeded`        Resource limit          Reduce request or
                                                   change approved limit

  `provider_unavailable`   Dependency              Bounded retry/fallback

  `provider_timeout`       Dependency              Retry only if budget
                                                   and semantics permit

  `tool_failed`            Tool                    Follow tool-specific
                                                   retry policy

  `approval_required`      Policy                  Await approval

  `cancelled`              Lifecycle               Do not automatically
                                                   restart

  `internal_error`         Unexpected              Use correlation ID; no
                                                   sensitive details
  ------------------------------------------------------------------------

The public error code and retryability should be mapped centrally. Avoid
ad hoc error strings across endpoints.

## 18. Persistence and Transaction Boundaries

### 18.1 Start-turn transaction

Atomically: - Insert user message. - Insert turn. - Insert idempotency
record. - Insert outbox record if asynchronous execution is needed.

### 18.2 Tool execution

Do not hold a database transaction open while waiting on an external
tool. Instead: 1. Persist invocation intent and state. 2. Execute
outside the transaction. 3. Persist normalized outcome. 4. Reconcile
uncertain outcomes through idempotency/status checks where supported.

### 18.3 Final answer

Persist the final assistant message, usage record, and terminal turn
state atomically where possible. Emit completion only after commit.

### 18.4 Event delivery

Use an outbox or durable event store if clients must be able to
reconnect and replay events. If streaming is best-effort only, document
that and provide a status endpoint as the source of truth.

## 19. Idempotency Contract

An idempotency key should be: - Scoped to the principal and operation. -
Stored durably. - Associated with a canonical request fingerprint. -
Retained for a documented period. - Protected by a unique constraint.

Behavior: - Same key + same fingerprint: return the original logical
result. - Same key + different fingerprint: return
`idempotency_conflict`. - Key expires: follow a documented policy and
avoid accidental duplicate side effects.

For external actions, also use provider/tool idempotency support where
available.

## 20. Cancellation and Deadline Propagation

Every operation receives a cancellation token and a deadline.

-   API request deadline bounds synchronous work.
-   Turn deadline bounds all model and tool steps.
-   Tool deadline is no later than the turn deadline.
-   Background task deadline is persisted.
-   Cancellation is propagated to provider adapters and tools when
    supported.

Cancellation is cooperative. A completed external side effect cannot
necessarily be undone. The system should persist this distinction and
report partial completion accurately.

## 21. Memory and Retrieval Contracts

### 21.1 Memory record

``` text
MemoryRecord
  id
  owner_id
  optional organization_id
  category
  value
  source_reference
  review_status
  created_at
  updated_at
  optional expires_at
  version
```

### 21.2 Retrieval result

``` text
RetrievedPassage
  source_id
  source_version
  chunk_id
  text
  location_reference
  score
  access_scope
```

Retrieval filters must be applied before content reaches the model.
Scores are ranking signals, not proof of truth. Preserve enough
provenance to support citations and deletion.

### 21.3 Deletion propagation

Deleting a source should enqueue durable cleanup for: - Original
object. - Extracted text. - Chunks and embeddings. - Search indexes. -
Cached results. - Derived artifacts where policy requires. - References
in memory and summaries where feasible.

Track cleanup completion and retry failures.

## 22. Artifact Contract

``` text
Artifact
  id
  owner_id
  optional organization_id
  project_id?
  name
  media_type
  size_bytes
  content_hash
  current_version
  object_reference
  created_at
  deleted_at?
```

Requirements: - Store bytes in object storage. - Store metadata and
version history in the database. - Authorize before returning metadata
or download URLs. - Use content hashes for integrity checks. - Set
output size and media-type limits. - Prevent unsafe active content from
executing in the primary app origin. - Keep provenance for generated
artifacts.

## 23. Observability Contracts

### 23.1 Correlation

Propagate: - `trace_id` - `request_id` - `conversation_id` - `turn_id` -
`model_call_id` - `tool_invocation_id` - `task_id`

Use opaque identifiers. Avoid placing user text, emails, or sensitive
identifiers in trace names or metric labels.

### 23.2 Trace spans

Useful spans: - `api.request` - `conversation.authorize` -
`context.build` - `retrieval.query` - `model.route` - `model.call` -
`tool.authorize` - `tool.execute` - `output.validate` - `turn.persist` -
`task.enqueue`

### 23.3 Metrics

-   Request and turn counts.
-   Turn completion/error rate.
-   Time to first token.
-   End-to-end turn duration.
-   Provider latency and errors.
-   Tool latency and failures.
-   Input/output token usage.
-   Estimated cost.
-   Context assembly duration.
-   Queue depth and age.
-   Cancellation and timeout counts.

Do not use unbounded user IDs or conversation IDs as metric labels.

## 24. Configuration Contracts

Configuration categories: - `ProviderPolicy` - `ModelRoutingPolicy` -
`TurnBudgetPolicy` - `ToolPolicy` - `ApprovalPolicy` -
`RetentionPolicy` - `UploadPolicy` - `RateLimitPolicy` -
`FeatureFlagSet`

Requirements: - Validate configuration at startup and on reload. -
Version policy changes. - Record the policy version used for a turn and
consequential action. - Fail closed when critical security configuration
is missing or invalid. - Do not permit a client to override server-owned
security limits. - Separate feature flags from authorization.

## 25. Test Contract

### 25.1 Unit tests

Test: - State transitions and invalid transitions. - Context priority
and token-budget behavior. - Model route selection. - Error mapping. -
Approval binding. - Idempotency fingerprint behavior. - Tool schema and
policy checks.

### 25.2 Contract tests

Test: - Provider adapter event normalization. - API request/response
schemas. - SSE event schemas. - Tool input/output schemas. - Persistence
compatibility.

### 25.3 Integration tests

Test: - Start-turn transaction. - Outbox delivery and duplicate
messages. - Cancellation during model streaming. - Tool failure and
retry. - Worker crash and lease recovery. - Artifact access control. -
Deletion propagation. - Event reconnect/replay if supported.

### 25.4 AI evaluation tests

Each case should include: - Stable case ID. - Task category. - Input and
controlled context. - Expected behavioral properties. -
Required/forbidden tool use. - Citation requirements where relevant. -
Scoring method. - Model/prompt version. - Result and reviewer notes.

Use deterministic assertions when possible. Human-review ambiguous
results. A model-based grader is only one evaluation signal.

## 26. Definition of Done for a Contract

A contract is ready when: - Field names and types are specified. -
Required/optional semantics are clear. - Validation rules are
documented. - Authorization requirements are defined. - Error behavior
is defined. - Versioning and compatibility are considered. - Sensitive
data handling is documented. - Unit and contract tests exist. -
Observability requirements are defined. - The owner of the contract is
known.

## 27. Implementation Order

1.  Define identifiers, errors, principals, and domain states.
2.  Implement persistence and transactional start-turn behavior.
3.  Implement one provider adapter and normalized events.
4.  Implement the turn manager and cancellation.
5.  Implement SSE events and status endpoint.
6.  Add idempotency and usage accounting.
7.  Add context builder and token budgeting.
8.  Add tool registry, validation, and authorization.
9.  Add approval flow for consequential actions.
10. Add retrieval, files, and citations.
11. Add durable tasks and recovery.
12. Add evaluation, security, and load-test gates.

Keep each milestone small enough to test end-to-end.

## 28. Reference Acceptance Scenarios

The following scenarios should become automated tests.

### Scenario A: Normal conversation

Given an authorized user and valid conversation, when a user submits a
message, then the turn is persisted, the response streams, and the final
assistant message is stored before `turn.completed` is emitted.

### Scenario B: Duplicate request

Given a previously accepted idempotency key, when the same request is
repeated, then the original logical turn is returned without creating a
second turn.

### Scenario C: Unauthorized conversation

Given a conversation owned by another user, when the caller attempts to
read or append to it, then access is denied without exposing its
content.

### Scenario D: Invalid tool call

Given a model request with unknown tool or invalid arguments, then the
executor is not called and a safe failure is recorded.

### Scenario E: Approval-required action

Given a tool that requires approval, when the model proposes the action,
then execution waits; approval is bound to exact parameters; changed
parameters require new approval.

### Scenario F: Provider timeout

Given a provider timeout, then the request obeys retry and budget
policy, records a normalized error, and does not leave the turn
indefinitely active.

### Scenario G: Cancellation

Given an active turn, when cancellation is requested, then the state
changes, cancellation is propagated, no new tool calls start, and the
final state is recorded accurately.

### Scenario H: Worker restart

Given a durable task with an expired lease, when a worker restarts, then
the task is safely recovered or marked failed according to policy,
without duplicate external side effects.

### Scenario I: Prompt injection in a document

Given a retrieved document that asks the model to reveal secrets or
invoke unrelated tools, then the content is treated as untrusted and no
unauthorized action occurs.

### Scenario J: Data deletion

Given a deleted source document, when cleanup completes, then it is no
longer retrievable through the original object, extracted text,
embedding index, or cache covered by the deletion policy.

## 29. Open Decisions

Resolve these for each product before implementation: - Is the
application single-tenant or multi-tenant? - Which data classifications
are permitted? - Which model providers and regions are approved? - Which
tool risk classes are allowed? - Which actions require approval? - Does
streaming need durable replay? - Which tasks require background
execution? - What are the retention and deletion guarantees? - What are
the initial budgets and quotas? - What are the required SLOs and
recovery objectives? - Which modalities are in scope for the first
release?

Record decisions in architecture decision records and keep this document
synchronized with the actual implementation.

## 30. Final Guidance

The highest-value part of this design is the consistency of contracts:
authorization, budgets, state transitions, tool execution, persistence,
and events should follow the same rules across every feature.

Start with a small end-to-end implementation, write acceptance tests for
the scenarios above, and expand only when the previous layer is
reliable. Avoid treating pseudocode in this document as production-ready
code without adapting it to the chosen framework, database, provider
APIs, and threat model.
