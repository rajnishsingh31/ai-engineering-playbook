# Chapter 16 — State Management

## Memory answers:

What should the assistant remember?

## State answers:

Where is the system right now?

**State management becomes critical when an AI application performs multi-step work rather than answering a single question.**

A Simple Stateless Request

Consider:

Explain BM25.

The application sends the question to the LLM and returns the response.

```text
Request
   ↓
LLM
   ↓
Response
```

Once the response is returned, the application does not need to preserve much operational information.

This is largely stateless.

A Stateful Workflow

Now consider:

Compare three uploaded companies, calculate revenue growth, identify the healthiest company, and email me the report.

This requires several steps:

```text
Upload documents
      ↓
Extract content
      ↓
Identify companies
      ↓
Extract metrics
      ↓
Validate values
      ↓
Compare companies
      ↓
Generate report
      ↓
Request approval
      ↓
Send email
```

Suppose the system fails after extracting two companies.

Should it restart everything?

No.

It should know:

which files were processed,
which extraction steps succeeded,
which company remains,
what intermediate results exist,
whether an email was already sent.

That information is workflow state.

## State vs Memory

These concepts overlap, but they serve different purposes.

### Memory

Used to improve future interactions.

User prefers Python examples.

## State

Used to continue or control an active system operation.

Company A extraction completed.
Company B extraction completed.
Company C extraction pending.

A useful distinction:

Memory
→ personalization and historical knowledge

State
→ operational progress and control
Major Types of State

## We can classify state into five useful categories:

Conversation state
Task state
Workflow state
Tool execution state
Application state

### 1. Conversation State

Conversation state helps interpret the current dialogue.

Example:

Current topic:
Tool Executor design

Current user intent:
Continue learning

Last question asked:
Design the Tool Executor

It may also contain:

recent messages,
current entities,
references such as “it” or “that design,”
unresolved questions,
current response mode.

Conversation state belongs primarily to the Conversation Manager.

### 2. Task State

Task state describes the current user task.

Example:

{
  "task_id": "task-481",
  "task_type": "compare_companies",
  "companies": [
    "Company A",
    "Company B",
    "Company C"
  ],
  "requested_metrics": [
    "revenue",
    "ARR",
    "growth_rate"
  ],
  "status": "in_progress"
}

This is more structured than general conversation history.

The task state answers:

What is being attempted?
What inputs belong to it?
What output is expected?
Is it complete?

### 3. Workflow State

Workflow state tracks progress through multiple steps.

{
  "workflow_id": "wf-903",
  "current_step": "extract_company_c",
  "completed_steps": [
    "extract_company_a",
    "extract_company_b"
  ],
  "pending_steps": [
    "extract_company_c",
    "compare_metrics",
    "generate_report"
  ],
  "status": "running"
}

This allows the application to resume after interruption.

### 4. Tool Execution State

Tool execution state tracks individual external operations.

Example:

{
  "tool_call_id": "tool-712",
  "tool": "extract_pdf_tables",
  "status": "running",
  "attempt": 2,
  "started_at": "2026-07-30T23:10:00+05:30",
  "idempotency_key": "wf-903-company-c-extraction"
}

Later:

{
  "tool_call_id": "tool-712",
  "status": "succeeded",
  "result_reference": "result-1882"
}

This is important for retries, auditing, and duplicate prevention.

### 5. Application State

Application state includes broader system-level information.

Examples:

authenticated user,
tenant,
selected repository,
active document collection,
feature flags,
subscription tier,
rate-limit usage.

This state may influence what workflows and tools are available.

## State Machine Thinking

A strong way to design workflows is as a state machine.

Suppose an email must be generated and approved before sending.

Possible states:

Drafting
   ↓
Awaiting Approval
   ↓
Approved
   ↓
Sending
   ↓
Sent

Failure states may include:

Draft Failed
Approval Rejected
Send Failed
Cancelled

A transition is allowed only when its conditions are satisfied.

For example:

Awaiting Approval
        ↓
User approves exact draft
        ↓
Approved

The system should not jump directly from Drafting to Sent.

State Transition Example
Created
   ↓
Extracting Documents
   ↓
Validating Metrics
   ↓
Comparing Companies
   ↓
Generating Report
   ↓
Awaiting Approval
   ↓
Sending
   ↓
Completed

Each transition should define:

preconditions,
action,
resulting state,
failure behavior,
retry policy.

This is conventional workflow engineering applied to AI systems.

Why Not Let the LLM Track State?

You could repeatedly send the full history and ask:

What step are we currently on?

That is dangerous.

The model might:

forget a completed step,
repeat a write operation,
misread an error,
claim progress that never occurred,
interpret ambiguous text incorrectly.

**Operational state should be stored deterministically.**

**The LLM may suggest the next step, but the workflow engine should know the authoritative current state.**

Authoritative State

Suppose the LLM says:

The report was emailed successfully.

But the email tool timed out before confirming delivery.

What is the truth?

Not the LLM statement.

The authoritative source is the tool result or email provider.

LLM claim
≠
Authoritative state

A production system must distinguish:

Proposed state
Observed state
Committed state

Example:

Proposed:
Send email

Observed:
Email API returned success

Committed:
Workflow state updated to sent

## Checkpoints

A checkpoint is a durable snapshot of workflow progress.

Example:

{
  "workflow_id": "wf-903",
  "checkpoint_version": 4,
  "current_step": "compare_metrics",
  "completed_outputs": {
    "company_a_metrics": "result-101",
    "company_b_metrics": "result-102",
    "company_c_metrics": "result-103"
  },
  "created_at": "2026-07-30T23:15:00+05:30"
}

If the service crashes, it can restart from this checkpoint.

Without checkpoints:

Crash
  ↓
Restart entire workflow

With checkpoints:

Crash
  ↓
Load latest checkpoint
  ↓
Resume from unfinished step
When Should You Checkpoint?

Not after every tiny operation.

That could be expensive and noisy.

Useful checkpoint boundaries include:

after expensive document extraction,
after a successful external write,
after human approval,
after completing a workflow stage,
before a risky action.

Think of checkpointing as a trade-off:

More checkpoints
→ better recoverability
→ more storage and coordination cost

Fewer checkpoints
→ simpler
→ more repeated work after failure
Resumability

A resumable workflow can continue after:

process crash,
timeout,
service restart,
user disconnect,
external dependency outage,
human approval delay.

Example:

Generate deployment plan
        ↓
Wait for manager approval for 8 hours
        ↓
Resume deployment workflow

The process should not hold a server thread or LLM session open for eight hours.

Instead:

Persist state
   ↓
Stop execution
   ↓
Approval event arrives
   ↓
Reload state
   ↓
Continue

This is especially important for long-running agents.

## Idempotency

Idempotency means repeating an operation does not create unintended duplicate effects.

Consider:

Send email

The email API accepts the request, but the network response is lost.

The system does not know whether it succeeded.

It retries.

Without idempotency:

Recipient receives two emails

With an idempotency key:

{
  "idempotency_key": "wf-903-send-final-report"
}

The email service or Tool Executor recognizes that the operation was already processed.

Result:

Same logical action
executed at most once
Exactly Once Is Difficult

Distributed systems rarely provide true exactly-once execution across arbitrary services.

## A more realistic design combines:

at-least-once delivery,
idempotent handlers,
deduplication,
durable state,
reconciliation.

Example:

Message may arrive more than once
        ↓
Consumer checks operation ID
        ↓
Already completed?
   ┌────┴────┐
   ▼         ▼
 Yes         No
Return     Execute
stored     and save
result     result

This should be familiar from distributed systems.

State and Retries

Suppose extraction fails transiently.

The state might be:

{
  "step": "extract_company_c",
  "status": "failed",
  "attempts": 1,
  "last_error": {
    "code": "OCR_TIMEOUT",
    "retryable": true
  }
}

The workflow engine decides whether to retry.

After retry:

{
  "step": "extract_company_c",
  "status": "succeeded",
  "attempts": 2,
  "result_reference": "result-103"
}

The LLM should not be the authoritative retry counter.

State and Human Approval

Human approval is also state.

Example:

{
  "approval_id": "approval-701",
  "action": "send_email",
  "resource": "financial-comparison-report",
  "status": "pending",
  "requested_by": "user-123",
  "expires_at": "2026-07-31T23:00:00+05:30"
}

After approval:

{
  "status": "approved",
  "approved_by": "user-123",
  "approved_at": "2026-07-30T23:20:00+05:30"
}

The approval must be bound to:

exact action,
exact content,
exact resource,
exact user,
expiration time.

If the email content changes after approval, the old approval may no longer be valid.

## State Consistency

Suppose two workers process the same workflow simultaneously.

Both read:

Current step:
send_email

Both execute.

Now two emails are sent.

To prevent this, use concurrency controls such as:

optimistic concurrency,
row versions,
distributed locks,
leases,
compare-and-swap,
idempotency keys.

Example:

Worker A reads version 7
Worker B reads version 7

Worker A updates version 7 → 8
Worker B tries version 7 → conflict

Worker B must reload rather than executing stale work.

Optimistic Concurrency

State record:

{
  "workflow_id": "wf-903",
  "status": "awaiting_approval",
  "version": 7
}

Update request:

Update only if version = 7

If another worker already changed it to version 8, the update fails.

This is often simpler than locking the workflow for its entire lifetime.

## Event-Driven State

Instead of one service controlling everything synchronously, workflows can respond to events.

DocumentUploaded
      ↓
ExtractionCompleted
      ↓
MetricsValidated
      ↓
ComparisonCompleted
      ↓
ReportApproved
      ↓
EmailSent

Each event advances state.

This is useful when:

operations are long-running,
multiple services participate,
failures are expected,
human approval is asynchronous.

### Event History vs Current State

There are two related storage patterns.

#### Current-state storage

Store the latest state:

{
  "workflow_id": "wf-903",
  "status": "awaiting_approval"
}

Easy to query.

#### Event history

Store every transition:

WorkflowCreated
DocumentExtractionStarted
DocumentExtractionCompleted
ReportGenerated
ApprovalRequested

Useful for:

audits,
debugging,
reconstruction,
analytics.

Many systems store both:

Event log
+
Current materialized state

### State Persistence Options

Different state belongs in different stores.

Recent conversation state
→ cache or conversation database

Workflow state
→ durable relational or NoSQL database

Large tool outputs
→ object storage

Events
→ event stream or append-only log

Searchable memories
→ vector and metadata stores

Avoid placing everything into a single vector database.

Vector databases are good for similarity retrieval, not authoritative workflow transitions.

State Schema Example
{
  "workflow_id": "wf-903",
  "workflow_type": "financial_comparison",
  "tenant_id": "tenant-42",
  "user_id": "user-123",
  "status": "running",
  "current_step": "compare_metrics",
  "version": 5,
  "inputs": {
    "document_ids": [
      "doc-a",
      "doc-b",
      "doc-c"
    ]
  },
  "step_results": {
    "extract_doc_a": "result-101",
    "extract_doc_b": "result-102",
    "extract_doc_c": "result-103"
  },
  "pending_approval": null,
  "created_at": "2026-07-30T22:40:00+05:30",
  "updated_at": "2026-07-30T23:15:00+05:30"
}

Notice that large extracted data is referenced rather than copied directly into the state record.

Keep State Small

Workflow state should contain:

identifiers,
statuses,
references,
timestamps,
decisions,
versions.

It should not necessarily contain:

entire PDFs,
huge prompts,
full model responses,
massive extracted tables.

Store large objects separately and refer to them.

Workflow state
→ result reference

Object storage
→ actual result
Deterministic vs Probabilistic State

Some state comes from deterministic systems:

Email sent: true
Tool returned HTTP 200
Approval received

Other state is model-derived:

User intent classified as financial comparison
Document appears to contain ARR
Confidence = 0.82

Do not treat these equally.

Model-derived state should often include:

confidence,
model version,
prompt version,
supporting evidence,
validation status.

Example:

{
  "classification": "financial_report",
  "confidence": 0.82,
  "validated": false,
  "model_version": "model-x",
  "prompt_version": "classifier-v3"
}

## State Reconciliation

Suppose your workflow state says:

Email status:
sending

But the service crashed.

Later, you need to determine whether the email was sent.

The reconciliation process might:

query the email provider using an operation ID,
compare external state with local state,
update local state,
retry only when safe.
Local state
+
External authoritative state
        ↓
Reconciled state

This is essential for uncertain outcomes.

## State Expiration and Cleanup

Not all state should live forever.

Examples:

Conversation state
→ expire after inactivity

Temporary upload state
→ delete after workflow completion

Approval request
→ expire after configured deadline

Completed workflow
→ archive based on retention policy

