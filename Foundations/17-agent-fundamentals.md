# Chapter 17 — Agent Fundamentals

You now understand the main building blocks:

LLM
RAG
Memory
Structured outputs
Tool calling
State management
Workflow orchestration

**An agent combines some of these pieces to pursue a goal through multiple decisions.**

The important word is:

## Decisions

A normal application follows a path designed in advance.

An agent can decide which path to take next.

## Deterministic Workflow vs Agent

Consider a financial document workflow:

Upload documents
      ↓
Extract text
      ↓
Extract metrics
      ↓
Compare companies
      ↓
Generate report

The sequence is fixed.

Even if each step uses an LLM, this is still primarily a workflow, not an agent.

Why?

Because the application already knows:

which steps exist,
their order,
when the workflow is complete.

The LLM performs work inside the steps, but it does not control the overall path.

Now consider this request:

Investigate why Company A’s profitability declined and produce a supported explanation.

The system may need to decide:

Should I inspect revenue?
Should I inspect gross margin?
Should I compare operating expenses?
Should I search management commentary?
Should I check prior periods?
Should I inspect restructuring charges?

The next action depends on what earlier actions reveal.

That is more agent-like.

A Working Definition

## Agent

An agent is a system that:

receives a goal,
observes the current state,
decides the next action,
executes that action through tools,
observes the result,
repeats until it reaches a stopping condition.
Goal
  ↓
Observe
  ↓
Decide
  ↓
Act
  ↓
Observe result
  ↓
Continue or stop

This loop is the foundation of agentic behavior.

### The Agent Loop

```text
                   ┌────────────────────┐
                   │        Goal        │
                   └─────────┬──────────┘
                             ▼
                   ┌────────────────────┐
                   │ Observe state and  │
                   │ available evidence │
                   └─────────┬──────────┘
                             ▼
                   ┌────────────────────┐
                   │ Choose next action │
                   └─────────┬──────────┘
                             ▼
                   ┌────────────────────┐
                   │ Execute via tool   │
                   │ or reasoning step  │
                   └─────────┬──────────┘
                             ▼
                   ┌────────────────────┐
                   │ Evaluate result    │
                   └─────────┬──────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
            Goal complete           Continue loop
```

The agent is not just the LLM.

The full agent includes:

LLM
+
tools
+
state
+
control loop
+
policies
+
stopping conditions

Why an LLM Alone Is Not an Agent

Suppose you ask an LLM:

Give me a plan for comparing three companies.

It returns:

1. Extract revenue.
2. Calculate growth.
3. Compare margins.

That is planning, but not an agent.

The model did not:

execute tools,
inspect results,
adapt its next step,
track progress,
determine completion.

It produced text.

An agent must participate in an execution loop.

### Agent vs Chatbot

A chatbot usually follows:

User message
    ↓
   LLM
    ↓
Response

An agent follows:

User goal
    ↓
LLM selects action
    ↓
Tool execution
    ↓
New observation
    ↓
LLM selects another action
    ↓
...
    ↓
Final result

A chatbot answers.

An agent acts and adapts.

### Agent vs Workflow

The difference is not binary. Systems exist on a spectrum.

Highly deterministic workflow
A → B → C → D

The path is fixed.

Conditional workflow
A
↓
If document is scanned → OCR
Else → text extraction
↓
C

The application still defines every branch.

Agent-controlled workflow

Goal
↓
LLM chooses one of several tools
↓
Result changes next decision
↓
LLM chooses again

The model controls some transitions.

#### A Practical Test

Ask:

**Who decides the next step?**

If the answer is:

application code → workflow,
predefined state machine → workflow,
**LLM based on observations → agentic behavior.**

Another useful question:

**Could the path differ substantially for two similar requests?**
**
If yes, the system may be agentic.**

##### Example: Repository Investigation Agent

Goal:

Find why CreateSubmissionAsync began failing after the latest deployment.

The agent may start with:

1. Search symbol definition
2. Inspect recent commits
3. Search deployment logs
4. Compare failure timestamps
5. Inspect callers
6. Search related incidents

But after examining logs, it may discover:

Authentication failures began immediately after certificate rotation.

It may then change direction:

Search certificate configuration
↓
Inspect secret version
↓
Compare deployment configuration

The path was not fully known in advance.

