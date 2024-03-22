---
title: "Overcoming ELK"
date: 2024-03-20.md
tags:
- Elasticsearch
- Logging
- Mapping explosion
- Schema conflict
---

Following on from my last post demonstrating the benefits of Factory Modules for concerns such as logging, I wanted to share some tips for working with the Elasticsearch, Logstash and Kibana (ELK) stack. My first tip is contentious - if you can afford to, don't use ELK. I say this because of two near fatal flaws - [Mapping Explosion](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-explosion.html) and [Type Conflict](https://opster.com/guides/elasticsearch/glossary/elasticsearch-conflicting-field). 

**Mapping Explosion** occurs when Elasticsearch fails to keep pace with indexing. Logs will increasingly lag, making them useless for monitoring and live issue resolution. You will start losing shards and eventually the entire cluster. Mapping Explosion occurs because Elasticsearch's default behaviour is to index every attribute of every document you log, and your engineering teams will inadvertantly log a wide variety of large documents. 

**Type Conflict** occurs when an attribute is logged with a different type than before, e.g.

```json
{ "error": "Danger Will Robinson!" }
```

```json
{ "error": { "message": "Danger Will Robinson!" }
```

In this case the second record is dropped.

There is an arguement that both problems will self resolve with improved diligence. In practice however, any system which relies on human infalibility is doomed to fail. Another approach is to enforce a centrally managed schema. Unfortunately, this solution would create a developmental bottleneck and introduce a change management and domain modelling nightmare akin to sharing a single database between all of your applications.

A more practical approach is to solve Mapping Explosion by restricting Elasticsearch's indexes to a single root path (say "@indexes", and to copy select paths from the logged context to a sub-document beneath this path, e.g.

```
const indexes = [ "crew.id", "crew.username" ];
const logger = new Logger({ indexes });
```

```json
{
  "crew": {
    "id": 123,
    "username": "drsmith",
    "job": "science officer",
    "personality": "untrustworthy"
  },
  "@indexes": {
    "crew": {
      "id": 123,
      "username": "drsmith"
    }
  }
}
```

The paths must resolve to a restricted set of types (string, number, boolean, date, etc), rather than an object or array to avoid recreating the opportunity for Mapping Explosion all over again. By maintaining a common list of paths, and allowing the application developers to supplement this them from the applications, we solve the problem of Mapping Explosion and encourage a consistent schema.

To sovle 
