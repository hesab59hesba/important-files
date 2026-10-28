# Lumio AI Harness
## Production-Grade System Architecture & Technical Specification

**Document type:** System architecture and implementation blueprint  
**Status:** Proposed baseline — validate decisions against product requirements before implementation  
**Version:** 1.0  
**Date:** 2026-10-28

---

## 1. Executive Summary

Lumio is an AI assistant application designed to provide reliable conversational experiences, tool use, retrieval, memory, multimodal inputs, and controlled multi-step task execution. This document defines a production-oriented architecture for the software surrounding language models: the **AI harness**.

The harness is responsible for assembling context, selecting a compatible model, managing model calls, executing authorized tools, persisting execution state, streaming updates to clients, applying limits, and producing observable, recoverable outcomes.

This blueprint uses a **modular-monolith-first, hybrid architecture**:

- Flutter client for mobile UX.
- TypeScript backend using NestJS or Fastify.
- PostgreSQL as the canonical application database.
- Redis for ephemeral state, caching, and rate-limit coordination where appropriate.
- S3-compatible object storage for uploaded files and generated artifacts.
- A durable job queue for long-running work.
- A provider abstraction around model APIs.
- A custom Lumio orchestration/product layer, using provider SDKs or an agent SDK where useful.
- Explicit tool permissions, isolation, auditing, and evaluation.

This is a reference architecture, not a claim to reproduce any proprietary internal OpenAI or ChatGPT implementation. It uses general patterns suitable for building a capable assistant. Final technology and capacity choices should be confirmed against Lumio's actual team, traffic, budget, compliance, and feature requirements.

## 2. Goals and Non-Goals

### 2.1 Goals

1. Support multi-turn conversations with persistent history.
2. Stream assistant output to Flutter clients.
3. Use tools through a controlled registry and executor.
4. Support provider/model selection without coupling the product to one provider.
5. Retrieve relevant conversation context, memories, and user files.
6. Support images and documents through extensible content abstractions.
7. Run long tasks independently of the original HTTP request.
8. Recover safely from interruptions and transient failures.
9. Apply authorization, rate limits, budgets, and tool-specific permissions.
10. Record traces, usage, errors, and evaluation results.
11. Enable prompt/model/configuration versioning and controlled rollouts.
12. Provide a path to split selected modules into services when justified by load or team ownership.

### 2.2 Non-goals for the initial release

- Training a foundation model.
- Recreating proprietary model internals or hidden prompts.
- Starting with dozens of microservices.
- Allowing unrestricted autonomous access to user accounts, shell commands, or databases.
- Guaranteeing that model output is factually correct solely because it passed schema validation.
- Building a multi-agent framework before single-agent execution is reliable.
- Treating vector similarity as authorization or as proof that retrieved content is correct.

## 3. Architecture Principles

- **Server authority:** The backend is authoritative for identities, permissions, turn state, tool execution, and persisted results.
- **Least privilege:** Every tool receives only the permissions and resources it needs.
- **Explicit state:** Model calls, tool calls, approvals, and task transitions are represented as structured records.
- **Provider independence:** Product-domain code depends on Lumio interfaces, not provider-specific payloads.
- **Capability-aware routing:** Never assume all models support the same tools, modalities, context sizes, or structured output features.
- **Durability where needed:** Persist state that must survive process restarts; keep transient streaming buffers ephemeral.
- **Idempotency:** Design retries to avoid duplicate user-visible messages and repeated side effects.
- **Bounded execution:** Every turn and tool has explicit time, cost, token, and iteration limits.
- **Observable by default:** Each turn has a trace/correlation ID and structured status/error information.
- **Privacy by design:** Minimize data collection, define retention policies, isolate tenants, and support deletion.
- **Human approval for consequential actions:** The model may propose an action, but application policy determines whether it can execute.
- **Evaluate continuously:** Model, prompt, tool, and retrieval changes must be tested against representative scenarios.

## 4. System Context

### 4.1 High-level diagram

```text
┌──────────────────────────────┐
│        Flutter Client        │
│ Chat UI · Files · Settings   │
└──────────────┬───────────────┘
               │ HTTPS / SSE
               ▼
┌──────────────────────────────┐
│       API / Edge Layer       │
│ Auth · Validation · Quotas   │
│ Request IDs · Rate limiting  │
└──────────────┬───────────────┘
               ▼
┌─────────────────────────────────────────────────────┐
│                  Lumio Backend                      │
│                                                     │
│  ┌──────────────┐   ┌───────────────────────────┐   │
│  │ Conversations│   │      AI Orchestrator      │   │
│  │ Messages     │──▶│ Turn manager / state      │   │
│  │ Attachments  │   │ Context / model routing   │   │
│  └──────────────┘   │ Tool policy / event stream│   │
│                     └─────────────┬─────────────┘   │
│                                   │                 │
│  ┌──────────────┐   ┌─────────────▼─────────────┐   │
│  │ Memory & RAG │   │      Tool Executor        │   │
│  │ Retrieval    │   │ Schemas · Permissions     │   │
│  └──────────────┘   │ Sandboxing · Audit        │   │
│                     └─────────────┬─────────────┘   │
└───────────────────────────────────┼─────────────────┘
                                    │
                ┌───────────────────┼─────────────────┐
                ▼                   ▼                 ▼
         Model Providers       External Tools     Job Workers
                │                   │                 │
                └───────────────────┼─────────────────┘
                                    ▼
┌─────────────────────────────────────────────────────┐
│ Infrastructure: PostgreSQL · Redis · Object Storage │
│ Queue · Secrets · Telemetry · Evaluation Datasets   │
└─────────────────────────────────────────────────────┘
```

### 4.2 Main runtime paths

