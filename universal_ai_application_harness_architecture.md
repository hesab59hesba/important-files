# Universal AI Application Harness
## Reusable Production-Grade System Architecture & Technical Specification

**Document type:** Reusable reference architecture and engineering blueprint  
**Version:** 1.0  
**Status:** Reference baseline — adapt and validate for each product  
**Last updated:** 2026-10-28  
**Scope:** AI-powered applications across mobile, web, desktop, internal tools, and API products

---

## Document Purpose

This document describes a reusable architecture for building the software system around one or more AI models. It is designed to be adapted for many kinds of applications: personal assistants, customer support, education, productivity, research, coding, document analysis, business workflows, and domain-specific copilots.

The architecture is deliberately **product-neutral**. It separates the common AI platform from application-specific features, policies, workflows, and user experiences.

It is not a claim to reproduce any proprietary implementation of ChatGPT or another AI system. It describes widely applicable engineering patterns for orchestration, model integration, tool use, context, retrieval, memory, security, reliability, evaluation, and operations.

### How to use this document

1. Treat the architecture as a menu of capabilities, not a mandate to implement everything.
2. Complete the application profile in Section 3.
3. Mark each component as **Required**, **Later**, or **Not applicable**.
4. Decide which capabilities belong in the core platform and which belong in product modules.
5. Validate provider APIs, legal requirements, operational targets, and cost assumptions before implementation.
6. Keep the first deployment as simple as possible while preserving good boundaries.

---

# Part I — Architecture and Product Boundaries

## 1. Executive Summary

A production AI application is more than a chat UI connected to a model API. It is a coordinated system responsible for identity, conversation state, context assembly, model selection, tool execution, data access, user permissions, persistence, streaming, safety, observability, and recovery.

This reference architecture uses a **modular-core, adapter-based design**:

- One or more client applications.
- An API and identity boundary.
- A product/application layer.
- An AI runtime and orchestration engine.
- Provider adapters for models and other external services.
- A controlled tool registry and execution layer.
- Optional memory and retrieval subsystems.
- Persistent storage, caching, and durable jobs.
- Security, policy enforcement, observability, and evaluation.
- An explicit extension mechanism for product-specific capabilities.

A modular monolith is a sensible default for an early product. Split components into independent services only when scale, reliability, security isolation, deployment needs, or team ownership justify the added complexity.

## 2. Goals and Non-Goals

### 2.1 Goals

The architecture should enable an application to:

1. Accept user requests from one or more clients.
2. Authenticate users and enforce resource ownership.
3. Integrate one or more model providers behind stable interfaces.
4. Manage multi-turn interactions and long-running tasks.
5. Assemble context from messages, user settings, files, and authorized data sources.
6. Stream progress and responses when supported.
7. Execute tools under explicit permissions and resource limits.
8. Validate structured outputs and tool arguments.
9. Store canonical records and recover from interruptions.
10. Support optional retrieval-augmented generation (RAG) and memory.
11. Handle multimodal content through extensible content contracts.
12. Measure reliability, latency, quality, usage, and cost.
13. Test prompt, model, tool, and retrieval changes before release.
14. Evolve from one model and a few tools to a more capable platform without a complete rewrite.

### 2.2 Non-goals

This document does not require:

- Training a foundation model.
- Using a particular cloud provider or programming language.
- Implementing every capability in the first release.
- Starting with a microservice architecture.
- Making every workflow autonomous.
- Giving the model direct, unrestricted access to databases, shell commands, or user accounts.
- Assuming model output is true because it is fluent or well-formed.
- Assuming all model providers share the same capabilities.
- Treating vector similarity as authorization or evidence of truth.

## 3. Application Profile and Adaptation Worksheet

Complete this section for each product before finalizing the design.

| Decision | Product-specific answer |
|---|---|
| Application name and purpose | `[Fill in]` |
| Target clients | `[Mobile / web / desktop / API / internal]` |
| Main user groups | `[Fill in]` |
| Authentication method | `[Fill in]` |
| Backend and language | `[Fill in]` |
| Model provider(s) | `[Fill in]` |
| Primary capabilities | `[Chat / tools / files / search / workflow / other]` |
| Required modalities | `[Text / images / audio / video / documents]` |
| External systems | `[Fill in]` |
| Data sensitivity | `[Public / internal / confidential / regulated]` |
| Expected traffic and concurrency | `[Fill in]` |
| Latency expectations | `[Fill in]` |
| Budget limits | `[Fill in]` |
| Retention and deletion requirements | `[Fill in]` |
| Deployment environment | `[Fill in]` |
| Human approval requirements | `[Fill in]` |
| Availability and recovery targets | `[Fill in]` |

### 3.1 Capability selection

For each capability, assign one status:

- **Required now**
- **Required later**
- **Not applicable**

| Capability | Typical default |
|---|---|
| Authentication and authorization | Required now for private data |
| Basic model integration | Required now |
| Conversation persistence | Required for conversational products |
| Streaming | Optional; valuable for interactive UX |
| Tool registry | Required if the model can call tools |
| Durable task queue | Later unless work outlives HTTP requests |
| Memory | Optional and user-controlled |
| RAG/document retrieval | Required only for relevant product needs |
| Multiple model providers | Later unless portability/failover is needed |
| Specialist agents | Later; justify with evaluations |
| Code execution sandbox | Only if product needs code execution |
| External account connectors | Only for defined integrations |
| Advanced analytics | Based on product and operational needs |

## 4. Architecture Principles

1. **Product-neutral core:** Generic runtime concerns belong in the platform; business rules belong in product modules.
2. **Server authority:** The backend is authoritative for identity, permissions, state, and persisted results.
3. **Least privilege:** Each tool and worker receives only the access it requires.
4. **Explicit state:** Represent turns, tasks, model calls, tool calls, approvals, and failures as structured state.
5. **Adapter boundaries:** Avoid leaking provider-specific schemas into core domain logic.
6. **Capability-aware behavior:** Check actual model and tool capabilities at runtime or configuration time.
7. **Durability where required:** Persist state that must survive restarts; do not make every token write a database transaction.
8. **Idempotent retries:** Prevent duplicate turns and side effects.
9. **Bounded execution:** Enforce time, cost, token, iteration, concurrency, and output-size limits.
10. **Observable behavior:** Assign correlation identifiers and record useful execution metadata.
11. **Privacy by design:** Minimize, scope, protect, and expire personal data.
12. **Human control:** Use approval workflows for consequential or irreversible actions.
13. **Evaluation before release:** Test behavior changes instead of relying only on anecdotal conversations.
14. **Graceful degradation:** Explain failure accurately and preserve safe progress where possible.
15. **Simplicity first:** Add infrastructure only when requirements or evidence justify it.

