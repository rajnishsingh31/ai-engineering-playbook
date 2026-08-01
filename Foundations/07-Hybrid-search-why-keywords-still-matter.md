## Hybrid Search

Hybrid search combines both signals.

Conceptually:

Question
      │
      ├─────────────► Keyword Search
      │
      └─────────────► Vector Search
                │
                ▼
         Candidate Merge
                │
                ▼
           Reranker
                │
                ▼
              Top K

Notice:

The reranker doesn't care whether a document came from keyword search or vector search.

It only cares:

Which candidate best answers the user's question?

### A Real Example

Suppose the user asks:

Windows error 0x80070005

Keyword search returns:

Error Code Reference

Vector search returns:

Permission denied while accessing resource

Both are useful.

The merged candidate list becomes:

Error Code Reference

Permission denied guide

Troubleshooting article

Security documentation

The reranker then decides the final order.

Why Enterprise Search Uses Hybrid

Imagine you're building Microsoft Learn.

Users search for:

AKS

Others search for:

Azure Kubernetes Service

Keyword search works well for:

AKS

Semantic search understands:

Azure Kubernetes Service

=

AKS

Now imagine:

Windows 11 24H2

Keyword search is excellent.

Semantic search alone may not prioritize the exact version.

Hybrid covers both cases.

Another Example

Suppose I ask:

How do I dispose a stream in C#?

The documentation says:

using (...)

There is no word:

dispose

But semantically:

using statement

↓

resource disposal

Vector search helps.

Now ask:

IDisposable

That's an exact type name.

Keyword search excels.

How Results Are Combined

Different systems merge results differently.

One simple idea:

Keyword Rank

+

Vector Rank

↓

Combined Score

Example:

Document	Keyword Rank	Vector Rank
A	1	5
B	4	1
C	2	2

A merge algorithm decides which candidates move forward.

Reciprocal Rank Fusion (RRF)

One of the most widely used approaches is Reciprocal Rank Fusion (RRF).

Don't worry about the math.

Understand the intuition.

Suppose:

Keyword search says:

A
B
C

Vector search says:

B
D
A

RRF rewards documents that appear near the top of multiple ranked lists.

The merged ranking might become:

B

A

C

D

Notice:

B appeared high in both lists.

That's a strong signal.

Why Not Average Scores?

You might think:

Average(keyword_score, vector_score)

Unfortunately:

Keyword score:

98

Vector score:

0.83

These scores come from completely different systems and scales.

They're not directly comparable.

That's why ranking-based fusion methods like RRF are popular.

**Metadata Still Applies**

Hybrid search doesn't replace metadata filtering.

## Pipeline:

Authentication

↓

Metadata Filters

↓

Keyword Search

+

Vector Search

↓

Merge

↓

Rerank

↓

LLM

Suppose a document belongs to:

Finance

A user from HR should never retrieve it—even if it's a perfect keyword or semantic match.

Security comes first.

Trade-offs
Keyword Search	Vector Search
Exact identifiers	Semantic meaning
Fast	Slightly slower
Great for codes	Great for natural language
Misses synonyms	Finds conceptual matches

Hybrid combines both.

## Architect Perspective

As an architect, ask:

Which queries are identifier-heavy?
How many users search by product names?
How often do users ask natural-language questions?
Should every query use hybrid search?
How should results be merged?
What latency budget remains after running two retrieval methods?
Engineer Perspective

### Typical implementation:

Question
      │
      ├────────────► BM25
      │
      └────────────► Vector Search
              │
              ▼
         Merge Results
              │
              ▼
          Reranker
              │
              ▼
             LLM

Notice the new term:

### BM25

This is the classic ranking algorithm behind many keyword search engines.

We'll learn it shortly.

### Updated Architecture

Our RAG pipeline now becomes:

### OFFLINE

Document
    │
    ▼
Extraction
    │
    ▼
Chunking
    │
    ▼
Embedding
    │
    ▼
Vector DB


### ONLINE

User Question
      │
      ▼
Authentication
      │
      ▼
Metadata Filters
      │
      ├──────────────► BM25 Search
      │
      └──────────────► Vector Search
                 │
                 ▼
           Merge Candidates
                 │
                 ▼
             Reranker
                 │
                 ▼
          Prompt Builder
                 │
                 ▼
                LLM

Notice that we now have two retrieval engines working together.

# When to use keyword search, vector search? hybrid search? Other search?


Keyword search answers:

"Find this."

Vector search answers:

"Find something about this idea/concept."

Hybrid search combines exact lexical matches with semantic similarity, increasing the chance of retrieving all relevant evidence before reranking.

A New Layer: Query Routing


Imagine a router in front of everything.

                    User Question
                          │
                          ▼
                   Query Classifier
        ┌────────────┬────────────┬────────────┐
        ▼            ▼            ▼            ▼
  Structured DB   Keyword     Hybrid RAG    Live API

Examples:

Query	Route
Error code	Structured DB / Keyword
Driver publishing	Hybrid
API symbol	Keyword
Build failures	Logs + APIs + RAG
Calendar meetings	Calendar API
GitHub PR	GitHub API
Weather	Weather API
SQL query	Database

This pattern is becoming increasingly common in modern AI systems.