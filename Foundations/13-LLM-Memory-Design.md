# 13. LLM Memory Design

             Memory

      ┌─────────┼─────────┐
      │         │         │
 Session    Working   Long-Term
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      Semantic      Episodic      Procedural

**Memory Is Not Inside the LLM**

Think of an LLM like a CPU.

A CPU doesn't remember yesterday's calculations after power is removed.

It only processes what's currently in RAM.

Similarly:

LLM

has no persistent memory between API calls.

Instead:

Application
        │
Memory Store
        │
       LLM

**The application decides what information to retrieve and include.**

This is one of the biggest architectural shifts.

## Different Types of Memory

I like to classify memory into five categories.

Session Memory

↓

Working Memory

↓

Semantic Memory

↓

Episodic Memory

↓

Procedural Memory

Each serves a different purpose.

### 1. Session Memory

This is the easiest.

It exists only during the current conversation.

Example:

User:
We're discussing Driver Servicing.

...

User:
How does it compare to HDC?

The assistant remembers:

"it" = Driver Servicing

End the session.

Gone.

Characteristics
short-lived
conversational
inexpensive
frequently updated

### 2. Working Memory

This is often confused with session memory.

Working memory stores information needed to complete the current task.

Imagine you're filling out a travel booking.

Destination:
Tokyo

Dates:
October 12

Passengers:
2

These values are not long-term preferences.

They're temporary state for completing the workflow.

Once the booking finishes...

Delete them.

Example

Financial assistant.

User uploads:

Company A.pdf
Company B.pdf
Company C.pdf

Working memory might store:

Current uploaded files

Comparison in progress

Intermediate calculations

Those disappear when the task ends.

### 3. Semantic Memory

Now we're entering long-term memory.

Semantic memory stores facts.

Example:

User prefers Python.

User works on Azure.

User likes first-principles explanations.

These aren't tied to one conversation.

They're stable over time.

This is exactly the kind of memory I use while teaching you.

For example, I remember that:

you're learning AI Engineering systematically,
you prefer first-principles explanations,
you have a strong Azure and distributed systems background.

That lets me avoid restarting from scratch every lesson.

Good semantic memories
Preferred programming language

Preferred explanation style

Timezone

Favorite IDE

Usual cloud provider
Bad semantic memories
User said thanks.

User asked about BM25 yesterday.

User greeted me.

User had lunch.

Those aren't stable preferences.

### 4. Episodic Memory

Humans remember events.

AI assistants can too.

Example:

Last week
↓

We built your RAG architecture.

That's an event.

Another:

You previously chose
Option A.

Also an event.

Unlike semantic memory:

User prefers Python.

Episodic memory answers:

What happened?

Semantic memory answers:

What is generally true?

Example

Semantic

User prefers Python.

Episodic

User implemented
a Bank application
using Python OOP.

### 5. Procedural Memory

This one is less common in LLM applications but very useful.

It stores how something should be done.

Imagine your GitHub assistant learns:

Repository X

↓

Always analyze architecture documents
before reading code.

Or:

Company policy

↓

Always require human approval
before production deployment.

These are procedures.

Not facts.

Not events.

Rules.

## Memory Lifecycle

Think of every piece of information as having a lifecycle.

New Information

↓

Should we keep it?

↓

No

↓

Discard

OR

↓

Yes

↓

Which memory?

↓

How long?

A good memory system is mostly about deciding what not to keep.

Memory Isn't Free

Many beginners think:

Just save everything.

That creates problems.

Imagine storing:

every greeting,
every typo,
every joke,
every temporary preference.

Soon you have:

Memory

↓

Huge

↓

Noisy

↓

Contradictory

Retrieval quality declines.

This is exactly the same lesson we learned with RAG.

Better retrieval starts with better curation.

### Memory Conflicts

Suppose memory contains:

Preferred language

↓

Java

Six months later:

User:
I now write mostly Python.

Now what?

Should memory contain:

Java

Python

?

Probably not.

#### Memory systems need conflict resolution.

Usually:

replace,
merge,
version,
ask for clarification.
Stale Memory

Imagine:

User lives in Seattle.

Three years later:

They moved.

Old memory becomes harmful.

Good systems often:

timestamp memories,
assign confidence,
allow expiration,
support updates.
Memory Confidence

Instead of:

User likes Python.

Store:

Fact:
User prefers Python

Confidence:
High

Last confirmed:
2026-07

Now the system can decide whether to trust it.

Memory Retrieval

Just because memory exists...

doesn't mean it belongs in every prompt.

Imagine memory:

User likes dark mode.

Current question:

Explain BM25.

Dark mode is irrelevant.

Don't retrieve it.

Exactly like RAG.

**Memory retrieval should be relevance-based.**

## Memory Architecture

New Conversation

↓

Memory Extractor

↓

Memory Classifier

↓

Memory Store

↓

Future Question

↓

Memory Retriever

↓

Prompt Builder

↓

LLM

Notice the symmetry.

**Memory behaves almost exactly like another knowledge source.**

A Common Mistake

Many people build:

Conversation

↓

Summary

↓

Memory

This causes almost every conversation to become permanent memory.

**Instead**:

Conversation

↓

Memory Extractor

↓

Only stable facts

↓

Memory

This distinction is incredibly important.