## 5. System Context

### 5.1 Logical architecture

```text
┌──────────────────────────────────────────────────────────┐
│                       Client Layer                       │
│     Mobile · Web · Desktop · API Consumer · Admin UI     │
└────────────────────────────┬─────────────────────────────┘
                             │ HTTPS / streaming transport
                             ▼
┌──────────────────────────────────────────────────────────┐
│                  API and Identity Boundary               │
│ Authentication · Validation · Authorization · Quotas     │
│ Request IDs · Rate limits · Upload intents · API errors  │
└────────────────────────────┬─────────────────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────┐
│                     Product Application                  │
│ Domain services · User settings · Plans · Workflows      │
│ Business rules · Application-specific integrations       │
└────────────────────────────┬─────────────────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────┐
│                     AI Platform Core                     │
│ Turn manager · Context builder · Model router            │
│ Runtime state · Tool policy · Output validation           │
│ Event streaming · Budgets · Cancellation                 │
└─────────────┬───────────────────┬────────────────────────┘
              │                   │
              ▼                   ▼
┌──────────────────────┐  ┌────────────────────────────────┐
│ Context Services     │  │ Controlled Execution            │
│ History · Memory     │  │ Tool registry · Policy checks   │
│ Retrieval · Files    │  │ Executors · Approval · Sandbox  │
└────────────┬─────────┘  └───────────────┬────────────────┘
             │                            │
             └──────────────┬─────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│                 Provider / Adapter Layer                 │
│ Model APIs · Embeddings · Search · External connectors    │
│ Speech · Vision · Other approved services                │
└────────────────────────────┬─────────────────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────┐
│                     Infrastructure                       │
│ SQL DB · Cache · Object storage · Queue · Secrets        │
│ Logs · Metrics · Traces · Evaluation datasets            │
└──────────────────────────────────────────────────────────┘
```

### 5.2 Runtime flows

**Interactive request**
1. Client submits a validated request.
2. API authenticates the caller and checks resource access.
3. Application creates or resumes a turn.
4. Context services return permitted context.
5. Runtime chooses a compatible model.
6. Model emits text, a final result, or a tool request.
7. Tool requests are validated and authorized before execution.
8. Results are fed back into the model when appropriate.
9. Final output is validated, persisted, and streamed.
10. Turn is marked terminal and usage is recorded.

**Document ingestion**
1. Client requests a constrained upload.
2. File uploads to private object storage.
3. Backend validates ownership, type, size, and upload completion.
4. A worker scans and parses the file in isolation.
5. Text is chunked and indexed if applicable.
6. Processing status is persisted.
7. Retrieval becomes available only within authorized scope.

**Long-running task**
1. API creates a durable task.
2. Queue dispatches a worker.
3. Worker executes bounded steps and persists progress.
4. Client polls or subscribes to progress.
5. Results are saved and task status becomes terminal.

## 6. Logical Layers and Boundaries

### 6.1 Client layer

Owns presentation, local interaction state, secure session handling, upload UX, streaming display, and reconnection. It is not a security authority.

### 6.2 API layer

Owns request validation, authentication middleware, authorization entry points, quotas, request IDs, and stable API contracts.

### 6.3 Product/domain layer

Owns product-specific entities, business rules, plans, user preferences, domain workflows, and integrations. Keep this layer independent from model-provider payload formats.

### 6.4 AI runtime layer

Owns turn lifecycle, context assembly coordination, model selection, tool-call continuation, budgets, cancellation, normalized events, and finalization.

### 6.5 Adapter layer

Wraps external models and services. Converts requests and responses, normalizes errors, reports capabilities, and hides provider transport details from core logic.

### 6.6 Infrastructure layer

Owns persistence implementations, queues, object storage, caching, secrets, telemetry, and deployment concerns. Domain modules depend on interfaces where doing so creates a useful testable boundary.

---

# Part II — Runtime and Model Integration

## 7. Recommended Technology Choices

No single stack is right for every application. Choose based on team experience, deployment environment, data sensitivity, workload, and maintenance capacity.

| Concern | Common options | Selection guidance |
|---|---|---|
| Client | Flutter, React, native mobile, desktop | Use the platform that best fits UX and team |
| Backend | TypeScript, Python, Go, Java/Kotlin, C# | Prefer strong ecosystem and team familiarity |
| HTTP framework | NestJS, Fastify, FastAPI, ASP.NET Core, Spring | Choose one primary framework |
| Relational database | PostgreSQL, managed SQL alternatives | Use for canonical transactional records |
| Cache | Redis or managed equivalent | Use only for clear ephemeral/caching needs |
| Object storage | S3-compatible or cloud-native | Private objects and constrained signed URLs |
| Queue | Managed queue, BullMQ, RabbitMQ, cloud task service | Choose based on delivery and retry semantics |
| Vector retrieval | pgvector, dedicated vector DB, managed retrieval | Benchmark against real datasets |
| Schema validation | Zod, Pydantic, JSON Schema validators, language equivalents | Validate every untrusted boundary |
| Telemetry | OpenTelemetry-compatible stack | Instrument cross-service execution |
| Deployment | Managed containers, serverless, Kubernetes | Avoid unnecessary operational complexity |
| Secrets | Cloud secrets manager or equivalent | Never ship server secrets to clients |

A modular monolith is usually the simplest useful starting point. The architecture should not force a specific database, queue, cloud provider, or model vendor.

## 8. Core Domain Concepts

Use consistent definitions throughout code and documentation.

