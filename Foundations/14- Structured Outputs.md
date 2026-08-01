# Chapter 14 — Structured Outputs

Until now, we mostly treated an LLM response as text:

"The user prefers Python and wants architecture diagrams first."

That works when a human reads the answer.

It becomes unreliable when software must consume it.

Suppose your application asks an LLM to extract a memory and receives:

The user seems to prefer Python. This is probably a semantic memory.

Your program now has to infer:

the memory text,
its type,
its confidence,
whether it should expire,
whether it replaces an existing memory.

Free-form text is a poor software contract.

The Core Idea

**Instead of asking the model for prose, require a predictable structure:**

{
  "memory": "User prefers Python examples.",
  "type": "semantic",
  "confidence": 0.98,
  "persistent": true
}

Now the response can be:

parsed,
validated,
stored,
tested,
passed to another service.

Structured output turns an LLM response into an application contract.

Why Prompting for JSON Is Not Enough

A beginner may write:

Return the answer as JSON.

The model might return:

Here is the JSON you requested:

{
  "language": "Python"
}

Or:

{
  "language": "Python",
}

That trailing comma makes it invalid JSON.

It may also produce:

{
  "preferred_language": "Python"
}

when your application expected the field language.

The response looks reasonable to a human but may break software.

## Three Levels of Output Reliability

### Level 1 — Free-form prompting
Extract the user's preferred language.

Output:

The user prefers Python.

Useful for humans, weak for automation.

### Level 2 — Prompted JSON
Return JSON with a field named "language".

Output is more predictable, but the model can still violate the format.

### Level 3 — Schema-constrained output

Define an explicit schema:

{
  "type": "object",
  "properties": {
    "language": {
      "type": "string"
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    }
  },
  "required": ["language", "confidence"],
  "additionalProperties": false
}

The model or platform is constrained to produce data matching that shape.

This is much safer.

Schema vs Example

An example says:

{
  "language": "Python",
  "confidence": 0.9
}

A schema says:

language must be a string,
confidence must be a number,
confidence must be between 0 and 1,
both fields are required,
extra fields are forbidden.

Examples demonstrate intent.

Schemas define a contract.

Common Data Types

Structured outputs commonly use:

string
number
integer
boolean
array
object
null
enum

For example:

{
  "query_type": "identifier",
  "requires_retrieval": true,
  "retrieval_sources": ["symbol_index", "bm25"],
  "max_results": 10
}

Here:

query_type is a string or enum,
requires_retrieval is a Boolean,
retrieval_sources is an array,
max_results is an integer.

#### Enums

Enums restrict values to an allowed set.

For a query router:

{
  "query_type": "identifier"
}

The schema might allow only:

identifier
semantic
structured_data
live_data
multi_part

Without an enum, the model might return:

code lookup

or:

symbol search

Both are understandable but may not match your routing code.

**Enums make downstream behavior deterministic.**

#### Structured Output Example: Query Router

Suppose the user asks:

Where is CreateSubmissionAsync() implemented?

The classifier could return:

{
  "query_type": "identifier",
  "route": "symbol_index",
  "transform_query": false,
  "confidence": 0.99
}

Your application can now execute:

route == symbol_index
        ↓
call symbol search

No natural-language interpretation is needed.

#### Structured Output Example: Memory Extractor

From:

From now on, show architecture diagrams before code.

The model might return:

{
  "should_store": true,
  "memory_type": "procedural",
  "memory_text": "Present architecture diagrams before code.",
  "scope": "persistent",
  "confidence": 1.0,
  "expires_at": null
}

This maps directly to the architecture we just designed.

#### Structured Output Example: Financial Extraction

Input:

Annual recurring revenue increased from $80 million to $100 million.

Output:

{
  "metric": "ARR",
  "period_start_value": 80000000,
  "period_end_value": 100000000,
  "currency": "USD",
  "growth_rate": 0.25,
  "source_section": "Annual Recurring Revenue"
}

Now software can compare companies numerically instead of asking an LLM to repeatedly interpret prose.

## Validation

Even with structured output, validate everything.

