---
title: "Welcome to My Tech Blog"
date: 2025-03-11
author: "Usman Asif"
layout: home
---

# Introduction

Hi! I'm a developer working on backend systems. This blog is where I share updates about my work, insights into RestAPIs, Databases, Background jobs, RabbitMQ, Elasticsearch, and other cool tech I'm working with.

---
<p></p>
## Latest Posts  
{% for post in site.posts limit:3 %}
- **[{{ post.title }}]({{ post.url | relative_url }})** ({{ post.date | date: "%B %d, %Y" }})
{% endfor %}

*See all posts **[here]({{ "/posts" | relative_url }})**.*

---
<p></p>
## ✨ Why This Blog?  
This journal is where I document my learnings, technical deep dives, and best practices in backend engineering.  

---
<p></p>
## 🔗 Follow My Work  
- GitHub: [github.com/usmanasifbutt](https://github.com/usmanasifbutt)  
- LinkedIn: [linkedin.com/in/usman-asif-ua2208/](https://www.linkedin.com/in/usman-asif-ua2208/)  

Stay tuned for more updates!