- **Conversation:** A durable grouping of related messages or interactions.
- **Message:** A user, assistant, tool, or system-generated content record.
- **Turn:** The lifecycle of one user request; may contain several model calls and tool calls.
- **Model call:** One request to a model provider.
- **Tool invocation:** One requested and policy-checked execution of a tool.
- **Task:** Durable work that may outlive the original request.
- **Event:** A structured runtime occurrence or client-facing update.
- **Memory:** A retained, permissioned piece of user/project context.
- **Artifact:** A generated or uploaded file/result with a controlled reference.
- **Approval:** A user's explicit authorization for a specific consequential action.
- **Trace:** A correlated record of operations for one request or workflow.

Do not collapse these into a single generic JSON object. They have different lifecycles, permissions, and retry semantics.

## 9. Turn Manager

The Turn Manager owns one user-request lifecycle.

Responsibilities:
- Create and authorize a turn.
- Establish a deadline and execution budget.
- Coordinate context assembly and model calls.
- Process model events.
- Dispatch authorized tool invocations.
- Track approval pauses.
- Persist required state transitions.
- Support cancellation and safe recovery.
- Finalize output and usage.
- Ensure each turn reaches a terminal state or an explicit recovery state.

A turn and a model call are different records. A single turn can invoke a model multiple times, including after tool results.

## 10. Runtime State Machine

### 10.1 Suggested states

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

### 10.2 Typical transitions

```text
RECEIVED → VALIDATING → ASSEMBLING_CONTEXT → MODEL_RUNNING
MODEL_RUNNING → TOOL_PENDING → TOOL_RUNNING → MODEL_RUNNING
MODEL_RUNNING → FINALIZING → COMPLETED
TOOL_PENDING → AWAITING_APPROVAL → TOOL_RUNNING
Active state → CANCELLED / TIMED_OUT / FAILED
Uncertain external side effect → RECOVERY_REQUIRED
```

Transitions must be validated. Record transition timestamps and sanitized failure categories. Use transactions for related state updates when practical.

### 10.3 Execution limits

Configure limits for:
- Total elapsed time.
- Model calls per turn.
- Tool calls per turn.
- Tool concurrency.
- Input and output tokens where supported.
- Estimated cost per turn/task.
- Tool output bytes.
- Retries.
- Background task lifetime.
- Per-user concurrent turns and tasks.

The model must not be able to override runtime limits.

## 11. Model Execution Loop

### 11.1 Conceptual pseudocode

```text
executeTurn(turnId):
    turn = loadTurnAndVerifyOwnership(turnId)
    acquireLease(turn)

    try:
        validateRequest(turn)
        enforceInitialBudget(turn)
        context = contextBuilder.build(turn)

        while withinLimits(turn):
            checkCancellation(turn)
            checkDeadline(turn)

            result = modelRouter.execute(context, turn)
            publishAndPersistRequiredEvents(result.events)

            if result.isFinal:
                final = outputValidator.validate(result)
                persistFinalMessageAndUsage(final)
                markCompleted(turn)
                return

            calls = toolRequestValidator.validate(result.toolCalls)
            if calls is empty:
                failWithProtocolError()

            for call in calls:
                decision = toolPolicy.evaluate(turn, call)

                if decision.requiresApproval:
                    createApprovalBoundToExactAction(call, decision)
                    markAwaitingApproval(turn)
                    return

                validateToolArguments(call)
                authorizeToolAgainstTrustedContext(turn, call)
                toolResult = toolExecutor.executeWithLimits(call)
                persistToolResult(toolResult)
                context.append(toolResult)

        failWithExecutionLimitError()

    catch cancellation:
        cancelSafeOperations()
        markCancelled(turn)
    catch timeout:
        markTimedOut(turn)
    catch error:
        recordSanitizedFailure(error)
        markFailedOrRecoveryRequired(turn, error)
    finally:
        releaseLeaseIfOwned(turn)
```

This is conceptual pseudocode, not production-ready code. The implementation must specify transaction boundaries, provider continuation semantics, streaming, leases, concurrent tool calls, and recovery behavior.

### 11.2 Parallel calls

Parallel tool calls must be explicitly supported by the tool and runtime. Check for shared state, conflicting writes, rate limits, and dependencies. Do not parallelize consequential operations merely because the model returned them together.

### 11.3 Lease and concurrency model

Prevent two workers from executing the same turn simultaneously. Use database leases, queue ownership, or another well-defined coordination mechanism. Leases need expiry and safe renewal. A lease alone does not make side effects idempotent.

## 12. Context Assembly

The context builder decides what information is relevant and permitted for one model call.

### 12.1 Context components

- Trusted platform/product instructions.
- Current request.
- Recent conversation messages.
- Relevant older messages or summaries.
- Relevant memories, if enabled and authorized.
- Retrieved document passages.
- Tool definitions permitted for this user/task.
- Required output schema.
- Provider-specific metadata.

### 12.2 Responsibilities

1. Discover the selected model's actual context constraints.
2. Reserve room for output and tool continuation.
3. Allocate budgets to instructions, history, retrieval, and tool definitions.
4. Rank context by relevance and recency.
5. Avoid duplicate or contradictory snippets where possible.
6. Preserve source identifiers and provenance.
7. Summarize older context only when appropriate.
8. Mark untrusted external content as data.
9. Record truncation/summarization decisions.
10. Avoid including personal information that is not needed.

Summaries are derived artifacts. They should not replace canonical source messages.

## 13. Model Router

The router selects a compatible model/provider based on configured policies.

Possible inputs:
- Task type.
- Required capabilities.
- Context size.
- Latency target.
- Cost budget.
- Provider health and availability.
- Quality evaluation results.
- User plan or product constraints.
- Data handling requirements.

Maintain a capability registry. Do not assume that every model supports tool calling, images, audio, structured outputs, the same context size, or the same streaming semantics.

Begin with a small number of explicit routing policies. Expand only when measurements show a benefit.

## 14. Provider Adapter Contract

Core application logic should depend on a normalized interface rather than provider-specific request bodies.

Illustrative TypeScript contract:

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

These types are examples. Define production types for multimodal content, provider continuation, cancellation, usage, errors, and tool-call identifiers.

### 14.1 Normalized events

Suggested events:
- `response.started`
- `text.delta`
- `tool_call.started`
- `tool_call.arguments_delta`
- `tool_call.completed`
- `usage.reported`
- `response.completed`
- `response.failed`

Retain provider-specific diagnostic metadata when useful, but do not leak it into public APIs unnecessarily.