**Interactive turn:** Flutter → API → authenticate/authorize → create turn → assemble context → model call → optional authorized tool calls → finalization → persist → stream events.

**File ingestion:** Client requests upload authorization → uploads to object storage → API validates metadata → durable job extracts and chunks content → optional embedding generation → index becomes searchable.

**Background task:** API creates task → queue dispatches worker → worker resumes controlled execution → events/results persisted → client polls or receives notification.

**Memory update:** A policy-controlled process identifies a candidate → checks relevance, sensitivity, user settings, and retention policy → persists only if permitted → retrieval index updated.

## 5. Recommended Technology Stack

| Area | Baseline recommendation | Notes |
|---|---|---|
| Mobile | Flutter / Dart | Cross-platform client |
| Backend language | TypeScript | Shared schemas and broad ecosystem |
| HTTP framework | NestJS or Fastify | Choose one; NestJS offers structure, Fastify is lean |
| API style | REST + SSE initially | WebSockets only where bidirectional needs justify them |
| Model API | Provider SDK behind adapter | Keep provider payloads out of domain modules |
| Database | PostgreSQL | Canonical transactional data |
| Vector search | pgvector initially | Move to a separate vector store only if measured needs require it |
| Cache | Redis | Cache, ephemeral coordination, distributed rate limits |
| Object storage | S3-compatible | Private buckets and short-lived signed URLs |
| Jobs | Durable queue (e.g. BullMQ/Redis or managed queue) | Pick based on operational reliability requirements |
| Schema validation | Zod or equivalent | Validate API and tool boundaries |
| ORM/query layer | Prisma, Drizzle, or SQL | Choose based on team experience and transaction needs |
| Telemetry | OpenTelemetry-compatible tracing/metrics/logging | Avoid logging sensitive payloads by default |
| Deployment | Containers on a managed platform | Start simple; add orchestration when needed |
| Secrets | Managed secrets service or secure deployment secrets | Never ship provider secrets to Flutter |
| CI/CD | Automated tests, migrations, staged deployments | Include rollback procedures |

**Choice note:** This stack is a recommended starting point, not a prerequisite. Existing infrastructure should be evaluated before migrations. Verify current provider API and SDK capabilities in official documentation before implementation.

## 6. Logical Modules and Responsibilities

### 6.1 API and identity

Responsibilities:
- Authenticate requests.
- Resolve user and tenant context.
- Validate payloads and attachment references.
- Apply request-size limits and rate limits.
- Enforce ownership on every conversation, file, memory, and task operation.
- Create correlation and idempotency identifiers.
- Return consistent errors.
- Never trust a user ID supplied in a request body as proof of identity.

### 6.2 Conversation service

Responsibilities:
- Create/list/archive conversations.
- Persist messages and ordering.
- Enforce ownership and retention rules.
- Store summaries as derived artifacts.
- Provide bounded history retrieval.
- Support deletion workflows.

### 6.3 Turn manager

Responsibilities:
- Create a turn record.
- Enforce lifecycle and deadline.
- Coordinate context assembly, model calls, tool execution, and finalization.
- Track model-call and tool-call budgets.
- Support cancellation and safe recovery.
- Emit normalized events.
- Guarantee a terminal state or clearly report that recovery is required.

### 6.4 Context assembler

Responsibilities:
- Resolve trusted instruction versions.
- Load recent conversation messages.
- Retrieve relevant prior messages, memories, and document chunks.
- Include only tools permitted for the current user and task.
- Track token budgets and provider-specific context constraints.
- Label untrusted retrieved content as data.
- Record context provenance without logging unnecessary private content.

### 6.5 Model router and adapters

Responsibilities:
- Choose a provider/model based on capabilities, task policy, health, latency, cost, and availability.
- Convert Lumio requests to provider-specific formats.
- Normalize streaming events and errors.
- Report token/usage data where available.
- Respect provider-specific tool and modality support.
- Implement bounded retry policy for transient errors.
- Avoid retries for invalid requests or policy rejections without a deliberate reason.

### 6.6 Tool registry and executor

Responsibilities:
- Register named tools with schemas and policy metadata.
- Validate model-supplied arguments.
- Resolve user/tenant authorization from trusted server context.
- Check resource ownership and scopes.
- Apply timeout, concurrency, and output-size limits.
- Require approval when policy says so.
- Execute in a restricted environment when risk warrants it.
- Persist invocation status and an audit record.
- Return structured success/failure results.

### 6.7 Memory service

Responsibilities:
- Store user-approved or policy-permitted durable preferences and facts.
- Deduplicate and expire memories.
- Support view, edit, delete, and disable operations.
- Retrieve only relevant memories.
- Track source/provenance and confidence where useful.
- Propagate deletion to derived indexes and caches.
- Exclude sensitive categories unless explicitly and appropriately supported by product policy.

### 6.8 Retrieval and document service

Responsibilities:
- Manage uploaded file metadata.
- Extract text safely in isolated workers.
- Chunk documents with source offsets/metadata.
- Create embeddings when configured.
- Retrieve relevant chunks with ownership filters.
- Provide citations/locations to source documents when possible.
- Handle corrupt, oversized, unsupported, and potentially malicious files.

### 6.9 Task and job service

Responsibilities:
- Create durable long-running tasks.
- Store status, progress, cancellation requests, and results.
- Dispatch idempotent jobs.
- Retry only safe operations.
- Recover stalled tasks.
- Keep worker privileges limited.
- Notify clients or expose task polling endpoints.

### 6.10 Observability and evaluation

Responsibilities:
- Trace each turn and nested operations.
- Record durations, error categories, usage, and cost estimates.
- Redact secrets and minimize sensitive content in logs.
- Track product quality and safety metrics.
- Run prompt/model/retrieval regression suites.
- Provide dashboards and alerting.

