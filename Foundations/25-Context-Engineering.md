# Production Fundamentals #4 — Context Engineering

## The Evolution

Think of the evolution like this:

Prompt Engineering
        ↓
RAG
        ↓
Memory
        ↓
Agents
        ↓
Context Engineering

Early AI applications asked:

"How should I write my prompt?"

Modern AI systems ask:

"What information should the model see?"

That's a much bigger question.

The Biggest Misconception

People think this is the important part:

You are a helpful assistant...

It isn't.

This is.

System Prompt
+
Retrieved Documents
+
Conversation History
+
Memory
+
Tool Results
+
Current Workflow State
+
User Question

That entire package is called Context.

The prompt is only one small piece.

The LLM Doesn't Know Reality

The LLM only knows what is inside the current context window.

Think of it as a human taking an exam.

Suppose I give you:

5 books
3 emails
2 meeting notes
Architecture diagram
Customer question

You answer based only on those.

Exactly the same for an LLM.

So the quality of the answer depends on the quality of the context, not just the model.

Context = Working Memory

Remember when we studied memory?

Long-term Memory
        │
        ▼
Retriever
        ▼
Working Memory
        ▼
LLM

## Context Engineering is essentially building the LLM's working memory for one request.

What Goes Into Context?

A production system may assemble context from many sources.

###               Context Builder
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
User Question     Conversation      Memory
      ▼               ▼                ▼
Retrieval        Workflow State     Tool Results
      ▼               ▼                ▼
          Prompt Assembly
                 ▼
                LLM

Notice:

The Context Builder becomes one of the most important components.

The Context Builder

Earlier we designed:

Planner
State Manager
Memory
Retriever

Now imagine another component:

Context Builder

### Its responsibility is:

**Construct the smallest, highest-quality context that enables the model to answer correctly.**

That's it.

Why Bigger Context Isn't Better

Many beginners think:

More information = Better answer.

Usually false.

Imagine asking:

Revenue?

and giving the model:

500 pages of annual reports
200 Slack messages
50 meeting notes
100 API docs

Most of that is noise.

Large context causes:

higher cost,
higher latency,
distraction,
"Lost in the Middle",
more opportunities for prompt injection.

**Good Context Engineering removes irrelevant information.**

### Context Sources

Think in layers.

User Request
        │
        ▼
Conversation
        │
        ▼
     Memory
        │
        ▼
       RAG
        │
        ▼
      Tools
        │
        ▼
  Workflow State

Not every request needs every layer.

Example 1 — Financial Comparison

User:

Which company has highest ARR growth?

Need:

✓ Annual reports

✓ ARR metrics

✓ Growth numbers

✗ GitHub repository

✗ Customer emails

✗ Meeting notes

The Context Builder should include only relevant information.

Example 2 — Incident Investigation

Need:

✓ Deployment history

✓ Logs

✓ Metrics

✓ Previous incidents

✓ Current hypotheses

✗ HR policy

✗ Financial reports

Again, selective context.

### Context Prioritization

Suppose the token budget allows only 8 chunks.

Retriever found:

12 good chunks

How do we choose?

Possible signals:

Retrieval score

Reranker score

Freshness

Authority

Recency

Document type

Importance

Workflow stage

This is Context Engineering.

### Context Compression

Suppose:

Meeting transcript:

25 pages

Should you pass all 25?

Usually not.

Instead:

Meeting

↓

Summarizer

↓

2-page summary

↓

LLM

Compression reduces:

cost,
latency,
distraction.
Context Ordering

We already touched this with "Lost in the Middle."

### Ordering matters.

Bad:

Question

Random Chunks

Instructions

Better:

Instructions

Question

Most Relevant Evidence

Supporting Evidence

Examples

Less Relevant Information

The model reads sequentially.

Order influences attention.

### Context Freshness

Imagine two documents.

2023 API

2026 API

Question:

Latest CreateSubmissionAsync parameters?

The Context Builder should strongly prefer:

2026 API

Freshness becomes a ranking signal.

### Context Trust

Not all context is equally trustworthy.

Official API
        High

Approved Architecture
        High

Wiki
        Medium

Slack
        Medium

Customer Upload
        Low

Internet
        Lowest

The model should know the trust level.

Example metadata:

{
  "source": "customer_upload",
  "trust": "low"
}
Context Isolation

Imagine multi-tenant AI.

User A

↓

Retriever

↓

Documents

↓

LLM

User B should never see:

User A's documents.

**Context isolation is just as important as database isolation.**

### Dynamic Context

Context changes during execution.

Example:

Planner:

Need revenue

Tool returns:

Revenue = 100M

Now the Context Builder updates:

Context v1

↓

Tool Result

↓

Context v2

The context evolves throughout the workflow.

### Context Versioning

Exactly like prompts.

Store:

Prompt Version

Retriever Version

Embedding Version

Context Version

This makes debugging much easier.

Context Window Is a Budget

Think of context like RAM.

Limited.

Expensive.

Valuable.

You don't fill RAM with unnecessary files.

You don't fill context with unnecessary tokens.

Architect's Mental Model

This is the one diagram I want you to remember.

          Available Information
                  │
                  ▼
          Context Builder
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
 Remove Noise  Rank  Compress
      ▼           ▼            ▼
          Assemble Context
                  ▼
                 LLM

The Context Builder is essentially an intelligent operating system for the LLM's working memory.

Prompt Engineering vs Context Engineering

| Aspect            | Prompt Engineering  | Context Engineering       |
| ----------------- | ------------------- | ------------------------- |
| Primary focus     | Writes instructions | Builds working memory     |
| Main concern      | Focuses on wording  | Focuses on information    |
| Nature            | Mostly static       | Dynamic                   |
| Scope             | Single prompt       | Entire request pipeline   |
| Production impact | Small impact        | Huge impact in production |

That's why the industry is shifting terminology.

AI Architect Rule

If I gave you two choices:

Spend a week improving the prompt.
Spend a week improving context quality.

For most enterprise AI systems, improving context quality produces a much larger improvement.

That's why companies invest heavily in retrieval, ranking, compression, filtering, freshness, and context assembly.

One New Insight (not in most courses)

I want to connect this with everything we've built.

Remember our ContextBuilder inside AgentRuntime?

It started as:

State
+
Memory
+
Retriever

After today's lesson, it becomes much richer:

Conversation
+
Workflow State
+
Memory
+
Facts
+
Hypotheses
+
Retrieved Chunks
+
Tool Results
+
Policies
+
Budgets
+
Current Task
+
User Question
        ↓
Context Builder
        ↓
Optimized Context
        ↓
LLM

Notice something interesting:

The Context Builder has quietly become one of the most important components in the entire architecture—almost as important as the Planner itself.