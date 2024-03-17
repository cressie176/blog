---
title: "Factory Modules"
date: 2024-03-16
---

A key benefit of microservice architectures is that multiple teams can work different parts of the system without tripping over each other. This isn't without its problems however. One common issue is the proliferation of 3rd party libraries like logging libraries, http clients, and persistance frameworks, etc. The first step to mitigate this is to standardise on a single library for each function, but this alone is insufficient. Take logging for example. Imagine if you standardised on [pino](https://github.com/pinojs/pino), installing it directly in 50 different service, only then to find your logs include sensitive HTTP headers; you would have to add the redaction code to each one of those 50 repositories.

A better approach is to wrap 3rd party libraries like [pino](https://github.com/pinojs/pino) within a custom factory module that applies a set of tested, sensible conventions, safeguards and features, thus creating a [Pit of Success](https://learn.microsoft.com/en-us/archive/blogs/brada/the-pit-of-success). Not only will this reduce the effort when you discover a new convention, but is the most effective way of leveraging the years of hard won lessons, you and your senior engineers have accumulated over their careers. Quality truly is [the knife-edge of experience](https://en.wikipedia.org/wiki/Pirsig%27s_Metaphysics_of_Quality).

For a logging library sensible conventions, safeguards and features might be:

- Support redaction of sensitive content
- Support for machine friendly logging
- Support for human friendly logging
- Support for emitting log records as events
- Support for async context tracking
- Reporting the source of empty log messages
- Allowing logging to be suppressed in tests
- Relocation of the context to a subdocument to avoid name clashes (level, time, etc)
- Testing errors are serialised correctly ([pino#862](https://github.com/pinojs/pino/issues/862), [winston#1338](https://github.com/winstonjs/winston/issues/1338), [bunyan#514](https://github.com/trentm/node-bunyan/issues/514))
- Testing circular references ([pino#990](https://github.com/pinojs/pino/issues/990), [winston#1946](https://github.com/winstonjs/winston/issues/1946),  [bunyan#427](https://github.com/trentm/node-bunyan/issues/427))
- Testing unserialisable context objects

I've created a example factory module for Pino implementing these conventions. You can find it [here](https://github.com/acuminous/module-acme-logging).