## 7. Runtime State Machine

### 7.1 States

- `RECEIVED`
- `VALIDATING`
- `ASSEMBLING_CONTEXT`
- `MODEL_RUNNING`
- `TOOL_PENDING`
- `AWAITING_APPROVAL`
- `TOOL_RUNNING`
- `FINALIZING`
- `COMPLETED`
- `FAILED`
- `CANCELLED`
- `TIMED_OUT`
- `RECOVERY_REQUIRED`

### 7.2 Typical transitions

```text
RECEIVED → VALIDATING → ASSEMBLING_CONTEXT → MODEL_RUNNING
MODEL_RUNNING → TOOL_PENDING → TOOL_RUNNING → MODEL_RUNNING
MODEL_RUNNING → FINALIZING → COMPLETED
TOOL_PENDING → AWAITING_APPROVAL → TOOL_RUNNING
Any non-terminal state → CANCELLED (if safely cancellable)
Any active state → TIMED_OUT (deadline exceeded)
Active state → FAILED (non-recoverable error)
Uncertain side-effect outcome → RECOVERY_REQUIRED
```

Only explicitly permitted transitions should be accepted. State updates should be transactional where practical, and transitions should be recorded for debugging and recovery.

### 7.3 Turn versus model call

A **turn** is one user-request lifecycle. It can include multiple model calls and tool invocations. Store each separately so retries, usage, and failures remain understandable.

### 7.4 Execution limits

Make limits configurable per plan, task type, provider, and environment:
- Maximum elapsed time.
- Maximum model calls.
- Maximum tool calls.
- Maximum tool concurrency.
- Maximum input/output tokens where supported.
- Maximum accumulated estimated cost.
- Maximum tool output bytes.
- Maximum retries.
- Maximum background task lifetime.

Do not let model-generated instructions override these limits.

## 8. Model Execution Loop

### 8.1 Pseudocode

```text
executeTurn(turnId):
    turn = loadAndAuthorizeTurn(turnId)
    acquireTurnLease(turn)

    try:
        transition(turn, VALIDATING)
        validateRequestAndBudget(turn)

        transition(turn, ASSEMBLING_CONTEXT)
        context = contextAssembler.build(turn)

        while withinTurnLimits(turn):
            checkCancellation(turn)
            checkDeadline(turn)

            transition(turn, MODEL_RUNNING)
            response = modelRouter.streamOrComplete(context, turn)

            for event in response.events:
                persistRequiredEvent(event)
                publishNormalizedEvent(event)

            if response.isFinal:
                transition(turn, FINALIZING)
                final = validateAndFinalize(response)
                persistFinalMessageAndUsage(final)
                transition(turn, COMPLETED)
                return

            calls = validateToolRequests(response.toolCalls)
            if calls.isEmpty:
                failWithProtocolError()

            for call in calls:
                policy = toolPolicy.evaluate(turn, call)

                if policy.requiresApproval:
                    persistApprovalRequest(call, policy)
                    transition(turn, AWAITING_APPROVAL)
                    return

                transition(turn, TOOL_PENDING)
                validateToolArguments(call)
                authorizeToolCall(turn.user, call)

                transition(turn, TOOL_RUNNING)
                result = toolExecutor.executeWithLimits(call)
                persistToolResult(result)
                context.appendToolResult(result)

        failWithExecutionLimitError()

    catch cancellation:
        safelyCancelWork()
        transition(turn, CANCELLED)
    catch timeout:
        transition(turn, TIMED_OUT)
    catch error:
        recordSanitizedFailure(error)
        transition(turn, FAILED)
    finally:
        releaseTurnLeaseIfOwned(turn)
```

This is conceptual pseudocode, not drop-in code. Production code must define transaction boundaries, leases, event delivery, concurrency behavior, provider-specific tool-call continuation, and recovery semantics.

### 8.2 Parallel tool calls

Parallel execution should be opt-in per tool and per invocation group. Do not run calls concurrently merely because a model returned several calls. Check whether they are independent, whether they share mutable resources, and whether any call has side effects.

### 8.3 Streaming and persistence

Persist enough state to recover and audit the turn, but avoid writing every token synchronously to the database. A practical approach is to stream deltas to clients and persist durable message state, selected events, and final content. Define what may be lost during a process crash and make that trade-off explicit.

## 9. Prompt and Instruction Management

### 9.1 Instruction categories

1. Platform and safety constraints.
2. Lumio core assistant behavior.
3. Product/feature configuration.
4. Task-specific guidance.
5. Tool descriptions and schemas.
6. Conversation history.
7. Retrieved documents and other untrusted content.
8. Current user request.

This is a logical precedence model. Actual provider APIs may represent instructions using different message roles and formats. Never assume user messages or retrieved content can change trusted policy.

### 9.2 Versioned prompt registry

Each prompt version should record:
- Stable key and version.
- Content checksum.
- Owner and change description.
- Created/released timestamps.
- Evaluation suite and results.
- Environment/release status.
- Rollback target.

### 9.3 Prompt safety

- Treat web pages, files, tool results, and user-provided text as untrusted input.
- Use explicit delimiters/metadata where supported.
- Do not rely on prompt text alone to enforce access control.
- Do not expose hidden platform instructions or secrets to users or tools.
- Avoid inserting unnecessary personal data.
- Keep tool permissions outside prompt control.
- Test direct and indirect prompt-injection scenarios.

### 9.4 Prompt evaluation and rollout

Use offline evaluations before rollout, staged release where practical, monitoring after release, and a quick rollback path. Compare quality, latency, refusal behavior, tool correctness, and cost.

## 10. Model Provider Abstraction

### 10.1 Internal contract

