# Chapter 8 — BM25: How Keyword Search Ranks Documents

You already know keyword search is useful for:

0x80070005
CreateSubmissionAsync
Windows 11 24H2
CVE-2026-12345

But keyword search must solve a harder problem:

If several documents contain the same keyword, which one should rank first?

That is where BM25 comes in.

## Start with a simple example

Suppose the query is:

driver signing certificate

We have three documents.

Document A
Driver signing requires a valid certificate.
Document B
A certificate is required for driver signing.
The signing certificate must not be expired.
Document C
This document explains certificates, certificate renewal,
certificate storage, certificate authorities, and certificate policies.

All three contain relevant words.

Which should rank highest?

Probably:

Document B

Why?

Because it contains all query terms and discusses them together in the right context.

BM25 tries to calculate this relevance.

## BM25 is not just keyword counting

A naive keyword system might say:

Count how many times each word appears.

But that creates problems.

Document C repeats certificate many times.

Should it automatically rank highest?

No.

BM25 improves on simple counting by considering three main ideas:

Term frequency
Document frequency
Document length

Let's study each.

## Term Frequency

Term frequency means:

How often does the query term appear in this document?

Suppose the query is:

certificate

Document A contains it once.

Document B contains it twice.

Document C contains it five times.

More occurrences are generally useful.

But BM25 does not assume:

5 occurrences = 5 times more relevant

The benefit gradually reduces.

Conceptually:

First occurrence
→ very useful

Second occurrence
→ useful

Tenth occurrence
→ adds little

This is called term-frequency saturation.

Why saturation matters

Imagine a document contains:

certificate certificate certificate certificate certificate

Repeating the word should not make it infinitely relevant.

BM25 prevents keyword stuffing from dominating results.

Conceptually:

Relevance
   │
   │        ─────────
   │      /
   │    /
   │  /
   └──────────────────
       Term frequency

The score rises, but eventually levels off.

## Inverse Document Frequency

Some words are more informative than others.

Suppose the corpus contains 1 million documents.

The word:

the

appears in almost every document.

The word:

CreateSubmissionAsync

appears in only 20 documents.

Which term is more useful for identifying the correct document?

Clearly:

CreateSubmissionAsync

BM25 gives more importance to rare terms.

This idea is called:

Inverse Document Frequency

or:

IDF

The intuition is:

Rare term
→ high importance

Common term
→ low importance
Example

Query:

Windows driver signing

Suppose:

Windows
appears in 500,000 documents

driver
appears in 50,000 documents

signing
appears in 5,000 documents

Then signing may contribute more to the score because it is more selective.

A highly specific term narrows the search space.

## Document Length Normalization

Long documents naturally contain more words.

Therefore, they have a greater chance of matching any query accidentally.

Suppose:

Document A
Driver signing requires a certificate.

Length:

5 words
Document B

A 500-page document contains the same sentence somewhere inside it.

Should both receive the same score?

Usually not.

Document A is highly focused.

Document B may only mention the topic briefly.

BM25 adjusts scores based on document length.

Conceptually:

Same number of keyword matches
+
Short focused document
→ usually ranks higher

This is called document-length normalization.

## The BM25 intuition

You do not need to memorize the complete formula yet.

The core idea is:

BM25 score
=
importance of query term
×
strength of term match
×
document-length adjustment

For each query term:

How rare is this term?
How often does it occur here?
How long is this document?

Then BM25 adds the term scores together.

## Simplified formula

The common BM25 formula looks roughly like this:

Score(D, Q)
=
Σ IDF(q)
×
TF-adjustment

A more complete version is:

Score(D,Q) =
Σ IDF(qᵢ) ×
[f(qᵢ,D) × (k₁ + 1)]
/
[f(qᵢ,D) + k₁ × (1 - b + b × |D| / avgdl)]

This may look complicated, but every part represents something we've already discussed.