## The Core Components of an Agent

A useful enterprise agent has at least seven components:

Goal
Planner / Decision Maker
State
Tools
Executor
Evaluator
Control Policy

### 1. Goal

The goal defines the desired outcome.

Bad goal:

Look at these documents.

Better goal:

Compare ARR growth across the uploaded companies and identify
the strongest performer using cited evidence.

A good goal should clarify:

expected result,
scope,
constraints,
completion criteria.

Agents perform poorly with vague goals because they do not know when to stop.

###2. Planner or Decision Maker

The planner decides what to do next.

Example structured output:

{
  "action": "search_financial_metric",
  "reason": "ARR values are missing for Company B",
  "arguments": {
    "company": "Company B",
    "metric": "ARR"
  },
  "expected_information": "Current-period ARR and prior-period ARR"
}

The planner should return a controlled action, not unrestricted prose.

The application validates the action before executing it.

### 3. State

The agent needs authoritative state such as:

{
  "goal": "Compare ARR growth",
  "completed_actions": [
    "extract_company_a_arr",
    "extract_company_c_arr"
  ],
  "known_facts": {
    "company_a_arr_growth": 0.25,
    "company_c_arr_growth": 0.18
  },
  "missing_information": [
    "company_b_arr_growth"
  ],
  "iteration": 4
}

The LLM should not reconstruct all of this from conversation text on every loop.

### 4. Tools

Tools expose controlled capabilities:

search_documents
extract_financial_metrics
calculate_growth
search_repository
read_logs
send_email

The agent chooses among them, but the Tool Executor still controls:

validation,
authorization,
execution,
retries,
rate limits,
auditing.

Agentic does not mean uncontrolled.

### 5. Executor

The executor turns an approved action proposal into a real operation.

LLM proposes:
search_company_metric

        ↓

Tool Executor validates:
tool exists
arguments valid
user authorized

        ↓

Worker executes search

        ↓

Normalized observation returned

The executor remains deterministic.

### 6. Evaluator

The evaluator asks:

Did this action move us toward the goal?
Is the evidence sufficient?
Is the result trustworthy?
Should we continue?

Example:

{
  "goal_satisfied": false,
  "evidence_sufficient": false,
  "missing_information": [
    "Company B prior-year ARR"
  ],
  "recommended_next_step": "search_company_b_prior_arr"
}

Without evaluation, an agent may endlessly call tools without knowing whether it is making progress.

### 7. Control Policy

The control policy imposes hard limits.

Examples:

Maximum 8 tool calls
Maximum 2 minutes
Maximum ₹10 estimated model cost
No write tools without approval
No access outside the current tenant
Stop after three consecutive low-value actions

The control policy is deterministic application logic.

The LLM does not get to remove these limits.

Plan-Then-Execute

One common pattern is:

Goal
 ↓
Generate plan
 ↓
Execute step 1
 ↓
Execute step 2
 ↓
Execute step 3
 ↓
Final answer

Example plan:

{
  "steps": [
    {
      "id": "step-1",
      "action": "extract_arr",
      "company": "Company A"
    },
    {
      "id": "step-2",
      "action": "extract_arr",
      "company": "Company B"
    },
    {
      "id": "step-3",
      "action": "compare_growth"
    }
  ]
}

This provides predictability and observability.

But it has a weakness:

The initial plan may become obsolete after new evidence appears.

Replanning

Suppose the agent discovers:

Company B reports bookings, not ARR.

The initial plan may no longer work.

A robust system can replan:

Initial plan
    ↓
Execute
    ↓
Unexpected observation
    ↓
Update state
    ↓
Create revised plan

Replanning should be bounded.

Otherwise, the agent may continuously rewrite its plan instead of completing work.

Step-by-Step Agent Loop

Another pattern chooses only the next action:

Observe current state
        ↓
Choose one action
        ↓
Execute
        ↓
Update state
        ↓
Repeat

This is more adaptive than a fixed plan.

But it can also be:

slower,
more expensive,
harder to predict,
more prone to loops.
Hybrid Pattern

### A strong design often combines both:

Create high-level plan
        ↓
Execute one step at a time
        ↓
Evaluate after each step
        ↓
Replan only when necessary

This balances structure and adaptability.

### ReAct

A common conceptual pattern is called ReAct:

Reason
  ↓
Act
  ↓
Observe
  ↓
Reason again

Example:

Observation:
Company B ARR is not in the investor presentation.

Action:
Search annual filing.

Observation:
Annual filing reports subscription revenue but not ARR.

Action:
Search earnings-call transcript.

Observation:
Management states ARR reached $90M.

Final:
Use the transcript value with citation and lower confidence.

The important concept is not exposing internal reasoning text.

The important concept is the operational cycle:

Observation → action → new observation

### What Counts as an Observation?

An observation is a normalized result available to the agent.

Examples:

{
  "status": "success",
  "data": {
    "metric": "ARR",
    "value": 120000000,
    "currency": "USD"
  },
  "source": {
    "document_id": "company-a-report",
    "page": 18
  }
}

Or:

{
  "status": "not_found",
  "searched_sources": [
    "investor_deck",
    "annual_report"
  ]
}

#### Observations should be:

structured,
concise,
relevant,
sanitized,
traceable.

Do not dump massive raw tool output into every agent iteration.

### Agent Scratchpad

The agent needs a working representation of progress.

Conceptually:

{
  "goal": "Identify strongest ARR growth",
  "facts": [
    {
      "company": "A",
      "arr_growth": 0.25,
      "source": "doc-a-page-18"
    }
  ],
  "open_questions": [
    "What is Company B ARR growth?"
  ],
  "failed_actions": [
    "search Company B investor deck"
  ]
}

This is often called a scratchpad.

But treat it as workflow state, not mysterious LLM memory.

It should be:

structured,
persisted when needed,
compacted,
validated,
bounded.
Completion Criteria

#### An agent needs an objective stopping condition.

Weak criterion:

Stop when you think you are done.

Better:

Complete when:
- ARR growth is available for all companies, or explicitly unavailable;
- every metric has provenance;
- values pass business validation;
- companies are ranked;
- uncertainty is disclosed.

A deterministic evaluator can check much of this.

{
  "all_companies_processed": true,
  "all_claims_have_sources": true,
  "business_validation_passed": true,
  "comparison_generated": true
}

Then the system can safely stop.

Why Agents Loop Forever

Agents may loop when:

the goal is vague,
completion criteria are unclear,
tools repeatedly return no data,
the model keeps trying similar searches,
there is no action budget,
failed actions are not recorded.

Example:

Search ARR
↓
Not found
↓
Search ARR again with similar wording
↓
Not found
↓
Search again

Controls should include:

Action deduplication
Maximum iterations
Progress detection
Failure counters
Alternative-strategy limits
Time and cost budget
Progress Detection

After every action, determine whether new useful information was gained.

Example:

{
  "new_facts_added": 0,
  "open_questions_resolved": 0,
  "duplicate_action": true,
  "progress_score": 0
}

If several consecutive actions make no progress:

Stop
↓
Return partial result
↓
Explain missing evidence

This is better than endless searching.

#### Tool Selection Errors

An agent may choose the wrong tool.

User asks:

What changed in the latest commit?

The agent may incorrectly choose:

vector_search_documentation

instead of:

git_commit_history

##### Reduce this through:

precise tool descriptions,
narrow tools,
structured routing,
examples,
tool eligibility rules,
evaluator feedback.

The application can also restrict the available tools for each workflow stage.

###### Dynamic Tool Availability

Do not expose every tool on every iteration.

For a read-only investigation:

Available:
search_documents
search_logs
search_commits
calculate_metric

Hide:

delete_repository
deploy_build
send_email

This reduces:

incorrect tool selection,
prompt size,
security risk,
accidental side effects.

Tool exposure is part of authorization and workflow design.

###cAutonomy Levels

Agentic systems can have different autonomy levels.

#### Level 0 — Answer only
LLM produces text.

#### Level 1 — Recommend actions
LLM suggests tools or steps.
Human executes.

#### Level 2 — Execute read-only actions
Agent searches documents and systems automatically.

#### Level 3 — Execute reversible writes with approval
Agent drafts email or creates ticket.
Human approves before submission.
#### Level 4 — Execute bounded low-risk writes automatically
Agent adds an internal label or updates temporary workflow state.

#### Level 5 — Broad autonomous action
Agent independently executes consequential operations.

Most enterprise systems should stay around Levels 2–4, depending on risk.