A provider adapter should support the capabilities that Lumio needs, not pretend all providers are identical.

```typescript
type ModelCapability =
  | "text"
  | "streaming"
  | "tool_calls"
  | "structured_output"
  | "image_input"
  | "audio_input"
  | "audio_output";

interface ModelRequest {
  model: string;
  instructions?: string;
  input: ModelInputItem[];
  tools?: ToolDefinition[];
  responseFormat?: ResponseFormat;
  maxOutputTokens?: number;
  metadata?: Record<string, string>;
}

interface ModelAdapter {
  getCapabilities(model: string): Set<ModelCapability>;
  generate(request: ModelRequest): AsyncIterable<ModelEvent>;
}
```

The types are illustrative. The production contract must model provider-specific continuation semantics, usage, cancellation, errors, and multimodal content.

### 10.2 Normalized events

Suggested event types:
- `response.started`
- `text.delta`
- `tool_call.started`
- `tool_call.arguments_delta`
- `tool_call.completed`
- `usage.reported`
- `response.completed`
- `response.failed`

Normalize common behavior while retaining provider metadata where necessary for debugging and correctness.

### 10.3 Routing policy

Routing inputs can include:
- Task category.
- Required capabilities.
- Context length.
- Latency target.
- Budget.
- Provider health.
- Quality evaluation results.
- User plan or product policy.

Do not route solely based on a model's marketing label. Maintain a capability registry and validate routing decisions.

### 10.4 Fallback

Fallback is permitted only when the alternate model supports the required task and policy. Do not use fallback to bypass a safety refusal or access restriction. For ambiguous side effects, reconcile operation status before retrying.

## 11. Tool System

### 11.1 Tool contract

Each tool should define:
- `name`
- `description`
- Input JSON schema
- Output schema
- Required scopes
- Resource ownership requirements
- Risk level
- Approval policy
- Timeout
- Concurrency policy
- Idempotency behavior
- Rate/cost limits
- Audit requirements

### 11.2 Example tool definition

```typescript
interface ToolDefinition {
  name: string;
  description: string;
  inputSchema: unknown;
  requiredScopes: string[];
  risk: "read" | "write" | "external_side_effect";
  approval: "never" | "conditional" | "always";
  timeoutMs: number;
  maxOutputBytes: number;
  idempotent: boolean;
}
```

The `unknown` schema placeholder must be replaced with the selected validation library's schema type.

### 11.3 Tool execution stages

1. Parse the model's requested tool call.
2. Validate the name and arguments.
3. Load authenticated user/tenant context from server-side state.
4. Check tool availability and permission scopes.
5. Enforce resource ownership.
6. Determine whether approval is required.
7. Apply quotas, deadlines, and concurrency controls.
8. Execute using a least-privilege identity.
9. Validate and size-limit the result.
10. Persist result and audit metadata.
11. Return a sanitized result to the model.
12. Resume the model loop if within limits.

### 11.4 Initial tool catalogue

Begin with low-risk tools:
- Calculator.
- Search within a user's authorized files.
- Retrieve relevant conversation messages.
- Read metadata for a user's uploaded file.
- Search the web through a vetted provider, if the product supports it.

Later additions may include:
- Webpage retrieval.
- Document analysis.
- Code execution in a sandbox.
- Calendar/email connectors with explicit scopes and user approval.
- Data-analysis workers.

Do not ship arbitrary shell access or unrestricted network access as a general-purpose model tool.

## 12. Context Assembly and Token Budgeting

### 12.1 Context components

- Trusted instructions.
- Current user input.
- Recent conversation messages.
- Relevant older messages or summaries.
- Relevant memories.
- Retrieved document passages.
- Tool definitions.
- Required output schema.
- Provider-specific metadata.

### 12.2 Budget strategy

The context assembler should:
1. Discover the selected model's actual context constraints.
2. Reserve space for output and tool continuation.
3. Allocate a budget to instructions, history, retrieval, and tools.
4. Rank candidate context by relevance and recency.
5. Avoid duplicating messages and retrieved passages.
6. Preserve source IDs for citations and debugging.
7. Summarize older history when appropriate.
8. Record truncation/summarization decisions.

Summaries are derived, not canonical. The original conversation should remain available according to retention policy.

### 12.3 Retrieval safeguards

- Filter by user/tenant authorization before exposing results to the model.
- Treat retrieved text as untrusted.
- Keep document/source identifiers.
- Avoid retrieving more personal data than needed.
- Do not interpret semantic similarity as permission.
- Provide source references when the feature requires verifiability.

## 13. Memory Architecture

### 13.1 Memory categories

- **Session context:** Current execution state; usually short-lived.
- **Conversation history:** Canonical messages in a conversation.
- **User preferences:** Durable, useful settings and preferences.
- **Project context:** User-authorized facts or decisions tied to a project.
- **Document knowledge:** Indexed file content, not necessarily a user memory.

Keep these concepts separate in storage and policy.

### 13.2 Memory lifecycle

```text
Candidate → Policy checks → Deduplication → Persist
   → Index/update → Retrieve when relevant → User control
```

### 13.3 Memory policy

Before saving, consider:
- Is it useful beyond this turn?
- Is it already stored?
- Is it sensitive or unnecessary?
- Has the user disabled memory?
- Is the source trustworthy enough?
- Is there a clear deletion and expiry path?

Users should be able to view, correct, and delete stored memories, and disable memory. Deletion should cover the canonical record plus relevant derived indexes and caches. Define the relationship between conversation deletion and memory retention clearly.

## 14. Conversation and Data Model

The following is a logical schema. Exact SQL types, indexes, partitioning, and constraints should be validated during implementation.

### 14.1 Core tables

