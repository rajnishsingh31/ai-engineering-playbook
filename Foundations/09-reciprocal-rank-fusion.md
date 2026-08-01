# Chapter 9 — Reciprocal Rank Fusion (RRF)

Let's start with a problem.

Suppose the query is:

How do I renew my driver signing certificate?

BM25 returns:

| Rank | Document | BM25 Score |
| ---- | -------- | ---------- |
| 1    | A        | 18.2       |
| 2    | B        | 15.6       |
| 3    | C        | 12.4       |

Vector search returns:

| Rank | Document | Cosine Similarity |
| ---- | -------- | ----------------- |
| 1    | B        | 0.94              |
| 2    | D        | 0.91              |
| 3    | A        | 0.89              |

Now we have a problem.

Can we average the scores?

Someone might suggest:

Average(BM25 Score, Cosine Similarity)

Sounds reasonable.

Unfortunately...

It is fundamentally wrong.

Why?

Because the numbers mean completely different things.

BM25:

18.2
15.6
12.4

Cosine similarity:

0.94
0.91
0.89

One system could even produce:

200
150
100

while another produces:

0.91
0.88
0.75

There is no universal scale.

It's like averaging:

Temperature in Celsius

+

Height in meters

The numbers exist, but they measure different things.

So what should we combine?

Instead of combining the scores...

We combine the ranks.

Notice something.

BM25 says:

1. A
2. B
3. C

Vector search says:

1. B
2. D
3. A

Even if their scoring systems are different...

Both systems agree that:

A is important.
B is important.

That agreement is valuable.

Enter Reciprocal Rank Fusion

Instead of asking:

"What score did each system assign?"

RRF asks:

"How high did each system rank this document?"

This makes the systems comparable.

The intuition

Suppose we assign points like this:

| Rank | Points |
| ---- | ------ |
| 1    | 100    |
| 2    | 50     |
| 3    | 33     |

This isn't the real formula.

It's just intuition.

Now:

BM25:

| Doc | Points |
| --- | ------ |
| A   | 100    |
| B   | 50     |
| C   | 33     |

Vector:

| Doc | Points |
| --- | ------ |
| B   | 100    |
| D   | 50     |
| A   | 33     |

Total:

| Doc | Total |
| --- | ----- |
| A   | 133   |
| B   | 150   |
| C   | 33    |
| D   | 50    |

Result:

B
A
D
C

Notice something beautiful.

Neither BM25 nor vector search "won."

The merged ranking rewards agreement.

The real RRF formula

The actual formula is surprisingly simple.

For each ranked list:

Contribution =
1 / (k + rank)

where:

rank starts at 1
k is usually around 60

Then:

Final Score

=

Sum of contributions

Don't worry about memorizing it.

Let's understand why it works.

Why divide?

Imagine:

Rank 1

gets

1/(60+1)

Rank 2 gets

1/(60+2)

Rank 20 gets

1/(60+20)

Notice:

Higher-ranked documents contribute more.

Lower-ranked documents contribute less.

The decrease is smooth rather than abrupt.

Why use k = 60?

This is an excellent architect question.

Why not:

k = 0

?

Because then:

Rank 1 = 1

Rank 2 = 0.5

Rank 3 = 0.333

Rank 1 dominates too strongly.

Using:

k = 60

compresses the differences.

Approximate values:

| Rank | Contribution |
| ---- | ------------ |
| 1    | 1/61         |
| 2    | 1/62         |
| 5    | 1/65         |
| 10   | 1/70         |

Notice:

Rank 1 is still better than Rank 10...

but not overwhelmingly so.

This makes RRF robust.

Example

BM25

1. A
2. B
3. C

Vector

1. B
2. A
3. D

Let's calculate approximately.

Document A

1/61

+

1/62

Document B

1/62

+

1/61

Nearly identical.

Document C

1/63

Document D

1/63

Result:

A

B

C

D

Notice:

A and B consistently appear near the top.

That's exactly what RRF rewards.

Why not just take the union?

Suppose:

BM25

A
B
C

Vector

B
D
A

Union becomes:

A
B
C
D

But...

Which should go first?

The reranker performs best when its candidate list is already high quality.

RRF improves the ordering before reranking.

What if only one search finds a document?

Example:

BM25

A
B
C

Vector

B
D
E

Document C still receives a contribution.

So does E.

A document doesn't need to appear in every list.

Appearing in multiple lists simply increases confidence.

A production pipeline

```text
User Query
   │
   ├──────────────┬──────────────┐
   ▼              ▼              ▼
 BM25         Vector Search
 Top 50 docs   Top 50 docs
   └──────────────┬──────────────┘
                  ▼
      Reciprocal Rank Fusion
                  │
          Top 30 merged docs
                  │
                  ▼
              Reranker
                  │
              Top 5 docs
                  │
                  ▼
                  LLM
```
Notice the numbers.

We don't usually rerank:

all BM25 results,
all vector results.

We first merge and trim the list.

That saves latency and cost.

Why is RRF so popular?

As an architect, this is the key question.

RRF has several attractive properties:

1. Independent of score scales

It doesn't matter whether:

BM25 gives:

12.7

or

600

or

2.1

Only the ranking matters.

2. Easy to add new retrieval methods

Imagine tomorrow you add:

SQL retrieval,
Graph search,
Knowledge graph,
Code symbol search.

Each can produce its own ranked list.

RRF can combine them all.

3. Very robust

Many academic evaluations have shown that RRF performs surprisingly well across different datasets without requiring complex tuning.

This is one reason you'll find it in many modern search systems.

An architect's thought process

Suppose you're designing your financial document extraction system from the assignment we discussed earlier.

A user asks:

"Which company had the highest ARR growth?"

You might retrieve candidates from:

Financial tables

↓

Narrative sections

↓

Executive summaries

↓

Metadata filters

↓

Semantic search

Each retrieval strategy contributes useful evidence.

Rather than trying to compare unrelated scores, you merge their ranked results using RRF before passing them to the reranker.

This is exactly the kind of architecture used in sophisticated enterprise systems.

Mental model

Keep this simple picture in your head:

```text
Retriever A
    │
    ▼
Ranked List

Retriever B
    │
    ▼
Ranked List

Retriever C
    │
    ▼
Ranked List

        ▼
        RRF
        ▼
Merged Ranked List
        ▼
   Reranker
        ▼
       LLM
```

Notice what RRF combines:

Order, not scores.

That single sentence captures the essence of RRF.

Engineer Perspective

If you were implementing this, you wouldn't write the algorithm from scratch unless you were building a search engine.

Platforms such as:

Azure AI Search
Elasticsearch
OpenSearch
Vespa

either support hybrid retrieval directly or provide mechanisms that implement or approximate RRF.

As an AI engineer, your focus is usually:

choosing which retrieval methods participate,
how many candidates each returns,
whether to apply metadata filters before retrieval,
how many documents to send to the reranker,
and how to evaluate the end-to-end quality.