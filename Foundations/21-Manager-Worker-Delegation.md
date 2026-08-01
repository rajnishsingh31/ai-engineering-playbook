# Chapter — Manager–Worker Delegation

The manager–worker pattern is usually the safest starting point for multi-agent systems because one component owns the overall goal.

                     User Goal
                         │
                         ▼
                   Manager Agent
             ┌───────────┼───────────┐
             ▼           ▼           ▼
       Security Agent  Legal Agent  Finance Agent
             │           │           │
             └───────────┼───────────┘
                         ▼
                   Manager Agent
                         │
                         ▼
                    Final Result

The manager does not necessarily perform every analysis itself. It decides:

what work should be delegated,
which worker should receive it,
what result is expected,
whether the worker’s answer is sufficient,
whether more work is required,
when the overall goal is complete.

## 1. The manager owns the global goal

Suppose the user asks:

Can we launch this AI product in Europe next quarter?

The global goal spans several domains:

Launch readiness
├── Technical readiness
├── Security readiness
├── Legal and privacy readiness
├── Financial viability
└── Delivery schedule

No individual worker owns the entire launch decision.

Each worker receives a bounded subgoal.

Legal worker
{
  "objective": "Assess European legal and privacy readiness",
  "scope": [
    "GDPR obligations",
    "data residency",
    "customer contract implications"
  ],
  "required_output": [
    "readiness_status",
    "blocking_issues",
    "evidence",
    "recommended_actions"
  ]
}
Security worker
{
  "objective": "Assess security readiness for European launch",
  "scope": [
    "identity controls",
    "data protection",
    "threat model",
    "incident readiness"
  ],
  "required_output": [
    "readiness_status",
    "risks",
    "evidence",
    "mitigations"
  ]
}

The manager later combines those local conclusions into the global decision.

## 2. Delegation should produce bounded subgoals

A bad delegation request is:

Analyze security.

It is vague. The worker does not know:

what system is in scope,
which environment matters,
what output is expected,
how deeply to investigate,
when to stop.

A better request is:

{
  "task_id": "security-readiness-01",
  "objective": "Determine whether the product satisfies production security requirements for a European launch",
  "system": "AI Customer Support Platform",
  "environment": "production",
  "questions": [
    "Are customer prompts and documents encrypted?",
    "Are tenant boundaries enforced?",
    "Is privileged tool execution approval-controlled?",
    "Are audit logs retained according to policy?"
  ],
  "completion_criteria": [
    "all required controls assessed",
    "blocking gaps identified",
    "every claim has evidence",
    "readiness status assigned"
  ],
  "budget": {
    "maximum_tool_calls": 12,
    "maximum_duration_minutes": 8
  }
}

The worker still has freedom to investigate, but only inside a clear boundary.

The manager delegates an outcome, not unrestricted autonomy.

## 3. Who creates the subgoals?

Usually, the manager’s planner generates a proposed decomposition.

Global Goal
    ↓
Manager Planner
    ↓
Proposed Subgoals
    ↓
Delegation Validator
    ↓
Worker Assignment

Example planner output:

{
  "subgoals": [
    {
      "capability": "legal_analysis",
      "objective": "Assess GDPR and data-residency readiness"
    },
    {
      "capability": "security_analysis",
      "objective": "Assess security and privacy controls"
    },
    {
      "capability": "financial_analysis",
      "objective": "Evaluate launch cost and expected return"
    },
    {
      "capability": "engineering_readiness",
      "objective": "Assess delivery and operational readiness"
    }
  ]
}

But deterministic application logic should validate:

whether each capability exists,
whether the subgoals overlap excessively,
whether any required dimension is missing,
whether the user is authorized to invoke those agents,
whether the expected cost is within budget.

## 4. Selecting the right worker

The manager should not select workers only by name.

Use a capability registry.

{
  "agent_id": "security-agent-v3",
  "capabilities": [
    "threat_model_review",
    "identity_assessment",
    "data_protection_review"
  ],
  "allowed_data_classes": [
    "internal",
    "confidential"
  ],
  "tools": [
    "read_security_documents",
    "query_security_findings",
    "read_architecture"
  ],
  "risk_level": "read_only",
  "supported_output_schema": "security-readiness-v2"
}

