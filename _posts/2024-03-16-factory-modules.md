---
title: "Factory Modules"
date: 2024-03-16
---

A key benefit of microservice architectures is that multiple teams can work different parts of the system without tripping over each other. This isn't without its problems however. One common issue is the proliferation of 3rd party libraries like logging libraries, http clients, and persistance frameworks, etc. The first step to mitigate this is to standardise on a single library for each function, but this alone is insufficient. Take logging for example. Imagine if you standardised on [Pino](https://github.com/pinojs/pino), installing it directly in 50 different service, only then to find when you logged an AxiosError it included sensitive HTTP headers; you would have to add the redaction code to each one of those 50 repositories. A better approach is to wrap 3rd party libraries like Pino within a factory module and to apply a set of tested, sensible, organisational conventions, thus creating a [Pit of Success](https://learn.microsoft.com/en-us/archive/blogs/brada/the-pit-of-success). For a logging library these might be:

- Redact sensitive content
- Use a human friendly format locally
- Disable log messages during tests runs by default
- Support [async context tracking](https://nodejs.org/api/async_context.html)
- Report the source of empty log messages
- Relocate the context to a subdocument to avoid name clashes (level, time, etc)
- Test errors are serialised correctly (https://github.com/pinojs/pino/issues/862, https://github.com/winstonjs/winston/issues/1338, https://github.com/trentm/node-bunyan/issues/514)
- Test for circular references (https://github.com/pinojs/pino/issues/990, https://github.com/winstonjs/winston/issues/1946, https://github.com/trentm/node-bunyan/issues/427)
- Test for unserialisable context objects

I've created a example factory module for Pino implementing these conventions. You can find it [here](https://github.com/cressie176/module-acme-logging).
