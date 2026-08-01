# Chapter 3 — Embeddings

## Table of Contents

- [Traditional Search](#traditional-search)
- [Embedding](#embedding)
- [Vector DB](#vector-db)

This is the concept that makes RAG, semantic search, vector databases, recommendation engines, and modern AI search possible.

Once you understand embeddings, you'll finally understand why you asked me about LanceDB a few days ago.

Let's Start With a Problem

Suppose you have a document that says:

"The physician prescribed antibiotics."

The user searches:

"What did the doctor prescribe?"

Will a traditional keyword search find it?

Maybe not.

Why?

Because:

doctor ≠ physician

The words are different.

Computers compare text literally.

## Traditional Search

Traditional search is usually based on matching words (plus techniques like stemming, ranking, etc.).

Example:

Search:

doctor

Document:

physician

No exact match.

The search engine may not return it.

Humans Think Differently

When you read:

doctor

You immediately think:

physician
surgeon
medical professional
clinician

Your brain understands meaning, not just spelling.

Can we teach computers to do the same?

## Embedding

That is exactly why embeddings exist.

Imagine a Giant Map

Suppose I asked you to arrange these words on a whiteboard.

doctor

physician

hospital

nurse

banana

airplane

Where would you place them?

Probably something like:

doctor      physician

     nurse

 hospital



banana



airplane

Why?

Because similar concepts belong close together.

Humans do this naturally.

Embeddings try to capture that idea mathematically.

Embeddings Are Coordinates

Imagine every word has coordinates.

Instead of:

doctor = 17

We now have:

doctor

(0.82, 0.15, -0.63, ...)

Physician:

(0.80, 0.17, -0.61, ...)

Very similar.

Banana:

(-0.25, 0.90, 0.43, ...)

Very different.

Wait...

Why multiple numbers?

Because meaning is complex.

One number can't represent:

profession
medicine
human
education
anatomy
treatment

Instead we use hundreds or even thousands of dimensions.

Don't panic.

You don't need to imagine 1,536 dimensions.

Just imagine a much richer version of x, y, z coordinates.

Analogy

Think about your home.

Two-dimensional coordinates:

x = 18

y = 42

Easy.

Now imagine describing a person.

How many dimensions?

Height
Weight
Age
Salary
Experience
Skills
Languages
Education

Suddenly one number isn't enough.

Language is even richer.

So embeddings use many dimensions.

How Similarity Works

Suppose:

doctor

(1.2, 3.5)

Physician

(1.1, 3.6)

Very close.

Banana

(8.4, -7.1)

Far away.

**The closer two vectors are, the more similar their meanings are likely to be.**

This is the foundation of semantic search.

Real Example

Suppose we embed these sentences.

Sentence A

How do I reset my password?

Sentence B

I forgot my login credentials.

Different words.

Same meaning.

Their embedding vectors should be close.

Now:

Sentence C

How do I bake a cake?

Far away.

This Changes Search Completely

Traditional search asks:

Do these words match?

Semantic search asks:

Do these meanings match?

That's a huge shift.

Where Do Embeddings Come From?

You might ask:

Do we manually assign these coordinates?

No.

The embedding model learns them during training.

You call an embedding model like:

embedding = model.embed("doctor")

It returns something like:

[0.182,
-0.413,
0.772,
...
1536 numbers]

Those numbers become the vector representation of the text.

Where Are These Stored?

Imagine your company has:

10 million documents

Each document becomes a vector.

Where do you store them?

In a normal SQL table?

You could...

But finding the "closest" vectors among millions efficiently is difficult.

## Vector DB

That's why **vector databases** exist.

Examples:

LanceDB
Pinecone
Qdrant
Weaviate
Milvus
pgvector (PostgreSQL extension)

Their job is to efficiently find vectors that are most similar to a query.

This Is Why LanceDB Exists

Remember when you asked:

"What is LanceDB?"

Now you know.

It's not "an AI database."

It's a database optimized for storing vectors and performing fast similarity search.

The Complete Flow

Suppose a user asks:

How can I reset my password?

The system works like this:

```text
User Question
      │
      ▼
Embedding Model
      │
      ▼
Question Vector
      │
      ▼
Vector Database
      │
Find Similar Documents
      │
      ▼
Top Matching Documents
      │
      ▼
LLM
      │
      ▼
Final Answer
```

Notice something.

The LLM wasn't searching.

The vector database searched.

The LLM answered.

This distinction is fundamental.

Architect Perspective

This is one of the biggest misconceptions in enterprise AI.

Many people think:

ChatGPT searches my documents.

No.

The retrieval layer searches.

The LLM reasons over the retrieved information.

That separation of responsibilities makes systems easier to scale, secure, and maintain.

Engineer Perspective

An embedding is simply another representation of your data.

Think about a customer in an application.

You might have:

{
  "id": 101,
  "name": "Rajnish",
  "email": "...",
  "role": "Manager"
}

Now imagine adding:

embedding:
[0.182,
-0.713,
0.441,
...]

The embedding isn't replacing the original data.

It's additional information that enables semantic search.