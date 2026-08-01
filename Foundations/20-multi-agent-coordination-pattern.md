# Chapter: Multi-Agent Coordination Patterns

Think of an organization.

You have:

CEO
Managers
Engineers
HR
Finance

There are many ways these people can coordinate.

Exactly the same is true for AI agents.

We'll cover five major patterns:

1. Manager–Worker
2. Hierarchical
3. Peer-to-Peer
4. Blackboard
5. Event-Driven

By the end of this chapter, you'll be able to look at CrewAI, LangGraph, AutoGen, or Microsoft's frameworks and immediately recognize which pattern they implement.

## Pattern 1 — Manager–Worker (The most common)

This is the easiest pattern to understand.

```text

              Manager Agent
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Research Agent  Finance Agent  Writer Agent
      │             │             │
      └─────────────┼─────────────┘
                    ▼
              Manager decides

```

**The workers do not talk to each other.

Everything flows through the manager.**

### Responsibilities

Manager

Owns:

overall goal
workflow state
planning
delegation
conflict resolution
completion decision

Think of it as the AgentRuntime we built earlier.

Workers

Workers own only their local task.

For example:

Research Agent:

Goal:
Find all revenue numbers.

Finance Agent:

Goal:
Normalize currencies.

Writer Agent:

Goal:
Write executive summary.

Notice something important:

Each worker has its own mini AgentRuntime.

They have:

planner
tools
evaluator
state

They are simply bounded to a much smaller goal.

Example

Suppose the user asks:

Analyze this company's annual report.

Manager:

Need:
• revenue
• profit
• risks
• ESG

Manager delegates:

Revenue Agent

Risk Agent

ESG Agent

Each returns structured output.

Manager combines.

No worker ever contacts another worker.

Advantages

Simple.

Easy to debug.

Easy to audit.

Clear ownership.

Workers are reusable.

Parallel execution.

Very common in enterprise systems.

Disadvantages

Manager becomes bottleneck.

Single point of failure.

Manager must understand every worker.

Large context may accumulate at manager.

## Pattern 2 — Hierarchical

Looks similar but is actually different.

```text

                  Director Agent
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        Finance Manager     Engineering Manager
              │                   │
         ┌────┴────┐        ┌─────┴─────┐
         ▼         ▼        ▼           ▼
     Revenue   Expenses  Build     Test Agents

```

**Notice:**

Managers themselves become workers of another manager.

This scales much better.

Imagine 5,000 financial reports.

One manager cannot coordinate all of them.

Instead:

Top Manager

↓

Region Managers

↓

Country Managers

↓

Extraction Agents

Exactly like distributed systems.

## Pattern 3 — Peer-to-Peer

No manager.

     Agent A ◄────► Agent B
        ▲             │
        │             ▼
     Agent D ◄────► Agent C

Agents negotiate.

They request work.

They exchange findings.

They resolve conflicts.

Nobody owns the whole workflow.

Example:

Research Agent:

"I found ARR."

Finance Agent:

"Need currency."

Research Agent:

"Here."

Finance Agent:

"Converted."

Writer Agent:

"I need both."

Advantages

Flexible.

Self-organizing.

No bottleneck.

Disadvantages

Hard to debug.

Lots of messages.

Deadlocks.

Loops.

Duplicate work.

Much harder to test.

## Pattern 4 — Blackboard

This is one of my favorites because it resembles how many enterprise AI systems work.

Nobody talks directly.

Instead, everyone shares a common workspace.

```text

          Shared Blackboard

        ARR = $10B

        Currency = USD

        Confidence = 0.92

        Missing:
        EBITDA

     ▲        ▲         ▲
     │        │         │
Research  Finance   Writer

```

Research Agent writes:

Revenue found.

Finance Agent reads:

Revenue found.

Need currency conversion.

Finance Agent writes:

Converted.

Writer reads latest board.

Nobody messages each other.

Sound familiar?

Our architecture already contains something similar.

We built:

Fact Store
Hypothesis Store
Workflow State

That is essentially a specialized blackboard.

Instead of agents chatting, they collaborate through shared structured state.

This scales much better.

## Pattern 5 — Event-Driven

Very common in cloud architectures.

```text
Research Finished
        │
        ▼
    Event Bus
        │
 ┌──────┴────────┐
 ▼               ▼
Writer      Finance
```

Nobody knows who is listening.

They simply publish events.

DocumentExtracted

RevenueFound

RiskDetected

ApprovalReceived

Interested agents subscribe.

This is how many enterprise workflows are built.

Comparing them

| Pattern        | Best For                 | Weakness                |
| -------------- | ------------------------ | ----------------------- |
| Manager–Worker | Most enterprise copilots | Manager bottleneck      |
| Hierarchical   | Large organizations      | More coordination       |
| Peer-to-Peer   | Research/experimentation | Hard to control         |
| Blackboard     | Knowledge synthesis      | Shared-state complexity |
| Event-Driven   | Long-running workflows   | Event orchestration     |

Which pattern do you think we have already built?

Let's compare it with our runtime.

We have:

Workflow State
Fact Store
Observation Store
Task Queue
Specialized Executors
Planner
Evaluator

Question:

Are our specialized executors actually agents?

Or are they just tools/workers?

That distinction is extremely important