Worker selection should consider:

Required capability
+
Authorization
+
Data access
+
Availability
+
Cost
+
Latency
+
Output contract compatibility

For example, two finance agents may exist:

Fast Finance Agent
→ lower cost, routine extraction

Deep Finance Agent
→ more expensive, complex analysis

The manager may choose based on task complexity.

## 5. A worker should return structured results

The worker should not return a long unstructured essay.

Prefer a contract such as:

{
  "task_id": "security-readiness-01",
  "status": "completed_with_findings",
  "local_conclusion": "not_ready",
  "confidence": 0.93,
  "findings": [
    {
      "severity": "high",
      "statement": "Tenant-aware filtering is not enforced for cached retrieval results.",
      "evidence_refs": [
        "security-review-44"
      ]
    }
  ],
  "blocking_issues": [
    "Cross-tenant cache isolation is missing."
  ],
  "recommended_actions": [
    "Include tenant ID and authorization scope in every cache key."
  ],
  "unresolved_questions": []
}

This makes aggregation much easier.

## 6. Local completion vs global completion

A worker may finish its own task without completing the overall goal.

Security worker:
Completed

Legal worker:
Completed

Finance worker:
Still running

Global launch decision:
Not ready to finalize

Each worker owns local completion criteria.

The manager owns global completion criteria.

Worker completion
All assigned security controls evaluated
Evidence attached
Local status assigned
Manager completion
All mandatory domains assessed
Cross-domain contradictions resolved
Global blockers identified
Final recommendation generated

## 7. Manager aggregation is more than concatenation

A weak manager does this:

Security answer
+
Legal answer
+
Finance answer
=
Final response

A strong manager must reason across results.

Example:

Legal Agent:
European launch is allowed if data remains in-region.

Engineering Agent:
The current architecture stores embeddings in a US region.

Security Agent:
Cross-region access is technically possible.

Finance Agent:
Regional deployment increases cost by 18%.

The manager must synthesize:

The launch is not currently ready because the technical deployment violates the legal residency condition. Achieving compliance requires a European deployment, increasing projected cost by 18%.

That conclusion did not exist in any one worker result.

This is the manager’s main value:

Cross-domain synthesis and global decision-making.

## 8. Result sufficiency evaluation

When a worker returns, the manager should not automatically accept it.

A worker-result evaluator checks:

did the worker answer the assigned question?
did it satisfy the output schema?
are claims supported by evidence?
are required fields missing?
did it exceed its scope?
is confidence adequate?
are there unresolved blockers?

Example:

{
  "worker_result_sufficient": false,
  "missing_requirements": [
    "No evidence for data-retention assessment",
    "No conclusion on cross-border transfer"
  ],
  "recommended_action": "request_targeted_follow_up"
}

The manager can then issue a narrower follow-up:

Assess only cross-border transfer requirements and cite the controlling policy.

## 9. Follow-up vs retry

These are different.

Retry

Use when execution failed:

Agent unavailable
Tool timeout
Schema-generation failure

The same logical task is attempted again.

Follow-up

Use when the result succeeded technically but is incomplete:

Legal analysis completed,
but data residency was not assessed.

The manager creates a new, narrower task.

This distinction matters for state, cost tracking, and auditability.

## 10. Avoid over-delegation

Managers can delegate too much.

Bad design:

One agent decides document names.
One agent reads headings.
One agent calculates percentages.
One agent formats bullet points.

Most of these should be tools, deterministic steps, or simple model calls.

Delegation adds:

another planner call,
more state,
more context transfer,
additional latency,
new failure modes,
result validation,
coordination overhead.

Before delegating, ask:

Does this subtask need independent adaptive reasoning?

If not, keep it inside the manager workflow as a tool or worker task.

## 11. Delegation depth

Unbounded delegation is dangerous.

Manager
  ↓
Worker
  ↓
Sub-worker
  ↓
Another worker
  ↓
...

This can cause:

uncontrolled cost,
unclear ownership,
loops,
excessive latency,
hard-to-follow audit trails.

Profiles should define:

{
  "maximum_delegation_depth": 2,
  "maximum_total_agents": 6,
  "maximum_subtasks": 12
}

A worker may be allowed to use tools but not delegate further.

For example:

Top Manager
→ may delegate to domain managers

Domain Manager
→ may delegate to specialists

Specialist
→ tools only; no more delegation

## 12. Parallel delegation

Independent subgoals can run concurrently.

                    Manager
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Legal        Security      Finance
          └────────────┼────────────┘
                       ▼
                    Fan-in
                       ▼
                  Aggregation

The manager checkpoints state and waits asynchronously.

It should define fan-in policy:

Wait for all mandatory workers
Proceed with optional workers if available
Stop at deadline
Return qualified result if optional analysis is missing

Example:

{
  "mandatory_agents": [
    "legal",
    "security",
    "engineering"
  ],
  "optional_agents": [
    "finance"
  ],
  "deadline": "10 minutes"
}

## 13. Handling worker failure

Suppose the Legal Agent fails.

The manager has several choices:

Retry same worker
Use fallback legal agent
Return partial result
Request human legal review
Fail the whole workflow

The policy depends on whether legal analysis is mandatory.

{
  "worker": "legal-agent",
  "status": "failed",
  "criticality": "mandatory",
  "fallback": "human_legal_review",
  "global_workflow_action": "pause"
}

A manager should not silently omit a failed mandatory domain and still claim a complete answer.

## 14. Handling contradictions

Suppose:

Security Agent:
Launch is blocked because encryption is absent.

Engineering Agent:
Encryption is enabled in production.

The manager should preserve both claims and trigger a resolution task.

{
  "conflict_id": "conflict-12",
  "claim_a": {
    "statement": "Encryption is absent.",
    "source": "security-agent"
  },
  "claim_b": {
    "statement": "Encryption is enabled.",
    "source": "engineering-agent"
  },
  "resolution_question": "Is production customer data encrypted at rest and in transit?"
}

Resolution may use:

authoritative configuration,
an independent verifier,
deterministic policy checks,
human review.

The manager should not choose whichever agent has higher confidence without considering source authority.

## 15. Source authority

Not all worker evidence is equally authoritative.

Example:

Architecture document
→ intended design

Production configuration
→ current implemented state

Engineer comment
→ informal claim

A conflict-resolution policy might rank sources:

Authoritative runtime state
>
approved policy
>
current configuration
>
design documentation
>
informal commentary

This ranking is domain-specific and should be configured.

## 16. Preventing manager context overload

The manager should not receive every raw document and every worker trace.

Workers should return:

concise conclusions,
structured findings,
evidence references,
unresolved questions,
confidence,
cost and execution metadata.

Raw evidence remains in source stores.

Worker raw evidence
→ Evidence Store

Worker structured result
→ Manager context

The manager can fetch specific evidence when required.

## 17. Manager state

A manager’s state might look like:

{
  "global_goal": "Assess European launch readiness",
  "delegated_tasks": [
    {
      "task_id": "legal-01",
      "agent": "legal-agent",
      "status": "completed"
    },
    {
      "task_id": "security-01",
      "agent": "security-agent",
      "status": "completed"
    },
    {
      "task_id": "finance-01",
      "agent": "finance-agent",
      "status": "running"
    }
  ],
  "global_facts": [],
  "conflicts": [],
  "missing_dimensions": [
    "financial viability"
  ],
  "remaining_budget": {
    "tool_calls": 6,
    "duration_seconds": 280
  }
}

The manager should not reconstruct delegation state from chat messages.

## 18. The manager does not need to be an LLM for everything

Many manager responsibilities are deterministic:

Wait for mandatory workers
Check schemas
Track deadlines
Apply fan-in rules
Enforce delegation depth
Detect missing result fields

The LLM is most useful for:

decomposing ambiguous goals,
choosing specialist capabilities,
cross-domain synthesis,
generating targeted follow-ups,
identifying semantic contradictions.

A strong design combines both.

Recommended architecture
                         User Goal
                             │
                             ▼
                       Goal Normalizer
                             │
                             ▼
                      Manager Runtime
             ┌───────────────┼────────────────┐
             ▼               ▼                ▼
       Delegation Planner  Global State   Control Policy
             │
             ▼
      Delegation Validator
             │
             ▼
         Agent Registry
             │
      ┌──────┼─────────┐
      ▼      ▼         ▼
   Worker A Worker B Worker C
      │      │         │
      └──────┼─────────┘
             ▼
      Worker Result Evaluator
             │
       ┌─────┼──────────┐
       ▼     ▼          ▼
    Accept Follow-up  Resolve conflict
             │
             ▼
        Global Aggregator
             │
             ▼
    Global Completion Evaluator
             │
             ▼
         Final Response