Symbol	Meaning
qᵢ	A query term
f(qᵢ,D)	Frequency of that term in the document
`	D
avgdl	Average document length
k₁	Controls term-frequency saturation
b	Controls document-length normalization
IDF	Importance of rare terms

The formula is less important than the intuition.

### What does k₁ do?

k₁ controls how much repeated occurrences matter.

Typical value:

k₁ ≈ 1.2 to 2.0

Lower k₁:

Repeated terms saturate quickly

Higher k₁:

Term frequency continues contributing longer

Example:

If certificate appears:

1 time
2 times
10 times

k₁ controls how much extra score the repeated occurrences receive.

### What does b do?

b controls document-length normalization.

Typical value:

b ≈ 0.75

If:

b = 0

document length is ignored.

If:

b = 1

full length normalization is applied.

So b answers:

How much should longer documents be penalized?

#### Example ranking

Query:

certificate renewal

Documents:

A
Certificate renewal instructions.
B
This guide explains certificate renewal,
certificate enrollment, certificate storage,
certificate revocation and certificate policy.
C
Renew your signing certificate before it expires.

A simplistic exact-match system might rank A or B highest because both contain both exact words.

BM25 may prefer A because:

Both query terms appear
The document is short
The content is focused

But a semantic vector search may prefer C because:

renew
≈
renewal

This is another reason hybrid search is useful.

## BM25 is lexical, not semantic

BM25 matches terms.

It does not deeply understand meaning.

Query:

How do I fix an expired signing credential?

Document:

Certificate renewal procedure

BM25 may struggle because:

credential ≠ certificate
fix ≠ renewal
expired may be missing

Vector search may understand the relationship.

This is the core limitation of BM25.

## Tokenization matters

Before BM25 can score documents, the text is usually processed by an analyzer.

For example:

The certificates are expiring.

may become:

certificate
expire

This processing can include:

lowercasing,
removing stop words,
stemming,
lemmatization,
punctuation handling,
synonym expansion.

So keyword search is not always a literal string comparison.

### Stop words

Words such as:

the
is
a
of
and

appear almost everywhere.

They provide little search value.

Search engines often remove or heavily downweight them.

Query:

What is the process for publishing a driver?

Useful terms might become:

process
publishing
driver

### Stemming and lemmatization

These techniques normalize related word forms.

Example:

publish
publishing
published

may be reduced to a common root.

Similarly:

certificate
certificates

may be treated as equivalent.

This helps BM25 handle minor grammatical variations.

But it still does not provide full semantic understanding.

### Field-aware search

Enterprise documents often have fields:

title
body
tags
API name
product version
author

A match in the title is usually more important than a match buried deep in the body.

Example:

Query:

CreateSubmissionAsync

Document A:

Title: CreateSubmissionAsync API

Document B:

The term appears once in a long discussion.

A search engine can apply boosts:

Title match × 5
API symbol match × 10
Body match × 1

This is called field boosting.

### BM25 and metadata filters

BM25 ranking and metadata filtering solve different problems.

Metadata filtering answers:

Which documents are allowed or applicable?

BM25 answers:

Which allowed documents are most relevant?

Example:

product = Windows 11
version = 24H2
status = Published
user_has_access = True

Apply filters first.

Then BM25 ranks the remaining documents.

### BM25 in hybrid search

A production hybrid pipeline may look like:

Query
  │
  ├── BM25 search → Top 30
  │
  └── Vector search → Top 30
             │
             ▼
       Candidate merge
             │
             ▼
            RRF
             │
             ▼
          Reranker
             │
             ▼
           Top 5

Each component has a distinct job:

BM25
→ exact lexical relevance

Vector search
→ semantic relevance

RRF
→ merge ranked lists

Reranker
→ deeper final relevance

## Engineer perspective

In Elasticsearch or OpenSearch, keyword retrieval may look conceptually like:

query = {
    "query": {
        "match": {
            "content": "driver signing certificate"
        }
    }
}

The search engine analyzes the query and documents, then applies BM25 scoring internally.

You usually do not implement the formula manually.

You configure:

analyzers,
synonyms,
field boosts,
filters,
BM25 parameters,
result count.

## Architect perspective

As an architect, the most important questions are not:

What is the exact BM25 formula?

They are:

Which fields should be searchable?

Which fields should be boosted?

Which identifiers require exact matching?

Which synonyms should we configure?

How do we handle versions?

How do we handle acronyms?

Do we need language-specific analyzers?

How do we evaluate retrieval quality?

### Example: Driver Servicing assistant

Suppose the query is:

### How do I create a derived submission for an extension driver?

You might configure fields like:

title
content
document_type
driver_type
submission_type
version

Possible weighting:

title                × 4
submission_type      × 5
driver_type          × 3
content              × 1

Keyword matching may strongly identify:

derived submission
extension driver

Vector search may retrieve conceptually related workflow guidance.

The reranker then chooses the best evidence.

### BM25 strengths

BM25 is strong when queries include:

identifiers,
API names,
error codes,
product versions,
exact phrases,
uncommon terminology,
domain-specific nouns.

It is:

fast,
explainable,
mature,
scalable,
inexpensive.

### BM25 weaknesses

BM25 struggles with:

synonyms,
paraphrasing,
conceptual similarity,
implied meaning,
natural-language intent.

Example:

Query:
Why is access blocked?

Document:
Permission denied due to missing authorization.

The meaning matches, but the exact words differ.

Vector search is more likely to help.

Final mental model
BM25 asks:

Does this document contain the important query terms?

How often?

How rare are those terms?

Is the document focused or unusually long?

It does not ask:

Does this document mean the same thing as the question?

That is where vector search helps.

Together:

BM25
+
Vector Search
+
Reranker
=
Strong enterprise retrieval

# Recommended mental model - Strategy to store chunks in search and vector DB

Either use single DB that supports both keyword search and vector or use same canonical chunks and stableIDs.

One source document
        ↓
One chunking pipeline
        ↓
Canonical chunks with stable IDs
        ↓
   ┌────┴────┐
   ↓         ↓
BM25 index   Vector index
   └────┬────┘
        ↓
Merge and deduplicate by chunk ID
        ↓
Rerank candidate text

So the direct answer is:

Use the same canonical chunks and stable IDs for both retrieval methods whenever practical. They may live in one unified index or separate databases, but both should map back to the same original chunk text and metadata.

# BM25 relevace:

BM25 relevance
=
rare useful terms
+
meaningful term frequency
+
document-length adjustment