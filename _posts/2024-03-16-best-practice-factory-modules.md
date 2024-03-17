---
title: "Best Practice Factory Modules"
date: 2024-03-16
---

Microservice architectures offer the advantage of allowing multiple teams to work on different parts of a system simultaneously, thus reducing conflicts. However, this model presents its own set of challenges. A common issue is the proliferation of third-party libraries for tasks such as logging, HTTP requests, messaging and persistence. While standardising on a single library for each function is an important step, it is insufficient to address all concerns. Consider logging: Imagine if you standardised on [pino](https://github.com/pinojs/pino), installing it directly in over 100 services, only then to find your logs include sensitive HTTP headers; you would have to add the redaction code to each one of those repositories.

A better approach is to wrap 3rd party libraries like pino in custom factory module that applies a set of tested, sensible conventions, safeguards and features, thus creating a [pit of success](https://learn.microsoft.com/en-us/archive/blogs/brada/the-pit-of-success). For a logging library, a good set of best practices are:

- A conventional API
- Support for machine friendly logging
- Support for human friendly logging
- Support for test friendly logging
- Support for async context tracking
- Support for redaction of sensitive content
- Reporting the source of empty log messages
- Relocating the context to a subdocument to avoid name clashes (level, time, etc)
- Ensuring errors are serialised correctly ([pino#862](https://github.com/pinojs/pino/issues/862), [winston#1338](https://github.com/winstonjs/winston/issues/1338), [bunyan#514](https://github.com/trentm/node-bunyan/issues/514))
- Ensuring circular references ([pino#990](https://github.com/pinojs/pino/issues/990), [winston#1946](https://github.com/winstonjs/winston/issues/1946),  [bunyan#427](https://github.com/trentm/node-bunyan/issues/427)) are tolerated
- Ensuring unserialisable context objects are tolerated

You can find an example factory module implementing the above best practices [here](https://github.com/acuminous/module-acme-logging).

Factory modules will not only reduce the effort required to adopt a new best practice, but are the most effective way of leveraging all the hard won lessons, you and your colleagues have accumulated over their careers. Quality truly is [the knife-edge of experience](https://en.wikipedia.org/wiki/Pirsig%27s_Metaphysics_of_Quality).