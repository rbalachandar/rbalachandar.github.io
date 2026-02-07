---
layout: post
title: "Contexo: 95.22% on LoCoMo Benchmark"
date: 2025-02-07 10:00:00 +0000
categories: benchmark contexo
excerpt: "Contexo achieves 95.22% accuracy on the LoCoMo long-term conversational memory benchmark"
---

This post covers results from running Contexo on the **LoCoMo benchmark** — a test of long-term conversational memory retrieval accuracy.

## The Results

Contexo achieved **95.22% accuracy**, beating MemU's published baseline of 92.09%.

```
Accuracy on LoCoMo Benchmark (QA Retrieval)

100% ┤
     │
 95% ┤    ██ Contexo: 95.22%
     │    ██
 92% ┤    ██  ██ MemU: 92.09%
     │    ██  ██
     └────┴────┴─────
       Contexo MemU
```

## What is LoCoMo?

The LoCoMo benchmark tests whether an AI system can find relevant information from a long conversation history. It evaluates:

- **Long-term conversational memory** — Can the system retrieve information from hundreds of messages ago?
- **Temporal reasoning** — Can it understand "when did we discuss X?"
- **Multi-hop reasoning** — Can it connect information across different parts of the conversation?

## How Contexo Works

Contexo uses a **two-tier memory architecture**:

1. **Working Memory** — Fast, in-memory context with auto-compaction strategies
2. **Persistent Memory** — Long-term storage with semantic search

For this benchmark, Contexo's hybrid search combines:
- **Semantic search** (BAAI/bge-large-en-v1.5 embeddings)
- **BM25/FTS5** (keyword-based retrieval)
- **Substring matching** (exact phrase matching)
- **Temporal reranking** (boosts for recent sessions + temporal words)

## Why This Matters

Most AI applications today lose context when conversations get long. Contexo solves this by:

- **Searching across all conversations** — Not just the current session
- **Local-first** — Runs on SQLite, Neo4j, or PostgreSQL without cloud dependencies
- **Developer-friendly** — Drop-in Python library with async/await support

## Try It Out

```python
from contexo import Contexo
from contexo.config.defaults import local_config

ctx = Contexo(config=local_config())
await ctx.initialize()

# Add messages to memory
await ctx.add_message("user", "My favorite color is blue")

# Later, search and retrieve
results = await ctx.search_memory("favorite color")
# Returns the original message
```

**[View on GitHub →](https://github.com/rbalachandar/contexo)**