### 14.2 Retry and fallback

Retry only transient failures under a bounded policy. Respect rate-limit guidance. Do not retry invalid requests unchanged. Use a fallback only when it supports the required capabilities and product policy allows it. Never use fallback to bypass a safety refusal or authorization decision.

For uncertain external side effects, reconcile the operation before retrying.

## 15. Instruction and Prompt Management

### 15.1 Logical instruction categories

1. Platform and safety constraints.
2. Product identity and behavior.
3. Feature/task instructions.
4. Tool descriptions and schemas.
5. Conversation context.
6. Retrieved untrusted content.
7. Current user request.

The actual API message roles differ by provider. Keep the precedence policy explicit and do not let user content or retrieved content override trusted platform controls.

### 15.2 Versioning

Every production prompt version should record:
- Stable key and version.
- Content checksum.
- Owner and change description.
- Creation/release timestamps.
- Evaluation dataset and results.
- Release status and rollback target.

### 15.3 Prompt safety

- Treat files, webpages, user content, and tool output as potentially adversarial.
- Enforce permissions in code, not only through prompt instructions.
- Do not expose secrets or privileged instructions to tools or users.
- Include only necessary personal context.
- Test direct and indirect prompt-injection scenarios.
- Do not claim that prompt wording alone eliminates prompt injection.

### 15.4 Rollout

Test prompt changes offline, compare them with a baseline, stage releases where possible, monitor quality and cost, and retain a rollback path.

## 16. Structured Output and Validation

When a feature requires machine-readable output:
1. Define a schema.
2. Use provider-supported structured output when available.
3. Validate the returned result on the server.
4. Handle incomplete, malformed, or refused outputs.
5. Apply domain/business validation after schema validation.
6. Never treat schema validity as proof that content is correct.

For high-impact decisions, include domain-specific checks and, where appropriate, human review.

---

# Part III — Tools, Workflows, and Autonomy

## 17. Tool Registry and Executor

A model should request a tool through a structured contract. It should never have unrestricted access to the application process, database, shell, or user credentials.

### 17.1 Tool definition

Each tool should define:
- Unique name and description.
- Input schema and output schema.
- Required permission scopes.
- Resource ownership requirements.
- Risk category.
- Approval policy.
- Timeout and concurrency limits.
- Output-size limit.
- Idempotency behavior.
- Audit requirements.
- Network/filesystem access policy where relevant.

Illustrative interface:

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

Replace `unknown` with the selected schema library's actual type in implementation.

### 17.2 Execution pipeline

1. Parse requested tool call.
2. Validate tool name and arguments.
3. Load authenticated identity and tenant context from trusted server state.
4. Check tool registration and availability.
5. Verify permissions and resource ownership.
6. Evaluate risk and approval requirements.
7. Enforce budget, concurrency, and deadline.
8. Execute with least privilege.
9. Validate and size-limit output.
10. Persist invocation outcome and audit metadata.
11. Return a sanitized result.
12. Resume the model if permitted and within limits.

### 17.3 Tool categories

Examples:
- Read-only retrieval.
- Calculations.
- Search within authorized data.
- Web search/retrieval.
- File analysis.
- Database query through narrow domain APIs.
- External account actions.
- Sandboxed code execution.

Start with low-risk tools and add higher-risk tools only after permission, isolation, auditing, and recovery behavior have been tested.

## 18. Permissions and Approval

The model's request is not authorization.

The policy engine should decide whether an action is:
- Allowed automatically.
- Allowed only within a narrow scope.
- Blocked.
- Allowed only after explicit user approval.

For approval-based actions:
- Bind approval to the exact action and parameters.
- Show the user a clear description of the consequence.
- Set an expiry.
- Revalidate identity and authorization immediately before execution.
- Record the decision.
- Do not accept model-generated claims of approval as proof.
- Reconcile uncertain outcomes before retrying.

Approval policy should be product-specific and risk-based.

## 19. Multi-Step Workflows and Agents

A workflow is a bounded sequence of steps, some of which may call a model or tool. A specialist agent is not automatically required for every multi-step task.

Recommended progression:
1. Deterministic workflow for predictable tasks.
2. Single model with controlled tools for flexible tasks.
3. Explicit planner/executor separation when evaluation shows it helps.
4. Specialist agents only when they provide a measurable quality, isolation, or organizational benefit.

### 19.1 Workflow requirements

- Explicit task state.
- Step inputs and outputs.
- Per-step and total deadlines.
- Tool permissions.
- Retry policy.
- Cancellation.
- Progress reporting.
- Durable checkpoints for long tasks.
- Clear completion criteria.
- Human approval for consequential steps.

Avoid unbounded recursive delegation or agent-to-agent loops. Apply global budgets across all nested work.

## 20. Long-Running Tasks and Durable Jobs

Use a durable queue when work must outlive the HTTP request.

Examples include large document processing, bulk ingestion, research, report generation, and data analysis.

### 20.1 Task lifecycle

`QUEUED → RUNNING → SUCCEEDED | FAILED | CANCELLED | TIMED_OUT`

Optional states: `WAITING_FOR_APPROVAL`, `RETRY_WAIT`, `RECOVERY_REQUIRED`.

### 20.2 Requirements

- Stable task IDs.
- Idempotent job handlers.
- Bounded retries and exponential backoff with jitter where appropriate.
- Lease/visibility timeout handling.
- Cancellation checks.
- Dead-letter handling for repeated failures.
- Persisted progress.
- Per-user quotas.
- Worker isolation and least privilege.
- Reconciliation for side effects.
- Expiry and cleanup policies.

## 21. Sandboxed Execution

If the application runs model-generated code, processes untrusted files, or provides computer-use capabilities, isolate execution from the main API service.

Potential controls:
- Ephemeral workers/containers.
- CPU, memory, process, and wall-clock limits.
- Restricted filesystem.
- Network egress controls.
- No host secrets.
- Narrowly scoped temporary credentials where needed.
- Output-size limits.
- Cleanup after execution.
- Auditing and abuse monitoring.

Do not execute model-generated code directly inside a privileged API process. A sandbox reduces risk but does not eliminate it; threat-model the implementation and dependencies.

---

# Part IV — Memory, Data, and Retrieval