## Core principles
The manager owns the global goal.

Workers receive bounded local goals.

Workers return structured, evidence-backed results.

Local completion does not imply global completion.

Aggregation requires cross-result reasoning.

Delegation should be used only for independent adaptive work.

The manager validates worker results before accepting them.

Delegation depth, cost, and time must be bounded.

Mandatory worker failure must remain visible.

The manager controls final completion.

## Excersise:

The Legal Agent says data must remain in India. The Architecture Agent says data is currently stored in Singapore. The Finance Agent says an India deployment adds 20% cost.

How should the manager represent, resolve, and synthesize these results?

### Solution:


1. Goal normalization

Convert the user request into a structured global goal:

{
  "goal_type": "production_readiness_assessment",
  "system": "AI Customer Support Platform",
  "environment": "production",
  "target_launch_date": null,
  "required_decision": "ready | conditionally_ready | blocked",
  "constraints": {
    "all_material_claims_require_evidence": true,
    "critical_controls_cannot_be_waived_by_agent": true
  }
}

The product and governance teams define what production readiness means.

2. Goal decomposition

The Manager Planner proposes domain subgoals:

Security readiness
Privacy and legal readiness
Reliability and capacity
AI quality and safety
Operational readiness
Financial viability
Engineering and release readiness

A Delegation Validator checks:

all mandatory domains are covered,
no subgoals overlap unnecessarily,
each task has completion criteria,
the total plan fits budget,
suitable agents exist.
3. Worker selection

Use an Agent Registry containing:

capabilities,
supported task schemas,
accessible data,
authorization scope,
model and prompt versions,
expected cost and latency,
health and availability.

Example:

{
  "agent_id": "security-readiness-agent-v3",
  "capabilities": [
    "identity_review",
    "tenant_isolation_review",
    "ai_threat_assessment"
  ],
  "access_mode": "read_only",
  "output_schema": "security-readiness-v2"
}

The manager selects by capability and policy—not merely by agent name.

4. Task contracts

Each worker gets a bounded contract:

{
  "task_id": "security-assessment-01",
  "objective": "Assess production security readiness.",
  "scope": {
    "system": "AI Customer Support Platform",
    "environment": "production"
  },
  "required_checks": [
    "authentication",
    "authorization",
    "tenant isolation",
    "encryption",
    "tool execution controls",
    "prompt injection defenses",
    "auditability"
  ],
  "completion_criteria": [
    "all required checks have status",
    "all material claims have evidence",
    "blocking issues are identified",
    "local readiness decision is provided"
  ],
  "budget": {
    "max_tool_calls": 15,
    "max_duration_minutes": 10
  }
}

Workers may adapt their local investigation, but they cannot expand beyond the assigned scope.

5. Parallel execution

Independent assessments run concurrently:

                 Manager
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    Security     Legal    Reliability
        ▼          ▼          ▼
   AI Quality  Operations   Finance
        └──────────┼──────────┘
                   ▼
                 Fan-in

The Workflow Engine schedules tasks asynchronously and persists manager state while waiting.

6. Mandatory versus optional workers

Example policy:

{
  "mandatory_workers": [
    "security",
    "privacy_legal",
    "reliability",
    "ai_quality",
    "operations",
    "engineering_release"
  ],
  "optional_workers": [
    "finance_optimization",
    "user_experience"
  ]
}

A mandatory-worker failure prevents a complete readiness decision.

An optional-worker failure may permit a qualified result, provided it is disclosed.

7. Global state

The manager maintains authoritative structured state:

{
  "workflow_id": "readiness-101",
  "global_status": "assessing",
  "worker_tasks": {
    "security": "completed",
    "legal": "completed",
    "reliability": "running",
    "ai_quality": "completed"
  },
  "accepted_findings": [],
  "conflicts": [],
  "blocking_issues": [],
  "missing_dimensions": ["reliability"],
  "remaining_budget": {
    "duration_seconds": 420,
    "worker_calls": 3
  },
  "state_version": 12
}

