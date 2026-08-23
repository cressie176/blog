---
title: "Good mechanisms for Drizzle: Announcing drizzle-transact and drizzle-explain"
date: 2026-08-23
excerpt: "Announcing `drizzle-transact` and `drizzle-explain`: two small libraries that turn good intentions into good mechanisms."
tags:
- Drizzle ORM
- TypeScript
- JavaScript
- PostgreSQL
- MariaDB
- Transactions
- Query Plans
- Query Optimisation
- Software Engineering
- Claude Code
- AI-Assisted Development
---

There is a line from Jeff Bezos that I keep coming back to: good intentions don't work, good mechanisms do.

The point is that if something repeatedly goes wrong, asking people to be more careful is rarely a satisfactory answer. People forget things, take shortcuts, misunderstand conventions and make mistakes. Good intentions also don't scale particularly well. You have to communicate them to everyone who works on the code, communicate them well enough that they understand not just what to do but why, and then rely on that knowledge being retained. Even when the rule itself is easy to explain, its importance may depend on experience and context that is much harder to transfer.

A better solution is usually to change the system so that doing the right thing is the easiest option, or so that doing the wrong thing becomes immediately visible. The mechanism carries the knowledge instead of relying on every developer having it. There are two places where I think the Drizzle ORM ecosystem relies a little too heavily on good intentions, so I built libraries to address them.

## Transactions are too easy to escape

Drizzle requires the transaction object to be explicitly passed to anything that participates in a transaction:

```ts
const order = await db.transaction(async (tx) => {
  return createOrder(tx);
});

async function createOrder(tx: DbTransaction) {
  const [order] = await tx.insert(orders).values(...).returning();
  await createOrderItems(tx, order.id);
  return order;
}

async function createOrderItems(tx: DbTransaction, orderId: number) {
  await tx.insert(orderItems).values(...);
}
```

Passing `tx` around is slightly annoying, but that isn't the important problem. The important problem is that `db` still exists. Somewhere further down the call stack it is very easy to use `db.insert(...)` rather than `tx.insert(...)`. The query works perfectly well, but it is no longer participating in the transaction.

For a read that may not matter, but for a sequence of related writes it matters a great deal. One write can commit while another rolls back, leaving the database in a state the application never intended. Nothing about the code makes this mistake especially conspicuous: it compiles, functional tests may well pass, and the problem only becomes apparent when something fails at exactly the wrong point.

You can document the rule that database access inside a transaction must always use `tx`, but that brings us straight back to good intentions. Every developer has to know the rule, understand why it matters and recognise all the places where it applies. As the team changes and the codebase grows, that knowledge has to be communicated and retained.

### drizzle-transact

