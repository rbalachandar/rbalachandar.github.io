---
layout: default
title: Blog
---

# Blog

Thoughts on AI engineering, memory systems, and multi-agent architectures.

---

{% for post in site.posts %}

<div style="border: 1px solid #333; border-left: 3px solid #666; padding: 1rem; margin-bottom: 1.5rem; border-radius: 4px;">

<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>

<div style="color: #888; font-size: 0.85em; margin-bottom: 0.75rem;">
📅 {{ post.date | date: "%B %d, %Y" }}{% if post.categories %} · 🏷️ {{ post.categories | join: ", " }}{% endif %}
</div>

{{ post.excerpt }}

<div style="margin-top: 1rem;"><strong><a href="{{ post.url }}">Read more →</a></strong></div>

</div>

{% endfor %}