Cleanup must consider:

audit requirements,
data retention,
privacy,
troubleshooting needs,
legal holds.

## Architect Perspective

A robust AI workflow should look like this:

```text

User Request
     │
     ▼
Conversation Manager
     │
     ▼
Task Creator
     │
     ▼
Workflow Engine
     │
     ├── State Store
     ├── Checkpoint Manager
     ├── Tool Executor
     ├── Approval Service
     └── Event Log
              │
              ▼
             LLM
```

### The LLM participates in selected steps, such as:

classification,
planning,
extraction,
summarization,
evaluation.

But the workflow engine controls:

current state,
allowed transitions,
retries,
completion,
cancellation,
resumability.
Example: Financial Document Workflow

```text

Workflow Created
       ↓
Documents Registered
       ↓
Extraction Started
       ↓
 ┌─────┼─────┐
 ▼     ▼     ▼
A      B     C
       ↓
Extraction Results Persisted
       ↓
Metrics Normalized
       ↓
Business Validation
       ↓
Cross-Company Comparison
       ↓
Report Generated
       ↓
Awaiting User Approval
       ↓
Email Sent
       ↓
Workflow Completed

```

**At every stage, authoritative state is stored outside the LLM.**

## Engineer Perspective

A workflow record could be represented conceptually as:

class WorkflowState:
    workflow_id: str
    status: str
    current_step: str
    version: int
    completed_steps: list[str]
    failed_steps: list[str]
    result_references: dict[str, str]

Every operation should:

read the current state,
verify the transition is allowed,
execute the operation,
persist its result,
atomically update the state,
emit an event.
Manager Perspective

Good state management improves:

reliability,
recoverability,
auditability,
customer trust,
operational support,
cost control.

Without it, incidents become difficult to diagnose:

Did the LLM call the tool?

Did the tool succeed?

Was the result persisted?

Did the workflow retry?

Was the email sent twice?

State and event history provide concrete answers.

## Core Principles

The LLM is not the state store.

Conversation history is not workflow state.

A tool request is not a completed action.

A timeout does not prove failure.

Retries require idempotency.

Long-running work requires checkpoints.

State transitions must be deterministic.

External systems remain authoritative for their actions.

## Exercise:

Cover:

the workflow state store,
checkpoints,
events,
concurrency control,
tool execution records,
approval state,
result storage,
retries,
idempotency,
reconciliation,
retention.

Focus on component responsibilities and the boundaries between the LLM, Workflow Engine, Tool Executor, and external systems.

## Resumable workflow architecture

Your architecture is very good. You identified many of the right components:

lightweight orchestrator,
independently scalable task processors,
durable workflow and task state,
asynchronous operations,
per-task idempotency,
result reuse,
event-driven completion,
task-specific timeouts and retry behavior,
watchdog monitoring,
LLM reasoning as an ordinary task.

That last point is particularly important:

An LLM invocation should be modeled as a task with inputs, output schema, timeout, status, model metadata, and retry policy—not as a magical special operation.

Recommended architecture

```text

                         User Request
                              │
                              ▼
                       Workflow API
                              │
                              ▼
                    Durable Orchestrator
                              │
             ┌────────────────┼─────────────────┐
             ▼                ▼                 ▼
       Workflow Store     Event Broker     Timer Service
             │                │                 │
             │                ▼                 │
             │       Independently Scaled       │
             │          Task Workers            │
             │                │                 │
             │                ▼                 │
             │          Tool Executor           │
             │                │                 │
             │                ▼                 │
             │        External Systems          │
             │                                  │
             └──────────── Result Store ────────┘
                              │
                              ▼
                     Approval Service

```

### 1. Central orchestrator

Your “lightweight central orchestrator” is the right idea.

It should own:

workflow definition,
allowed state transitions,
dependency resolution,
fan-out and fan-in,
task scheduling,
workflow-level timeout,
cancellation,
compensation policy,
terminal workflow state.

It should not perform:

PDF extraction,
LLM inference,
email delivery,
large calculations,
long blocking operations.

Those belong in workers.

The orchestrator should replay or reload deterministic state after failure rather than depending on in-memory variables.

### 2. Workflow and task state

Your states:

Started
Passed
Failed
Timed Out

are a good start, but production systems usually need a richer lifecycle.

For a task:

Pending
Scheduled
Running
Retry Scheduled
Succeeded
Failed
Timed Out
Cancelled
Outcome Unknown
Compensated

Why distinguish Pending, Scheduled, and Running?

Because each tells operations teams something different:

Pending
→ dependency not yet satisfied

Scheduled
→ message emitted but worker has not started

Running
→ worker claimed and began execution

An Outcome Unknown state is essential for external writes such as sending an email.

### 3. Idempotency model

Your use of workflowId and requestId is good.

I would distinguish three identifiers:

{
  "workflow_id": "wf-903",
  "task_instance_id": "task-extract-company-a",
  "attempt_id": "attempt-2",
  "idempotency_key": "wf-903-extract-company-a"
}

They serve different purposes:

workflow_id identifies the overall workflow.
task_instance_id identifies the logical workflow step.
attempt_id identifies one execution attempt.
idempotency_key deduplicates repeated execution of the same logical step.

Retries should get a new attempt_id but retain the same logical idempotency key where appropriate.

### 4. Kafka partitioning and split-brain

Your idea to partition by workflowId is valuable because it preserves ordering for events from the same workflow.

Partition key = workflowId

This generally means events for one workflow are processed sequentially by one consumer at a given moment.

However, this alone does not eliminate split-brain risks because:

consumer-group rebalancing can move the partition,
a consumer may pause while its lease expires,
delayed workers may publish stale results,
duplicate events can be delivered,
a previous instance may still finish in-flight work.

Therefore, combine partitioning with:

Optimistic concurrency
{
  "workflow_id": "wf-903",
  "state_version": 18
}

Update only when the expected version matches.

Or leases
{
  "workflow_id": "wf-903",
  "owner": "orchestrator-7",
  "lease_expiry": "..."
}
And idempotent event handling

Every event should have a stable event ID:

{
  "event_id": "evt-782",
  "workflow_id": "wf-903",
  "task_instance_id": "extract-a",
  "event_type": "TaskSucceeded"
}

If the event was already applied, ignore it.

So:

Kafka ordering
+
workflow versioning
+
event deduplication
+
lease or ownership rules

provides stronger protection than partitioning alone.

### 5. Task retries: worker or orchestrator?

You proposed that each task should own its retry policy because it knows its behavior best. That is partly correct.

A useful division is:

Task or Tool Executor owns
which technical failures are transient,
API-specific retry rules,
HTTP retry behavior,
backoff and jitter,
request timeout,
provider rate-limit handling.
Orchestrator owns
maximum workflow-level attempts,
whether the workflow may continue after failure,
fallback to another processor,
whether partial results are acceptable,
escalation or human review,
workflow deadline and cancellation.

For example:

PDF worker:
OCR_TIMEOUT is retryable.

Orchestrator:
Allow at most three extraction attempts
before marking Company C unavailable.

This avoids two risks:

workers retrying indefinitely without workflow awareness,
orchestrator duplicating service-specific retry knowledge.

### 6. Schema registry

Publishing schemas to a registry is a sound enterprise design, especially for versioned task contracts.

Each request should include a contract version:

{
  "schema_version": "2.1",
  "workflow_id": "wf-903",
  "task_instance_id": "extract-company-a",
  "document_id": "doc-a"
}

However, I would be cautious about the orchestrator dynamically reading arbitrary schemas and inventing task requests at runtime.

The orchestrator should usually have a versioned workflow definition that explicitly declares:

Task type
Supported schema version
Required inputs
Expected output
Dependency rules
Retry policy reference

The registry validates compatibility; it should not make the orchestrator blindly trust newly published worker contracts.

Otherwise, a worker schema change could silently alter workflow behavior.

### 7. Task-completion events

Your event-based completion design is correct.

A task event should include:

{
  "event_id": "evt-901",
  "workflow_id": "wf-903",
  "task_instance_id": "extract-company-a",
  "attempt_id": "attempt-1",
  "event_type": "TaskSucceeded",
  "result_reference": "result-301",
  "occurred_at": "2026-07-31T10:30:00+05:30",
  "producer_version": "pdf-extractor-4.2"
}

Large results should not be placed directly on Kafka. Store them in a result store and publish only the reference.

This prevents:

oversized messages,
excessive duplication,
poor replay performance,
sensitive content spreading through the event system.

### 8. Timeout monitoring

