# When Multi-Agent is justified

## 1. Distinct domains and expertise

Example:

Enterprise Assistant
        │
        ├── Finance Agent
        ├── Security Agent
        ├── Legal Agent
        └── Engineering Agent

A request might be:

Can we launch this AI product in Europe next quarter?

This requires different reasoning:

Finance Agent
cost model,
pricing,
forecast,
expected margin.
Legal Agent
GDPR obligations,
contractual terms,
data residency.
Security Agent
threat model,
identity controls,
encryption,
risk acceptance.
Engineering Agent
system readiness,
dependencies,
delivery schedule.

Each domain has:

different tools,
different terminology,
different evaluators,
different policies,
different source systems.

A single agent could technically access all tools, but it becomes overly broad and harder to govern.

Multiple agents allow capability boundaries:

Legal Agent
→ legal documents only

Finance Agent
→ financial systems only

Security Agent
→ security telemetry and policies only

The coordinating agent receives structured conclusions rather than unrestricted access to everything.

## 2. Different trust and authorization boundaries

This is an even stronger justification than expertise.

Imagine a production incident system:

Investigation Agent
        ↓
Finds likely root cause
        ↓
Remediation Agent
        ↓
Proposes restart or rollback
        ↓
Approval Agent / Human
        ↓
Execution Agent

The Investigation Agent has read-only access:

logs,
metrics,
deployment history,
configuration.

The Remediation Agent can formulate a proposed action but cannot execute it.

The Execution Agent has tightly controlled production permissions.

Investigation Agent
Permission: Read only

Remediation Planner
Permission: Propose only

Production Executor
Permission: Approved actions only

Keeping these as separate agents or runtimes limits blast radius.

If one broad agent had every capability, a prompt injection or planning mistake could potentially move directly from reading a log to modifying production.

Therefore:

Multiple agents are justified when authority must be isolated, not merely when reasoning differs.

## 3. Independent goals owned by different teams or systems

Consider software release management:

Release Manager Agent
        │
        ├── Build Readiness Agent
        ├── Security Compliance Agent
        ├── Test Quality Agent
        └── Deployment Readiness Agent

Each sub-agent has its own goal:

Build Readiness Agent

Determine whether all required artifacts were produced.

Security Compliance Agent

Determine whether signing, SBOM, vulnerability, and policy requirements passed.

Test Quality Agent

Evaluate test results and unresolved regressions.

Deployment Readiness Agent

Validate environment, rollout plan, capacity, and rollback readiness.

These agents may run independently, on different schedules, and be owned by different engineering teams.

Each can produce a durable decision:

{
  "agent": "security-compliance",
  "decision": "blocked",
  "reasons": [
    "SBOM attestation missing"
  ],
  "evidence": [
    "artifact-193"
  ]
}

The Release Manager Agent does not redo their analysis. It aggregates their decisions using release policy:

Build = Ready
Security = Blocked
Tests = Ready
Deployment = Ready

Final release status = Blocked

This pattern is valuable because each agent is independently testable, deployable, versioned, and accountable.

Additional justified cases

There are a few other cases worth recognizing.

Different execution lifecycles

One agent may finish in seconds, while another may operate for days.

Example:

Hiring Coordinator
        │
        ├── Resume Screening Agent
        ├── Interview Scheduling Agent
        └── Candidate Follow-up Agent

These have different:

triggers,
schedules,
state lifetimes,
retries,
completion conditions.

It may be cleaner to model them as independent agents coordinated by events.

Adversarial review

Sometimes multiple agents are intentionally given opposing roles.

Proposal Agent
      ↓
Critic Agent
      ↓
Revision Agent
      ↓
Final Evaluator

For example:

architecture proposal versus security critic,
investment thesis versus risk critic,
code-generation agent versus code-review agent.

This is justified only when the independent review adds measurable quality. Simply asking the same model twice with different role names does not guarantee independence.

Organizational ownership

An enterprise may already have independently managed services:

Finance AI platform
Security AI platform
Developer AI platform
Support AI platform

A coordinating assistant may need to delegate to each rather than absorb all of their functionality.

This lets teams independently manage:

tools,
prompts,
models,
schemas,
policies,
release cycles,
compliance.

In this case, multi-agent architecture mirrors real organizational boundaries.

Strong decision rule

Before creating another agent, ask:

Does this component need its own:

1. Goal?
2. State?
3. Planning loop?
4. Tools?
5. Completion criteria?
6. Authorization boundary?
7. Independent lifecycle?
8. Operational owner?

If the answer is mostly no, it is probably not another agent.

It may instead be:

a tool,
a workflow step,
a worker,
an evaluator,
a model call,
a specialized service.

For your financial example:

Company extraction
→ worker task

Currency normalization
→ deterministic tool

Comparison
→ workflow step or calculation service

Citation validation
→ evaluator

Overall financial investigation
→ agent

That is a much cleaner architecture than assigning an agent to every box.

## Core lesson

Create multiple agents when you need independently governed reasoning systems—not merely separate pieces of work.

Next, the natural question is: when multiple agents are justified, how should they coordinate? The next section is the main coordination patterns: manager–worker, peer-to-peer, hierarchical, blackboard, and event-driven architectures.