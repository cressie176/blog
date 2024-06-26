---
title: "Overcoming ELK's Fundamental Flaws"
date: 2024-03-20
tags:
- Elasticsearch
- ELK
- Logging
- Mapping Explosion
- Type Conflict
---

Following on from my previous post highlighting the benefits of [Best Practice Factory Modules](https://cressie176.github.io/blog/2024/03/16/best-practice-factory-modules.html) for concerns such as logging, I wanted to share some tips for working with the Elasticsearch, Logstash and Kibana (ELK) stack, which suffers from two fundamental flaws - [Mapping Explosion](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-explosion.html) and [Type Conflict](https://opster.com/guides/elasticsearch/glossary/elasticsearch-conflicting-field). 

**Mapping Explosion** occurs when Elasticsearch fails to keep pace with indexing. Logs will increasingly lag, making them useless for monitoring and live issue resolution. Eventually you will start to lose shards, and maybe even the cluster. Mapping Explosion is most often a consequence of Elasticsearch's default behaviour, which is to index every attribute of every document you log. Since engineers may inadvertently log a variety of large documents, Mapping Explosion is almost inevitable for large teams if nothing is done.

**Type Conflict** occurs when an attribute is logged with a different type than before, e.g.

```json
{ "error": { "message": "Danger Will Robinson!" } }
```

```json
{ "error": "Danger Will Robinson!" }
```

In this above example, Elasticsearch's dynamic mapping feature will create an index for the first error attribute with type Object. Since the type is now fixed, it cannot insert a String value into the index for the second error attribute.

Theoretically, both problems will self resolve with increased diligence. In practice, any system which relies on human infallibility is doomed to fail. Another approach is to enforce a centrally managed schema, potentially using [dynamic templates](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html). Unfortunately, this would create a developmental bottleneck and introduce a version management and domain modelling nightmare akin to sharing a single database between all of your applications. 

A more practical way to prevent Mapping Explosion is by restricting Elasticsearch"s dynamic mapping behabiour to a single root attribute (say "@indexes"), and to copy select paths from the logged context to a sub-document beneath this attribute, e.g.

```js
const factory = require("./logger-factory");
const indexes = [ "staff.id", "staff.username" ];
const logger = factory({ indexes });
logger.info("Never fear, Smith is here", { staff });
```

```json
{
  "message": "Never fear, Smith is here",
  "staff": {
    "id": 123,
    "username": "drzsmith",
    "position": "Staff Psychologist",
    "startDate": "1965-10-02T00:00:00.000Z",
    "notes": ["untrustworthy","cowardly", "sabotage"]
  },
  "@indexes": {
    "staff": {
      "id": 123,
      "username": "drzsmith"
    }
  }
}
```

The paths must resolve to a restricted set of types (String, Number, Boolean, Date and null), but not an Object or Array to avoid recreating the opportunity for Mapping Explosion all over again. By maintaining a common list of paths, and allowing the developers to extend it from application code, we solve the problem of Mapping Explosion and encourage a more consistent schema.

This solution partially solves the Type Conflict problem too. Elasticsearch will convert numbers to strings (excluding NaN and Infinity if using JavaScript), but obviously cannot always convert strings to numbers. Furthermore, converting numbers to strings may affect sort order and prevents aggregation. An improvement is to differentiate between differently typed values with the same path by appending a type suffix and thus eradicating potential conflict, e.g.

```json
{
  "message": "Never fear, Smith is here",
  "staff": {
    "id": 123,
    "username": "drzsmith",
    "position": "Staff Psychologist",
    "startDate": "1965-10-02T00:00:00.000Z",
    "notes": ["untrustworthy","cowardly", "sabotage"]
  },
  "@indexes": {
    "staff": {
      "id": {
        "numberValue": 123
      },
      {
        "username": {
          "stringValue": "drzsmith"
        }
      }
    }
  }
}
```
The above solution can be implemented in [pino](https://github.com/pinojs/pino) as follows...

```js
const pino = require("pino");
const has = require("has-value");
const get = require("get-value");
const set = require("set-value");
const typeOf = require("which-builtin-type");

const DEFAULT_INDEXES = ["staff.id", "staff.username"];

module.exports = function(options) {

  const formatters = {
    log(context) {
      const indexes = {};
      const invalid = [];
      DEFAULT_INDEXES.concat(options?.indexes).forEach((path) => {
        if (!has(context, path)) return;
        const value = get(context, path);
        const type = typeOf(value);
        if (["Date", "BigInt", "String"].includes(type)) set(indexes, `${path}.stringValue`, value)
        else if (type === "Boolean") set(indexes, `${path}.booleanValue`, value)
        else if (type === "Number") set(indexes, `${path}.numberValue`, value)
        else if (value === null) set(indexes, `${path}.nullValue`, true)
        else invalid.push(path);
      }, {});
      const indexError = invalid.length > 0
        ? Object.assign(new Error("Indexes must be of type String, Number, Boolean, BigInt, Date or null"), { invalid })
        : undefined;
      return { "@indexes": indexes, "@indexesErr": indexError, ...context };
    }
  }

  return pino({
    base: null,
    formatters,
    serializers: {
      "@indexErr": pino.stdSerializers.err
    },
    transport: {
      target: "pino/file",
      options: {
        destination: 1
      }
    }
  });
}
```

```js
const factory = require("./logger-factory");
const indexes = [ "staff.startDate", "staff.notes" ];
const logger = factory({ indexes });
logger.info({ staff }, "Never fear, Smith is here");
```

```json
{
  "level": 30,
  "time": 1711117540332,
  "@indexes": {
    "staff": {
      "id": {
        "numberValue": 123
      },
      "username": {
        "stringValue": "drzsmith"
      },
      "startDate": {
        "stringValue": "1965-10-02T00:00:00.000Z"
      }
    }
  },
  "@indexesErr": {
    "type": "Error",
    "message": "Indexes must be of type string, number, boolean, bigint or date",
    "stack": "Error: Indexes must be of type string, number, boolean, bigint or date\n...",
    "invalid": [
      "staff.notes"
    ]
  },
  "msg": "Never fear, Smith is here",
  "staff": {
    "id": 123,
    "username": "drzsmith",
    "startDate": "1965-10-02T00:00:00.000Z",
    "position": "Staff Psychologist",
    "notes": [
      "Mutiny",
      "Treachery",
      "Sabotage"
    ]
  }
}
```

As per [my previous post](https://cressie176.github.io/blog/2024/03/16/best-practice-factory-modules.html), the logger configuration for safely copying the paths to the "@indexes" sub-document should be added to a Factory Module to encourage adoption and avoid duplication.

My final tip is contentious - if you can afford to, avoid using ELK for logging. It is the proverbial square peg in a round hole. While the above solution shaves ELK's sharpest corners, it alters the shape of the logged documents, breaking the [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) for those querying them. If however, you are too far gone to easily backtrack, or only have budget for a self-hosted logging platform, the above solution is an effective way to overcome ELK's two fundamental flaws.