More autonomy is not automatically better.

## Risk-Based Autonomy

Use autonomy based on:

Impact
Reversibility
Confidence
Authorization
Environment
Data sensitivity

Example:

Search documentation
→ automatic

Generate report draft
→ automatic

Send report internally
→ approval may be required

Publish earnings statement
→ mandatory human review

Delete financial records
→ likely prohibited
Agent Boundaries

## A safe agent operates inside a box:

Allowed tools
Allowed data
Allowed tenants
Maximum actions
Maximum cost
Maximum duration
Approval requirements
Completion criteria

Inside the box, it may adapt.

Outside the box, it cannot act.

Bounded autonomy
≠
Unlimited autonomy
Agent as a State Machine

Even an agent should have deterministic outer states.

Created
  ↓
Planning
  ↓
Executing
  ↓
Evaluating
  ├── Continue → Executing
  ├── Replan → Planning
  ├── Approval needed → Awaiting Approval
  ├── Complete → Completed
  └── Cannot proceed → Partial / Failed

**The LLM may decide among permitted transitions.

The workflow engine enforces whether those transitions are valid.**

Proposed Action vs Executed Action

Keep these separate.

{
  "proposed_action": {
    "tool": "send_email",
    "arguments": {
      "recipient": "alice@example.com"
    }
  }
}

This does not mean the email was sent.

The lifecycle is:

Proposed
  ↓
Validated
  ↓
Authorized
  ↓
Approved
  ↓
Executed
  ↓
Observed
  ↓
Committed to state

Never allow the model to treat a proposal as success.

## Agent Architecture
                       User Goal
                           │
                           ▼
                     Goal Validator
                           │
                           ▼
                 Agent Workflow Engine
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        State Store    Control Policy   Event Log
             │
             ▼
       Planner / LLM
             │
             ▼
       Proposed Action
             │
             ▼
        Action Validator
             │
             ▼
        Tool Executor
             │
             ▼
       External Systems
             │
             ▼
        Observation
             │
             ▼
          Evaluator
             │
       ┌─────┼────────┐
       ▼     ▼        ▼
   Continue Replan   Stop

This architecture separates probabilistic and deterministic responsibilities.

### Responsibility Boundaries

#### LLM / Planner

Responsible for:

interpreting the goal,
proposing a plan,
choosing among allowed actions,
adapting to evidence,
synthesizing findings.

Not responsible for:

authorization,
retries at the network level,
committing workflow state,
executing external operations,
deciding whether security policy applies.

#### Workflow Engine

Responsible for:

lifecycle,
state transitions,
action budgets,
scheduling,
cancellation,
checkpoints,
completion status.

#### Tool Executor

Responsible for:

schema validation,
authentication,
authorization,
policy enforcement,
deterministic execution,
retries,
normalization,
audit.

#### Evaluator

Responsible for:

checking completeness,
validating evidence,
assessing progress,
deciding whether another iteration is useful.

**The evaluator can use deterministic rules, an LLM, or both.**

##### Deterministic Evaluator vs LLM Evaluator

###### Deterministic checks

Good for:

Are all required companies processed?
Does every claim have a citation?
Are numeric fields valid?
Has the tool-call budget been exceeded?
LLM-based evaluation

Useful for:

Does the evidence adequately explain the decline?
Are there contradictions across sources?
Is the report coherent?

Use deterministic checks whenever possible.

Use LLM judgment only where semantic interpretation is required.

## Single-Agent Example

Financial investigation goal:

Determine why Company A’s operating margin declined.

The agent could do:

1. Extract current and prior operating margin.
2. Extract revenue growth.
3. Extract gross-margin movement.
4. Extract operating-expense categories.
5. Search management commentary.
6. Compare numeric changes with commentary.
7. Generate evidence-backed explanation.

Potential state:

{
  "goal": "Explain operating-margin decline",
  "facts": {
    "operating_margin_current": 0.12,
    "operating_margin_prior": 0.18,
    "gross_margin_change": -0.02,
    "sales_marketing_growth": 0.35
  },
  "hypotheses": [
    {
      "cause": "higher sales and marketing expense",
      "status": "supported"
    },
    {
      "cause": "gross-margin compression",
      "status": "partially_supported"
    }
  ],
  "missing_information": [
    "restructuring charges"
  ]
}