Your workflow_monitor concept is valid, but polling every workflow can become expensive at scale.

Alternatives include:

durable timers,
delayed messages,
timer-wheel services,
scheduled timeout events,
database queries indexed by next_deadline,
Kafka retry or delay topics.

For example, when a task starts:

TaskStarted
    ↓
Schedule TaskTimeoutDue at T+10 minutes

When the timeout event arrives:

Is task still running?
  ├── No → ignore stale timeout event
  └── Yes → transition to timed out

The timeout event must include the attempt ID. Otherwise, a timeout from attempt 1 might incorrectly terminate attempt 2.

### 9. Checkpoints

You proposed persisting at task start and completion. That is a sensible minimum.

Checkpoint additionally after:

fan-out scheduling,
fan-in aggregation,
LLM planning decisions that affect later execution,
human approval,
external side effects,
compensation steps.

The checkpoint should record references rather than large payloads:

{
  "workflow_id": "wf-903",
  "state_version": 21,
  "current_stage": "awaiting_approval",
  "completed_tasks": {
    "extract_a": "result-301",
    "extract_b": "result-302",
    "extract_c": "result-303",
    "generate_report": "result-410"
  },
  "approval_id": "approval-88"
}

### 10. Result storage and expiring references

You correctly noticed that SAS URLs may expire.

The durable state should not store a SAS URL as the canonical resource identity.

Store:

{
  "storage_account": "financialdocs",
  "container": "workflow-results",
  "blob_key": "wf-903/report.pdf"
}

Generate a fresh short-lived access URL only when required.

Durable object identity
        ↓
Authorization check
        ↓
Generate temporary SAS URL

This keeps capability tokens out of long-term workflow state and event logs.

### 11. Approval state

Approval should be a separate durable entity:

{
  "approval_id": "approval-88",
  "workflow_id": "wf-903",
  "action": "send_email",
  "content_hash": "sha256:...",
  "recipient_ids": ["contact-702"],
  "status": "pending",
  "expires_at": "...",
  "version": 1
}

Binding approval to the content hash ensures that if the report or email body changes, approval must be requested again.

### 12. Reconciliation

The workflow must reconcile operations whose outcome is uncertain.

Examples:

email send timed out,
financial API accepted a request but did not respond,
object upload completed but acknowledgement was lost,
an LLM provider returned after the worker lease expired.

A reconciliation worker should:

Read uncertain operations
       ↓
Query authoritative external system
       ↓
Resolve succeeded / failed / still unknown
       ↓
Atomically update task and workflow state

The LLM should not perform reconciliation reasoning from prose logs.

### 13. Retention

Retention should vary by data class:

Workflow metadata
→ retained for operational/audit period

Task events
→ retained for replay and audit period

Raw uploaded financial documents
→ tenant policy and user deletion rules

LLM prompts and outputs
→ privacy- and security-controlled retention

Temporary SAS URLs
→ never treated as durable state

Idempotency records
→ at least as long as duplicate delivery remains possible

Deletion must cascade carefully while preserving legally required audit records.

LLM as a task

Your final point is excellent.

An LLM task should look like:

{
  "task_type": "llm_extract_financial_metrics",
  "workflow_id": "wf-903",
  "task_instance_id": "extract-metrics-a",
  "prompt_version": "financial-extractor-v4",
  "model_policy": "financial-extraction",
  "input_reference": "doc-a-chunks",
  "output_schema_version": "3.0",
  "timeout_seconds": 45
}

Its result should include:

{
  "status": "succeeded",
  "result_reference": "result-501",
  "model": "selected-model",
  "prompt_version": "financial-extractor-v4",
  "confidence": 0.91,
  "validation_status": "passed"
}

This gives you:

model traceability,
prompt-version tracking,
replayability,
evaluation,
cost attribution,
controlled retries.
Final assessment

Your design already contains the backbone of a production-grade system:

Durable orchestrator
+
asynchronous workers
+
idempotent tasks
+
event-driven completion
+
persisted state
+
timeouts and retries
+
LLM treated as a task

The key refinements are:

Kafka partitioning does not replace concurrency control.

Unknown outcome is different from failure.

Worker retries and workflow retries have different owners.

Durable resource identity should not be an expiring SAS URL.

Task events must be deduplicated and version-aware.

Approval must bind to exact content and action.