## 22. Memory Architecture

Memory is optional. A product that does not need durable personalization can omit this subsystem.

### 22.1 Separate concepts

- **Session context:** Temporary execution state.
- **Conversation history:** Canonical interaction record.
- **User preferences:** Durable settings or preferences.
- **Project context:** Authorized facts and decisions tied to a project.
- **Document knowledge:** Searchable content from files.
- **Operational telemetry:** System data, not user memory.

Do not store all of these in one unstructured memory field.

### 22.2 Memory lifecycle

```text
Candidate
   ↓
Relevance and usefulness check
   ↓
Sensitivity and product-policy check
   ↓
User setting / consent check
   ↓
Deduplicate and validate source
   ↓
Persist with provenance and retention
   ↓
Retrieve only when relevant and authorized
   ↓
User can view, correct, disable, or delete
```

### 22.3 Memory controls

Provide appropriate user controls:
- View stored memories.
- Correct or delete individual entries.
- Disable memory.
- Delete conversation history.
- Understand the difference between memory and source records.
- Request export/deletion where applicable.

Deletion must propagate to relevant derived indexes and caches according to the retention design. Define whether a conversation deletion also deletes memories derived from it.

## 23. Retrieval-Augmented Generation (RAG)

Use RAG when the application needs to ground responses in private documents, a knowledge base, or external sources. It is not necessary for every assistant.

### 23.1 Ingestion

1. Validate upload and ownership.
2. Scan and parse in a restricted worker.
3. Extract text and metadata.
4. Chunk with source offsets.
5. Generate embeddings if configured.
6. Store chunks and index metadata.
7. Mark document ready or failed.
8. Expose it only to authorized retrieval queries.

### 23.2 Retrieval

1. Resolve identity and permitted data scope.
2. Search by keyword, vector, or hybrid methods.
3. Rank and deduplicate candidates.
4. Filter to the current context budget.
5. Preserve source references.
6. Label passages as untrusted data.
7. Return source identifiers for citations where supported.

### 23.3 Retrieval evaluation

Measure retrieval recall, relevance, source correctness, citation accuracy, latency, and cost. Test cross-user isolation. A high similarity score is not proof of correctness or authorization.

## 24. File and Artifact Management

Treat files as controlled resources, not arbitrary strings or public URLs.

Store:
- File ID and owner.
- Object storage key.
- MIME type and verified format.
- Size and checksum.
- Processing/scan status.
- Source conversation or task.
- Retention and deletion metadata.
- Access policy.

Use private object storage and short-lived signed URLs. Validate file type and size, handle malformed content safely, and do not execute active content from uploads. Keep large binary data out of the relational database unless there is a deliberate reason.

---

# Part V — Persistence and APIs

## 25. Logical Data Model

The following entities are a reference model, not a requirement to create every table. Adapt them to the product's actual requirements.

### 25.1 Core entities

**users / tenants**
- Identity and status.
- Tenant or organization membership.
- Plan/entitlements if needed.
- Creation and lifecycle metadata.

**conversations**
- Owner/tenant.
- Title and status.
- Created/updated/archived/deleted timestamps.

**messages**
- Conversation and turn identifiers.
- Sequence/order.
- Role and structured content.
- Status and timestamps.
- Safe metadata.

**turns**
- Owner and conversation.
- Status.
- Idempotency key.
- Start/end/deadline.
- Error category.
- Usage and configuration references.

**model_calls**
- Turn.
- Provider/model.
- Status and duration.
- Usage and cost estimates.
- Error category.

**tool_invocations**
- Turn and tool.
- Status.
- Redacted input/output references.
- Idempotency key.
- Start/end.
- Error category.

**tasks**
- Owner, conversation, or project.
- Status/progress.
- Deadline.
- Result reference.
- Error and cancellation metadata.

**approvals**
- Actor and target action.
- Exact action digest/parameters reference.
- Status and expiry.
- Decision timestamp.

**files / artifacts**
- Owner and storage reference.
- Type, size, checksum, scan status.
- Processing status and retention.

**memories**
- Owner, category, content/reference.
- Provenance.
- Created/updated/expiry/deletion timestamps.

**documents / chunks**
- Source file.
- Chunk content and offsets.
- Embedding/index metadata.
- Ownership and retention.

**prompt_versions / model_configurations**
- Versioned configuration and rollout metadata.

**audit_events**
- Actor, action, resource, outcome, timestamp, correlation ID, safe metadata.

### 25.2 Constraints and indexes

- Foreign keys for relationships.
- Unique message ordering within a conversation.
- Scoped idempotency constraints.
- Index common owner/status/time queries.
- Index active turns/tasks by status and deadline.
- Enforce ownership in all data-access paths.
- Consider row-level security as an additional layer, not a replacement for authorization.
- Use transactions for related state changes.
- Define retention, archival, and deletion.
- Avoid secrets and unnecessary private content in audit logs.

## 26. API Design

Use versioned APIs and stable contracts. REST plus SSE is a reasonable starting point for many interactive applications; use other transports when product requirements justify them.

### 26.1 Illustrative endpoint groups

**Conversation products**
- `POST /v1/conversations`
- `GET /v1/conversations`
- `GET /v1/conversations/{id}`
- `PATCH /v1/conversations/{id}`
- `DELETE /v1/conversations/{id}`

**Turns**
- `POST /v1/conversations/{id}/turns`
- `GET /v1/turns/{id}`
- `POST /v1/turns/{id}/cancel`
- `GET /v1/turns/{id}/events`

**Files**
- `POST /v1/files/upload-intents`
- `POST /v1/files/{id}/complete`
- `GET /v1/files/{id}`
- `DELETE /v1/files/{id}`

**Tasks**
- `GET /v1/tasks/{id}`
- `POST /v1/tasks/{id}/cancel`

**Approvals**
- `GET /v1/approvals/{id}`
- `POST /v1/approvals/{id}/decision`

**Memory, if enabled**
- `GET /v1/memories`
- `PATCH /v1/memories/{id}`
- `DELETE /v1/memories/{id}`
- `PATCH /v1/settings/memory`

Not every application needs conversations, memory, or approvals. Omit irrelevant endpoints rather than implementing empty abstractions.

### 26.2 Request rules

