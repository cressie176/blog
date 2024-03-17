---
title: "Best Practice Factory Modules"
date: 2024-03-16
---

A key benefit of microservice architectures is that multiple teams can work different parts of the system without tripping over each other. This isn't without its problems however. One common issue is the proliferation of 3rd party libraries for things like logging, http requests, and persistance, etc. The first step to mitigate this is to standardise on a single library for each function, but this alone is insufficient. Take logging for example. Imagine if you standardised on [pino](https://github.com/pinojs/pino), installing it directly in 50 different service, only then to find your logs include sensitive HTTP headers; you would have to add the redaction code to each one of those 50 repositories.

A better approach is to wrap 3rd party libraries like [pino](https://github.com/pinojs/pino) within a custom factory module that applies a set of tested, sensible conventions, safeguards and features, thus creating a [pit of success](https://learn.microsoft.com/en-us/archive/blogs/brada/the-pit-of-success). Not only will this reduce the effort when you discover a new convention, but is the most effective way of leveraging the years of hard won lessons, you and your colleagues have accumulated over their careers. Quality truly is [the knife-edge of experience](https://en.wikipedia.org/wiki/Pirsig%27s_Metaphysics_of_Quality).

For a logging library, a good set of best practices are:

- A conventional API
- Support for machine friendly logging
- Support for human friendly logging
- Support for test friendly logging
- Support for async context tracking
- Support for redaction of sensitive content
- Reporting the source of empty log messages
- Relocating the context to a subdocument to avoid name clashes (level, time, etc)
- Ensuring errors are serialised correctly ([pino#862](https://github.com/pinojs/pino/issues/862), [winston#1338](https://github.com/winstonjs/winston/issues/1338), [bunyan#514](https://github.com/trentm/node-bunyan/issues/514))
- Ensuring circular references ([pino#990](https://github.com/pinojs/pino/issues/990), [winston#1946](https://github.com/winstonjs/winston/issues/1946),  [bunyan#427](https://github.com/trentm/node-bunyan/issues/427))
- Ensuring unserialisable context objects are tolerated

You can find an example factory module that implements these conventions [here](https://github.com/acuminous/module-acme-logging).

