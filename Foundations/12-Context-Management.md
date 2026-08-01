# Chapter 12 — Context Management

## Types of Context

Think of context as four buckets.

Conversation Context

↓

User Context

↓

Retrieved Knowledge

↓

System Context

Let's examine each.

### 1. Conversation Context

This is what has happened in the current chat.

Example:

User:
Explain BM25.

Assistant:
...

User:
How does it compare to vector search?

The second question depends on the first.

Conversation context should usually be retained.

### 2. User Context

This is information about the user.

For example, if I know that you:

are an Engineering Manager,
are learning AI Engineering,
prefer first-principles explanations,

I don't need you to repeat that every session.

Notice something.

This is not conversation history.

It's longer-lived.

### 3. Retrieved Knowledge

Suppose earlier in the conversation we retrieved:

WHCP Guide

Section 4.2

The next question is:

What exceptions are there?

Should we retrieve again?

Maybe.

Or maybe we reuse the retrieved evidence.

This is another engineering decision.

### 4. System Context

Things like:

You are an enterprise assistant.

Do not hallucinate.

Use citations.

This usually changes very little.

### The Context Pyramid

A useful mental model is:

                 System
                    ▲
                 User
                    ▲
            Conversation
                    ▲
         Retrieved Evidence

Each layer changes at a different rate.

System → almost never
User → occasionally
Conversation → every turn
Retrieved evidence → depends on the question
**Context Isn't Memory**

This is one of the biggest misconceptions.

Many people think:

Context == Memory

They're different.

**Context

Temporary.**

Example:

We're discussing BM25.

Tomorrow:

Gone.

**Memory

Persistent.**

Example:

User prefers Python.

User is learning AI Engineering.

That may persist across sessions.

A Better Architecture

Instead of:

Entire Chat

↓

LLM

Production systems look more like:

Current Question

↓

Conversation Manager

↓

Relevant History

↓

Prompt Builder

↓

LLM

Notice something.

We introduced a brand new component.

## The Conversation Manager.**

Its job is:

**Decide what history is actually useful.**

### Strategy 1 — Sliding Window

The simplest strategy.

Keep only the last N turns.

Example:

Turn 96

Turn 97

Turn 98

Turn 99

Turn 100

Everything earlier is discarded.

Advantages

Very simple.

Very fast.

Disadvantages

Suppose:

Turn 5

We're talking about extension drivers.

Turn 100

Can they use derived submissions?

Oops.

We forgot what "they" refers to.

### Strategy 2 — Summarization

Instead of keeping every message...

Compress older history.

Example:

Instead of:

20 pages

Store:

Conversation Summary

User is designing a Driver Servicing assistant.

We discussed BM25,
RRF,
chunking,
reranking...

Now the prompt becomes:

Summary

+

Recent Messages

+

Current Question

This is one of the most common production approaches.

### Strategy 3 — Semantic Retrieval Over History

This is where it gets interesting.

Instead of storing chat history as plain text...

Treat it like documents.

Sound familiar?

Exactly.

We chunk it.

Embed it.

Store it.

Then retrieve only relevant conversation history.

Pipeline:

Conversation History

↓

Chunk

↓

Embed

↓

Vector Store

Now user asks:

How does that compare to BM25?

The system retrieves only previous BM25 discussion.

Not the unrelated conversation about GitHub.

Notice Something Beautiful

Everything we've learned about RAG...

Can also be applied to conversation history.

Conversation history is just another knowledge source.

### Strategy 4 — Hybrid Context

Many systems combine multiple strategies.

Current Turn

+

Recent Messages

+

Conversation Summary

+

Retrieved Conversation

+

Retrieved Documents

This provides both recency and long-term continuity.

Example

Imagine this conversation.

Yesterday:

We discussed BM25.

Today:

How is it different from RRF?

The assistant might retrieve yesterday's BM25 explanation instead of requiring you to explain it again.

## Context Compression

Suppose we have:

100 pages

