# Model Selection & Cost Optimization

One of the biggest misconceptions is:

"Which model is the best?"

Production teams almost never ask that.

They ask:

"Which model is the best for this task?"

The Biggest Mistake

Imagine your application:

Upload Financial Reports
        ↓
Extract Tables
        ↓
Summarize
        ↓
Compare Companies
        ↓
Generate Report

Many beginners do:

Everything
      ↓
GPT-5.5

It works.

It is also expensive.

Think Like a Software Architect

You wouldn't build every component in C++.

Or every service in SQL.

Different components use different technologies.

AI should be the same.

AI Pipeline

Suppose:

Question
     ↓
Classify
     ↓
Retrieve
     ↓
Extract
     ↓
Compare
     ↓
Generate

Do these require the same intelligence?

No.

AI is a Pipeline of Different Cognitive Tasks

Think of different "thinking levels":

Level 1
Classification

Level 2
Extraction

Level 3
Summarization

Level 4
Reasoning

Level 5
Planning

Not every task needs Level 5.

Example

Financial Project

User uploads PDF

Step 1

Extract text

Need LLM?

No.

OCR/parser.

Step 2

Find Revenue

Need strongest model?

Probably not.

Small model or deterministic parser may be enough.

Step 3

Compare 10 companies

Now reasoning matters.

Use stronger model.

Step 4

Generate report

Medium or strong model.

One workflow.

Different intelligence requirements.

## Model Routing

Instead of:

Everything
      ↓
One Model

Production systems do:

               Router
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
 Small Model  Medium Model  Large Model

### The router chooses.

Routing Signals

What can the router consider?

Task type

Difficulty

Latency

Cost

Tenant

SLA

Context size

Reasoning needed

Example

Question:

Revenue?

Route:

Fast model

Question:

Compare all acquisitions over five years and identify strategic trends.

Route:

Reasoning model

## Multi-Model Architecture

Modern AI platforms increasingly look like this:

Application
      │
      ▼
Model Gateway
      │
 ┌────┼────┐
 ▼    ▼    ▼
GPT Claude Gemini

The application doesn't care which model answers.

The gateway decides.

Why a Model Gateway?

### Benefits:

fallback if one provider fails,
A/B testing,
cost optimization,
regional routing,
compliance,
model upgrades without changing application code.

Exactly like an API Gateway.

## Cost Optimization

Most cost comes from:

Input Tokens

Output Tokens

Embeddings

Repeated Calls

Not from requests.

This connects back to our Context Engineering discussion.

Biggest Cost Saver

Surprisingly, it's usually:

Better context.

Why?

Because:

fewer irrelevant tokens,
fewer retries,
shorter prompts,
fewer hallucinations,
fewer follow-up questions.

Good Context Engineering often saves more money than switching models.

## Latency Optimization

Where does latency come from?

Retrieval

↓

Reranker

↓

LLM

↓

Tool Calls

↓

Additional LLM Calls

The largest contributor is often:

large prompts,
multiple LLM calls,
slow tools.

Observability helps identify bottlenecks.

## Caching

One of the easiest wins.

Different cache layers:

Embedding Cache

↓

Retrieval Cache

↓

Prompt Cache

↓

LLM Response Cache

↓

Tool Result Cache

Example:

100 users ask:

What is ARR?

No reason to regenerate the same explanation 100 times.

Fallback Strategy

Suppose GPT is unavailable.

Application
      │
      ▼
Gateway
      │
GPT fails
      │
      ▼
Claude

The application continues working.

## Production systems should plan for provider failures.

### A/B Testing

You have:

Prompt v1

Prompt v2

or

GPT

Claude

Send a percentage of traffic to each and compare using the evaluation pipeline.

Don't switch based on intuition.

Build vs Buy

As an architect, you'll often ask:

Should we:

call a hosted model,
use Azure/OpenAI,
run an open-source model,
fine-tune,
use Retrieval-Augmented Generation instead?

The answer depends on:

latency,
privacy,
cost,
regulation,
workload,
expertise.

There is rarely one universally correct answer.

The AI Platform View

## An enterprise platform might look like:

                Applications
                      │
                      ▼
               AI Gateway
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 Context Service  Evaluation     Observability
      │
      ▼
 Retrieval
      │
      ▼
 Model Router
      │
 ┌────┼────┬────┐
 ▼    ▼    ▼    ▼
GPT Claude Gemini Local

Notice something:

The model is just one service in the architecture.

Architect's Mental Model

Instead of asking:

Which model is best?

Ask:

Which task am I solving?
What reasoning level is required?
What latency is acceptable?
What cost is acceptable?
What compliance constraints exist?
Can a simpler solution work?

Those questions lead to better architectures than chasing benchmark leaderboards.

Five Rules I Use

If I were reviewing an architecture, these are the rules I'd look for:

Rule 1

Don't use the strongest model everywhere.

Rule 2

Route by task complexity, not user.

Rule 3

Improve context before upgrading models.

Rule 4

Measure with evaluation before switching providers.

Rule 5

Hide providers behind a model gateway.

That keeps your application portable and future-proof.

AI Architect Insight

Here's something that took many companies a while to realize:

Early AI systems were model-centric.

Best Model
     ↓
Everything else

Modern production AI systems are system-centric.

Context
+
Evaluation
+
Security
+
Observability
+
Routing
+
Models

The model is one component of a larger engineering system.