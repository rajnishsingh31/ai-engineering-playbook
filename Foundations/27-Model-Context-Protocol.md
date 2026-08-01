# Model Context Protocol (MCP)

What problem does MCP solve?

Imagine today.

You build an AI agent.

It needs access to:

GitHub
Jira
Slack
SQL
Files
Azure
AWS
Kubernetes

Without MCP, every application writes its own integrations.

My Agent
    ├── GitHub SDK
    ├── Jira SDK
    ├── Slack SDK
    ├── SQL Client
    ├── Azure SDK
    └── ...

Now another team builds another agent.

They write the same integrations again.

No standard.

Think about web browsers.

Every website doesn't invent its own way to communicate.

They use:

HTTP
HTML
JSON

Those standards made the web possible.

MCP is trying to do something similar for LLMs and tools.

## Simple Definition

**MCP is a standard protocol that allows AI models to discover and use external capabilities in a consistent way.**

Notice the important word:

Standard

It is not another framework like LangChain.

It is a communication protocol.

### Before MCP
Agent A
    ├── GitHub integration
    ├── SQL integration
    └── Jira integration

Agent B
    ├── GitHub integration
    ├── SQL integration
    └── Slack integration

Everyone rewrites everything.

### After MCP
Agent
    │
    ▼
MCP Client
    │
──────── Protocol ────────
    │
    ▼
MCP Servers
    ├── GitHub
    ├── Jira
    ├── SQL
    ├── Files
    ├── Kubernetes
    └── Azure

The agent only speaks MCP.

Servers expose capabilities.

Think of USB

This analogy is surprisingly accurate.

Before USB:

Every device:

different connector
different driver
different protocol

After USB:

One standard.

Keyboard.

Mouse.

Printer.

Camera.

All plug in.

MCP is becoming the "USB for AI tools."

Core Components

There are only three.

Host

Client

Server
Host

The application.

Examples:

ChatGPT Desktop
Claude Desktop
VS Code
Cursor
Your AI application

The Host owns:

UI
Conversation
User
Client

Lives inside the host.

Speaks MCP.

Responsibilities:

discover capabilities
send requests
receive results

Think of it as the protocol implementation.

Server

Exposes capabilities.

Example:

GitHub MCP Server

Provides:

Search repositories

Read issues

Create PR

Read commits

Another server:

SQL

Provides:

Execute query

List tables

Describe schema

The agent doesn't know implementation details.

Discovery

One of MCP's biggest advantages.

Instead of hardcoding:

search_github()

The client can ask:

What tools do you provide?

Server responds:

search_repo

read_issue

create_pr

Exactly like API discovery.

Tool Schema

Notice something familiar.

We already designed tool schemas.

Example:

{
  "name": "search_logs",
  "description": "...",
  "input": {
      ...
  },
  "output": {
      ...
  }
}

MCP formalizes this.

When we studied:

JSON schemas
Tool registry
Capability registry

We unknowingly built the same concepts.

Resources

## MCP exposes more than tools.

### It can expose resources.

Examples:

documents
files
logs
code
prompts

Instead of asking a tool:

Read README

The model may simply access a resource.

### Prompts

MCP even allows servers to expose reusable prompts.

Imagine:

Security Server

Offers:

Threat Model Review

Architecture Server

Offers:

Architecture Critique

These become discoverable.

Why Architects Should Care

Suppose you're building:

Enterprise AI Assistant.

Without MCP:

Every connector:

GitHub
Jira
Azure DevOps
SQL
Slack

Requires custom integration.

With MCP:

If vendors expose MCP servers,

your application speaks one protocol.

Much easier.

### Security

Remember our security chapter?

MCP fits perfectly.

LLM

↓

Planner

↓

Policy Engine

↓

MCP Client

↓

MCP Server

↓

External System

Notice:

The LLM still shouldn't directly execute tools.

Authorization still belongs outside the model.

MCP Doesn't Replace Architecture

Some beginners think:

We use MCP now.

No.

MCP replaces:

Connector implementations.

It does not replace:

Planner
Evaluator
Memory
State
Context Builder
Policy Engine
Workflow

Those remain your architecture.

Enterprise Example

Imagine Microsoft.

They have:

Azure DevOps
GitHub
OneDrive
SharePoint
Outlook
Teams

Instead of every AI assistant integrating separately,

each service can expose an MCP server.

Now:

Copilot

Visual Studio

GitHub Copilot

Internal assistants

All speak one protocol.

Huge reduction in duplicated work.

Where MCP Fits

Let's place it in the architecture we've been building.

                User
                  │
                  ▼
             Agent Runtime
        ┌─────────┼─────────┐
        ▼         ▼         ▼
 Planner   Context Builder  Memory
        │
        ▼
 Policy Engine
        │
        ▼
    MCP Client
        │
──────── Protocol ────────
        │
        ▼
   MCP Servers
        ├── GitHub
        ├── SQL
        ├── Files
        ├── Slack
        ├── Azure
        └── Jira

Notice:

**MCP sits where our Tool Executor / Connector Layer used to be.**

It standardizes that layer.

What MCP Doesn't Solve

It doesn't answer:

Which tool should I use?
Which model should I use?
Is the answer correct?
Is the plan good?
Is the user authorized?
Should I call this tool?

Those remain your responsibility.

MCP is plumbing, not intelligence.

## Definition

If someone asks:

"What is MCP?"

A strong answer is:

Model Context Protocol is an open standard that allows AI applications to discover and interact with external tools, resources, and prompts through a common protocol. It standardizes integrations but does not replace planning, memory, security, or orchestration.

That's much stronger than:

"It's a way for AI to call tools."