- Authenticate every private request.
- Derive identity from verified credentials.
- Validate body, query, and path parameters.
- Apply size limits and rate limits.
- Do not accept arbitrary system prompts, provider credentials, or tool scopes from untrusted clients.
- Use idempotency keys for retryable creation operations.
- Verify ownership for every referenced resource.

### 26.3 Error contract

```json
{
  "error": {
    "code": "REQUEST_LIMIT_EXCEEDED",
    "message": "The request could not be completed within the configured limits.",
    "requestId": "req_example",
    "retryable": false
  }
}
```

Do not expose stack traces, secrets, hidden instructions, or internal tool outputs to clients.

## 27. Streaming and Event Contracts

SSE is a useful starting point for server-to-client incremental updates. WebSockets may be appropriate for bidirectional realtime interactions.

### 27.1 Suggested event types

- `request.accepted`
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

### 27.2 Event contract

Each event should have:
- Event ID where replay/deduplication is supported.
- Event type.
- Turn/task/message identifier.
- Timestamp.
- Validated payload.
- Correlation ID.

Example:

```text
event: message.delta
id: evt_123
data: {"turnId":"turn_123","messageId":"msg_123","delta":"Hello"}

```

### 27.3 Delivery semantics

Do not assume exactly-once delivery. Design for reconnection, duplicates, and lost connections. If replay is supported, define event retention and cursor semantics. The client should reconcile with canonical server state after reconnecting.

## 28. Idempotency and Transactions

Idempotency is essential for reliable retries.

- Scope idempotency keys to a user/tenant and operation.
- Store the outcome associated with the key.
- Reject reuse with incompatible payloads.
- Use unique constraints to resolve concurrent duplicate requests.
- Make background jobs idempotent.
- Use operation IDs for external side effects.
- Reconcile unknown outcomes before repeating non-idempotent actions.

Use transactions for related database changes, but do not hold long database transactions open across model calls or external network requests.

---

# Part VI — Security, Privacy, and Reliability

## 29. Security Model

### 29.1 Authentication and authorization

- Authenticate all private endpoints.
- Verify resource ownership on every read/write.
- Use least-privilege service identities.
- Separate administrative permissions.
- Protect expensive endpoints with quotas and rate limits.
- Never trust identity or scope claims that the client can freely edit.

### 29.2 Prompt injection

- Treat user content, files, webpages, and tool results as untrusted.
- Separate trusted policy from untrusted context.
- Restrict available tools by task and user permissions.
- Enforce authorization in code.
- Validate tool arguments and results.
- Test direct and indirect injection scenarios.

No prompt can guarantee that a model will ignore all adversarial instructions. Security controls must remain effective even when the model is manipulated.

### 29.3 Secrets

- Keep model-provider keys on the server.
- Use a managed secrets system or secure deployment configuration.
- Rotate credentials and scope them narrowly.
- Redact credentials from logs and error reports.
- Do not pass secrets into model context unless strictly required and approved.

### 29.4 Data protection

- Encrypt traffic and stored data.
- Minimize personal data collection.
- Define retention and deletion.
- Restrict operational access.
- Redact telemetry.
- Separate environments and credentials.
- Document model-provider data handling and retention.
- Consider jurisdictional and contractual requirements for sensitive data.

### 29.5 Upload safety

- Enforce size and type limits.
- Verify actual content rather than trusting filenames.
- Scan where appropriate.
- Parse in an isolated worker.
- Defend against decompression bombs and malformed files.
- Prevent cross-user file access.
- Use private storage and short-lived URLs.

### 29.6 Audit

Record security-relevant actions: permission decisions, approvals, external actions, file access where appropriate, administrative changes, and deletion requests. Audit records should be tamper-resistant enough for the product's risk level and should not contain unnecessary sensitive payloads.

## 30. Reliability and Failure Handling

### 30.1 Failure categories

- Invalid request.
- Authentication/authorization failure.
- Provider transient error.
- Provider permanent error.
- Rate limit.
- Tool validation or execution error.
- Timeout.
- Cancellation.
- Storage/queue failure.
- Unknown side-effect outcome.

### 30.2 Retry policy

- Use bounded backoff with jitter for transient errors.
- Respect provider retry guidance.
- Do not retry invalid input unchanged.
- Avoid retry storms.
- Use circuit breakers or concurrency limits where helpful.
- Do not repeat non-idempotent operations without reconciliation.

### 30.3 Recovery

On restart, find stale active turns/tasks. Check lease ownership, recorded events, and side-effect status. Resume only when safe; otherwise retry an idempotent step, reconcile, or move to `RECOVERY_REQUIRED`.

### 30.4 Graceful degradation

When a dependency fails:
- Communicate the failure accurately.
- Preserve safe completed work.
- Offer retry where appropriate.
- Use a compatible fallback only if allowed.
- Avoid falsely reporting success.
- Keep the client able to inspect current task status.

## 31. Cancellation and Timeouts

Cancellation should be cooperative and state-aware.

- Accept cancellation requests only from authorized actors.
- Propagate cancellation to providers and workers when supported.
- Stop scheduling new tool calls after cancellation.
- Allow safe cleanup.
- Record whether a side effect may already have occurred.
- Distinguish cancellation requested from cancellation completed if necessary.
- Enforce deadlines even if a provider ignores cancellation.

## 32. Privacy and Data Lifecycle

Define lifecycle behavior for:
- Conversations and messages.
- Memories.
- Uploaded files and extracted text.
- Embeddings and indexes.
- Generated artifacts.
- Tool logs.
- Model request/response metadata.
- Evaluation and analytics data.
- Backups.

A deletion workflow should identify derived data, indexes, caches, and copies. Define realistic backup expiry and deletion guarantees. Do not promise immediate deletion from systems where retention behavior is controlled by a provider unless that guarantee is verified.

---

# Part VII — Operations, Evaluation, and Delivery

## 33. Observability

### 33.1 Trace model

```text
Trace: request / turn
  ├── authentication and validation
  ├── context assembly
  │    ├── history retrieval
  │    ├── memory retrieval
  │    └── document retrieval
  ├── model call #1
  ├── tool invocation #1
  ├── model call #2
  ├── output validation
  └── event delivery
```

### 33.2 Metrics

