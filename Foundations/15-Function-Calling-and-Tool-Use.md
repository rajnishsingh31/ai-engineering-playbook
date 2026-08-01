# Chapter 15 — Function Calling & Tool Use

Let's start with a simple question.

Suppose a user asks:

"What's the weather in Bangalore?"

Can the LLM answer this?

No.

Why?

Because weather changes every minute.

The model was trained on historical data.

It has no idea whether it's raining right now.

So what should it do?

Not guess.

It should ask another system.

That's exactly what tool use is.

Before Tool Calling

Old pipeline:

User
   │
   ▼
LLM
   │
   ▼
Answer

The LLM could only answer from:

training knowledge
prompt
retrieved context

Nothing else.

## After Tool Calling

Now the pipeline becomes:

User
   │
   ▼
LLM
   │
   ▼
Should I use a tool?
   │
 ┌─┴────────────┐
 │              │
No             Yes
 │              │
 ▼              ▼
Answer      Tool Request
                 │
                 ▼
         External System
                 │
                 ▼
          Tool Result
                 │
                 ▼
               LLM
                 │
                 ▼
             Final Answer

Notice something important.

The LLM does not execute the tool.

The application does.

What Is a Tool?

**A tool is simply a capability the LLM cannot perform itself.**

Examples:

Search documents

Read SQL

Call REST API

Read GitHub

Execute Python

Check Calendar

Send Email

Weather

Stock prices

ERP system

CRM

The LLM reasons.

The tools perform actions.

Analogy

Think of an architect.

The architect doesn't:

pour concrete
install wiring
paint walls

Instead:

The architect decides:

We need an electrician.

Similarly:

The LLM decides:

I need the Calendar API.

The electrician still does the work.

## Tool Definition

**Every tool has three parts.**

### Name

Example:

search_symbol

### Description
Search repository symbols by exact name.

This helps the LLM decide when to use it.

### Parameters

Example:

{
  "symbol_name": "CreateSubmissionAsync"
}

That's all.

Example

User:

Where is CreateSubmissionAsync()?

LLM reasons:

"I don't know."

"I should use search_symbol."

Instead of answering:

It returns:

{
  "tool": "search_symbol",
  "arguments": {
      "symbol_name": "CreateSubmissionAsync"
  }
}

Notice...

This is not the answer.

It is a request.

Important Separation

Many beginners imagine:

LLM
↓

GitHub

Actually:

LLM
↓

Tool Request

↓

Application

↓

GitHub API

↓

Application

↓

LLM

↓

Answer

#### The application is always in control.

Why?

Because only the application knows:

credentials
authentication
retries
authorization
rate limits
audit logging

The LLM should never know your GitHub token.

Function Calling Lifecycle

Step 1

User

↓

Where is CreateSubmissionAsync()?

Step 2

LLM

↓

Need search_symbol tool.

Step 3

Application

↓

Calls GitHub.

Step 4

GitHub

↓

Returns:

DriverSubmission.cs

Line 219

Step 5

LLM

↓

Generates:

CreateSubmissionAsync() is defined in DriverSubmission.cs...

This is the complete lifecycle.

Multiple Tools

Suppose user asks:

Show failed submissions from SQL and summarize the related GitHub code.

Need one tool?

No.

Need two.

Pipeline:

User

↓

LLM

↓

SQL Tool

↓

GitHub Tool

↓

LLM

↓

Answer

## The LLM becomes an orchestrator.

### Parallel vs Sequential

Sometimes tools are independent.

Example:

Weather

Calendar

Can execute together.

Tool A

||

Tool B

Parallel.

Sometimes not.

Example:

Find Customer

↓

Customer ID

↓

Get Orders

Need sequential execution.

Tool Choice

Imagine available tools:

search_symbol

calendar

weather

send_email

github

sql

User:

What time is my meeting tomorrow?

Should it use:

GitHub?

No.

Weather?

No.

Calendar?

Yes.

### Tool selection is itself a reasoning problem.

#### Bad Tool Design

Imagine:

tool1

tool2

tool3

tool4

Descriptions:

Does stuff

General helper

Useful tool

Search

The LLM has no idea.

Descriptions should be specific.

#### Good:

Search repository symbols by exact name.

Search documentation using semantic search.

Retrieve SQL metrics from analytics database.

Read Azure DevOps work items.

Tool descriptions are effectively prompts.

One Tool vs Many

Bad:

enterprise_tool()

Arguments:

mode

operation

type

action

system

query

...

50 parameters.

The LLM struggles.

Better:

search_symbol()

search_document()

search_work_item()

search_sql()

Small focused tools.

Single responsibility.

Exactly like good software engineering.

Tool Results

Never return:

Success

Return useful information.

Bad:

{
    "success": true
}

Good:

{
    "file": "DriverSubmission.cs",
    "line": 219,
    "repository": "DriverServicing"
}

The LLM needs information to continue reasoning.

Tool Errors

Suppose GitHub times out.

Tool returns:

{
   "status":"timeout"
}

Should LLM hallucinate?

No.

Instead:

I couldn't retrieve the repository because GitHub timed out.

Tool failures are part of reasoning.

Retry Responsibility

Who retries?

The LLM?

Usually no.

The application.

Pipeline:

Tool Request

↓

Application

↓

Retry

↓

Retry

↓

Fail

↓

LLM

The application understands:

exponential backoff
HTTP status
retry budgets

The LLM shouldn't reinvent networking.

Authentication

Imagine:

Delete Repository

Tool exists.

Should LLM always use it?

No.

Application checks:

Identity

↓

Permission

↓

Allowed?

↓

Execute

The LLM cannot authorize users.

Tool Calling vs RAG

This confuses many people.

Suppose user asks:

Explain BM25.

Need tool?

No.

Need retrieval?

Maybe.

Suppose user asks:

What's the weather?

Need retrieval?

No.

Need tool?

Yes.

## Difference:

### RAG: Bring knowledge to the LLM.

Retriever

↓

Context

↓

LLM

### Tool Calling

Send work out of the LLM.

LLM

↓

Tool

↓

Result

↓

LLM

One imports knowledge.

The other performs actions.

Combining Both

Repository assistant.

User:

Why is CreateSubmissionAsync failing?

Pipeline:

LLM

↓

Search Symbol

↓

Retrieve Docs

↓

Retrieve Logs

↓

LLM

↓

Answer

Notice:

Some tools perform retrieval.

Others perform actions.

## Stateless Tools

**A tool should behave like a normal function.**

Example:

get_weather(city)

Input:

Bangalore

Output:

Weather.

No hidden state.

Predictable.

Side Effects

Some tools change the world.

Examples:

Send Email

Delete File

Approve PR

Create Ticket

Deploy Build

These require much stronger controls.

**Read-only tools are easier.

Write tools are dangerous.**

### Read vs Write

Read

Search

Get

Fetch

Lookup

Usually safe.

Delete

Approve

Transfer

Purchase

Deploy

High risk.

Never treat them equally.

## Tool Categories

I classify tools into **four groups**.

### Retrieval

Search docs

Search GitHub

SQL read

Vector search

### Computation
Python

Calculator

Simulation

### External APIs
Calendar

Weather

CRM

ERP

Payments

### Action

Send email

Deploy

Delete

Approve

Purchase

### Every category needs different safeguards.

## Architect Perspective

Imagine building your Financial Document Assistant.

Possible tools:

Extract PDF

Read DOCX

Read XLSX

OCR Image

Hybrid Search

SQL Metrics

Currency Converter

Financial API

The LLM shouldn't know how PDFs are parsed.

It only decides:

I need the PDF extraction tool.

## Updated Architecture
                    User
                      │
                      ▼
              Conversation Manager
                      │
                      ▼
              Query Classifier
                      │
                      ▼
             Prompt Builder
                      │
                      ▼
                    LLM
                      │
          Should I use a tool?
          ┌───────────┴───────────┐
          ▼                       ▼
      No Tool                Tool Request
                                  │
                                  ▼
                        Tool Executor (Application)
                                  │
                                  ▼
                         External System / API
                                  │
                                  ▼
                            Tool Response
                                  │
                                  ▼
                                 LLM
                                  │
                                  ▼
                             Final Answer

Notice the **Tool Executor**.

This is one of the most important architectural components in AI systems.

It **isolates**:

authentication,
retries,
logging,
authorization,
rate limiting,
monitoring,
circuit breakers,
caching.

The LLM should never implement those concerns.

A Design Insight

You might have noticed something interesting.

So far we've introduced:

Retriever
Reranker
Prompt Builder
Conversation Manager
Memory Retriever
Tool Executor

None of these are AI models.

They're all software engineering components.

This is why strong AI Engineers are also strong software engineers. The LLM is only one component in a much larger distributed system.