Suppose the model returns:

{
  "growth_rate": 25
}

Did it mean:

25%,
or 25.0 as a decimal, meaning 2,500%?

Your schema should define the representation clearly:

growth_rate:
decimal fraction between -1 and 10
Example: 0.25 means 25%

### Validation checks two dimensions.

#### Structural validation

Does it match the schema?

required fields present,
types correct,
enums valid,
no unexpected fields.

#### Business validation

Does it make sense for the domain?

end date is after start date,
currency is supported,
ARR is non-negative,
cited company exists,
calculated growth matches the values.

A schema can prove that a field is a number.

It cannot prove that the number is correct.

Important Principle

Valid JSON is not the same as valid business data.

For example:

{
  "employee_count": -500
}

This may be valid JSON and may match a simple integer schema.

But it is not a valid employee count.

Your application still needs domain rules.

### The Validation Pipeline
LLM
 │
 ▼
Schema Validation
 │
 ├── Invalid → retry or fail
 │
 ▼
Business Validation
 │
 ├── Invalid → correct, retrieve more evidence, or escalate
 │
 ▼
Trusted Application Object

Do not let an LLM response directly update databases or invoke sensitive tools without validation.

### Retry Strategy

Suppose validation fails because a required field is missing.

A weak approach is:

Try again.

A better repair prompt contains the exact error:

Your previous response failed validation:

- "currency" is required.
- "growth_rate" must be between 0 and 10.

Return a corrected object only.

Limit retries.

For example:

Attempt 1
   ↓ invalid
Repair attempt
   ↓ invalid
Fail safely or request human review

Unlimited retries increase cost and can create loops.

### Missing Information

Suppose a report states:

Revenue increased to $100 million.

But it does not give the previous value.

A dangerous output is:

{
  "previous_revenue": 80000000,
  "current_revenue": 100000000
}

The model invented the missing value.

A safer schema allows uncertainty:

{
  "previous_revenue": null,
  "current_revenue": 100000000,
  "growth_rate": null,
  "status": "insufficient_information"
}

#### Your contracts should explicitly support:

unknown,
not applicable,
ambiguous,
conflicting evidence.

Otherwise, the model may feel forced to fabricate a value.

Null vs Missing Field

These mean different things.

Missing field:

{
  "current_revenue": 100000000
}

Could mean:

the model forgot the field,
the schema did not require it,
it is unavailable.

Explicit null:

{
  "previous_revenue": null
}

means:

The field is known to be unavailable.

For important fields, it is often better to require them and allow null.

### Evidence and Citations

For extraction, don't return only a value.

Return provenance:

{
  "metric": "ARR",
  "value": 100000000,
  "currency": "USD",
  "evidence": {
    "document_id": "company-a-fy2026",
    "page": 12,
    "section": "Key Metrics",
    "quote": "ARR reached $100 million."
  }
}

This gives you:

traceability,
easier debugging,
human verification,
citation generation.

The evidence quote should remain short and directly support the value.

## Structured Outputs and Tool Calling

These concepts are related, but not identical.

## Structured output

The model returns application data:

{
  "query_type": "identifier"
}
Tool calling

The model asks the application to perform an operation:

{
  "tool": "search_symbol",
  "arguments": {
    "symbol": "CreateSubmissionAsync"
  }
}

Structured output answers:

What data should the application receive?

Tool calling answers:

What action should the application perform?

## Tool calling is our next major topic.

Versioning Contracts

Suppose version 1 returns:

{
  "language": "Python"
}

Later you add confidence:

{
  "language": "Python",
  "confidence": 0.95
}

Downstream services may depend on the old format.

Treat LLM schemas like API contracts:

version them,
document them,
test compatibility,
avoid silently renaming fields,
support controlled migrations.

For example:

{
  "schema_version": "2.0",
  "language": "Python",
  "confidence": 0.95
}

## Architect Perspective

Structured outputs create a clean boundary:

Probabilistic component
        │
        ▼
Validated schema
        │
        ▼
Deterministic software

The LLM remains probabilistic.