Track:
- API volume, availability, and errors.
- Time to first token.
- End-to-end turn latency.
- Model-call latency and failure rate.
- Tool success rate and duration.
- Queue depth and task age.
- Token usage and estimated cost.
- Cost per successful turn/task.
- Context size and truncation.
- Cancellation and timeout rates.
- Retrieval and citation quality.
- Safety incidents and authorization failures.
- Evaluation results and user feedback signals.

### 33.3 Logging

Use structured logs and correlation IDs. Avoid full prompts, private documents, raw personal data, and credentials by default. Diagnostic payload capture should be narrowly scoped, access-controlled, and governed by retention policy.

### 33.4 Alerts

Create alerts for provider failure spikes, queue backlog, latency regressions, unusual cost, elevated authorization denials, worker crashes, and data-store health. Tune thresholds to real baselines rather than arbitrary values.

## 34. Cost and Capacity Management

- Set per-user and per-plan quotas.
- Set per-turn and per-task cost ceilings.
- Limit output tokens and tool iterations.
- Budget context.
- Cache only when correctness, freshness, and privacy allow it.
- Route to lower-cost models only when evaluation supports acceptable quality.
- Monitor provider spend and anomalies.
- Bound concurrency.
- Use backpressure when workers or providers are saturated.
- Establish clear policies for paid, free, and internal workloads if relevant.

Do not set production limits without measuring representative workloads.

## 35. Evaluation and Testing Strategy

### 35.1 Test layers

1. Unit tests for state transitions, schemas, permissions, budgeting, and retry rules.
2. Integration tests for databases, queues, storage, and model adapters.
3. Contract tests for provider events and tool schemas.
4. End-to-end tests for complete requests and streaming.
5. Security tests for cross-user access, prompt injection, and tool authorization.
6. AI evaluations for correctness, relevance, tool selection, and groundedness.
7. Load tests for concurrency, queue pressure, and provider limits.
8. Operational drills for cancellation, provider outage, and recovery.

### 35.2 Evaluation dataset

Include cases for:
- Basic requests.
- Multi-turn continuity.
- Ambiguity and missing information.
- Structured output.
- Tool choice and malformed arguments.
- Tool failure and timeout.
- Prompt injection in files/webpages.
- Unauthorized resource access.
- Long-context summarization.
- Memory retrieval and deletion.
- Provider outages and fallback.
- Cancellation and duplicate requests.
- Multimodal cases when supported.

### 35.3 Release gates

Before deploying a prompt, model, tool, or retrieval change:
- Run relevant automated tests.
- Compare evaluation scores with a baseline.
- Review safety regressions.
- Check latency and cost.
- Use a staged rollout when practical.
- Keep rollback available.
- Record the version/configuration used by each execution.

## 36. Deployment and Scaling

### 36.1 Initial deployment

```text
Clients
  │
TLS / API gateway
  │
Stateless API instances
  ├── SQL database
  ├── Cache (optional)
  ├── Private object storage
  ├── Durable queue (if needed)
  └── External model/service APIs

Workers (as needed)
  ├── Document ingestion
  ├── Embeddings/indexing
  └── Long-running workflows

Telemetry
  ├── Logs
  ├── Metrics
  └── Traces
```

### 36.2 Environments

Use separate development, staging, and production environments with separate credentials and data. Do not copy production data into development without an approved sanitization process.

### 36.3 Scaling

Start by scaling stateless API instances horizontally. Scale workers based on queue depth and job latency. Monitor database connections and query performance. Split a module into a service when measurements or ownership boundaries justify it.

### 36.4 Migrations

Use versioned schema migrations. Test with representative data. Prefer backward-compatible changes during rolling deployments. Define rollback or forward-fix procedures and backup/restore expectations.

## 37. Configuration and Feature Flags

Separate:
- Deployment configuration.
- Secrets.
- Versioned prompts and policies.
- Model routing configuration.
- Per-user or per-plan limits.
- Feature flags.

Validate configuration at startup. Record relevant versions per turn. Feature flags must not bypass authentication, authorization, or other mandatory controls.

## 38. Client Application Guidance

Client-specific implementation varies, but the client should generally:
- Render streamed updates incrementally.
- Track local optimistic state separately from confirmed server state.
- Support reconnection and reconciliation.
- Display tool/task progress and approval requests clearly.
- Upload files through authorized flows.
- Handle cancellation and errors.
- Avoid storing provider secrets.
- Use secure local storage for sensitive session credentials.
- Never rely on client-side checks as the security boundary.

Keep UI state models separate from provider-specific payload formats.

---

# Part VIII — Implementation and Governance

## 39. Suggested Repository Structure

Adapt to the language and framework. This is a logical example for a TypeScript backend.

```text
ai-application/
├── apps/
│   ├── api/
│   ├── workers/
│   └── admin/                 # optional
├── packages/
│   ├── domain/
│   ├── api-contracts/
│   ├── ai-runtime/
│   │   ├── turn-manager/
│   │   ├── context/
│   │   ├── model-router/
│   │   ├── providers/
│   │   ├── tools/
│   │   ├── prompts/
│   │   ├── policies/
│   │   └── events/
│   ├── product-modules/
│   ├── infrastructure/
│   │   ├── database/
│   │   ├── queue/
│   │   ├── storage/
│   │   └── telemetry/
│   └── shared/
├── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── security/
│   ├── load/
│   └── evaluations/
├── docs/
│   ├── architecture/
│   ├── runbooks/
│   └── decisions/
└── README.md
```

Do not create every folder on day one. Introduce modules when they own a clear responsibility and contract.

## 40. Implementation Roadmap

### Phase 0 — Discovery
- Complete the application profile.
- Identify data sensitivity and access boundaries.
- Select initial provider and backend stack.
- Define launch features and success metrics.
- Document retention, budget, and latency expectations.

**Exit:** Approved product and architecture decisions.

### Phase 1 — Reliable core
- Authentication and ownership checks.
- API contracts.
- Conversation/task persistence as required.
- Provider adapter.
- Turn lifecycle and bounded execution.
- Basic streaming.
- Idempotency, cancellation, and structured errors.
- Initial tracing.

**Exit:** Reliable end-to-end request handling.

