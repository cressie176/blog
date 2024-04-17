A friend recently drew my attention to a [dangerous issue](https://github.com/prisma/prisma/issues/20169) with [Prisma](https://www.prisma.io/). For the unfamiliar, Prisma is a popular Object Relational Mapper (ORM) for Node.js with excellent TypeScript support and great documentation. The issue, which Prisma's developers say [is by design](https://github.com/prisma/prisma/issues/20169#issuecomment-1631989456), is that Prisma will delete all rows in a table if you specify a where clause attribute with an undefined value, e.g.

```js
await prisma.theme.deleteMany({
  where: {
    shop: undefined,
  }
});
```

Given the likelihood of accidentally undefined values in Node.js applications, even with the compile time protection of TypeScript, the potential consequence of this design decision is terrifying. The issue was reported in July 2023 and Prisma's developers agree that the behaviour is dangerous, but have no plans to address it. Prisma's `updateMany` suffers from the same problem.

This is not the first time a Prisma design decision has surprised me. When I first evaluated Prisma I was surprised that transactions appeared to be an afterthought. Instead of beginning a transaction, executing some code, and committing or rolling back the transaction, Prisma took a batch query approach like the Redis [MULTI](https://redis.io/docs/latest/commands/multi/) command. Batch queries have the major limitation that all statements must be determined in advance and cannot be conditionally performed based on the result of a previous query. Prisma added a traditional transaction API by way of a preview feature in v2.29.0, which has been generally available since v4.7.0 but I still find it bewildering that transactions were not better supported from the start. Prisma also lacks support for transparent transaction management, which other ORMs achieve through [AsyncLocalStorage](https://nodejs.org/api/async_context.html) and [cls](https://www.npmjs.com/package/cls). Based on [this discussion](https://github.com/prisma/prisma/issues/5729) I am unsure if the Prisma developers understand the concept of transparent transaction management and its benefits.

Then there was [this damning article](https://codedamn.com/news/product/dont-use-prisma) posted in April 2023 by Mehul Mohan of Codedamn, highlighting the problems of Prisma's in-memory joins. Prisma added a [preview feature](https://www.prisma.io/blog/prisma-orm-now-lets-you-choose-the-best-join-strategy-preview) in February 2024 allowing the choice of in-memory/application-level or database-level joins, but once again, I find the original design decision to eschew database-level joins in favour of in-memory/application-level ones bizarre.

In summary, Prisma has a history of surprising, limiting and sometimes dangerous design decisions that make it unfit for typical enterprise applications. So why is it popular? I can only imagine that Prisma's excellent TypeScript support, a pleasing website and use of Rust, enamours it to software engineers who care about those things, but

(i) don't use Prisma in typical enterprise applications, or<br>
(ii) don't understand what features are most important for such applications, or<br>
(iii) don't check whether Prisma adequately supports these features<br>

Prisma's popularity, combined with its inflated marketing claims, will ensure those in the second and third categories will continue to adopt it, putting them and their customers at risk of massive data loss. The problem is not restricted to Prisma. While we continue to trust the naivety of uninformed crowds, the software we produce will continue to depend on libraries that are unfit for purpose. Instead of selecting a library on popularism alone, I recommend writing a checklist of key criteria, then evaluating the library against them. For an ORM the list might be...

- Model Definition
  - Columns
  - Relationships
  - Type Mapping
- Schema Definition
  - Objects (Tables, Views, Functions, etc)
  - Indexes / Constraints
  - Cascades
  - SQL Case
- Query API
  - CRUD,
  - Upsert
  - Bulk
  - Where clause
  - Joins
  - Aggregation
  - Functions
- Raw Query Support
  - Execution
  - Binding
  - Result mapping
- Transaction API
  - Explict Transactions
  - Transparent Transactions
  - Custom Isolation Levels
- Generated SQL
  - Correctness
  - Performance
  - Debuggable
- TypeScript Support
- Migrations

I also suggest reviewing open and closed GitHub issues, paying particular attention to any that cause surprise. Finally, search for articles that are objectively critical of the library, as well as those that advocate for it.

If during your evaluation, you encounter bizarre design decisions, such as an ORM that doesn't support transactions, or reckless ones, such a laissez-faire attitude to deleting all of your data, then I recommend abandoning the evaluation and looking elsewhere.
