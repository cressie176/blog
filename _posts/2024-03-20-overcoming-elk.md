---
title: "Overcoming ELK"
date: 2024-03-20.md
tags:
- Elasticsearch
- Logging
- Mapping explosion
- Schema conflict
---

Following on from my last blog post about Factory Modules and logging, I thought it worth sharing a few tips for working with the Elasticsearch, Logstash and Kibana (ELK) stack. My first tip is contentious - if you can afford to, don't use ELK. I say this because of two almost fatal flaws - [Mapping Explosion](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-explosion.html) and [Type Conflict](https://opster.com/guides/elasticsearch/glossary/elasticsearch-conflicting-field). Mapping Explosion occurs when Elasticsearch is unable to keep pace creating indexes for the documents being logged. The initial symptom is that logs increasing lag behind reality, until you eventually start losing shards or the entire cluster. The gotcha is that by default Elastic is configured to index every attribute of every document you send it, and engineering teams will inadvertantly log a wide variety of large documents. The second problem, Type Conflict, occurs when an attribute is logged with a different type than previously indexed attribute with the same name, e.g.

```json
{ "error": "Danger Will Robinson!" }
```

```json
{ "error": { "message": "Danger Will Robinson!" }
```

In this case the second record is dropped.

One naive way of resolve these issues is for engineers to log more carefully. Good luck with that! Another is to enforce a tightly controlled shared schema. 

