# Chapter 6 — Reranking: Finding the Best Answer Among Good Candidates

Imagine your Microsoft engineering assistant.

The user asks:

"How do I renew my driver signing certificate?"

The vector database returns these chunks:

1. Driver Signing Overview
2. Certificate Renewal Process
3. Driver Submission Workflow
4. Security Policies
5. Certificate Expiration FAQ

At first glance, this looks good.

But ask yourself:

Should all five chunks be sent to the LLM?

Probably not.

Some are much more relevant than others.

The challenge is:

Vector search is optimized for speed, not perfect ranking.

Why Isn't Vector Search Enough?

Remember ANN?

Instead of checking every vector, it searches efficiently.

That means it usually finds good candidates, not necessarily the perfect ordering.

Think of Google.

You search:

"Python enumerate"

Imagine Google returns:

1. Python Tutorial
2. Python enumerate()
3. Python Lists
4. Python Functions
5. Python FAQ

All are related.

But you'd expect enumerate() to be first.

Vector search is similar.

It retrieves a candidate set.

Something else can improve the ranking.

Stage 1 vs Stage 2 Retrieval

## Production search often has two stages.

### Stage 1 — Retrieval

Fast.

Goal:

Don't miss relevant documents.

Question
      │
      ▼
Vector Search
      │
      ▼
Top 50 Chunks

Notice:

Not Top 5.

Top 50.

Why?

**Because retrieval is trying to maximize recall**.

Remember:

Recall = Don't miss useful information.

# Stage 2 — Reranking

Now we slow down a little.

Top 50
     │
     ▼
Reranker
     │
     ▼
Top 5

The reranker carefully compares:

Question

vs

Chunk

one pair at a time.

Its only job is:

**Which chunk best answers THIS question?**

Think Like a Hiring Manager

Imagine interviewing 100 candidates.

Would you hire the first person who applied?

No.

You would:

Collect resumes

↓

Shortlist 20

↓

Conduct interviews

↓

Select Top 3

Vector search is the resume screening.

Reranking is the interview.

How Does a Reranker Work?

Unlike embeddings...

which compare vectors...

a reranker reads both texts together.

Example:

Question:

How do I renew a signing certificate?

Candidate chunk:

Renewing a certificate requires...

The reranker sees:

Question:

How do I renew a signing certificate?

Candidate:

Renewing a certificate requires...

It then predicts:

Relevance = 0.98

Another chunk:

Driver submission workflow...

might receive:

0.42

Notice something?

The reranker isn't trying to understand the entire knowledge base.

It's only judging:

How well does this specific chunk answer this specific question?

## Embeddings vs Rerankers

This distinction is one of the most important in modern RAG.

### Embedding Model

Input:

Document

Output:

One vector

Goal:

Represent semantic meaning.

The document is processed independently.

### Reranker

Input:

Question

+

Document

Output:

Relevance score

Goal:

Estimate how well this document answers this question.

Notice the difference?

Embedding:

Document only

Reranker:

Question + Document

That extra context often produces much better ranking.

Why Is It More Accurate?

Suppose the question is:

How do I revoke a certificate?

Candidate A:

Certificate renewal process...

Candidate B:

Certificate revocation procedure...

Embedding similarity might rate both highly because:

certificate
process
security

are semantically related.

But the reranker reads both texts together and notices:

renew

≠

revoke

So it gives Candidate B the higher score.

This is one of the biggest reasons rerankers improve retrieval quality.

Why Not Use the Reranker on Every Document?

Suppose you have:

50 million chunks

Could we compare:

Question

with

every chunk?

Technically yes.

Practically no.

Imagine:

50,000,000 comparisons

for every user query.

Latency would be terrible.

Instead:

Vector Search

↓

Top 50

↓

Reranker

↓

Top 5

The reranker only processes a small shortlist.

Cross-Encoder vs Bi-Encoder

This is where things get interesting.

Embedding Model

Sometimes called a bi-encoder.

Question

↓

Vector A
Document

↓

Vector B

Similarity:

Compare vectors

Question and document are encoded separately.

Reranker

Usually a cross-encoder.

Question

+

Document

↓

Transformer

↓

Score

The model sees both together.

That lets it understand relationships like:

negation
intent
exact wording
subtle distinctions

It is slower but usually more accurate.

Trade-offs
Retrieval	Reranker
Very fast	Slower
Scales to millions	Works on small candidate sets
Higher recall	Higher precision
Uses vectors	Reads text pairs

Notice:

Earlier we discussed:

**Recall

vs

Precision.

This is where they meet.

Retrieval:**

Maximize recall.

Reranker:

Maximize precision.

A Real Pipeline

Let's revisit our certificate example.

User:

Renew signing certificate

Vector search returns:

Certificate renewal

Driver lifecycle

Security policy

Submission guide

Troubleshooting

Reranker scores them:

Renewal            0.98

Submission         0.85

Troubleshooting    0.60

Security           0.42

Lifecycle          0.20

Now only the top few go to the LLM.

Instead of sending five noisy chunks, the prompt builder sends the highest-quality evidence.

Architect Perspective

As an architect, ask:

How many candidates should retrieval return?
How many should reranking keep?
Is the quality improvement worth the added latency?
Should reranking be skipped for very simple queries?
Should different rerankers be used for code versus legal documents?
What happens if the reranker disagrees with retrieval?
Engineer Perspective

