# Chapter 11 — Prompt Construction

First, let's clear up a common misconception.

When people hear Prompt Engineering, they often think of prompts like:

You are the world's best software engineer.
Think step by step.
Answer carefully.

These techniques can have some effect, but in enterprise AI they are not where most of the value comes from.

A production prompt is much more like assembling a legal brief than writing a clever instruction.

## A Mental Model

Imagine you're interviewing a candidate.

Would you ask:

"Answer this question."

Or would you first hand them the relevant documents?

Obviously the second.

The LLM is similar.

It reasons over the information you give it.

Garbage context → garbage answers.

Good context → good answers.

## Prompt Builder

The Prompt Builder

Remember our pipeline:

User Question
      │
Query Transformation
      │
Retrieval
      │
RRF
      │
Reranker
      │
Prompt Builder
      │
LLM

We've arrived here.

The Prompt Builder's job is not to be creative.

Its job is to package information.

Prompt Builders often perform content extraction, not just document selection.

## What goes into a production prompt?

A typical enterprise prompt contains four major sections.

System Instructions

↓

User Question

↓

Retrieved Context

↓

Output Instructions

Let's examine each.

### System Instructions

These define the assistant's behavior.

Example:

You are an enterprise assistant.

Answer only using the supplied context.

If the answer cannot be determined,
say you don't know.

Do not invent information.

Notice what isn't there.

No:

"You are the smartest AI."

Those fluffy instructions rarely add value.

The useful instructions are about constraints.

### User Question

Exactly what the user asked.

Example:

Can extension drivers be submitted
as derived submissions?

Nothing surprising here.

### Retrieved Context

This is the most important section.

Example:

Document: WHCP Guide

Section:
Derived submissions...

...

Document:
Partner FAQ

Section:
Extension drivers...

Notice:

The LLM isn't searching anymore.

We've already done that.

Now we're simply presenting evidence

### Output Instructions

These tell the model how to respond.

Example:

Summarize in three bullets.

Include citations.

If multiple documents disagree,
mention the disagreement.

This controls formatting and behavior.

## Putting it together

A simplified production prompt might look like:

System:
You are an enterprise assistant.
Answer only from supplied context.

Question:
Can extension drivers be submitted as derived submissions?

Context:
[Chunk 1]

[Chunk 2]

[Chunk 3]

Instructions:
Answer briefly.
Include citations.

Simple.

But there's a lot of engineering hidden behind those [Chunk] placeholders.

## The Biggest Constraint: Context Window

Remember tokens?

Suppose your model accepts:

128,000 tokens

Does that mean you should always stuff in 128k tokens?

Absolutely not.

Why?

Three reasons:

Cost
Latency
Model performance

Even if the model can read everything, more isn't always better.

### Example

Example

Suppose retrieval gives you:

Top 50 chunks

Should you include all 50?

Probably not.

Maybe only:

Top 5

Why?

Because the reranker already identified the strongest evidence.

The prompt builder trusts that work.

## Prompt Construction is a Budgeting Problem

Think of the context window as a budget.

Suppose your model supports 32,000 tokens.

You might allocate them like this:

Component	Tokens
System prompt	500
User question	100
Retrieved context	28,000
Instructions	200
Safety margin	3,200

Notice something.

The majority of the budget goes to context, not instructions.

## Architect Principle

The LLM should spend its compute reasoning, not filtering.

Filtering is the retriever's job.

## Ordering Matters

Suppose we have three retrieved chunks.

Chunk A

Definition

Chunk B

Detailed Procedure

Chunk C

Exception

Should we order them randomly?

No.

The order influences comprehension.

A natural order might be:

Definition

↓

Procedure

↓

Exceptions

The LLM reads more like a human than a database.

Logical flow helps.

## The "Lost in the Middle" Problem

This is one of the most important discoveries in modern LLM research.

Imagine a prompt like this:

Important

...

...

...

Very long context

...

...

Critical fact

...

...

...

End

Models often remember:

the beginning
the end

But perform worse on information buried in the middle.

This phenomenon is called:

Lost in the Middle

### Why does this matter?

Suppose your most relevant chunk is ranked first.

Your second-most relevant chunk is ranked second.

Then you append 80 pages of documentation.

That second chunk is now buried.

The model may pay less attention to it.

### Production Solutions

Several strategies exist.

Strategy 1

Only include the highest-quality chunks.

Often the best solution.

Strategy 2

Put the most important chunks first.

Strategy 3

Sometimes repeat critical facts in a summary.

Not by duplicating large sections, but by synthesizing the key evidence before presenting the full context.

## Citations

Suppose your chunk contains:

Section 4.2

Derived submissions
are supported only...

Instead of answering:

Yes.

The model can answer:

Yes. According to Section 4.2, derived submissions are supported only under specific conditions.

Notice something.

The citation wasn't generated magically.

The retrieval system already supplied:

document name
page
section
chunk metadata

The prompt builder included it.

## Grounding

This is a crucial concept.

Bad prompt:

Answer the question.

Good prompt:

Answer only using the supplied context.

If the answer isn't present,
say you don't know.

This is called grounding.

You're anchoring the model to evidence.

Grounding reduces hallucinations, though it doesn't eliminate them entirely.

## The Prompt Builder is More Than String Concatenation

Many beginners think:

prompt = system + context + question

In reality, a production Prompt Builder often:

selects which chunks to include,
orders them,
removes duplicates,
enforces token budgets,
adds citations,
injects metadata,
formats tables,
preserves code blocks,
truncates safely.

It's a real software component.

## Applying This to Your GitHub Assistant

User asks:

Where is CreateSubmissionAsync() implemented?

The Prompt Builder might include:

the source file,
surrounding method,
interface definition,
XML comments,
call sites (if needed).

It would not dump the entire repository.

## The End-to-End Picture

 User Question
                           │
                    Query Classifier
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
     Direct Lookup              Query Transformation
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

##  Retriever vs Reranker vs Prompt Builder

Entire Knowledge Base
        │
        ▼
Retriever
(Find candidates)
        │
        ▼
Reranker
(Find best evidence)
        │
        ▼
Prompt Builder
(Find the best presentation)
        │
        ▼
       LLM