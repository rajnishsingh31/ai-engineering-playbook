# Chapter 2 - Tokens and Context

## Table of Contents

- [What is a Token?](#what-is-a-token)
- [Context Window](#context-window)
- [Architect Perspective](#architect-perspective)

Let's Start with a Simple Question

Imagine I ask you:

Write a Python function to sort a list.

How do you answer?

Let's think about what happens inside your brain.

You don't search a database.

You don't scan a textbook.

Instead:

You read the sentence.
You understand the meaning.
You remember similar code.
You predict what code should come next.
You continue until complete.

LLMs work surprisingly similarly at a high level.

But Computers Don't Understand Words

This is the first major concept.

Computers don't understand:

Hello
Python
Apple
India

To a computer these are just characters.

Eventually...

Everything becomes numbers.

Always.

Why?

Suppose I ask you to compare

Apple

Orange

A computer cannot compare words directly.

It needs numbers.

Everything in computing ultimately becomes binary.

010011010101

Neural networks only work on numbers.

So before training starts...

Words must become numbers.

First Attempt

Imagine assigning IDs.

Apple  → 1

Orange → 2

Banana → 3

Python → 4

Simple?

Yes.

Useful?

Not at all.

Why?

Because:

King = 10

Queen = 11

Car = 12

The computer doesn't know

King and Queen are related.

It only sees

10

11

12

The numbers have no meaning.

We Need Something Better

Instead of IDs...

We want numbers that capture meaning.

Example:

King

Queen

Prince

Princess

Should somehow be "close."

Whereas

King

Banana

Should be far apart.

That idea leads us toward embeddings, which we'll study in depth later.

For now, just remember:

We want numbers that preserve relationships.

Before Embeddings Comes Tokenization

This is one of the most misunderstood topics.

People think:

ChatGPT reads words.

It doesn't.

It reads tokens.

## What is a Token?

**A token is a unit of text processed by the model.**

Sometimes:

One word

Hello

One token.

Sometimes:

Engineering

May become

Engineer

ing

Two tokens.

Sometimes punctuation is its own token.

.
,
?

Each may be separate.

Sometimes spaces matter too.

Why Split Words?

Imagine storing every possible word.

English alone has hundreds of thousands of words.

Now add:

Python
Java
Hindi
Japanese
SQL
URLs
Emojis

The vocabulary would explode.

Instead...

LLMs learn reusable pieces.

Example:

Play

Player

Playing

Played

Instead of learning four unrelated words...

The tokenizer can reuse pieces.

This makes the vocabulary more manageable and lets the model generalize to words it hasn't seen exactly before.

Think LEGO

Instead of storing every toy...

You store LEGO bricks.

play

ing

er

ed

Then build

Player

Playing

Played

This is far more efficient.

Exercise

Suppose the tokenizer knows:

play

er

ing

ed

How might it tokenize:

Player

Playing

Played

Think before reading on.

Answer:

Player

↓

play

er
Playing

↓

play

ing
Played

↓

play

ed

Notice the reuse.

Why Should You Care?

Because everything in AI is billed, limited, and measured in tokens, not words.

For example:

Not

10,000 words

Instead:

128,000 tokens

200,000 tokens

1 million tokens

When people talk about context windows, they're talking about token capacity.

Interview Question

Why do AI providers charge per token instead of per word?

Think about it.

The model processes tokens, not words.

Some words become one token.

Some become many.

The computational work scales with the number of tokens.

## Context Window

This is another foundational concept.

Imagine I'm telling you a story.

John bought a laptop.

He traveled to Bangalore.

He lost it.

What does

it

refer to?

You immediately know.

The laptop.

Because you still remember the earlier sentences.

Now imagine I continue for another 500 pages.

Eventually...

You'll forget earlier details unless you revisit them.

LLMs have a similar limitation.

**Context Window = Working Memory**

Think of it as the model's short-term memory for a single interaction.

Everything inside the context window is available for reasoning.

Everything outside it is invisible unless you provide it again.

Imagine This

Suppose an LLM has a context window of:

8 tokens

Now send:

I
love
to
learn
AI
with
ChatGPT
every
day

The model can only keep the last 8 tokens.

One token falls off the left side.

love
to
learn
AI
with
ChatGPT
every
day

"I" is gone.

This is a simplified illustration, but it captures the core idea: context capacity is finite.

This Explains So Much

Why can't we put an entire enterprise database into the prompt?

Because it won't fit within the available context window.

Why does RAG exist?

Because we retrieve only the most relevant pieces of information and place those into the context.

Why do agents use tools?

Because they can fetch additional information instead of relying only on what already fits in the prompt.

Engineer Perspective

Suppose you're building an AI assistant.

Your prompt might include:

System instructions
Conversation history
Retrieved documents
User question
Tool outputs

All of these consume context.

Managing that budget is part of building reliable AI applications.

## Architect Perspective

One common misconception is:

"Models with larger context windows make RAG unnecessary."

Not necessarily.

A larger context window helps, but RAG still offers advantages:

You don't send irrelevant information.
You reduce cost by sending only what's needed.
You improve focus by providing targeted context.
You can work with knowledge that changes frequently.

We'll revisit this in detail when we study RAG.