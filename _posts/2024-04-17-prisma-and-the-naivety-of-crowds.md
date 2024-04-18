A [friend](https://www.rouanw.com) recently drew my attention to a [dangerous issue](https://github.com/prisma/prisma/issues/20169) with [Prisma](https://www.prisma.io/). For the unfamiliar, Prisma is a popular Object Relational Mapper (ORM) for Node.js with excellent TypeScript support and great documentation. The issue, which the developers say is [by design](https://github.com/prisma/prisma/issues/20169#issuecomment-1631989456), is that Prisma will delete all rows in a table if you specify a where clause attribute with an undefined value, e.g.

```js
await prisma.theme.deleteMany({
  where: {
    shop: undefined,
  }
});
```

Given the likelihood of accidentally undefined values in Node.js applications, even with the compile time protection of TypeScript, the potential consequence of this design decision is terrifying. The issue was reported in July 2023 and Prisma's developers [agree the behaviour is dangerous](https://github.com/prisma/prisma/issues/20169#issuecomment-1631760913), but have no plans to address it. Prisma's `updateMany` suffers from the same problem.

This is not the first time a Prisma design decision has surprised me. When I first evaluated Prisma I was surprised that transactions appeared to be an afterthought. Instead of beginning a transaction, executing some code, and committing or rolling back the transaction, Prisma took a batch query approach like the Redis [MULTI](https://redis.io/docs/latest/commands/multi/) command. Batch queries have the major limitation that all statements must be determined in advance and cannot be conditionally performed based on the result of a previous query. Prisma added a transaction API by way of a preview feature in v2.29.0, which has been generally available since v4.7.0 but I still find it bewildering that transactions were not better supported from the start. Prisma also lacks support for transparent transaction management, which other ORMs achieve through [AsyncLocalStorage](https://nodejs.org/api/async_context.html) and [cls](https://www.npmjs.com/package/cls). Based on [this discussion](https://github.com/prisma/prisma/issues/5729) I am unsure if the Prisma's developers understand the concept of transparent transaction management and its benefits.

Then there was [this damning article](https://codedamn.com/news/product/dont-use-prisma) posted in April 2023 by Mehul Mohan of Codedamn, highlighting its poor performance, largely but not exclusively due to Prisma's reliance on application-level joins. Prisma added a [preview feature](https://www.prisma.io/blog/prisma-orm-now-lets-you-choose-the-best-join-strategy-preview) in February 2024 allowing the choice of application-level or database-level joins, but once again, I find the original design decision to eschew database-level joins in favour of application-level ones bizarre.

In summary, Prisma has a history of surprising, limiting and sometimes dangerous design decisions that make it unfit for typical enterprise applications. So why is it popular? I can only imagine that Prisma's excellent TypeScript support, a pleasing website and use of Rust, enamours it to software engineers who care about those things, but

&nbsp;&nbsp;&nbsp;(i) don't use Prisma in typical enterprise applications, or<br>
&nbsp;&nbsp;&nbsp;(ii) don't understand what features are most important for such applications, or<br>
&nbsp;&nbsp;&nbsp;(iii) don't check whether Prisma adequately supports these features<br>

Prisma's popularity, combined with its claims of simplicity and a great developer experience will ensure those in the second and third categories keep choosing it, putting them and their customers at risk of massive data loss. The problem is not restricted to Prisma. While we in the software industry continue to trust the misguided wisdom of uninformed crowds, the software we produce will continue to depend on libraries that are unfit for purpose.

When the consequence of a poor choice is severe, more rigour is called for. Instead of selecting a library by popularism alone, I recommend identifing key screening and ranking criteria by reviewing the documentation of mainstream candidates. Screening criteria are essential features, which for an ORM might include transactions and adequate performance. If a candidate fails a single screening criteria it must be rejected immediately. In constrast, ranking criteria is desirable, but not essential, and helps you find the best choice from a shortlist of screened candidates. An example might be "TypeScript", support for which could be "Excellent", "Good", "Fair", "Poor" or "Terrible".

Screening is usually simpler than ranking, and since it removes candidates, should be done first. Ranking is easier when using a consistent [rating scale](https://en.wikipedia.org/wiki/Rating_scale) and if the criteria are of equal importance. If the criteria are not equally important, you can weight them, but this increases complexity. Instead, I prefer to start with the most important ranking criteria, and only consider less important ones in the event of a tie. Finally, criteria may be both screening and ranking, since you may exclude candidates that don't meet a minimum threshold, and prefer those that excel.

### Screening Example
<table>
<thead>
<tr>
<th></th>
<th>Candidate A</th>
<th>Candidate B</th>
<th>Candidate C</th>
</tr>
</thead>
<tbody>
<tr>
<td>CRUD Operations</td>
<td>✓</td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Documentation</td>
<td>✓</td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Maintained</td>
<td>✓</td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>PostgreSQL</td>
<td>✓</td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Showstopper Issues</td>
<td>✗</td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Test Coverage</td>
<td></td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Transactions</td>
<td></td>
<td>✓</td>
<td>✓</td>
</tr>
<tr>
<td>Performance</td>
<td></td>
<td>✓</td>
<td>✓</td>
</tr>
</tbody>
</table>

Candidate A was rejected due to showstopping issues. Screening criteria that is hard to measure (e.g. Performance) should be evaluated last to minimse the candidates it needs to be done for.

### Ranking Example
<table>
<thead>
<tr>
<th></th>
<th>Candidate B</th>
<th>Candidate C</th>
</tr>
</thead>
<tbody>
<tr>
<td>Model Definition</td>
<td>Excellent</td>
<td>Excellent</td>
</tr>
<tr>
<td>Schema Definition</td>
<td>Excellent</td>
<td>Excellent</td>
</tr>
<tr>
<td>CRUD Operations</td>
<td>Good</td>
<td>Excellent</td>
</tr>
<tr>
<td>Raw Query Support</td>
<td>Fair</td>
<td>Good</td>
</tr>
<tr>
<td>Transactions</td>
<td>Good</td>
<td>Good</td>
</tr>
<tr>
<td>Maintenance</td>
<td>Excellent</td>
<td>Fair</td>
</tr>
<tr>
<td>TypeScript Support</td>
<td>Excellent</td>
<td>Poor</td>
</tr>
</tbody>
</table>

Candidate B has far better TypeScript support and is significantly better maintained, so assuming the criteria is of equal importance, is the best choice. When the decision is closer, extend the table to include less important criteria.

For my screening criteria, I always include a review of a candidate's GitHub issues (both open and closed), paying particular attention to any that cause surprise. Finally, I suggest actively looking for articles that are objectively critical of the library, as well as those that advocate for it. If during your evaluation, you encounter bizarre or reckless design decisions, such as an ORM that doesn't support transactions, or worse, a has laissez-faire attitude to protecting data, then I recommend rejecting that candidate and looking elsewhere.
