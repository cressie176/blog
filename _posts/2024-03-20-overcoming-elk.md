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

**Mapping Explosion** occurs when Elasticsearch fails to keep pace indexing the documents being logged. The initial symptom is the logs will increasingly lag, making them useless for monitoring and live issue resolution. Eventually you will start losing shards or the entire cluster. Mapping Explosion is common because the default behaviour is to index every attribute of every document, and engineering teams will inadvertantly log a wide variety of large documents. 

**Type Conflict** occurs when an attribute is logged with a different type than before, e.g.

```json
{ "error": "Danger Will Robinson!" }
```

```json
{ "error": { "message": "Danger Will Robinson!" }
```

In this case the second record is dropped.

Some might argue the both problems can be resolved if only engineers would be more diligent - and they would be right! However, any system which relies on human infalibility is doomed to fail. Another approach is to enforce a centrally controlled schema, not only fixing mapping explosion and type conflicts but also benefiting from data consistency. Unfortunately this solution would not only create a huge bottleneck, but introduce a change/version management and domain modelling nightmare, akin to using a shared database for every one of your applications!

A more practical approach is to solve Mapping Explosion by restricting dynamic mapping to a single key, and to copy select paths from the logged context to this sub-document beneath this key, e.g.

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
  "@indexed": {
    "crew": {
      "id": 123,
      "username: "drsmith"
    }
  }
}
```

There is a further restriction in that the paths must resolve to a restricted set of types (string, number, boolean, date, etc), rather than an object or array otherwise we recreate the opportunity for Mapping Explosion all over again. By maintaining a common list of paths, and allowing the application developers to supplement this from their applications, we solve the problem of Mapping Explosion and encourage a consistent schema.

To sovle 