But once its response passes structural and business validation, the rest of your application can work with a predictable object.

This is how you safely integrate an LLM into conventional distributed systems.

## Where Structured Outputs Fit

User Question
      │
Conversation and Memory
      │
Query Classifier
      │
Structured Routing Decision
      │
Retrieval / Tools
      │
Prompt Builder
      │
LLM
      │
Structured Answer
      │
Schema Validation
      │
Business Validation
      │
UI / Database / Next Workflow Step

## Structured outputs may appear multiple times:

query classification,
memory extraction,
retrieval planning,
document extraction,
answer generation,
evaluation.

They are not only for the final response.

Engineer Perspective

A production implementation would usually define a typed model.

Conceptually:

class QueryDecision:
    query_type: QueryType
    route: Route
    transform_query: bool
    confidence: float

Your application then:

sends the schema to the model,
receives structured data,
parses it into the typed object,
applies validation,
executes deterministic logic.

We will implement this later in Python using a validation library such as Pydantic, but the contract design matters more than the library.

## Manager Perspective

Structured outputs improve:

integration reliability,
testability,
auditability,
ownership boundaries,
incident diagnosis.

They also help teams divide responsibility:

## AI team
→ model prompt and schema performance

Platform team
→ validation, retries, observability

Domain team
→ business rules

Security team
→ authorization and safe execution

This is much easier to operate than passing free-form prose between services.

## Dangerous tool execution

Human approval is essential, but it should be the last major control, not the only one.

A structurally valid tool call is not automatically:

authorized,
safe,
intended,
properly scoped,
executable.

The system should apply these checks:

### 1. Tool allowlist

Determine whether the assistant is permitted to invoke delete_repository at all.

Many assistants should never have access to destructive tools.

Available tools for this assistant:
- search_repository
- read_file
- create_issue

delete_repository:
not available

The safest dangerous tool is often one the model cannot see.

### 2. User identity and authorization

Verify that the authenticated user has permission to delete that exact repository.

User permission
+
repository permission
+
organization policy

The LLM must never decide authorization.

### 3. Intent verification

Confirm that the user explicitly requested deletion.

A question such as:

What happens if this repository is deleted?

must not trigger deletion.

The system should distinguish:

Discuss an action
≠
Perform an action

### 4. Resource validation

Resolve the exact repository using a stable identifier rather than only a display name:

{
  "repository_id": "repo-78492",
  "repository_name": "production-driver-service",
  "organization": "windows-platform"
}

This prevents deleting a similarly named resource.

### 5. Environment and policy checks

Production resources may require stronger controls than test resources.

Development
→ possibly automated

Production
→ mandatory approval and change-management policy

### 6. Impact preview

Before execution, provide a dry-run or impact summary:

Repository:
production-driver-service

Branch protections:
4

Open pull requests:
17

Dependent pipelines:
6

Deletion:
irreversible after retention period

### 7. Explicit human confirmation

The confirmation must name the precise action and resource:

Permanently delete windows-platform/production-driver-service?

A generic confirmation such as “Continue?” is insufficient.

For extremely sensitive actions, the user may need to type the resource name or use a privileged approval workflow.

### 8. Time-bound approval token

The approved action should generate a short-lived token bound to:

user,
tool,
exact repository,
action,
expiration time.

This prevents approval from being reused for another operation.

### 9. Idempotency and concurrency control

Prevent duplicate execution caused by retries or repeated model calls.

Also recheck that the repository has not changed materially between approval and execution.

### 10. Audit logging

Record:

requesting user,
original request,
model decision,
authorization result,
approval identity,
resolved resource,
execution result,
timestamp.

The final architecture is:

LLM tool proposal
        ↓
Schema validation
        ↓
Tool allowlist
        ↓
Intent verification
        ↓
Identity and authorization
        ↓
Policy and environment checks
        ↓
Resource resolution
        ↓
Impact preview
        ↓
Human approval
        ↓
Short-lived execution token
        ↓
Deterministic tool executor
        ↓
Audit log

The key principle is:

**The LLM may propose an action, but deterministic application controls must authorize and execute it.**