### Phase 2 — Controlled tools
- Tool registry and schemas.
- Permission policy.
- Execution limits.
- Audit events.
- Approval flow if needed.
- Security and contract tests.

**Exit:** Tools operate within verified boundaries.

### Phase 3 — Context and retrieval
- Context assembler and budgeter.
- History retrieval and summaries.
- RAG/document ingestion if needed.
- Memory only if product value warrants it.
- Retrieval and isolation evaluations.

**Exit:** Relevant context can be included without unauthorized access.

### Phase 4 — Durability and operations
- Durable queue and workers if needed.
- Recovery and reconciliation.
- Dashboards and alerts.
- Cost controls.
- Retention and deletion workflows.
- Load and failure testing.

**Exit:** Defined operational objectives and recovery procedures.

### Phase 5 — Advanced features
- Additional modalities.
- Provider routing/fallback.
- External integrations.
- Sandboxed execution.
- Specialist agents or delegated workflows.
- Advanced personalization.

**Exit:** Each feature demonstrates measurable value and passes its security/evaluation gates.

## 41. Architecture Decision Records (ADRs)

Record important decisions in short, version-controlled documents.

Each ADR should include:
- Title and status.
- Context/problem.
- Options considered.
- Decision.
- Consequences and trade-offs.
- Security and operational implications.
- Date and owners.

Suggested initial ADRs:
1. Backend language/framework.
2. Initial model provider and adapter contract.
3. SQL database and data model.
4. Streaming transport.
5. Queue and worker strategy.
6. Memory and retention policy.
7. Tool permission and approval model.
8. Deployment environment.
9. Logging and privacy policy.

## 42. Service Objectives

Define measurable targets after collecting workload data. Potential objectives:
- API availability.
- Time to first token.
- Interactive request completion latency.
- Successful task completion rate.
- Tool execution success rate.
- Background task deadline success.
- Data durability and recovery point.
- Recovery time after dependency failure.

Separate simple interactions from workflows that depend on external systems. Do not promise one latency target for every task type.

## 43. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Prompt injection | Untrusted-content handling and code-enforced permissions |
| Cross-user data leakage | Ownership filters, tenant isolation, adversarial tests |
| Infinite tool loops | Hard call, time, token, and cost limits |
| Duplicate side effects | Idempotency, operation records, reconciliation |
| Provider outage | Bounded retries and compatible fallback |
| Excessive context/cost | Context budgets, retrieval ranking, spend limits |
| Incorrect memory | Provenance, user controls, expiry, deletion |
| Malicious files | Validation, scanning, isolated parsing |
| Queue backlog | Backpressure, scaling, queue alerts |
| Secret leakage | Secret manager, redaction, least privilege |
| Provider incompatibility | Capability registry and contract tests |
| Unsafe actions | Risk tiers, approvals, sandboxing |
| Prompt regression | Versioning and automated evaluations |
| Premature complexity | Modular monolith first; split by evidence |
| Unclear ownership | Module contracts and architecture decision records |
| Incomplete deletion | Data lineage, index/cache cleanup, retention tests |

## 44. Production Readiness Checklist

### Product and design
- [ ] Application profile completed.
- [ ] Required capabilities selected.
- [ ] Data sensitivity classified.
- [ ] Latency, cost, and availability goals defined.
- [ ] Provider capabilities and data terms verified.

### Runtime
- [ ] Turn lifecycle and terminal states implemented.
- [ ] Model calls and tool invocations tracked separately.
- [ ] Time, token, cost, and iteration limits enforced.
- [ ] Cancellation behavior tested.
- [ ] Duplicate request handling implemented.
- [ ] Recovery behavior defined.

### Security
- [ ] Authentication and resource ownership enforced server-side.
- [ ] Tool permissions validated independently of model output.
- [ ] Secrets kept out of clients and logs.
- [ ] Untrusted content is isolated from trusted instructions.
- [ ] File processing is constrained.
- [ ] Consequential actions have appropriate approval.
- [ ] Cross-user isolation tests pass.

### Data and privacy
- [ ] Retention policies documented.
- [ ] Deletion covers relevant derived data.
- [ ] Logs are minimized and redacted.
- [ ] Backups and recovery are tested.
- [ ] Provider data handling reviewed.

### Operations
- [ ] Traces, metrics, and alerts available.
- [ ] Provider outage and queue failure tested.
- [ ] Cost ceilings and quotas enforced.
- [ ] Migrations and rollback procedures documented.
- [ ] Incident and recovery runbooks exist.

### Quality
- [ ] Unit and integration tests pass.
- [ ] Tool contracts tested.
- [ ] Prompt/model evaluations run.
- [ ] Security and load tests run.
- [ ] Staged rollout and rollback path available.

## 45. Official Documentation and Further Reading

Verify current API behavior, SDK versions, capabilities, pricing, and data handling before implementation. These links are starting points, not substitutes for product-specific review.

- OpenAI API documentation: https://developers.openai.com/api/docs/
- OpenAI Responses API reference: https://developers.openai.com/api/reference/responses/overview
- OpenAI function calling: https://developers.openai.com/api/docs/guides/function-calling
- OpenAI Agents SDK: https://developers.openai.com/api/docs/guides/agents/sdk
- OpenAI prompt engineering: https://developers.openai.com/api/docs/guides/prompt-engineering
- Flutter documentation: https://docs.flutter.dev/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- OpenTelemetry documentation: https://opentelemetry.io/docs/

Use equivalent official documentation for the selected model provider, framework, database, queue, cloud, and identity provider.

## 46. Final Recommendation

Use this document as a **reusable architecture baseline**, not a rigid template.

The most transferable foundation is:
1. Clear separation between product logic and AI runtime.
2. A bounded, observable turn lifecycle.
3. Provider adapters and capability-aware routing.
4. Controlled tools with server-side authorization.
5. Explicit context assembly and data ownership.
6. Durable state and safe retry/recovery behavior.
7. User-appropriate privacy, retention, and deletion.
8. Automated evaluations and operational monitoring.

For a small application, implement only the minimum subset that provides real value. For a high-risk or high-scale application, add the stronger isolation, approval, audit, and recovery controls required by its threat model and service objectives.

**Architecture status:** Reusable reference baseline. Complete the application profile and validate all assumptions before production implementation.
