# Chapter 18 — Single-Agent Architecture

Today we're going to answer one question:

If I had to build ChatGPT, GitHub Copilot Agent, Microsoft 365 Copilot, or an enterprise investigation agent, what does one agent actually look like?

We'll ignore frameworks (LangGraph, CrewAI, AutoGen, Semantic Kernel, etc.) and design the architecture ourselves.

First Principle

**An agent is not:**

an LLM
a prompt
a while loop
ReAct
tool calling

An agent is an application whose decision maker happens to be an LLM.

Think of it like a self-driving car.

Is the self-driving car...

Just the steering wheel?

No.

Is it the camera?

No.

Is it the GPU?

No.

The car is an entire system.

Similarly,

Agent
≠
LLM

**The LLM is one subsystem.**

Think Like an Operating System

Instead of thinking:

```text
User
 ↓
GPT
 ↓
Answer
```

Think:

```text
                User Goal
                     │
                     ▼
             Agent Runtime
                     │
      ┌──────────────┼───────────────┐
      ▼              ▼               ▼
  State Store    Tool Executor    Memory
      │
      ▼
     LLM
      │
      ▼
 Evaluator
      │
      ▼
 Workflow Engine
```

Notice something.

The LLM is not at the top.

It's inside the runtime.

That's how Microsoft, Google, Anthropic, OpenAI, etc. think.

## The Agent Runtime

Imagine removing the LLM.

What remains?

Probably:

State

Tool execution

Memory

Logging

Retry

Evaluation

Workflow

Policies

These are all traditional software engineering.

Now insert the LLM.

The LLM becomes:

The reasoning engine.

Not the application.

## Responsibilities of an Agent Runtime

Let's list everything the runtime owns.

Goal management

State management

Planning

Tool execution

Memory retrieval

Policy enforcement

Checkpointing

Retry

Evaluation

Stopping

Logging

Audit

Cost tracking

Timeouts

Human approval

Observability

Notice that most of these are things we've already learned.

## The Agent Cycle

Every agent repeatedly executes the same high-level loop.

```text
Goal
↓
Understand current situation
↓
Decide next action
↓
Validate action
↓
Execute
↓
Observe
↓
Evaluate
↓
Update state
↓
Stop?
↓
No
↓
Repeat
```

Every agent framework in the industry is implementing some variation of this loop.

### Step 1 — Goal

Everything starts with a goal.

Example:

Investigate why Driver Submission latency increased.

The runtime creates:

{
  "goal":"Investigate Driver Submission latency",
  "workflow_id":"wf-123",
  "status":"running"
}

Notice:

No planning yet.

No tools yet.

Just the goal.

### Step 2 — Build Context

Before asking the LLM anything:

Collect context.

Example:

Conversation

Current workflow state

Memory

Previous observations

Budget

Available tools

Policies

Prompt Builder creates:

Goal

+

Current facts

+

Remaining budget

+

Allowed tools

+

Previous failed actions

↓

LLM

The LLM never sees the database.

It sees a curated context.

### Step 3 — Planning

The planner asks:

What should I do next?

Notice the wording.

Not:

"What is the answer?"

Instead:

"What action moves me toward the goal?"

Example output:

{
    "tool":"search_logs",
    "arguments":{
        "service":"DriverSubmission"
    },
    "reason":"Need latency evidence"
}

The answer isn't the final response.

It's an action proposal.

### Step 4 — Validation

Never trust the planner.

Everything gets validated.

Tool exists?

Allowed?

Arguments valid?

Tenant valid?

Budget remaining?

Already attempted?

Approval needed?

If validation fails:

Back to planner.

### Step 5 — Execute

Tool Executor runs:

Search Logs

↓

Result

Example:

{
    "status":"success",
    "latency_p95":18.2,
    "baseline":4.5
}

### Step 6 — Observe

The runtime transforms tool output into observations.

Instead of:

Huge JSON

It creates:

{
    "fact":"P95 latency increased from 4.5s to 18.2s",
    "confidence":1,
    "source":"Azure Monitor"
}

Observations are much easier for the planner to reason over.

### Step 7 — Evaluate

The evaluator asks:

Did we make progress?

Did we answer the question?

Is more evidence needed?

Should we stop?

Example:

{
    "goal_complete":false,
    "new_information":true,
    "missing_information":[
        "deployment history"
    ]
}

### Step 8 — Update State

The runtime updates:

Completed actions

Known facts

Failed attempts

Remaining budget

Current iteration

Notice:

The LLM isn't remembering this.

The runtime is.

### Step 9 — Loop

Now repeat.

This time the planner sees:

Known:

Latency increased.

Unknown:

Which deployment caused it?

Next action:

{
    "tool":"search_deployments"
}

