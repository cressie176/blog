---
title: "Overcoming ELK"
date: 2024-03-20.md
tags:
- Elasticsearch
- ELK
- Logging
- Mapping Explosion
- Type Conflict
---

Following on from [my previous post](https://cressie176.github.io/blog/2024/03/16/best-practice-factory-modules.html) demonstrating the benefits of Factory Modules for concerns such as logging, I wanted to share some tips for working with the Elasticsearch, Logstash and Kibana (ELK) stack, which suffers from two, near fatal flaws - [Mapping Explosion](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-explosion.html) and [Type Conflict](https://opster.com/guides/elasticsearch/glossary/elasticsearch-conflicting-field). 

**Mapping Explosion** occurs when Elasticsearch fails to keep pace with indexing. Logs will increasingly lag, making them useless for monitoring and live issue resolution. You will start losing shards and eventually the entire cluster. Mapping Explosion occurs because Elasticsearch's default behaviour is to index every attribute of every document you log, and your engineering team will inevitably log a wide variety of large documents. 

**Type Conflict** occurs when an attribute is logged with a different type than before, e.g.

```json
{ "error": "Danger Will Robinson!" }
```

```json
{ "error": { "message": "Danger Will Robinson!" }
```

In this case the second record is dropped.

There is an argument that both problems will self resolve with improved diligence. In practice however, any system which relies on human infallibility is doomed to fail. Another approach is to enforce a centrally managed schema. Unfortunately, this would create a developmental bottleneck and introduce a version management and domain modelling nightmare akin to sharing a single database between all of your applications.

A more practical approach is to solve Mapping Explosion by restricting Elasticsearch's indexes to a single root attribute (say "@indexes", and to copy select paths from the logged context to a sub-document beneath this attribute, e.g.

```js
const indexes = [ "staff.id", "staff.username" ];
const logger = new Logger({ indexes });
```

```json
{
  "staff": {
    "id": 123,
    "username": "drsmith",
    "job": "science officer",
    "personality": "untrustworthy"
  },
  "@indexes": {
    "staff": {
      "id": 123,
      "username": "drsmith"
    }
  }
}
```

The paths must resolve to a restricted set of types (string, number, boolean, date, etc), rather than an object or array to avoid recreating the opportunity for Mapping Explosion all over again. By maintaining a common list of paths, and allowing the developers to extend it from application code, we solve the problem of Mapping Explosion and encourage a more consistent schema.

This solution partially solves the Type Conflict problem too. Elasticsearch will convert numbers to strings (excluding NaN and Infinity if using JavaScript), but obviously cannot always convert strings to numbers. Furthermore, converting numbers to strings may affect sort order and prevents aggregation. An improvement is to differentiate between differently typed values with the same path by append a type suffix, eradicating potential conflict, e.g.

```json
{
  "staff": {
    "id": 123,
    "username": "drsmith",
    "job": "science officer",
    "personality": "untrustworthy"
  },
  "@indexes": {
    "staff": {
      "id": {
        "numberValue": 123
      },
      {
        "username": {
          "stringValue": "drsmith"
        }
      }
    }
  }
}
```

As per [my previous post](https://cressie176.github.io/blog/2024/03/16/best-practice-factory-modules.html), the logger configuration for creating the indexes should be added to a Factory Module to avoid duplication.

My final tip is contentious - if you can afford to, avoid using ELK for logging. It is the proverbial square peg in a round hole. While the above solution shaves ELK's sharpest corners, it alters the shape of the logged documents, breaking the [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) for those querying them. If however, you are  too far down the ELK route to easily backtrack, or only have budget for a self-hosted logging platform, the above solution is an effective way to overcome ELK's two greatest flaws.
