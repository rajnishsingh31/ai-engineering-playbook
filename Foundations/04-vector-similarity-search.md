# The vector database needs a mathematical way to measure that similarity.

## Table of Contents

- [The vector database needs a mathematical way to measure that similarity.](#the-vector-database-needs-a-mathematical-way-to-measure-that-similarity)
  - [Table of Contents](#table-of-contents)
  - [Euclidean distance](#euclidean-distance)
  - [Cosine similarity](#cosine-similarity)
  - [Dot product](#dot-product)
- [dot product](#dot-product-1)
  - [Normalized Vectors](#normalized-vectors)
  - [A concrete example:](#a-concrete-example)
  - [Similarity Does Not Mean Correctness](#similarity-does-not-mean-correctness)
  - [Metadata Filtering](#metadata-filtering)
  - [Top-K Retrieval](#top-k-retrieval)
  - [Small K](#small-k)
  - [Large K](#large-k)
  - [Similarity Threshold](#similarity-threshold)
  - [Exact Search vs Approximate Search](#exact-search-vs-approximate-search)
  - [ANN means:](#ann-means)
    - [Analogy: Finding a Restaurant](#analogy-finding-a-restaurant)
    - [HNSW — A Preview](#hnsw--a-preview)
  - [Retrieval pipeline so far](#retrieval-pipeline-so-far)
  - [Important Distinction](#important-distinction)
  - [Recall](#recall)
    - [Recall versus precision](#recall-versus-precision)
- [Engineer Perspective](#engineer-perspective)
- [Architect Perspective](#architect-perspective)
- [Popular Vector DB](#popular-vector-db)

The three common approaches are:

## Euclidean distance

Euclidean distance is ordinary physical distance. The distance between them is the straight-line distance on a graph.
Smaller distance means greater similarity.

Imagine: A = [1,2] B=[4,6] Conceptually: Distance(A,B) = 5

Limitation: Euclidean distance considers both vector's direction and magnitude. 

For text meaning, magnitude may not always be the feature we care most about.


## Cosine similarity

Cosine similarity measures the angle between two vectors. It focuses primarily on whether the vectors point in the same direction.


Imagine 2 points: Vector A  ↗ Vector B  ↗ They point in the same direction, so their cosine 
similariy is high

Typical cosine similarity values are:

 1    → same direction
 0    → unrelated or perpendicular
-1    → opposite direction

Why Direction Can Represent Meaning

Suppose we have: A = [1,2]  B=[10,20]

These vectors have very different magnitudes.

But they point in exactly the same direction.

Conceptually, they may represent the same semantic pattern at different scales.

Cosine similarity would treat them as maximally similar.

Euclidean distance would consider them far apart.

That is one reason cosine similarity is frequently used for semantic embeddings.


## Dot product

The dot product combines corresponding dimensions and adds the results.

Example: A= [1,2] B=[3,4] Dot Product = 1 * 3 + 2 * 4 = 11

A larger dot product often indicates greater similarity.

The dot product is closely related to cosine similarity:


dot product
=
magnitude of A
×
magnitude of B
×
cosine of the angle

So the dot product considers both direction and magnitude.

If vectors are normalized to length 1, dot product and cosine similarity become equivalent.

For normalized vectors, cosine similarity and dot-product ranking produce the same ordering.

## Normalized Vectors

Normalization changes a vector’s length to 1 while keeping its direction unchanged.

Before normalization: [10, 20]  After normalization: [0.447, 0.894]. The direction is same. Only magnitude changes.

Why do this? Because it makes comparisons more consistent and allows efficient dot-product search while preserving cosine-style similarity.

## A concrete example:

Suppose the user asks:

How do I renew my signing certificate?

The query embedding is:

Q = [0.82, 0.74, ...]

The vector database contains these chunks:

Chunk A:
Renewing a code-signing certificate requires...
Vector: [0.80, 0.76, ...]

Chunk B:
Certificate expiration notifications are sent...
Vector: [0.68, 0.65, ...]

Chunk C:
How to submit a Windows driver package...
Vector: [0.20, -0.15, ...]

The search engine calculates similarity:

Similarity(Q, A) = 0.96
Similarity(Q, B) = 0.84
Similarity(Q, C) = 0.19

It may return:

1. Chunk A
2. Chunk B

The vector database then returns the actual text and metadata associated with those vectors.

## Similarity Does Not Mean Correctness

This is crucial.

A vector database finds:

The chunks whose meanings appear most similar to the question.

It does not guarantee:

that the document is correct,
that the document is current,
that the document is authoritative,
that the document answers the question fully,
that the user has permission to access it.

For example, a search for:

Current driver-signing process

might retrieve:

Legacy driver-signing process from 2021

The content is semantically similar but outdated.

That is why production retrieval also needs:

- metadata filtering,
- authorization filtering,
- document versioning,
- freshness information,
- reranking.

## Metadata Filtering

When storing a document chunk, we normally store more than text and a vector.

Conceptually:

{
  "chunk_id": "driver-signing-42",
  "text": "To renew the certificate...",
  "embedding": [0.18, -0.72, 0.41],
  "source": "Driver Signing Handbook",
  "version": "2026.2",
  "department": "Windows",
  "classification": "Internal",
  "allowed_groups": ["DriverServicing"],
  "updated_at": "2026-06-14"
}

Before or during vector search, the application may add filters:

department = Windows
classification permitted for current user
version = latest
updated_at after a defined date

Then semantic search occurs only over acceptable candidates.

This is where vector search becomes an enterprise retrieval system rather than a demo.

## Top-K Retrieval

A vector search usually asks for the top K results.

For example:

top_k = 5

This means:

Return the five most similar chunks.

Choosing K creates a trade-off.

## Small K
top_k = 2

- Advantages:

lower token usage,
lower latency,
less irrelevant context.

- Risk:

relevant evidence may be missed.

## Large K
top_k = 20

- Advantages:

greater chance of including the answer,
more supporting evidence.

- Risks:

higher cost,
increased noise,
more contradictory information,
larger prompts.

There is no universally correct value.

It must be evaluated using representative questions.

## Similarity Threshold

Some systems also require a minimum similarity score.

For example:

Return result only when score ≥ 0.75

Without a threshold, the vector database may always return something—even when nothing is genuinely relevant.

Imagine asking an HR knowledge base:

How do I repair a motorcycle engine?

The database might still return the mathematically closest HR document.

But “closest” does not necessarily mean “relevant.”

A threshold helps the system say:

I could not find sufficient supporting information.

However, similarity scores vary by embedding model and distance metric, so 0.75 is not a universal standard. Thresholds should be calibrated using evaluation data.

## Exact Search vs Approximate Search

Suppose you have 100 vectors.

The database can compare the query against every vector.

This is called exact search.

Query
  ↓
Compare with all 100 vectors
  ↓
Return nearest

Easy.

Now suppose you have 50 million vectors.

Comparing the query against every vector can become expensive and slow.

Instead, vector databases commonly use **Approximate Nearest Neighbor**, or ANN, search.

## ANN means:

Find very likely nearest vectors without comparing against every vector.

It trades a small amount of perfect accuracy for much better speed and scalability.

### Analogy: Finding a Restaurant

Imagine finding the closest Indian restaurant in a city.

An exact approach would be:

Calculate your distance from every restaurant in the city.

An approximate approach would be:

First search nearby neighborhoods,
then compare restaurants within those areas.

You may occasionally miss the mathematically closest restaurant, but the result is much faster and usually good enough.

### HNSW — A Preview

One popular ANN index is HNSW:

Hierarchical Navigable Small World graph.

Do not worry about the name yet.

Think of it as a network of vectors connected to nearby vectors.

```text
A —— B —— C
|    |     |
D —— E —— F
```

When searching, the database navigates through the graph instead of scanning every vector.

It begins with broad jumps and progressively moves into a more local neighborhood.

HNSW often provides:

high search quality,
fast queries,
relatively high memory usage,
slower and more expensive indexing.

We will cover index trade-offs in the vector database chapter.

## Retrieval pipeline so far

```text

User Question
      │
      ▼
Authentication
      │
      ▼
Question Embedding
      │
      ▼
Metadata / ACL Filters
      │
      ▼
ANN Vector Search
      │
      ▼
Top-K Candidate Chunks
      │
      ▼
Similarity Threshold
      │
      ▼
Original Text + Metadata
      │
      ▼
Prompt Builder
      │
      ▼
     LLM
      │
      ▼
Grounded Answer + Citations

```

## Important Distinction

Keep these responsibilities separate:

Embedding model
    Creates vectors

Similarity metric
    Measures vector closeness

Vector index
    Organizes vectors for efficient search

Vector database
    Stores vectors, text, metadata and executes queries

LLM
    Reasons over retrieved original content

## Recall

In retrieval systems, recall means how many of the truly relevant results the system successfully found.

Formula:

Recall =
Relevant results retrieved
──────────────────────────
Total relevant results that exist
Example

Suppose there are 10 support tickets in the database that genuinely match the user’s issue.

Your vector search retrieves 8 of them.

Recall = 8 / 10 = 80%

The remaining 2 relevant tickets were missed.

In ANN search

Approximate Nearest Neighbor search is faster because it does not compare the query against every vector.

That creates the possibility that it misses some genuinely close vectors.

Exact search:
Finds all true nearest candidates
Higher latency

ANN search:
Usually finds most nearest candidates
Lower latency
May miss some

So when we say ANN has a recall trade-off, we mean:

It gains speed by accepting a chance of missing relevant neighbors.

A system with 95% recall found about 95 out of every 100 relevant candidates it could have found under the evaluation setup.

### Recall versus precision

These two are often discussed together.

Recall

Of all relevant documents that existed, how many did we retrieve?

Precision

Of all documents we retrieved, how many were actually relevant?

Example:

The database contains 10 relevant documents.

The system retrieves 8 documents:

6 are relevant
2 are irrelevant

Then:

Recall = 6 / 10 = 60%

Precision = 6 / 8 = 75%

A simple mental model:

Recall    → Did we miss useful results?
Precision → Did we retrieve noisy results?

Increasing top_k often improves recall because you retrieve more candidates, but it may reduce precision because more irrelevant content can enter the results.

# Engineer Perspective

As an engineer, you need to decide:

Which embedding model generates the vectors?
Which similarity metric does that model support?
Are vectors normalized?
What value should top_k use?
Should there be a similarity threshold?
Which metadata filters apply?
How will retrieval quality be tested?

# Architect Perspective

As an architect, ask:

Does semantic similarity alone satisfy the use case?
Is hybrid keyword and vector search needed?
How will access control be enforced?
How will old and conflicting documents be handled?
What happens when retrieval returns no strong match?
How much latency and recall loss are acceptable?
Should the system use exact or approximate search?
How will embedding model upgrades and reindexing be managed

# Popular Vector DB

| Database        | Best For                                    | Trade-offs                                                                           |
| --------------- | ------------------------------------------- | ------------------------------------------------------------------------------------ |
| LanceDB         | Local development, embedded apps, analytics | Simple, lightweight, not a managed cloud service                                     |
| pgvector        | Existing PostgreSQL systems                 | One database to operate, but not always the best fit for very large vector workloads |
| Pinecone        | Managed cloud vector search                 | Easy to use, but vendor-managed and potentially higher cost                          |
| Qdrant          | Open-source production deployments          | Strong filtering and retrieval features                                              |
| Milvus          | Very large-scale deployments                | Powerful but operationally more complex                                              |
| Azure AI Search | Azure enterprise ecosystems                 | Integrates well with Azure services, combines keyword and vector search              |