The agent continues only if the missing information materially affects the conclusion.

When Not to Use an Agent

Do not use an agent simply because it sounds modern.

## Avoid an agent when:

the steps are known,
the sequence is stable,
predictability is important,
the task is high volume,
latency and cost matter,
deterministic code can solve it,
risk is high.

For example:

Upload PDF
→ extract text
→ chunk
→ embed
→ store

This should normally be a workflow.

There is little value in asking an LLM:

What should I do after extracting text?

The application already knows.

When an Agent Helps

Use an agent when:

the path cannot be fully predefined,
evidence changes the next step,
several tools may be relevant,
the task involves investigation,
iterative search is necessary,
partial information must be handled adaptively.

Examples:

Root-cause investigation
Research synthesis
Repository debugging
Complex support triage
Financial due diligence
Incident analysis
Best Enterprise Pattern

## A strong default is:

**Deterministic workflow outside, bounded agent inside.**

Example:

Upload and register documents
        ↓
Deterministic extraction pipeline
        ↓
Deterministic validation
        ↓
Bounded investigation agent
        ↓
Deterministic report checks
        ↓
Human approval
        ↓
Deterministic delivery

The agent is used only where adaptability adds value.

This is safer and easier to operate than making the entire application autonomous.

## Architect Perspective

For the Financial Document Assistant:

```text

                Deterministic Workflow
                         │
       ┌─────────────────┼──────────────────┐
       ▼                 ▼                  ▼
Document ingestion   Metric extraction   Validation
                                              │
                                              ▼
                                  Bounded Analysis Agent
                                              │
                              ┌───────────────┼───────────────┐
                              ▼               ▼               ▼
                        Search evidence   Calculate      Test hypotheses
                                              │
                                              ▼
                                  Evidence-backed findings
                                              │
                                              ▼
                                  Deterministic citation check
                                              │
                                              ▼
                                         Final report

```

The agent does not control:

file authorization,
tenant boundaries,
data retention,
email delivery,
financial calculations that deterministic code can perform.

It controls the investigative path within approved evidence sources.

## Manager Perspective

When someone proposes an agent, ask:

Why is a deterministic workflow insufficient?
Which decisions require model judgment?
What tools can the agent access?
Which actions create side effects?
What is the action, time, and cost budget?
How do we know it has completed?
How do we evaluate whether its work is correct?
What happens when it cannot proceed?
Where is human approval required?
Can every action be audited?

These questions separate production architecture from an impressive demo.

## Core Principles

An LLM is not automatically an agent.

A workflow can use LLMs without being agentic.

An agent chooses actions based on observations.

The agent loop must have state and stopping conditions.

The LLM proposes; deterministic systems authorize and execute.

More autonomy is not always better.

Use deterministic workflows whenever the path is known.

Use bounded agents where adaptation creates real value.

## Important Responsibility:

Humans define success, safety, and limits. The LLM decides the adaptive path. The workflow engine decides whether execution may continue or must stop.

## Q4 — Bounded Financial-Analysis Agent

You did not answer this part, so here is a reference architecture based on the concepts you have learned.

### 1. Goal representation
{
  "goal_id": "goal-481",
  "objective": "Compare three companies and identify the financially strongest",
  "companies": [
    "Company A",
    "Company B",
    "Company C"
  ],
  "required_dimensions": [
    "growth",
    "profitability",
    "cash_generation",
    "liquidity"
  ],
  "constraints": {
    "use_only_authorized_sources": true,
    "all_claims_require_provenance": true,
    "do_not_invent_missing_values": true
  }
}

The goal should define what “financially strongest” means. Otherwise, the model may choose an arbitrary interpretation.

### 2. Deterministic outer workflow
Register documents
        ↓
Extract text and tables
        ↓
Normalize standard metrics
        ↓
Validate units and periods
        ↓
Bounded analysis agent
        ↓
Deterministic citation validation
        ↓
Generate report
        ↓
Human approval for external delivery

Only the investigative portion needs agentic behavior.

### 3. Planner

The planner proposes one next action from an approved set.

{
  "action": "search_metric_evidence",
  "company": "Company B",
  "metric": "operating_cash_flow",
  "source_type": "annual_report",
  "reason_code": "REQUIRED_METRIC_MISSING",
  "expected_information": "Current and prior period operating cash flow"
}

