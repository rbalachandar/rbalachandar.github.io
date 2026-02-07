---
layout: default
---

# AI Engineering

**Building production-ready AI systems, from memory to multi-agent architectures.**

Welcome! I write about AI engineering, from memory systems to developer tooling.

---

## 📝 Recent Posts

{% for post in site.posts limit:3 %}
### [{{ post.title }}]({{ post.url }})
{{ post.date | date: "%B %d, %Y" }}

{{ post.excerpt }}

[Read more →]({{ post.url }})
{% endfor %}

---

## 🧠 Featured: Contexo

A Python library for managing memory in AI applications.

**Key Features:**
- Two-tier memory: Fast session + Persistent storage
- Hybrid search: Semantic + FTS + Temporal reranking
- Local-first: SQLite, Neo4j, or PostgreSQL
- Working memory snapshots for crash recovery
- Multi-agent support with private thoughts

**Benchmark Results:**
95.22% on LoCoMo benchmark (beats MemU's 92.09%)

[**View on GitHub →**](https://github.com/rbalachandar/contexo)

---

## 📚 Topics

- **AI Engineering** — Building production-ready AI systems
- **Memory Systems** — Context management for LLM applications
- **Multi-Agent** — Architectures for collaborative AI agents
- **Observability** — Tracing, debugging, and monitoring AI systems
- **Developer Tools** — Tooling for AI application development