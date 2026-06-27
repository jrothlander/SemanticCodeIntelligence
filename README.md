# Semantic Code Intelligence Layer

## Core Idea
Transform code from raw text into structured meaning that AI can reason over efficiently. This system turns your codebase into a queryable intelligence layer rather than a collection of files.

## Overview
The **Semantic Code Intelligence Layer** is a system that transforms source code into a structured, semantic representation that both developers and AI agents can efficiently understand and reason over.

Instead of treating code as raw text, this layer models:
- **Behavior** (via IR)
- **Relationships** (via graph)
- **Similarity** (via embeddings)

This enables AI systems (e.g., Claude) to operate on **meaning, not syntax**, improving accuracy, reducing token usage, and enabling system-level reasoning.

## Is anything similar being done? Not exactly. 
I started calling my code embedding layer code2vec back in 2008-2011, based on word2vec and seq2seq. It wasn't until 2018 that someone else named their technology code2vec and published it. That is a similar idea, in that it uses an AST to generate embeddings at the code level, which is useful at some level. But the intent there was to create meaningful function names based on what the code was actually doing. While we did look at that and we did use the AST to generate code, the idea to use embeddings was not at the code level and we would often say it was "above the code level". We were looking for the "intent of the original developer" and the "tribal knowledge" that only the original developers of the language where aware of. We wanted to pull this out of the code and felt like we could. 

## Benefits
- Faster onboarding
- Better AI-assisted development
- Reduced token usage
- Improved system understanding

## Architecture
```mermaid
flowchart TD

%% =========================
%% NODES
%% =========================

A[Codebase]
B[Canonical IR]
C[Neo4j Relationships]
D[Qdrant Embeddings]
E[MCP Tools]
F[Claude + Skills]
Q["User Query\nFind user lookup logic"]

%% =========================
%% ARCHITECTURE FLOW
%% =========================

A --> B
B --> C
B --> D
C --> E
D --> E
E --> F

%% =========================
%% QUERY FLOW (NUMBERED)
%% =========================

Q --> F
F -->|1. semantic search| D
D -->|2. match IR meaning| B
B -->|3. trace relationships| C
C -->|4. expand context| E
E -->|5. minimal context| F

%% Optional path
B -.->|optional source fetch| A

%% =========================
%% STYLING (SAFE FOR GITHUB)
%% =========================

classDef src fill:#90CDF4,stroke:#2B6CB0,color:#000
classDef ir fill:#9AE6B4,stroke:#2F855A,color:#000
classDef intel fill:#FAF089,stroke:#D69E2E,color:#000
classDef ai fill:#D6BCFA,stroke:#6B46C1,color:#000
classDef query fill:#FED7D7,stroke:#E53E3E,color:#000

class A src
class B ir
class C intel
class D intel
class E ai
class F ai
class Q query
```

```
Source Code
   ↓
Parsing (Roslyn / Custom Parsers)
   ↓
Canonical IR
   ↓
IR Graph
   ↓
Path Extraction
   ↓
Embedding Layer (Qdrant)
   ↓
Graph Layer (Neo4j)
   ↓
MCP Tools
   ↓
Claude + Skills
```
---

## Problem
Modern codebases are:
- Large, distributed, and difficult to understand
- Dependent on tribal knowledge
- Hard for AI tools to reason about due to limited context

Existing approaches fall short:

| Approach | Limitation |
|--|--|
| Keyword Search | Finds text, not intent |
| RAG for Code | High token cost, low structure |
| Static Analysis | Limited semantic reasoning |
| AI Copilots | Fragmented, incomplete context |

## Solution
Build a **Semantic Code Intelligence Layer** that converts code into a **queryable semantic model**.

At its core:
> Transform code from raw text → into structured meaning → accessible for reasoning.

## IR Indexing + Query Flow Sequence Diagrams 

### 1. IR Indexing Diagram

```mermaid
sequenceDiagram
    participant Dev as Developer / CI
    participant Parser as Roslyn / Parser
    participant IR as IR Builder
    participant Graph as Neo4j
    participant Embed as Embedding Service
    participant Qdrant as Qdrant

    Dev->>Parser: Provide source code
    Parser->>IR: AST + Semantic Model
    IR->>IR: Build Canonical IR

    IR->>Graph: Store nodes + edges
    Graph-->>IR: Persisted

    IR->>Embed: Extract IR paths
    Embed->>Embed: Generate embeddings

    Embed->>Qdrant: Store vectors
    Qdrant-->>Embed: Acknowledged
```

### 2. Query + Retrieval Flow (AI-Optimized)

```mermaid
sequenceDiagram
    participant User
    participant Skill as Claude Skill
    participant MCP
    participant Qdrant
    participant Graph
    participant Source

    User->>Skill: Find user lookup logic

    Skill->>MCP: search_ir(query)
    MCP->>Qdrant: Vector search
    Qdrant-->>MCP: Top matches
    MCP-->>Skill: IR summaries

    Skill->>MCP: trace_flow(symbol)
    MCP->>Graph: Traverse graph
    Graph-->>MCP: Related nodes
    MCP-->>Skill: Graph context

    alt Need source
        Skill->>MCP: get_source(symbol)
        MCP->>Source: Fetch code
        Source-->>MCP: Code snippet
        MCP-->>Skill: Minimal code
    end

    Skill-->>User: Answer
```

### 3. Token-Efficient Reasoning Flow

```mermaid
sequenceDiagram
    participant Claude
    participant System

    Claude->>System: Ask question

    System-->>Claude: IR only
    Note over Claude,System: Small, structured context

    Claude->>System: Expand
    System-->>Claude: Graph neighbors

    Claude->>System: Fetch source (optional)
    System-->>Claude: Minimal snippet
```