**users**
- `id`
- `created_at`
- `status`
- `plan_id` (if applicable)
- `settings_json` (only for suitable non-sensitive settings)

**conversations**
- `id`
- `user_id`
- `title`
- `status`
- `created_at`
- `updated_at`
- `archived_at`
- `deleted_at` (if soft deletion is used)

**messages**
- `id`
- `conversation_id`
- `turn_id`
- `sequence_number`
- `role`
- `content_json`
- `status`
- `created_at`
- `completed_at`
- `metadata_json`

**turns**
- `id`
- `conversation_id`
- `user_id`
- `status`
- `idempotency_key`
- `started_at`
- `completed_at`
- `deadline_at`
- `error_code`
- `usage_json`

**model_calls**
- `id`
- `turn_id`
- `provider`
- `model`
- `status`
- `started_at`
- `completed_at`
- `usage_json`
- `latency_ms`
- `error_code`

**tool_invocations**
- `id`
- `turn_id`
- `tool_name`
- `status`
- `input_redacted_json`
- `result_redacted_json`
- `idempotency_key`
- `started_at`
- `completed_at`
- `error_code`

**attachments**
- `id`
- `user_id`
- `conversation_id`
- `storage_key`
- `mime_type`
- `size_bytes`
- `checksum`
- `scan_status`
- `created_at`
- `deleted_at`

**memories**
- `id`
- `user_id`
- `category`
- `content`
- `source_reference`
- `created_at`
- `updated_at`
- `expires_at`
- `deleted_at`

**documents / document_chunks**
- Document metadata, processing state, storage reference, chunk text, source offsets, embedding reference, ownership, and retention metadata.

**tasks**
- `id`
- `user_id`
- `conversation_id`
- `status`
- `progress_json`
- `created_at`
- `updated_at`
- `deadline_at`
- `result_reference`
- `error_code`

**prompt_versions**
- `id`
- `prompt_key`
- `version`
- `content_checksum`
- `release_status`
- `created_at`
- `evaluation_reference`

**audit_events**
- `id`
- `actor_id`
- `action`
- `resource_type`
- `resource_id`
- `outcome`
- `created_at`
- `correlation_id`
- `safe_metadata_json`

### 14.2 Database constraints and indexes

- Foreign keys for relationships.
- Unique `(conversation_id, sequence_number)` for message order.
- Unique `(user_id, idempotency_key)` or an equivalent scoped constraint for turn creation.
- Index conversation queries by owner and update time.
- Index active turns/tasks by status and deadline.
- Enforce ownership in every query path; consider PostgreSQL row-level security as an additional defense, not a substitute for sound application authorization.
- Use transactions for related state changes.
- Define retention, archival, and deletion behavior before production.
- Avoid storing raw secrets or unnecessary sensitive payloads in logs or audit records.

## 15. API Design

Prefix public APIs with `/v1`. Use consistent error shapes and validate all inputs.

### 15.1 Conversations

- `POST /v1/conversations` — create.
- `GET /v1/conversations` — list the authenticated user's conversations.
- `GET /v1/conversations/{conversationId}` — get metadata.
- `PATCH /v1/conversations/{conversationId}` — update supported metadata.
- `DELETE /v1/conversations/{conversationId}` — request deletion.

### 15.2 Turns

- `POST /v1/conversations/{conversationId}/turns` — create a turn.
- `GET /v1/turns/{turnId}` — inspect status.
- `POST /v1/turns/{turnId}/cancel` — request cancellation.
- `GET /v1/turns/{turnId}/events` — reconnect to the event stream, if supported.

The turn-creation request should support an idempotency key and a validated message/content payload. Do not accept arbitrary model names, system instructions, or tool permissions from an untrusted client.

### 15.3 Files

- `POST /v1/files/upload-intents` — request a constrained upload.
- `POST /v1/files/{fileId}/complete` — confirm upload and enqueue validation.
- `GET /v1/files/{fileId}` — inspect metadata/status.
- `DELETE /v1/files/{fileId}` — request deletion.

Prefer short-lived signed uploads/downloads. Keep buckets private and validate file size, MIME type, ownership, and actual content where possible.

### 15.4 Memory

- `GET /v1/memories` — list authorized memories.
- `PATCH /v1/memories/{memoryId}` — edit permitted fields.
- `DELETE /v1/memories/{memoryId}` — delete.
- `PATCH /v1/settings/memory` — enable/disable memory.

### 15.5 Tasks and approvals

- `GET /v1/tasks/{taskId}` — status/progress.
- `POST /v1/tasks/{taskId}/cancel` — cancel.
- `GET /v1/approvals/{approvalId}` — inspect pending approval.
- `POST /v1/approvals/{approvalId}/decision` — approve/reject after revalidating identity, expiry, and the exact action being approved.

### 15.6 Error format

```json
{
  "error": {
    "code": "TURN_LIMIT_EXCEEDED",
    "message": "The request could not be completed within the configured limits.",
    "requestId": "req_example",
    "retryable": false
  }
}
```

Do not expose stack traces, provider secrets, internal prompts, or sensitive tool results to clients.

## 16. Streaming Protocol

SSE is a practical initial transport for streamed assistant output.

### 16.1 Example event

```text
event: message.delta
id: evt_123
data: {"turnId":"turn_123","messageId":"msg_123","delta":"Hello"}

```

### 16.2 Suggested event types

- `turn.started`
- `turn.status`
- `message.started`
- `message.delta`
- `tool.started`
- `tool.completed`
- `approval.required`
- `task.progress`
- `message.completed`
- `turn.completed`
- `turn.failed`
- `turn.cancelled`

### 16.3 Delivery semantics

Do not assume exactly-once network delivery. Use event IDs, turn/message IDs, and client-side duplicate detection. Persist sufficient event metadata to support reconnects if that feature is offered. Document retention limits for replay. The client must reconcile with canonical server state after reconnecting.