The reason should be represented using controlled reason codes where possible.

### 4. Agent state and scratchpad
{
  "iteration": 5,
  "known_facts": [
    {
      "company": "Company A",
      "metric": "revenue_growth",
      "value": 0.18,
      "source_reference": "doc-a-page-12"
    }
  ],
  "missing_information": [
    {
      "company": "Company B",
      "metric": "operating_cash_flow"
    }
  ],
  "failed_strategies": [
    {
      "strategy": "search_investor_deck_for_cash_flow",
      "result": "not_found"
    }
  ],
  "open_hypotheses": [
    {
      "hypothesis": "Company A has stronger growth but weaker liquidity",
      "status": "partially_supported"
    }
  ]
}

The scratchpad should store structured progress, not an unlimited transcript of model reasoning.

### 5. Tool set

Expose only relevant read-oriented tools:

search_document
extract_metric
search_financial_table
retrieve_prior_period
calculate_growth
normalize_currency
compare_metrics
retrieve_market_data

Potentially dangerous or irrelevant tools should remain hidden:

send_email
purchase_stock
delete_document
modify_financial_record

### 6. Action validator

Before execution, validate:

the tool is allowlisted,
the source belongs to the current tenant,
the company is in scope,
the metric is permitted,
the action has not already been attempted,
the remaining budget is sufficient,
no prohibited data source is requested.
Proposed action
      ↓
Schema validation
      ↓
Policy validation
      ↓
Scope validation
      ↓
Duplicate-action check
      ↓
Execute

### 7. Evaluator

Use deterministic rules first.

{
  "required_companies_processed": true,
  "metrics_comparable": true,
  "all_claims_have_sources": true,
  "numeric_validation_passed": true,
  "missing_critical_metrics": false
}

Use an LLM evaluator only for semantic questions such as:

Does the evidence support the explanation?
Are management claims contradicted by the numbers?
Is the final conclusion balanced?

### 8. Progress detection

After every action, calculate:

{
  "new_facts": 2,
  "resolved_questions": 1,
  "duplicate_information": false,
  "progress_score": 0.75
}

Stop or change strategy after:

3 consecutive zero-progress actions
2 failed attempts using the same source type
6 attempts for one missing metric

Exact thresholds depend on the workflow.

### 9. Budgets

Your proposed values can be represented as:

{
  "maximum_duration_seconds": 600,
  "maximum_llm_tokens": 20000,
  "maximum_tool_calls": 15,
  "maximum_replans": 2,
  "maximum_external_api_cost": 2.00,
  "maximum_consecutive_no_progress_actions": 3
}

A token budget alone is insufficient because tools and external APIs also consume time and money.

### 10. Approval boundaries

Read-only financial research may run automatically.

Require approval for:

sending the report externally,
sharing confidential documents,
publishing conclusions,
modifying source data,
triggering transactions,
using paid data sources above a cost threshold.
Research and draft
→ automatic

External distribution
→ approval

Financial transaction
→ outside agent scope

### 11. Completion criteria

The agent may claim a final ranking only when:

All required companies evaluated
All ranking dimensions available or explicitly unavailable
Periods and currencies normalized
Evidence is comparable
Every important claim has provenance
No unresolved critical contradiction remains
Scoring method is applied consistently
Uncertainty is disclosed

Otherwise, it must return a partial or qualified result.

### 12. Auditability

Record:

user goal,
model and prompt versions,
proposed actions,
policy decisions,
executed tools,
sanitized arguments,
observations,
citations,
budgets consumed,
replanning events,
final completion reason.

This lets an auditor answer:

Why did the system conclude Company A was strongest?

What Should Not Be Agent-Controlled

The agent should not control:

authentication,
authorization,
tenant boundaries,
schema enforcement,
network retries,
secret access,
currency arithmetic,
score calculation formulas,
workflow state commits,
approval enforcement,
external writes,
retention policy,
budget enforcement.

A strong production pattern remains:

```text

Deterministic ingestion
        ↓
Deterministic extraction and normalization
        ↓
Bounded agentic investigation
        ↓
Deterministic validation
        ↓
Controlled delivery

```

The key lesson is:

Use the agent to decide where to investigate next—not to control everything the system does.