We obviously can't send all of it.

Compression techniques include:

summarization
removing repetition
keeping only decisions
extracting entities
retaining unresolved questions

Notice that we're compressing meaning, not just deleting text.

## Architect Perspective

Imagine you're building your GitHub repository assistant.

The user spends an hour exploring:

submission workflow
authentication
retries
partner APIs

Then asks:

Where does retry happen?

Should the assistant reread the entire hour-long conversation?

No.

It should probably remember:

Current topic:
Submission workflow

Current repository:
Driver Servicing

Current component:
Submission Manager

That's enough.

An Important Design Principle

Not all previous messages are equally valuable.

Think about this exchange:

User:
Thanks!

Should that consume prompt tokens?

No.

But:

User:
Assume we're talking about extension drivers.

That absolutely should.

A Conversation Manager should understand the importance of history, not just its age.

Bringing Everything Together

Our architecture now looks like this:

                   User Question
                          │
                  Conversation Manager
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
 Recent History     Conversation      User Memory
                      Retrieval
                          │
                          ▼
                  Query Classifier
                          │
        ┌─────────────────┴─────────────────┐
        ▼                                   ▼
 Identifier Lookup                 Query Transformation
                                             │
                                             ▼
                                 BM25 + Vector Search
                                             │
                                             ▼
                                            RRF
                                             │
                                             ▼
                                         Reranker
                                             │
                                             ▼
                                       Prompt Builder
                                             │
                                             ▼
                                              LLM

Notice that the conversation itself has become another input to the system, just like documents.

## Handling the 250k-to-100k token problem

I would make the decisions in this order.

### 1. Resolve the current question

The Conversation Manager identifies:

current topic,
referenced entities,
pronouns such as “it” or “that service,”
whether the question depends on previous turns.

### 2. Preserve recent relevant turns

Keep a filtered sliding window, excluding noise such as:

greetings,
acknowledgements,
repeated answers,
irrelevant side discussions.

### 3. Retrieve relevant historical context

Search conversation summaries and older turns using the resolved question.

Do not retrieve old history merely because it exists.

#### 4. Retrieve external evidence

The Retriever gets relevant repository files, documents, code, logs, or structured records.

### 5. Rerank each evidence type

Historical conversation and repository evidence should not necessarily compete blindly in one list.

For example:

Conversation relevance ranking

Repository evidence ranking

Code evidence ranking

The system may allocate separate budgets to each.

### 6. Deduplicate

Remove cases where:

a summary repeats a recent message,
multiple chunks contain the same paragraph,
conversation history repeats a retrieved document,
old evidence has been superseded.

### 7. Compress safely

Compress lower-priority content while preserving:

decisions,
facts,
constraints,
identifiers,
numbers,
citations,
unresolved issues.

Source documents should usually be trimmed or extracted rather than freely summarized when exact wording matters.

### 8. Apply a token budget

For a 100k window, an illustrative allocation could be:

System and safety instructions       3k
Current question and recent turns    8k
Conversation summary                 5k
Retrieved historical discussion      8k
Repository/document evidence        55k
Prompt/output instructions           1k
Reserved output capacity            15k
Safety margin                        5k

These values are not fixed. They depend on the task.

For repository analysis, code evidence may receive more space. For a conversational coaching assistant, recent conversation may receive more.


## Refined architecture

Current User Message
        │
        ▼
Conversation Manager
        ├── Resolve references
        ├── Detect topic
        ├── Select recent turns
        └── Generate history-search query
        │
        ├─────────────────────┐
        ▼                     ▼
History Retriever       Knowledge Retriever
        │                     │
        ▼                     ▼
History Reranker        Evidence Reranker
        └──────────┬──────────┘
                   ▼
          Context Deduplicator
                   │
                   ▼
          Context Compressor
                   │
                   ▼
             Prompt Builder
        ├── Source labels
        ├── Logical ordering
        ├── Token budgeting
        └── Citation metadata
                   │
                   ▼
                  LLM