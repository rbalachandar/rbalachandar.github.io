---
layout: default
---

# AI Engineering

---

## 📝 Recent Posts

{% for post in site.posts limit:3 %}

<div style="border: 1px solid #333; border-left: 3px solid #666; padding: 1rem; margin-bottom: 1.5rem; border-radius: 4px;">

<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>

<div style="color: #888; font-size: 0.85em; margin-bottom: 0.75rem;">
📅 {{ post.date | date: "%B %d, %Y" }}{% if post.categories %} · 🏷️ {{ post.categories | join: ", " }}{% endif %}
</div>

{{ post.excerpt }}

<div style="margin-top: 1rem;"><strong><a href="{{ post.url }}">Read more →</a></strong></div>

</div>

{% endfor %}

<div style="text-align: center; margin: 2rem 0;"><strong><a href="/blog/">View all posts →</a></strong></div>

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
#context #memory #multi-agent #observability #tools
