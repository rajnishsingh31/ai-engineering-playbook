# Production Fundamentals #3 — AI Observability

Traditional software asks:

"Why did my code fail?"

AI systems ask:

"Why did the model make that decision?"

Those are very different problems.

Why AI Needs New Observability

Suppose your Financial Document Intelligence system returns:

Microsoft's ARR growth = 18%

The user says:

That's wrong.

Where do you start?

Traditional logs won't tell you.

You need to answer questions like:

Did retrieval fail?
Did reranker fail?
Was the prompt bad?
Did the model hallucinate?
Was the wrong document indexed?
Was the citation incorrect?

This is AI observability.

## The AI Pipeline

```text
User Question
      │
      ▼
Prompt Builder
      │
      ▼
Retriever
      │
      ▼
Reranker
      │
      ▼
LLM
      │
      ▼
Tool Calls
      │
      ▼
Final Answer
```

**Every step should be observable.
**
**The Five Things You Must Observe**

## This is my mental model.

1. Request
2. Context
3. Reasoning
4. Tools
5. Outcome

Remember these five.

### 1. Request

Capture:

Request ID
User (or Tenant)
Time
Model
Prompt Version
Workflow/Profile
Conversation ID

Exactly like distributed systems.

Everything should be traceable.

### 2. Context

This is unique to AI.

Record:

Retrieved documents

Chunk IDs

Similarity scores

Top-K

Reranker scores

Memory retrieved

Conversation summary

If someone asks:

Why did the model answer that?

You should know what information the model actually saw.

Example

User:

Highest ARR growth?

Retriever:

Chunk 12

Chunk 41

Chunk 83

Store:

Chunk IDs

Document IDs

Similarity

Reranker order

Tomorrow you can reproduce exactly what happened.

### 3. Reasoning

This is tricky.

Should we log Chain-of-Thought?

No.

Modern production systems generally do not persist or expose raw internal reasoning.

Instead, log observable decisions.

Example:

Planner:

Need revenue.

Need ARR.

Need EBITDA.

Good.

Not:

LLM internal reasoning...

Better to log:

Decision:
Invoke search_financial_metrics

Reason:
Revenue missing

Decision traces > reasoning traces.

### 4. Tool Calls

Every tool call should record:

Tool Name

Parameters

Latency

Retries

Status

Error

Duration

Example:

Tool:
SearchFinancials

Input:
Microsoft

Duration:
140ms

Status:
Success

If a workflow fails:

You'll immediately know which tool caused it.

### 5. Outcome

Record:

Latency

Cost

Tokens

Citations

Confidence

Evaluation score

User feedback

This feeds your evaluation system.

AI Traces

Think of this like OpenTelemetry.

Instead of:
```text
API A

↓

API B

↓

SQL

AI traces become:

Prompt

↓

Retrieval

↓

Reranker

↓

Planner

↓

Tool

↓

LLM

↓

Evaluation
```
Everything belongs to one trace.

Example Trace

```text
Request

↓

Retriever
128ms

↓

Reranker
42ms

↓

Planner
340ms

↓

Tool
SQL
90ms

↓

LLM
4.8 sec

↓

Citation Validation
30ms

↓

Response
```

Now debugging becomes easy.

### Prompt Versioning

One of the most overlooked production features.

Never log only:

Prompt

Instead:

Prompt Version

Prompt Template Version

Model Version

Retriever Version

Embedding Version

Example:

Prompt:
v17

Retriever:
v5

Embedding:
text-embedding-3-large

Model:
GPT-5.5

Months later you'll know exactly what produced the answer.

### Cost Tracking

Enterprise AI spends real money.

Per request capture:

Prompt Tokens

Completion Tokens

Embedding Tokens

Cost

Model

Latency

Example:

GPT

Input
2300

Output
450

Cost
$0.03

Over millions of requests this matters enormously.

AI-Specific Metrics

Traditional metrics:

CPU

Memory

Latency

Errors

## AI metrics:

Hallucination Rate

Groundedness

Citation Accuracy

Tool Success Rate

Retrieval Recall

Prompt Success Rate

Evaluation Score

Token Usage

Average Context Size

Memory Hits

These tell you whether the AI is healthy.

### Failure Analysis

Suppose users complain.

Observability helps isolate the problem.
```text
Wrong Answer
      │
      ├── Wrong Retrieval?
      │
      ├── Wrong Prompt?
      │
      ├── Wrong Tool?
      │
      ├── Hallucination?
      │
      └── Missing Document?
```
Without traces you're guessing.

### Dashboards

Imagine a production dashboard.

Today's Requests

12,340

Average Latency

2.8 sec

Average Cost

$0.021

Hallucination Rate

0.7%

Citation Accuracy

98.6%

Tool Failure

0.2%

Top Failed Prompt

Revenue Comparison

This is what AI operations teams monitor daily.

### Replay

One of the most powerful capabilities.

Given a Request ID:

Replay:
```text
Question

↓

Retrieved Chunks

↓

Prompt

↓

Model

↓

Tool Calls

↓

Response
```
You can reproduce customer issues.

Correlation IDs

Everything should share one ID.
```text
Workflow ID

↓

Prompt

↓

Retriever

↓

Planner

↓

Tool

↓

Evaluation

↓

Audit
```
Exactly like microservices.
```text
AI Observability Stack
                 Request
                    │
                    ▼
             Trace Context
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Prompt Logs   Tool Logs    Retrieval Logs
      ▼             ▼             ▼
      └─────────────┼─────────────┘
                    ▼
            Evaluation Logs
                    ▼
             Metrics Store
                    ▼
             Dashboards
```
### What Should NOT Be Logged

This is just as important.

Avoid logging:

raw API keys,
secrets,
customer passwords,
sensitive personal data,
full chain-of-thought,
confidential documents unnecessarily.

Instead:

redact,
tokenize,
hash,
mask.

Observability should never become a security risk.

### AI Architect Checklist

Every production AI system should answer:

Which prompt version handled this request?
Which chunks were retrieved?
Which tools were called?
How much did it cost?
Why did it fail?
Can I replay it?
Can I compare it with last week's version?

If you can answer those seven questions, your AI system is observable.

The Three Pillars of Production AI

We've now covered three of the most important topics:
```text
Evaluation
      │
Measures quality

Security
      │
Protects the system

Observability
      │
Explains the system
```
Almost every production AI platform is built around these three pillars.