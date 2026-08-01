Before AI, we need to answer one question.

What exactly are we building?

People often say:

"We're building an AI application."

That's too vague.

Instead, imagine we're building a software system whose intelligence comes from an LLM.

That's a subtle but important distinction.

Think of an e-commerce website.

                Amazon
──────────────────────────────────────

Frontend

↓

Backend APIs

↓

Database

↓

Payment Gateway

↓

Recommendation Engine

The recommendation engine is just one component.

Similarly, in an AI application:

              Enterprise AI App
──────────────────────────────────────

Web UI

↓

Backend APIs

↓

Authentication

↓

Business Logic

↓

LLM

↓

Vector DB

↓

Storage

↓

Observability

↓

Evaluation

The LLM is one component of a larger system.

Architect's takeaway: Don't think of "building with AI." Think of "building software systems that incorporate AI."

Why are LLMs such a big deal?

Let's go back 20 years.

Imagine building a chatbot in 2008.

You'd write rules like:

IF user says hello
    reply "Hello"

IF user asks password reset
    reply "Click Forgot Password"

IF user asks refund
    open refund workflow

Now imagine handling millions of ways humans ask the same thing.

Examples:

I forgot my password.
Can't log in.
My password isn't working.
Locked out of my account.
Help me access my profile.

You'd end up writing thousands of rules.

This approach doesn't scale.

The breakthrough

Researchers asked a different question.

Instead of programming language,

Can we teach a computer language by exposing it to massive amounts of text?

This is similar to how children learn.

A child isn't given a grammar rulebook first.

They hear language repeatedly and gradually recognize patterns.

LLMs are trained in a somewhat analogous way: they learn statistical relationships in language from enormous datasets rather than hand-written rules.

## The Core Idea

Everything in an LLM comes down to one surprisingly simple objective:

Predict the next token.

That's it.

Seriously.

When you type:

The capital of France is

The model predicts:

Paris

Then:

.

Then the next token after that.

One token at a time.

This simple objective, learned over vast amounts of data, leads to remarkably capable behavior.

Wait... just predicting the next token?

Yes.

This surprises almost everyone.

Let's see why it's powerful.

Suppose the model has seen millions of examples like:

2 + 2 = 4

The sky is blue.

Paris is the capital of France.

Python uses indentation.

Over time, it learns patterns.

Not just words.

Patterns.

Relationships.

Grammar.

Logic.

Programming syntax.

Reasoning structures.

It's still predicting the next token—but because the training data contains so many patterns, that prediction becomes highly informative.

A useful analogy

Imagine a friend who has read every programming book, every Stack Overflow post, every Wikipedia article, every novel, and millions of websites.

If you ask:

"Write a Python function to reverse a linked list."

They're not searching the internet in real time.

They're generating an answer based on patterns they've learned.

LLMs work differently from humans internally, but this analogy is useful for understanding why they can respond so fluently.

## AI vs Machine Learning vs Deep Learning

This hierarchy is worth memorizing conceptually.

Artificial Intelligence
│
├── Machine Learning
│
│      ├── Deep Learning
│      │
│      │      ├── Computer Vision
│      │      ├── Speech
│      │      ├── NLP
│      │      │
│      │      └── Large Language Models

### AI

The broad field.

Anything attempting to perform tasks associated with human intelligence.

Examples:

Chess engines
Self-driving cars
Recommendation systems
Spam filters
LLMs
Machine Learning

Instead of writing explicit rules,

we let the computer learn patterns from data.

Example:

Spam detection.

Instead of:

IF email contains "FREE MONEY"

    spam

We provide millions of labeled emails, and the model learns the patterns that distinguish spam from legitimate mail.

## Deep Learning

Machine Learning using large neural networks.

This made possible:

Image recognition
Speech recognition
Translation
LLMs
NLP

Natural Language Processing.

Everything involving human language.

Examples:

Translation
Summarization
Question answering
Sentiment analysis

LLMs are one part of NLP.

The Interview Perspective

Suppose an interviewer asks:

"Why are LLMs revolutionary?"

A strong answer could be:

Traditional NLP often relied on task-specific models and hand-engineered features or rules. LLMs learn broad language representations from massive datasets and can perform many different tasks—such as summarization, translation, coding, and question answering—using the same underlying model, often guided only by prompts.

That answer demonstrates understanding of the shift in approach.

## Engineer's Perspective

As an engineer, remember this:

LLM != Application

The application includes:

UI
APIs
Authentication
Databases
Logging
Monitoring
Security
Business logic
LLM integration

The LLM is an important component, but it's part of a larger system.

## Architect's Perspective

When designing AI systems, one of your first questions should be:

"Should this problem use an LLM at all?"

Not every problem needs one.

Examples:

Good fit:

Document summarization
Code explanation
Knowledge assistants
Draft generation

Poor fit:

Simple arithmetic
Deterministic business rules
Basic CRUD operations
Exact financial calculations without verification

Choosing not to use AI when it's unnecessary is often a mark of good architectural judgment.