## 17. File Ingestion and Retrieval (RAG)

### 17.1 Ingestion pipeline

1. Authenticate upload intent.
2. Upload to a private object-store location.
3. Verify completion and file ownership.
4. Validate size, file type, and content.
5. Scan where appropriate.
6. Process in an isolated worker.
7. Extract text/metadata.
8. Chunk with source offsets.
9. Generate embeddings if enabled.
10. Persist index records.
11. Mark the document ready or failed.
12. Make it retrievable only by authorized users.

### 17.2 Retrieval pipeline

1. Authenticate user and resolve scope.
2. Apply ownership filters.
3. Search using keyword, vector, or hybrid retrieval as appropriate.
4. Rank and deduplicate candidates.
5. Select within context budget.
6. Include source metadata.
7. Treat passages as untrusted input.
8. Preserve source references for citations.

### 17.3 Operational concerns

Handle password-protected files, malformed PDFs, enormous documents, parser vulnerabilities, archive bombs, unsupported formats, and extraction timeouts. Apply quotas per user and per file. Never execute active content from uploaded files.

## 18. Long-Running Tasks and Durable Jobs

Use a durable queue when work must outlive the initiating HTTP request.

Examples:
- Large document processing.
- Bulk ingestion.
- Report generation.
- Multi-step research.
- Data analysis.

### 18.1 Task lifecycle

`QUEUED → RUNNING → SUCCEEDED | FAILED | CANCELLED | TIMED_OUT`

Optional states may include `WAITING_FOR_APPROVAL`, `RETRY_WAIT`, and `RECOVERY_REQUIRED`.

### 18.2 Job requirements

- Stable job/task IDs.
- Idempotent handlers.
- Bounded retry policies.
- Visibility timeout or lease handling.
- Cancellation checks.
- Dead-letter handling for repeatedly failing jobs.
- Progress persistence.
- Per-user quotas.
- Worker isolation and least privilege.
- Safe reconciliation for side effects.

## 19. Security Architecture

### 19.1 Authentication and authorization

- Authenticate every private API request.
- Derive user identity from verified credentials.
- Check ownership on every resource read/write.
- Use short-lived access tokens and secure refresh practices appropriate to the identity provider.
- Protect administrative APIs with separate privileges.
- Use least-privilege service identities.
- Rate-limit sensitive endpoints and expensive operations.

### 19.2 Prompt injection and untrusted data

- Treat user messages, files, web pages, and tool outputs as potentially adversarial.
- Separate trusted instructions from untrusted content.
- Enforce permissions in code, not only in prompts.
- Restrict which tools are available in each context.
- Do not let retrieved content modify authorization rules.
- Test indirect prompt injection through documents and websites.

### 19.3 Tool isolation

For high-risk tools:
- Use a separate worker or sandbox.
- Restrict filesystem and network access.
- Limit CPU, memory, process count, and wall-clock time.
- Avoid exposing host credentials.
- Use allowlists for outbound connections where feasible.
- Enforce output size limits.
- Destroy ephemeral environments after execution.

### 19.4 External side effects

- Require approval according to risk policy.
- Bind approval to the exact action and parameters.
- Expire approvals.
- Recheck authorization immediately before execution.
- Use idempotency keys and reconciliation.
- Audit the actor, action, outcome, and relevant identifiers.
- Never treat a model's claim of user approval as proof.

### 19.5 Privacy and data lifecycle

- Minimize stored personal data.
- Encrypt in transit and at rest.
- Define retention periods.
- Redact logs.
- Support user data export/deletion where required.
- Propagate deletion to indexes, caches, and derived records.
- Document provider data handling and retention before launch.
- Keep development/test datasets free of real user secrets unless a controlled process explicitly permits them.

### 19.6 Abuse controls

Implement request quotas, upload limits, tool-specific budgets, anomaly detection, and account-level restrictions. Provide safe degradation when limits are reached.

## 20. Reliability, Retries, and Recovery

### 20.1 Error classes

Classify failures as:
- Invalid user input.
- Authentication/authorization failure.
- Provider transient error.
- Provider permanent error.
- Rate limit.
- Tool validation failure.
- Tool execution failure.
- Deadline exceeded.
- Cancellation.
- Storage/queue failure.
- Unknown outcome for a side effect.

### 20.2 Retry policy

- Use bounded exponential backoff with jitter for appropriate transient errors.
- Respect provider retry guidance where available.
- Do not retry invalid requests unchanged.
- Avoid retry storms through concurrency limits and circuit breakers where appropriate.
- Use idempotency for creation and side-effect operations.
- Reconcile unknown outcomes before retrying non-idempotent operations.

### 20.3 Recovery

On process restart, find active turns/tasks whose leases expired. Determine whether they can safely resume, should be retried, or require reconciliation. Do not simply restart every operation from the beginning.

### 20.4 Graceful degradation

When a provider is unavailable, Lumio may:
- Explain the temporary failure.
- Offer a compatible fallback if policy permits.
- Preserve the user's message for retry.
- Keep completed tool results where safe.
- Avoid claiming a task succeeded when completion is uncertain.

## 21. Observability and Cost Management

### 21.1 Trace hierarchy

```text
Trace: user turn
  ├── auth/validation
  ├── context assembly
  │    ├── conversation retrieval
  │    ├── memory retrieval
  │    └── document retrieval
  ├── model call #1
  ├── tool invocation #1
  ├── model call #2
  ├── finalization
  └── event delivery
```

### 21.2 Metrics

