# Production Fundamentals — Evaluation

If you ask me:

What is the single biggest difference between a demo AI app and a production AI system?

The answer is Evaluation.

Most beginners ask:

"Can the model answer?"

Production teams ask:

"How do we know it is getting better?"

That single mindset difference separates hobby projects from production systems.

## Why Evaluation Exists

Imagine you built your Financial Document Intelligence system.

User asks:

Which company has highest ARR growth?

System answers:

Microsoft.

Great.

Now tomorrow you:

changed chunking,
switched embedding model,
changed top-k from 5 to 10,
added reranking,
upgraded GPT model.

Question:

Did your system become better?

How do you know?

Without evaluation, you don't.

You only have anecdotes.

**Production AI is driven by evidence, not intuition.

Every AI Change Needs Evaluation**

Suppose you change any one of these:

Embedding model

Chunk size

Chunk overlap

Vector DB

Retriever

Reranker

Prompt

LLM

Temperature

Top-K

Memory strategy

Agent Planner

Tool descriptions

Every one of these can make the system:

better,
worse,
faster,
slower,
cheaper,
more expensive.

Without evaluation, you're flying blind.

The AI Engineering Loop

Traditional Software

Code

↓

Tests

↓

Deploy

AI Systems

Change

↓

Evaluate

↓

Compare

↓

Deploy

Notice:

Testing becomes Evaluation.

## Types of Evaluation

There are five major categories.

We'll cover all of them today.

1. Retrieval Evaluation

2. Generation Evaluation

3. Agent Evaluation

4. Business Evaluation

5. System Evaluation

### 1. Retrieval Evaluation

Suppose user asks:

Driver certificate revocation

Retriever returns:

Chunk A
Certificate renewal

Chunk B
Certificate revocation

Chunk C
Submission API

Question:

Was retrieval good?

We evaluate:

Retrieval Recall

Did retrieval find the right chunk?

Retrieval Precision

Did retrieval avoid irrelevant chunks?

Ranking Quality

Was the best chunk ranked first?

Example

Ground Truth

Correct chunk:

Certificate Revocation Policy

Retriever

1 Certificate Renewal

2 Certificate Revocation

3 Driver Packaging

Retriever technically found it.

But ranking is poor.

Evaluation catches this.

### 2. Generation Evaluation

Retriever works perfectly.

LLM says:

Revocation means certificate renewal.

Wrong.

Retriever succeeded.

Generator failed.

Now we evaluate:

factual correctness,
groundedness,
hallucination,
completeness,
citation correctness,
instruction following.

Example

Retrieved:

Revenue = $120M

LLM:

Revenue = $140M

Retriever perfect.

Generation failed.

### 3. Agent Evaluation

Suppose your investigation agent.

Planner decides:

Search logs

↓

Search deployment

↓

Compare versions

↓

DONE

Question:

Was that a good plan?

Maybe not.

It forgot:

Query Metrics

Evaluation now measures:

Planning quality

Tool selection

Tool ordering

Number of retries

Budget

Reasoning quality

Stop criteria

### 4. Business Evaluation

Ultimately business doesn't care if retrieval precision improved 2%.

They care about:

Resolution time

Support tickets

Customer satisfaction

Revenue

Developer productivity

This is highest-level evaluation.

### 5. System Evaluation

Measures:

Latency

Cost

Availability

Throughput

Token usage

Failure rate

## Offline vs Online Evaluation

One of the most important concepts.

### Offline

You already know expected answer.

Example

Question

Revenue?

Expected

120M

Run system.

Compare.

Offline.

Perfect for development.

###cOnline

Production.

Real users.

Unknown answers.

Measure:

#### Latency

#### Thumbs up

#### Task success

#### Human feedback

#### Escalation

## Golden Dataset

This is one of the most important production concepts.

Imagine your financial system.

Instead of random testing.

You create:

Question

Expected Answer

Expected Citations

Difficulty

Category

Example

Question

Highest ARR?

Expected

Microsoft

Citation

Annual Report pg 17

Now every code change runs against these questions.

Exactly like unit tests.

LLM-as-a-Judge

Huge trend today.

Instead of humans grading every answer.

Use another LLM.

Example.

Prompt

Question

Expected

Actual

Score correctness from 1-5.

Explain.

Judge returns

Correctness 4.8

Grounded 5

Missing EBITDA

Very common today.

Human Evaluation

Still necessary.

Especially for:

Safety

Legal

Medical

Financial

Enterprise

Humans score:

Correctness

Helpfulness

Tone

Completeness

Safety

## What do we actually measure?

Here are the most common production metrics.

### Correctness

Is answer correct?

### Faithfulness

Did answer come from retrieved documents?

This is huge.

Question:

Revenue?

Document

120M

Answer

120M

Faithful.

Question

Revenue?

Document

120M

Answer

140M

Not faithful.

Hallucination.

### Groundedness

Can every important statement be backed by evidence?

Different from correctness.

### Relevance

Did answer actually answer question?

### Completeness

Did it answer everything?

### Citation Accuracy

Huge.

Especially for enterprise.

Every citation should point to:

Correct document

Correct section

Correct page

Evaluation Pyramid

## Think of evaluation like a pyramid.

Business KPIs
       ▲
Agent Evaluation
       ▲
Generation Evaluation
       ▲
Retrieval Evaluation
       ▲
Infrastructure Metrics

You need every layer.

What companies actually do

OpenAI

Anthropic

Microsoft

Google

Meta

All have:

Thousands to millions of evaluation cases.

Every model change.

Every prompt change.

Every retriever change.

Everything gets evaluated.

Nobody relies on intuition.

Interview Questions

These are extremely common.

How would you know your RAG system improved?

How do you compare two prompts?

How do you evaluate a new embedding model?

How would you reduce hallucinations?

How would you test an AI agent before deployment?

Every one of those is fundamentally an evaluation question.

Architect's Mental Model

This is the one thing I want you to remember.

Traditional software asks:

Does it work?

AI systems ask:

Is this version measurably better than the previous version?

Everything in production AI revolves around that question.