# Chapter 10 — Query Transformation

## The Problem

Let's use your Driver Servicing domain.

Your documentation contains this sentence:

"A derived submission may only be created for extension drivers under specific conditions."

Now a user asks:

"Can I create a child submission for an extension INF?"

A human immediately understands:

child submission
≈
derived submission

extension INF
≈
extension driver

But BM25 may struggle because the words don't literally match.

Even vector search may not perfectly bridge all terminology, especially if your organization uses internal jargon.

## The Core Idea

Instead of changing the documents...

Change the query.

Pipeline becomes:

```text
User Question
        │
        ▼
Query Transformation
        │
        ▼
Retrieval
        │
        ▼
RRF
        │
        ▼
Reranker
        │
        ▼
LLM
```

Notice:

We're improving the input before retrieval.

## Why does this work?

**Users are surprisingly bad at asking questions.**


### Technique 1 — Query Rewriting

The simplest transformation.

Original:

How do I fix cert problems?

Rewrite:

How do I renew an expired driver signing certificate?

Notice:

The rewritten query is:

more specific
uses enterprise terminology
contains searchable terms

Another example

Original:

My build won't publish.

Rewrite:

Driver submission failed during publication.

Much better retrieval.

### Technique 2 — Query Expansion

Instead of replacing words...

Add related ones.

Original:

certificate renewal

Expanded:

certificate renewal
certificate expiration
certificate expiry
certificate replacement

Now BM25 has more opportunities to match.

Enterprise example

Original:

extension driver

Expanded:

extension driver
extension INF
derived submission
extension package

This is especially useful when different teams use different terminology.

### Technique 3 — Multi-Query Retrieval

This is extremely common in production.

Instead of one query...

Generate several.

Original:

How do I publish a driver?

Generate:

How do I submit a driver?

Driver publication workflow

Driver submission process

Publishing drivers to Microsoft

WHCP driver submission

Now retrieve using all of them.

Pipeline:

```text
Original Question
        │
        ▼
Generate Multiple Queries
        │
 ┌──────┼──────┬──────┐
 ▼      ▼      ▼      ▼
Q1     Q2     Q3     Q4
 │      │      │      │
 ▼      ▼      ▼      ▼
Retrieve Retrieve Retrieve Retrieve
        │
        ▼
Merge
        │
        ▼
RRF
```

Notice...

We already know how to merge ranked lists.

RRF works beautifully here too.

#### Why Multi-Query Works

Every query captures slightly different wording.

Imagine searching GitHub.

Original:

authentication failure

Alternative queries:

login failure

access denied

permission error

authorization issue

Different documents may use different terminology.

One query rarely captures everything.

#### Technique 4 — HyDE (Hypothetical Document Embeddings)

This is one of the coolest ideas in modern retrieval.

Instead of embedding the question...

Ask the LLM to imagine the ideal answer first.

Example:

User asks:

How do I revoke a signing certificate?

LLM first generates:

A signing certificate can be revoked by...

Now embed that hypothetical answer, not the original question.

Why?

Because documents are usually written like answers, not questions.

Documents often say:

Certificate revocation procedure...

not

How do I revoke my certificate?

Embedding something that resembles the document can improve semantic matching.

#### Why HyDE Works

Compare:

Question:

How do I...

Document:

The process for...

Different wording.

HyDE converts the question into something closer to the style of the documents.

#### Technique 5 — Decomposition

ome questions actually contain multiple questions.

Example:

Which company has the highest ARR,
the fastest growth,
and the lowest churn?

Instead of retrieving once...

Split it.

Query 1

Highest ARR

↓

Retrieve

Query 2

Growth

↓

Retrieve

Query 3

Churn

↓

Retrieve

Merge

This dramatically improves retrieval.

### Technique 6 — Step-Back Prompting

Sometimes users ask something extremely specific.

Why did Partner A fail certification yesterday?

Before searching...

Generate a broader question.

How does driver certification fail?

Retrieve:

general process
common failure reasons
policy

Then combine with:

Partner A logs
yesterday's events

Very useful in enterprise troubleshooting.

## Which techniques are used together?

Production systems rarely use just one.

```text

Example:

User Question
        │
        ▼
Rewrite
        │
        ▼
Expand
        │
        ▼
Generate 3 Queries
        │
        ▼
Retrieve
        │
        ▼
Merge
        │
        ▼
RRF
        │
        ▼
Reranker

```

### But isn't this slower?

Excellent architect question.

Yes.

Every transformation adds:

latency
cost
complexity

So ask:

Is it worth it?

Examples:

Where is CreateSubmissionAsync()?

Don't rewrite.

Don't generate five queries.

Go directly to the symbol index.

But:

Explain how to publish a driver.

Multi-query retrieval may significantly improve results.

Query Router Revisited

Remember our router?

It becomes smarter.

```text

                     User Query
                          │
                  Query Classifier
                          │
        ┌─────────────────┴─────────────────┐
        ▼                                   ▼
Identifier?                           Natural Language?
        │                                   │
        ▼                                   ▼
Direct Lookup                  Query Transformation
        │                                   │
        ▼                                   ▼
    Retrieve                            Retrieve

```

Not every query deserves transformation.

## Architect Perspective

This is one of the biggest mindset shifts.

Many people optimize:

Embedding model

or

Vector database

first.

Experienced AI architects often ask:

"Can we improve the query instead?"

Sometimes a simple rewrite increases retrieval quality more than switching to a larger embedding model.

## Where does the LLM fit?

You may have noticed something interesting.

We're using an LLM before retrieval.

Question
      │
      ▼
LLM
(Query Rewrite)
      │
      ▼
Retriever

### So an LLM can participate twice:

Before retrieval (rewrite, expand, decompose, HyDE)
After retrieval (answer generation)

This is why production AI systems often make multiple LLM calls for a single user request.

## The Complete Retrieval Pipeline (So Far)



                     User Question
                            │
                    Query Classifier
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
      Identifier Lookup         Query Transformation
                                            │
                                            ▼
                                 Multiple Queries / HyDE
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

This is recognizably similar to the architecture behind many state-of-the-art enterprise RAG systems.

## Query Transformation - Decision Tree

```text

                   User Query
                       │
               Query Classifier
                       │
        ┌──────────────┼──────────────┐
        │              │              │
 Identifier?      Ambiguous?     Multi-part?
        │              │              │
        ▼              ▼              ▼
 Symbol Index     Rewrite       Decompose
        │              │              │
        └──────────────┴──────────────┘
                       │
                  Retrieval
                       │
                 RRF + Reranker
                       │
                      LLM

```