Track:
- Request volume and error rate.
- Time to first token.
- End-to-end turn latency.
- Model-call latency and failures.
- Tool-call success and duration.
- Queue depth and task age.
- Token usage and estimated cost.
- Cost per completed turn.
- Context size and truncation.
- Cancellation and timeout rates.
- Retrieval relevance and citation correctness.
- Evaluation scores and safety incidents.

### 21.3 Logging

Use structured logs with request, conversation, turn, model-call, tool-call, and task identifiers. Avoid logging full prompts, private files, credentials, or raw personal data by default. Add explicit redaction and access controls to any diagnostic content retained.

### 21.4 Cost controls

- Per-user and per-plan quotas.
- Maximum output tokens.
- Context budgets.
- Tool execution budgets.
- Cache only when correctness and privacy permit.
- Route simple tasks to lower-cost compatible models when evaluation supports it.
- Monitor spend and alert on anomalies.
- Enforce a hard budget ceiling for each turn/task.

## 22. Evaluation and Quality Engineering

### 22.1 Evaluation layers

1. **Unit tests:** State transitions, schema validation, budgeting, permission rules.
2. **Integration tests:** Provider adapters, database, queue, object storage.
3. **Contract tests:** Normalized events and tool input/output formats.
4. **End-to-end tests:** Full user turns and streaming.
5. **Security tests:** Cross-user access, prompt injection, unsafe tool calls.
6. **Offline AI evaluations:** Correctness, relevance, tool choice, citation support, refusal behavior.
7. **Production monitoring:** Latency, errors, cost, user feedback, quality signals.

### 22.2 Evaluation dataset categories

Include representative examples for:
- General chat.
- Multi-turn continuity.
- Ambiguous requests.
- Missing information.
- Coding and structured output.
- Tool selection and malformed tool arguments.
- Tool failures.
- Prompt injection in files and webpages.
- Unauthorized resource access.
- Long-context summarization.
- Memory relevance and deletion.
- Provider outage and cancellation.
- Multimodal input when supported.

### 22.3 Release gates

Before deploying a new prompt, model, tool, or retrieval configuration:
- Run unit and integration tests.
- Run the relevant evaluation subset.
- Compare against a known baseline.
- Review safety regressions.
- Validate cost and latency.
- Roll out gradually where possible.
- Keep a rollback path.

## 23. Deployment Architecture

### 23.1 Initial production deployment

```text
Internet
   │
   ▼
TLS / Load Balancer
   │
   ▼
API containers (stateless)
   ├── PostgreSQL (managed)
   ├── Redis (managed if used)
   ├── Private object storage
   ├── Durable queue
   └── Model provider APIs

Worker containers
   ├── Document processing
   ├── Embeddings/indexing
   └── Long-running tasks

Telemetry pipeline
   ├── Logs
   ├── Metrics
   └── Traces
```

### 23.2 Environments

Maintain separate development, staging, and production environments. Use separate credentials, data, and storage buckets. Production data should not be copied into development without an approved sanitization process.

### 23.3 Scaling strategy

Start by scaling stateless API containers horizontally and sizing workers according to queue depth. Monitor PostgreSQL connection usage and use a pooler when appropriate. Add service boundaries only when profiling, reliability requirements, or team ownership justifies them.

### 23.4 Database migrations

Use versioned migrations, test them against representative data, and define rollback or forward-fix procedures. Prefer backward-compatible schema changes during rolling deployments.

## 24. Flutter Client Architecture

Suggested client modules:

```text
lib/
├── app/
│   ├── app.dart
│   ├── router.dart
│   └── theme/
├── core/
│   ├── networking/
│   ├── auth/
│   ├── errors/
│   ├── streaming/
│   └── storage/
├── features/
│   ├── chat/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── conversations/
│   ├── attachments/
│   ├── memory_settings/
│   └── tasks/
└── shared/
    ├── widgets/
    └── models/
```

Client responsibilities:
- Display messages and streamed deltas.
- Upload attachments through authorized flows.
- Track local optimistic state.
- Reconnect and reconcile turn state.
- Handle cancellation.
- Show tool/task progress.
- Avoid storing provider secrets.
- Use secure local storage for sensitive session credentials.
- Avoid assuming client-side checks provide security.

Keep domain models separate from provider-specific response structures.

## 25. Configuration and Feature Flags

Configuration should be validated at startup and divided into:
- Static deployment configuration.
- Secret configuration.
- Versioned product policy.
- Feature flags.
- Per-user/plan limits.

Feature flags may control staged rollout of memory, web search, file analysis, or alternate models. Flags must not bypass authorization or security policy.

Record which configuration and prompt versions were used for a turn to support debugging and reproducibility.

## 26. Suggested Repository Structure

```text
lumio-backend/
├── apps/
│   ├── api/
│   └── workers/
├── src/
│   ├── modules/
│   │   ├── identity/
│   │   ├── conversations/
│   │   ├── messages/
│   │   ├── attachments/
│   │   ├── memory/
│   │   ├── retrieval/
│   │   ├── tasks/
│   │   ├── approvals/
│   │   └── billing/
│   ├── ai/
│   │   ├── runtime/
│   │   ├── context/
│   │   ├── models/
│   │   ├── tools/
│   │   ├── prompts/
│   │   ├── safety/
│   │   └── evaluation/
│   ├── infrastructure/
│   │   ├── database/
│   │   ├── cache/
│   │   ├── queue/
│   │   ├── storage/
│   │   └── observability/
│   └── shared/
│       ├── types/
│       ├── schemas/
│       └── errors/
├── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── security/
│   ├── contract/
│   └── evaluations/
├── docs/
├── package.json
└── README.md
```

This structure is a guide. Avoid creating empty abstractions prematurely; introduce modules when they own a clear responsibility and contract.

## 27. Implementation Roadmap