**Not everything worth summarizing is worth remembering.**

## Updated Architecture

Our system now becomes:

                  User Question
                         │
                Conversation Manager
                         │
                Memory Retriever
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
      Conversation History      Long-term Memory
             │                       │
             └───────────┬───────────┘
                         ▼
                 Query Classifier
                         │
                 Retrieval Pipeline
                         │
                    Prompt Builder
                         │
                         ▼
                        LLM

Notice something elegant:

Conversation history and long-term memory are parallel inputs. They are related, but they are not the same thing.

## Architect Perspective

Imagine you're building a GitHub engineering assistant for your team.

A good memory system would remember things like:

Preferred programming language of each engineer.
Which repository they're currently working on.
That Alice prefers architectural diagrams before code.
That Bob usually asks for performance-focused explanations.

It would not permanently remember:

"Alice asked about retries yesterday."
"Bob said thanks."

The difference is whether the information will improve future interactions over time

## Refined Memory Extractor Architecture

A production Memory Extractor could use the following decision process.

###1. Candidate extraction

From the conversation, identify possible memory candidates:

Preferences
Constraints
Decisions
Goals
Relationships
Current entities
Workflow state
Important events
Behavioral instructions

**Do not store entire messages by default. Extract concise atomic statements.**

Instead of:

“I've been using Python lately because the AI course examples
are easier to follow than Java.”

store:

User currently prefers Python examples.

### 2. Ownership classification

Ask:

What is this information about?

About the user
→ possible long-term memory

About the current conversation
→ session memory

About the active task
→ working memory

About an external system or document
→ knowledge store, not user memory

This prevents domain facts from polluting user memory.

### 3. Stability assessment

Determine how long the information is likely to remain true.

Minutes or current topic
→ session

Until task completion
→ working

Months or years
→ semantic or procedural

Past event
→ episodic

For example:

User is currently debugging authentication.
→ short-lived

User prefers first-principles explanations.
→ long-lived

### 4. Explicitness and confidence

Confidence should depend on how clearly the user stated something.

“From now on, use Python.”
→ explicit, high confidence

“Python examples seem easier.”
→ inferred, medium confidence

User happened to use Python once.
→ weak inference, do not store

A possible confidence model:

Signal	Confidence
Explicit lasting instruction	High
Repeated preference	High
Single indirect statement	Medium
Inference from behavior	Low

Low-confidence memories should usually not be persisted automatically.

### 5. Sensitivity and permission check

Before persistent storage, check whether the information is:

sensitive,
unnecessarily personal,
irrelevant to future assistance,
something the user would reasonably expect to be retained.

The safest rule is:

Store only what has clear future utility and is appropriate to retain.

### 6. Conflict detection

Search existing memories for the same subject.

Example:

Existing:
Preferred language = Java

Candidate:
Preferred language = Python

Then choose among:

Replace
Version
Merge
Reject
Clarify

Typical policies:

Current preference changed → replace active value and archive old value.
Multiple compatible preferences → merge.
Direct contradiction with unclear timing → ask or retain both with uncertainty.
Newer explicit statement → prefer newer statement.

### 7. Expiration policy

Different memories should have different expiry rules.

Session memory
→ topic change or session end

Working memory
→ task completion or timeout

Semantic preference
→ no fixed expiry, but periodically reconfirm

Episodic event
→ retain selectively or decay over time

Procedural rule
→ retain until changed or revoked

Some memories can use a time-to-live value:

Current repository: Repo A
TTL: 24 hours

Current comparison companies: A, B, C
TTL: until task completion

### 8. Importance scoring

Not every valid event deserves long-term storage.

An episodic event such as:

User completed the retrieval module.

may matter because it affects future teaching.

But:

User answered homework question 2 correctly.

probably does not need permanent storage.

Importance can depend on:

future usefulness,
uniqueness,
impact on personalization,
whether it represents a milestone or decision.

### 9. Store with provenance

Each memory should include more than text:

Memory:
User prefers architecture diagrams before code.

Type:
Semantic preference

Source:
Explicit user statement

Created:
2026-07-30

Last confirmed:
2026-07-30

Confidence:
High

Status:
Active

Provenance helps explain where the memory came from and supports corrections.

### 10. Retrieval policy

Persistent memory should not automatically enter every prompt.

For each new request:

Current question
        ↓
Memory Retriever
        ↓
Only relevant active memories
        ↓
Prompt Builder

For example:

Question:
Explain an authentication architecture.

Relevant memory:
User prefers architecture diagrams before code.

But for:

Question:
Rewrite this sentence.

that memory might be unnecessary.

## Final architecture

Conversation
     │
     ▼
Candidate Extractor
     │
     ▼
Ownership Classifier
 ┌───┼───────────┬────────────┐
 ▼   ▼           ▼            ▼
Session Working  User Memory  External Knowledge
                    │
                    ▼
          Stability and Confidence
                    │
                    ▼
          Sensitivity and Utility
                    │
                    ▼
             Conflict Detector
                    │
         ┌──────────┼──────────┐
         ▼          ▼          ▼
      Replace     Version     Merge
                    │
                    ▼
           Expiry and Importance
                    │
                    ▼
                Memory Store

The core principle is:

Memory should preserve durable, useful state—not become an archive of everything the model and user have ever said.