Large worker evidence remains in evidence storage and is referenced by ID.

8. Worker-result evaluation

Every worker result passes through:

Schema validation

Are the required fields and types present?

Evidence validation

Does each important claim cite authoritative evidence?

Scope validation

Did the worker answer only the delegated question?

Completeness validation

Were all required checks assessed?

Domain validation

Do results satisfy domain-specific rules?

Confidence calibration

Does confidence align with evidence quality and unresolved questions?

Possible outcomes:

Accept
Targeted follow-up
Retry after technical failure
Use fallback worker
Create conflict-resolution task
Escalate to human
Reject result
9. Conflict handling

The manager records both supporting and contradictory claims rather than overwriting one.

{
  "conflict_id": "conflict-7",
  "topic": "production encryption",
  "claims": [
    {
      "agent": "security",
      "statement": "Encryption is not enabled.",
      "evidence": "security-scan-4"
    },
    {
      "agent": "architecture",
      "statement": "Encryption is configured.",
      "evidence": "design-doc-8"
    }
  ],
  "status": "unresolved"
}

The manager should then consult the most authoritative source, such as actual production configuration rather than intended architecture.

High-impact unresolved conflicts require human review.

10. Follow-ups

Follow-up tasks should be narrow:

{
  "objective": "Verify whether production customer-data storage has encryption enabled.",
  "allowed_sources": [
    "production_configuration",
    "cloud_resource_inventory"
  ],
  "required_output": [
    "enabled_status",
    "encryption_type",
    "resource_ids",
    "evidence"
  ]
}

This is preferable to rerunning the entire Security Agent.

11. Delegation limits

The profile should constrain:

{
  "maximum_delegation_depth": 2,
  "maximum_total_agents": 10,
  "maximum_follow_ups_per_worker": 2,
  "maximum_conflict_resolution_tasks": 3,
  "maximum_replans": 2
}

Specialist workers would usually be prohibited from further delegation unless explicitly allowed.

12. Budgets

Track:

total duration,
LLM tokens,
tool calls,
external service costs,
number of worker tasks,
follow-ups,
replans,
human-review deadlines.

The manager should reserve budget before delegating.

When budget is low, it should prioritize mandatory unresolved dimensions instead of launching broad optional analysis.

13. Global completion criteria

The manager may return Ready only when:

All mandatory workers completed successfully
All mandatory controls were assessed
No unresolved critical or high-severity blockers remain
All important findings have evidence
Cross-domain conflicts are resolved
Reliability and AI quality thresholds passed
Rollback and operational support are validated
Required approvals are present

Terminal states should include:

Ready

All mandatory criteria pass.

Conditionally ready

Only explicitly accepted non-critical gaps remain, with owners and deadlines.

Blocked

At least one mandatory blocker exists.

Incomplete

Required analysis or evidence is unavailable.

Failed

The assessment could not run because of a system or policy failure.

14. Human-approval boundaries

Human approval should be required for:

accepting security or legal exceptions,
approving residual high-risk findings,
authorizing production launch,
approving cost increases above thresholds,
external publication of readiness results,
remediation actions affecting production.

Agents can recommend; authorized humans accept risk.

15. Auditability

Persist:

original goal,
selected manager profile,
decomposition plan,
worker selections and versions,
every delegated contract,
tool and evidence references,
worker results,
validation decisions,
conflicts and resolutions,
follow-up tasks,
budget usage,
human approvals,
final readiness decision and rationale.

The system should be able to answer:

Why was the platform declared ready, and which evidence supported every required control?

Final architecture
User Goal
    ↓
Goal Normalizer
    ↓
Manager Planner
    ↓
Delegation Validator
    ↓
Agent Registry
    ↓
Parallel Domain Workers
    ↓
Worker Result Evaluators
    ↓
Shared Fact and Conflict Store
    ↓
Follow-up / Resolution Tasks
    ↓
Global Aggregator
    ↓
Global Completion Evaluator
    ↓
Human Launch Approval
    ↓
Final Readiness Decision

The core separation is:

Domain agents assess local readiness. The manager synthesizes global readiness. Deterministic policy and authorized humans control the final production-launch decision.