You'll eventually configure things like:

Vector Search

Top-K = 50

↓

Reranker

Top-K = 5

↓

LLM

These values are not fixed.

You'll measure:

latency,
answer quality,
token cost,
retrieval accuracy,

and tune them based on evaluation results.

Updated Architecture

## Our production pipeline now looks like this:

                   OFFLINE

Document
    │
    ▼
Text Extraction
    │
    ▼
Chunking
    │
    ▼
Embedding Model
    │
    ▼
Vector Database
         ▲
         │
         │
User Question
     │
     ▼
Embedding Model
     │
     ▼
Vector Search (Top 50)
     │
     ▼
Metadata Filtering
     │
     ▼
Reranker
     │
     ▼
Top 5 Chunks
     │
     ▼
Prompt Builder
     │
     ▼
LLM

Notice where the reranker sits:

After retrieval, before the LLM.

That's the sweet spot.

Real-World Insight

Many people assume:

Better LLM = Better RAG.

In practice, I've seen teams improve answer quality more by improving:

chunking,
metadata,
reranking,
retrieval evaluation,

than by switching from one frontier LLM to another.

A stronger retrieval pipeline gives the LLM better evidence to reason over.

## Well-known rerankers

They broadly fall into two categories.

### Managed reranking services

These are easiest to integrate because you send:

query + candidate documents

and receive relevance scores.

### Cohere Rerank

Cohere offers a managed Rerank API aimed at enterprise search and retrieval. Its current Rerank family processes the query and candidates more deeply than embedding similarity and returns a reordered result set.

Conceptually:

results = cohere.rerank(
    query="How do I renew a certificate?",
    documents=candidate_chunks,
    top_n=5
)

#### Good fit when:

you want a simple hosted API,
multilingual or enterprise search matters,
you do not want to host GPUs.

#### Trade-offs:

external service dependency,
usage cost,
data-governance review,
network latency.
Voyage AI Rerank

### Voyage

 provides managed reranking models such as rerank-2.5 and rerank-2.5-lite. The API accepts a query and candidate documents and returns relevance scores and rankings.

Typical decision:

rerank-2.5
→ stronger model

rerank-2.5-lite
→ lower cost and latency

Voyage also publishes domain-oriented embedding models, although you should still evaluate whether its general reranker performs well on your own domain.

### Jina AI Reranker

Jina offers several rerankers:

jina-reranker-v3.5: multilingual, listwise reranking
jina-reranker-v2-base-multilingual: multilingual text, code and function-search support
jina-reranker-m0: multimodal reranking for visually rich pages, tables and images

The multimodal option is especially interesting for PDFs whose meaning depends on page layout, diagrams or tables.

### Azure AI Search Semantic Ranker

In an Azure architecture, Azure AI Search can provide a second-stage semantic ranking capability on top of initial search results. It is attractive when retrieval, security filtering and ranking are already being implemented inside Azure AI Search.

Architecturally, this means:

Azure AI Search retrieval
        ↓
Semantic ranking
        ↓
Top results
        ↓
Azure OpenAI / another LLM

It reduces the number of separately operated components, though you trade some model-level control for platform integration.

### Pinecone Rerank

Pinecone also provides a reranker designed to improve the precision of initially retrieved candidates. Its reranker uses joint query-document processing rather than relying only on embedding distance.

This may be convenient when Pinecone already hosts your vector index.

Open or self-hosted rerankers

These are useful when:

documents cannot leave your environment,
predictable cost matters,
you need model customization,
you can operate model-serving infrastructure.
BGE rerankers

Popular examples include:

BAAI/bge-reranker-base
BAAI/bge-reranker-large
BAAI/bge-reranker-v2-m3

bge-reranker-v2-m3 is multilingual and accepts a query-document pair to produce a relevance score. The BGE documentation explicitly describes the common pattern of retrieving a larger candidate set and then reranking it down to a small final set.

### Example architecture:

Vector search: top 50
        ↓
BGE reranker
        ↓
Top 5
Sentence Transformers cross-encoders

The **sentence-transformers library** supports cross-encoder models for reranking. It offers pretrained models and can also be used to fine-tune a reranker on domain-specific relevance examples.

A simplified example:

from sentence_transformers import CrossEncoder

model = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

pairs = [
    ["How do I renew a certificate?", chunk]
    for chunk in candidate_chunks
]

scores = model.predict(pairs)

This is an excellent learning option because you can see the mechanics directly.

Jina and BGE through Hugging Face

Some Jina and BGE models can also be downloaded and self-hosted through Hugging Face, subject to each model’s licence. Be careful here: “available on Hugging Face” does not automatically mean unrestricted commercial use. For example, some Jina model variants use non-commercial licences when self-hosted, while their managed API may have different commercial terms.

A practical starting recommendation

For your learning project:

First:
BGE reranker through sentence-transformers

Then compare against:
Cohere, Voyage or Jina managed API

This gives you both perspectives:

Self-hosted
→ model serving, latency and infrastructure

Managed
→ integration simplicity and operational convenience

For a Microsoft-oriented enterprise design, I would also evaluate:

Azure AI Search semantic ranking

because identity, ACL filtering, indexing and ranking can potentially stay within the Azure architecture.