[`drizzle-transact`](https://www.npmjs.com/package/drizzle-transact) takes a different approach. The raw Drizzle instance is wrapped during application setup and is not exported for general use. Database access happens through functions such as `withTransaction` and `newTransaction`, while the current transaction is held using Node's `AsyncLocalStorage` so functions further down the call stack can automatically participate in it:

```ts
import { newTransaction, withTransaction } from './db';

const order = await newTransaction(() => createOrder());

async function createOrder() {
  return withTransaction(async (tx) => {
    const [order] = await tx.insert(orders).values(...).returning();
    await createOrderItems(order.id);
    return order;
  });
}

async function createOrderItems(orderId: number) {
  await withTransaction(async (tx) => {
    await tx.insert(orderItems).values(...);
  });
}
```

`createOrderItems` doesn't need to know that `createOrder` started the transaction. `withTransaction` simply joins the transaction that is already active. This removes the need to pass `tx` through layers of otherwise unrelated business logic, which is useful in itself, but the main reason for doing it is safety: application code doesn't have a convenient non-transactional database handle available to accidentally use.

The model is deliberately similar to transaction propagation in Spring. The underlying `transact` function supports:

* `Required` - join the current transaction, or create one
* `RequiresNew` - always create an independent transaction
* `Nested` - use a savepoint within the current transaction
* `RequiresExisting` - join the current transaction, or throw if there isn't one
* `Never` - require that no transaction is active

There are also shorthand functions for the common cases. I particularly like `RequiresExisting`: if a function such as `deductStock` is only safe when called as part of a larger transaction, that constraint can be expressed in code. Calling it incorrectly then fails immediately rather than relying on somebody knowing, and remembering, the convention.

## Why not use drizzle-transactional?

There is already another library solving a similar problem: [`drizzle-transactional`](https://www.npmjs.com/package/drizzle-transactional). I looked at it before deciding to write `drizzle-transact`, and it is worth considering. Its API wasn't quite what I wanted, though. I preferred explicit transaction propagation to its hook-based model, and although I initially liked its `@Transactional` decorator, implementing something similar required proxying the Drizzle client and routing calls invisibly between the active transaction and the underlying client. It worked, but felt more magical than I wanted for something whose purpose is to make transactional behaviour clear.

The other difference is dependencies. `drizzle-transactional` depends on `zod`, `reflect-metadata` and `drizzle-orm`, whereas `drizzle-transact` has no production dependencies and uses Node's `AsyncLocalStorage` plus the Drizzle instance the application already has. Dependencies bring upgrade work, security noise and additional supply-chain exposure, so when a small infrastructure library can reasonably avoid them, I think it should.

## Testable query plans

Consider this query:

```ts
db.select()
  .from(reservations)
  .where(eq(reservations.roomId, roomId));
```

Suppose `reservations` contains five million rows and there is no index on `room_id`. The query is perfectly correct, but PostgreSQL has little choice other than to scan the table. That may be invisible during development: against a small local database the query is fast, its functional tests pass and nothing suggests there is a problem. It only becomes expensive once it runs against realistic volumes.

There is another version of the same problem where the schema is correct and the necessary indexes exist, but the optimiser's statistics no longer represent the data particularly well. PostgreSQL chooses plans using estimates about things such as predicate selectivity and the number of rows produced by each operation. If those estimates are badly wrong, it can choose a plan that looks sensible for the data it thinks it has but performs badly against the data it actually has. In both cases, functional correctness tells us nothing useful because the query still returns the right answer.

The usual advice is to inspect important queries with `EXPLAIN`, but that is another good-intentions solution. It relies on developers identifying which queries deserve inspection, remembering to inspect them, understanding the resulting query plan and doing so against representative data. Query performance is also particularly easy to overlook in JavaScript and TypeScript teams, where many developers are weighted more towards frontend development and simply haven't accumulated years of database experience. That isn't a criticism, but it does make database performance a poor thing to depend on individual knowledge and memory for.

### drizzle-explain

[`drizzle-explain`](https://www.npmjs.com/package/drizzle-explain) makes query plans testable. It executes a Drizzle query using `EXPLAIN (ANALYZE, FORMAT JSON)` and checks the resulting query plan against configurable constraints:

```ts
import { createExplain } from 'drizzle-explain';
import { postgresDriver } from 'drizzle-explain/postgres';

const explain = createExplain(
  postgresDriver(pool),
  {
    maxCost: 100,
    rowEstimateTolerance: 10,
  },
);

const analysis = await explain(
  (db) => findReservationsByRoom(db, roomId),
);

assert.ok(analysis.passed, analysis.message);
```

Queries are executed inside a transaction which is always rolled back, so writes can also be analysed without leaving test data behind. The library currently checks three aspects of the query plan: estimated cost, the accuracy of row estimates and optionally the presence of disallowed operations.

A missing index will often produce a dramatic increase in the optimiser's estimated cost. A deliberately low cost threshold provides a useful tripwire: if a query crosses it, somebody needs to look at the plan and either improve the query or explicitly accept a higher cost for that particular case.

The second check compares estimated and actual row counts throughout the plan. Poor row estimates are an important cause of bad plan selection. Measuring the ratio rather than the absolute difference also means that normal growth in the amount of data doesn't inherently break the test. What matters is whether the shape of the data has changed enough that the optimiser's model of it is becoming inaccurate.

Finally, individual plan operations can be disallowed. If, for example, a query should never perform a sequential scan of a particular table, the test can say so directly. When something fails, the analysis includes a rendered version of the query plan showing the offending node:

```text
✘ cost 62431 exceeds limit 100

Seq Scan on reservations  (cost=0..62431 rows=10 actual=10)  ✘ cost 62431 > 100
  Filter: (room_id = 42)
```

One thing the library deliberately does not test is execution time. Timing depends on hardware, cache state and whatever else happens to be running on the machine, so a test that passes on a developer's laptop and intermittently fails in CI isn't particularly useful. Cost and row-estimate accuracy describe the query plan rather than the machine running it, which makes them much better candidates for automated tests.

## The test data matters

There is an important limitation to all of this: `EXPLAIN` can only tell you about the data it is given, and the optimiser doesn't make decisions based purely on the number of rows in a table. Distribution matters. A hotel booking system, for example, may have considerably more summer reservations than winter reservations and some room grades may be much more popular than others. Seed a database with uniformly distributed test data and the resulting query plans may bear little resemblance to production.

For that reason the `drizzle-explain` repository contains a worked example using [`drizzle-seed`](https://github.com/drizzle-team/drizzle-orm/tree/main/drizzle-seed) to construct a production-shaped hotel booking dataset rather than simply generating a large number of random rows. This is probably the most important prerequisite for useful query-plan testing. You don't necessarily need a copy of production, but you do need data with approximately the same characteristics.

`drizzle-explain` currently supports PostgreSQL and MariaDB. SQLite's `EXPLAIN QUERY PLAN` doesn't expose cost or estimated row counts, so the checks the library is built around aren't available there. As with `drizzle-transact`, the library adds no production dependencies: you provide the database client and the appropriate driver adapts its query plans into the common representation used by the analyser.

## The common idea

The two libraries solve different problems, but they came from the same observation. With transactions, we rely on developers remembering to use the right database handle throughout the call stack. With query performance, we rely on them remembering to inspect query plans, understanding them and doing so against representative data. Neither is something I want to depend on individual knowledge and memory for.

`drizzle-transact` makes transaction propagation the normal way to access the database, while `drizzle-explain` turns query-plan expectations into executable tests. In both cases, the mechanism captures knowledge that would otherwise have to be repeatedly communicated and remembered, and applies it consistently. That is what I mean by a good mechanism.

## How I built them

I built both libraries using Claude Code (Opus 4.8), following almost the same process each time. I first worked through the problem and API design with Claude, then had it write the README before any implementation. That README became the specification and source of truth for the rest of the work.

Claude then broke the implementation into GitHub issues that could be worked on in parallel using separate worktrees. `CLAUDE.md` and `CONTRIBUTING.md` contained the standing instructions, coding conventions and development rules. I focused mainly on the design and behaviour, then reviewed the implementation in detail towards the end.

Parallel development did introduce some drift. In `drizzle-explain`, the PostgreSQL and MariaDB drivers and examples developed slightly different approaches to the same behaviour and needed reconciling afterwards. Claude also introduced a normalised query-plan representation between the database-specific drivers and the core analyser, and identified the limitations of SQLite support from the information exposed by `EXPLAIN QUERY PLAN`.

Each library took roughly half a day of elapsed time to build, although I was doing other things at the same time. Most of my involvement was in defining the behaviour, refining the API and reviewing the result rather than writing the implementation itself.

Both libraries are MIT licensed and available on npm:

* [`drizzle-transact`](https://www.npmjs.com/package/drizzle-transact)
* [`drizzle-explain`](https://www.npmjs.com/package/drizzle-explain)