This continues until completion.

### The Planner is NOT Always the Same LLM

This surprises many engineers.

You don't need one model.

Example:

Small model

↓

Planning

Large model

↓

Complex reasoning

Small model

↓

Evaluation

Different models can perform different roles.

**Planning Does NOT Mean Creating a Full Plan
**

### There are three strategies.

#### Strategy 1

Full plan.

Plan A

↓

Execute everything

Fast.

Bad when information changes.

#### Strategy 2

One action.

Observe

↓

One action

↓

Observe

↓

One action

Adaptive.

More expensive.

#### Strategy 3

Hybrid.

High-level plan

↓

One-step execution

↓

Replan only when needed

Most enterprise agents use this.

#### Why Evaluation is Separate

Many beginners ask:

Why not ask the planner?

Because planners are optimistic.

Example:

Planner:

I think we're done.

Evaluator:

No.

Company B still missing.

Continue.

Think of code review.

Developer and reviewer are different roles.

#### Why Observations Matter

Suppose logs return:

{
   ... 2000 lines ...
}

Should the planner receive all 2000 lines?

No.

Observation layer extracts:

CPU jumped after deployment.

Memory stable.

No database errors.

The planner reasons over observations, not raw telemetry.

This dramatically reduces tokens.

### Fact Store

As the loop runs:

Facts accumulate.

Fact 1

Latency increased.

Fact 2

Deployment 102 happened 5 minutes earlier.

Fact 3

CPU doubled.

Fact 4

Database unchanged.

Instead of searching repeatedly:

Planner uses facts.

### Hypothesis Tracking

A mature agent doesn't only collect facts.

It also tracks hypotheses.

Example:

Hypothesis A

Deployment caused latency.

Status:

Supported.
Hypothesis B

Database issue.

Status:

Rejected.

Why?

To avoid repeatedly investigating rejected ideas.

### Action History

Every action gets recorded.

Search logs

Success

Search deployment

Success

Search SQL

No evidence

Search SQL

Don't repeat

This prevents loops.

Confidence Evolution

Initially:

Root cause:

Unknown.

Later:

Deployment:

Confidence 0.4

Then:

Certificate rotation:

Confidence 0.9

Confidence evolves as evidence accumulates.

### Budget Manager

Another runtime component.

Tracks:

Tool calls

LLM tokens

Time

Money

Example:

Budget:

20 tool calls

Currently:

17 used

Planner sees:

Only 3 actions left.

This changes strategy.

### Policy Manager

Planner proposes:

Delete repository.

Policy:

Denied.

Planner proposes:

Search logs.

Policy:

Allowed.

**Policy Manager protects the system.**

### Checkpoint Manager

Every few actions:

Checkpoint

If the process crashes:

Resume.

Not restart.

Exactly what we learned in State Management.

Human in the Loop

Suppose planner says:

Send email to CEO.

Runtime:

Approval required.

Workflow pauses.

Approval arrives.

Resume.

Again:

The runtime controls execution.

### Runtime Architecture

                   User Goal
                        │
                        ▼
               Goal Manager
                        │
                        ▼
                Prompt Builder
                        │
                        ▼
               Planner (LLM)
                        │
                        ▼
             Action Validator
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      Policy Manager        Budget Manager
             │                     │
             └──────────┬──────────┘
                        ▼
                Tool Executor
                        │
                        ▼
               External Systems
                        │
                        ▼
                Observation Layer
                        │
                        ▼
                 Fact Store
                        │
                        ▼
                 Evaluator
                        │
                        ▼
                 State Store
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        Goal Complete?          Continue

Why This Architecture Matters

Suppose next year:

GPT-7 arrives.

What changes?

Only this:

Planner

↓

New model

Everything else stays.

That is why production systems isolate the LLM behind a stable runtime.

## Engineer Perspective

If you were implementing this in Python, you might have classes like:

AgentRuntime
Planner
Evaluator
ToolExecutor
StateManager
ObservationProcessor
BudgetManager
PolicyManager
CheckpointManager

Each has a single responsibility and can be tested independently.

## Manager Perspective

If you're leading an AI platform team, different engineers can own different components:

Platform team → Runtime, state, execution, observability.
AI team → Prompts, planner, evaluator, model selection.
Security team → Policy manager, approvals, secrets.
Domain team → Tool definitions, business validation, completion criteria.

This separation lets teams evolve independently without coupling every change to the LLM.

## Core Principles
An agent is an application, not a prompt.

The runtime owns control.

The LLM owns reasoning.

Observations are better than raw tool output.

Facts are better than repeated searches.

Evaluation is a separate responsibility.

Planning proposes actions; the runtime decides whether they may execute.

A good runtime can replace the underlying model with minimal architectural change.