### Phase 0 — Confirm requirements
- Confirm existing backend and deployment.
- Choose initial model provider(s).
- Define launch features and user plans.
- Define data retention and privacy requirements.
- Establish expected traffic and budget.

**Exit criteria:** Decisions documented and reviewed.

### Phase 1 — Reliable core chat
- Authentication and ownership checks.
- Conversation/message persistence.
- Provider adapter.
- Turn manager and bounded execution.
- Streaming API.
- Cancellation and idempotent turn creation.
- Basic tracing and error handling.

**Exit criteria:** Stable multi-turn chat with recoverable client behavior.

### Phase 2 — Safe tools
- Tool registry and schemas.
- Policy engine.
- Tool executor.
- Per-tool permissions, timeouts, and output limits.
- Audit events.
- Unit/security tests.

**Exit criteria:** Tools can execute within enforced boundaries.

### Phase 3 — Context and memory
- Context assembler and budgeter.
- Conversation summaries.
- User-controlled memory.
- Retrieval and ownership filters.
- Memory deletion propagation.

**Exit criteria:** Relevant context is retrieved without cross-user leakage.

### Phase 4 — File ingestion and background work
- Private uploads.
- Validation and scanning.
- Isolated parsing workers.
- Chunking and retrieval.
- Durable jobs, retries, progress, cancellation.

**Exit criteria:** Files and long-running tasks survive transient failures safely.

### Phase 5 — Production readiness
- Provider/model routing and compatible fallback.
- Budget enforcement.
- Dashboards and alerts.
- Evaluation suite and release gates.
- Incident response and data deletion procedures.
- Load and security testing.

**Exit criteria:** Defined service objectives and tested operational procedures.

### Phase 6 — Advanced capabilities
- More modalities.
- Approved external integrations.
- Sandboxed code/data analysis.
- Delegated specialist agents if evaluations justify them.
- More advanced planning and resumable workflows.

**Exit criteria:** Each capability has a documented risk model, tests, and measured product value.

## 28. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Prompt injection | Treat external content as untrusted; enforce permissions in code |
| Cross-user data leakage | Ownership filters, scoped queries, isolation tests |
| Infinite tool loop | Hard call/time/token/cost budgets |
| Duplicate side effects | Idempotency keys, operation records, reconciliation |
| Provider outage | Timeouts, bounded retries, compatible fallback |
| Expensive context | Token budgets, retrieval ranking, usage tracking |
| Incorrect memory | User controls, provenance, deduplication, deletion |
| Malicious uploads | Size/type checks, scanning, isolated parsing |
| Queue backlog | Queue-depth alerts, worker scaling, quotas |
| Leaked secrets | Secret manager, redaction, least privilege |
| Model/tool incompatibility | Capability registry and contract tests |
| Unsafe autonomous actions | Risk tiers, approvals, sandboxing |
| Prompt regression | Versioned prompts and automated evaluations |
| Vendor lock-in | Provider adapter and capability-aware routing |
| Premature complexity | Modular monolith first; split services based on evidence |

## 29. Service-Level Objectives to Define

Set targets only after establishing workload and baseline measurements. Candidate SLOs include:
- API availability.
- Time to first streamed token.
- Interactive turn completion latency.
- Successful completion rate.
- Tool execution success rate.
- Background task completion within deadline.
- Data durability and recovery point.
- Time to recover from provider failure.

Define separate targets for ordinary chat and complex tasks. Avoid promising a single latency target for workflows whose duration depends on external tools.

## 30. Open Decisions Before Implementation

The following choices remain deliberately open because they depend on Lumio's current state:

1. Existing backend: none, Firebase, Supabase, Node.js, or another stack.
2. Model strategy: OpenAI-only initially, multi-provider, or hosted plus local.
3. Launch scope: chat only, advanced tools, or full agent/task platform.
4. Execution approach: custom engine, existing framework, or hybrid.
5. Authentication provider and subscription/billing requirements.
6. Expected users, concurrency, latency goals, and budget.
7. Data residency, retention, and regulatory requirements.
8. Required modalities at launch: text, images, PDFs, audio, or others.
9. Whether users can connect external accounts and which actions require confirmation.
10. Deployment target and operational ownership.

Do not treat unresolved choices as implemented features.

## 31. Official Documentation to Consult

Verify API details, model capabilities, SDK versions, pricing, and retention terms against current official documentation during implementation.

- OpenAI API documentation: https://developers.openai.com/api/docs/
- Responses API reference: https://developers.openai.com/api/reference/responses/overview
- Function calling: https://developers.openai.com/api/docs/guides/function-calling
- Agents SDK: https://developers.openai.com/api/docs/guides/agents/sdk
- Prompt engineering: https://developers.openai.com/api/docs/guides/prompt-engineering
- Conversation state: https://developers.openai.com/api/docs/guides/conversation-state
- Flutter documentation: https://docs.flutter.dev/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- OpenTelemetry documentation: https://opentelemetry.io/docs/

URLs are provided as starting points; confirm the current pages and capabilities before coding.

## 32. Final Recommendation

Build Lumio as a **modular, observable, policy-enforced AI platform** with a custom product/orchestration layer and existing provider SDKs where useful.

The recommended order is:

1. Reliable turn lifecycle and streaming.
2. Persistent conversations and context assembly.
3. Safe tool registry and execution.
4. User-controlled memory and document retrieval.
5. Durable jobs, recovery, and observability.
6. Model routing and advanced agent features.

The defining quality of Lumio should not be the number of tools or agents it contains. It should be the reliability with which it understands a request, uses authorized capabilities, preserves relevant context, explains uncertainty, and completes work safely.

**Architecture status:** Proposed baseline. Validate open decisions, security assumptions, provider capabilities, and operational targets before production implementation.
