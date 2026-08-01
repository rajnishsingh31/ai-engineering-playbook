
# Chunking: The Art of Preparing Knowledge for AI

## Table of Contents

- [Architect's Rule #1: A great embedding model cannot compensate for poor chunking.](#architects-rule-1-a-great-embedding-model-cannot-compensate-for-poor-chunking)
- [what is chunk?](#what-is-chunk)
- [Chunking pipeline](#chunking-pipeline)
- [Chunk Size Trade-offs](#chunk-size-trade-offs)
- [Metadata Matters](#metadata-matters)
- [Architect Perspective](#architect-perspective)
- [Engineer Perspective](#engineer-perspective)
- [Architecture Update](#architecture-update)

## Architect's Rule #1: A great embedding model cannot compensate for poor chunking.

I've seen production systems where teams spent weeks comparing embedding models, only to discover that the real issue was their chunking strategy.

## what is chunk?

A chunk is simply:

A small, self-contained piece of text that can stand on its own.

Example:

Employees may carry forward
up to 15 days of unused leave
into the next calendar year.

That's a good chunk.

Bad chunk:

...carry forward up to...

...

calendar year unless...

No context.

Impossible for the LLM to understand properly.

## Chunking pipeline

```text
PDF
↓
Extract Text
↓
Split into Chunks
↓
Generate Embeddings
↓
Store in Vector DB
```

Notice:

The embedding model never sees the entire document.

It sees one chunk at a time.

## Chunk Size Trade-offs

This is where architects earn their salary.

Suppose we choose:

Very Small Chunks
Sentence 1

Sentence 2

Sentence 3

Advantages:

Very precise retrieval
Low token usage

Problems:

Lose surrounding context
May separate explanation from definition

Example:

Chunk A

Employees may carry forward...

Chunk B

...unless employed less than one year.

These should probably stay together.

Very Large Chunks
Five pages together

Advantages:

Rich context

Problems:

Expensive
More irrelevant information
Lower retrieval precision
Larger prompts
The Sweet Spot

Many production systems use chunks around:

300–800 tokens

Some use:

200
500
1000

There is no universal best value.

It depends on:

document type,
question type,
model context window,
evaluation results.

### Fixed-Size Chunking

Simplest approach:

Every 500 tokens

Advantages:

Easy
Fast

Problem:

May split in the middle of a sentence or section.

### Semantic chunking

Instead of counting tokens, split by meaning.

For example:

Chapter

↓

Section

↓

Subsection

Or:

Heading

Paragraph

Paragraph

This preserves logical structure.

Usually better retrieval.

### Overlapping Chunks

One of the smartest tricks in RAG.

Suppose we split like this:

Chunk 1

Sentence 1
Sentence 2
Sentence 3
Sentence 4
Chunk 2

Sentence 4
Sentence 5
Sentence 6
Sentence 7

Notice:

Sentence 4 appears twice.

That's called overlap.

Why?

Imagine the answer sits right on the boundary.

Without overlap:

Definition

↓

Explanation

They become separated.

With overlap:

Both chunks retain enough context.

Typical overlap:

Chunk size

500 tokens

Overlap

50–100 tokens

Again, this is tuned based on experiments.

## Metadata Matters

Remember our vector database?

We don't just store:

Chunk

Embedding

We also store metadata.

Example:

{
  "document": "HR Handbook",
  "section": "Leave Policy",
  "page": 18,
  "version": "2026.2",
  "updated": "2026-06-01"
}

This becomes incredibly useful later for:

citations,
filtering,
debugging,
ranking.

## Architect Perspective

When designing a RAG system, ask:

What is the natural unit of knowledge?
Should chunks follow document structure?
Should tables stay intact?
Should code blocks stay together?
Should headings be included with paragraphs?
Should overlap be used?
How will we evaluate chunk quality?

These questions often have a larger impact than switching between embedding models.

## Engineer Perspective

When implementing chunking, you'll decide:

Chunk size
Overlap size
Token-based or semantic splitting
Metadata to store
Handling of tables, lists, code, images, and headings
Re-chunking strategy when documents change

## Architecture Update

```text
OFFLINE INDEXING
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
Vector DB
```

## Libraries and services for chunking

There are many tools that perform chunking. Fewer tools genuinely derive the best strategy automatically.

### LangChain text splitters

LangChain provides multiple splitting methods, including:

recursive character splitting,
token-based splitting,
language-aware code splitting,
HTML and Markdown-aware splitters,
semantic splitting through integrations.

Its text-splitter components are designed to make large documents individually retrievable while staying within model context limits.

Example:

from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=75
)

chunks = splitter.split_text(document_text)

This applies a strategy you configure; it does not prove that 500/75 is optimal.

### LlamaIndex

LlamaIndex provides node parsers and splitters such as:

sentence splitting,
token splitting,
semantic splitting,
Markdown-aware parsing,
hierarchical node parsing,
code-aware parsing.

A semantic splitter typically embeds smaller units such as sentences and creates a boundary when adjacent sections become semantically dissimilar.

Conceptually:

Sentence 1 ─ similar ┐
Sentence 2 ─ similar ├─ Chunk A
Sentence 3 ─ similar ┘

Large semantic change
        ↓

Sentence 4 ─ similar ┐
Sentence 5 ─ similar ┘ Chunk B

This can discover topic transitions, but it costs more than simple structural splitting and is not automatically superior for every domain.

Unstructured

Unstructured is especially valuable for document ingestion.

It first partitions files into structured elements such as:

Title
NarrativeText
ListItem
Table
Header
Footer

You can then chunk using that document structure. It supports combining partitioning and chunking in an ingestion request.

This is substantially better than extracting a PDF into one flat text string and splitting every 500 characters.

Example output:

Title: Certificate Renewal

NarrativeText: Certificates must be renewed...

Table: Certificate type | Validity period

Title: Troubleshooting

Chunking can then preserve:

heading + related paragraphs + table
Azure AI Search integrated chunking

Azure AI Search supports built-in indexing pipelines that can:

Read documents
    ↓
Split content
    ↓
Generate embeddings
    ↓
Populate the search index

Its built-in indexers and skillsets can automate both chunking and vectorization.

This is convenient for Azure-native systems, but you still configure choices such as:

splitting mode,
page or token length,
overlap,
extraction approach,
metadata propagation.

It automates the pipeline, not the architectural decision about what strategy is best.

### Document-specific parsers

For high-quality RAG, parsing and chunking often need to work together.

Examples of useful categories include:

PDF/layout parsing:
Unstructured, Docling, Azure Document Intelligence

Source code:
Tree-sitter-based parsers, language-aware splitters

Markdown:
Heading-aware splitters

HTML:
DOM-aware splitters

Tables:
Table extraction followed by table-aware chunking

For code, syntax-tree chunking is typically safer than blindly splitting tokens because it can preserve complete functions, classes and methods.

#### Can a service automatically find the best strategy?

Not reliably from only the document.

Suppose a system sees a legal contract. It may determine that clause-level chunking is structurally sensible. But it cannot know whether users will ask:

What is the termination notice period?

or:

Summarize all obligations spanning Sections 8–12.

Those two query patterns favour different chunk sizes and retrieval methods.

#### The “best” chunking strategy depends on:

Documents
+ expected questions
+ embedding model
+ retrieval method
+ reranker
+ top-k
+ answer requirements

Recent research also finds that chunking effectiveness varies significantly by domain; structure-aware and content-aware approaches often outperform naive fixed-length splitting, but no single method wins universally.

So the reliable solution is an evaluation-driven chunking optimizer.

How such a system works

Create several candidate indexes:

Strategy A:
300 tokens, 50 overlap

Strategy B:
600 tokens, 100 overlap

Strategy C:
Heading-aware

Strategy D:
Semantic chunking

Strategy E:
Parent-child hierarchical chunking

Then evaluate each strategy using representative questions.

For every question, measure:

Did the correct chunk appear in top 5?
Where was the correct chunk ranked?
How much irrelevant text was retrieved?
Did the final answer cite the right source?
What were latency and indexing cost?

Possible metrics:

Recall@K
Hit Rate@K
MRR
nDCG
Context precision
Context recall
Answer groundedness
Latency
Index size

The winner is not necessarily the most accurate strategy. It may be the one with the best acceptable balance of:

quality + latency + cost + maintainability
What I would recommend for your financial-document project

Since your earlier project includes PDFs, Word documents, spreadsheets and financial reports:

1. Parse structure
   Use Unstructured, Docling or Azure Document Intelligence.

2. Classify content elements
   Narrative, heading, table, footnote, financial statement.

3. Apply type-specific chunking
   Paragraphs by section.
   Tables kept intact or converted into structured Markdown.
   Footnotes linked to their associated table.
   Spreadsheet ranges grouped by logical financial statement.

4. Create two or three candidate chunking strategies.

5. Build 30–50 representative questions.

6. Evaluate Recall@K and final answer grounding.

7. Choose the strategy based on measured results.

For example, a financial table should not become:

Chunk 1:
Revenue | 2025 | 2026

Chunk 2:
$12M | $16M

The header and values must remain together:

Company: Acme
Statement: Revenue summary
Period: FY2025–FY2026

Metric | FY2025 | FY2026
Revenue | $12M | $16M
Final mental model
Chunking library
→ Implements splitting techniques

Document parser
→ Discovers structure and content types

Evaluation framework
→ Determines which strategy works best

Architect
→ Chooses the acceptable quality/cost trade-off

So yes, tools can automate much of the work—but today, the “best chunking strategy” is generally measured, not magically inferred.