---
title: "NPM search is broken"
date: 2024-12-23
tags:
- npm
- search
---
As an active user of npm and an author/maintainer of several libraries, I’ve recently encountered significant problems with npm’s new search experience. While I appreciate the effort the npm team has put into improving search, the current implementation has introduced serious issues that negatively impact discoverability and usefulness.

### What's the Issue?
The new search algorithm prioritises objective sorting criteria like relevance, download counts, dependency counts, and publication date. However, these changes have led to surprising and often unhelpful results in real-world usage. Here are some key concerns:

- **Irrelevant Results**: Searching for “RabbitMQ” now returns results like [vasync](https://www.npmjs.com/package/vasync) and [slugid](https://www.npmjs.com/package/slugid), which are unrelated to RabbitMQ. This behaviour was not observed with the previous search implementation.

- **Misranked Relevant Packages**: Established and actively maintained libraries have been pushed far down the rankings for relevant keywords, making them difficult to find.

- **Prominence of Obscure or Stale Packages**: In searches like “hierarchical configuration,” packages that are outdated, rarely downloaded, or both dominate the results, displacing widely used and current libraries.

### What might have gone wrong?
From my observations, it appears the new search algorithm may disregard or de-emphasise package metadata such as "keywords" in favour of the contents of the package’s README. The problem with prioritising the README is that it inadvertently emphasises irrelevant libraries. For example, many libraries include a section in their README for migrating from previous versions. This causes them to rank highly for searches like “database migration” even when they have nothing to do with databases migration. Similiarly, many libraries include a section titled "Config" or “Configuration” to explain how to set up the package. This means irrelevant libraries frequently rank highly when searching for “configuration”

### Demonstrating the problem
To see just how poorly the new search performs for these examples, I searched for "hierarchical configuration" and reviewed the top 10 results...

#### 'Default' search order
<table>
<tr><th>Rank</th><th>Library</th><th>Assessment</th><th>Relevant</th><th>Current</th><th>Popular</th></tr>
<tr><td>1</td><td>node-env-configuration</td><td>A hierarchical configuration library with 4K downloads, last published 8 years ago</td><td>✅</td><td>❌</td><td>❌</td></tr>
<tr><td>2</td><td>ngx-access</td><td>An access control library for Angular with 78 downloads, last published 4 years ago</td><td>❌</td><td>❌</td><td>❌</td></tr>
<tr><td>3</td><td>turing-config</td><td>A hierarchical configuration library with 30 downloads, last published 7 years ago</td><td>✅</td><td>❌</td><td>❌</td></tr>
<tr><td>4</td><td>typeconf</td><td>A hierarchical configuration library with 21 downloads, last published 7 years ago </td><td>✅</td><td>❌</td><td>❌</td></tr>
<tr><td>5</td><td>config</td><td>A hierarchical configuration library with 6M downloads, last published 5 months ago</td><td>✅</td><td>✅</td><td>✅</td></tr>
<tr><td>6</td><td>d3-hierarchy</td><td>A library containing layout algorithms for hierarchical data with 17.6M downloads, last published 3 years ago</td><td>❌</td><td>❌</td><td>✅</td></tr>
<tr><td>7</td><td>config-core</td><td>A hierarchical configuration library with 20 downloads, last published 4 years ago</td><td>✅</td><td>❌</td><td>❌</td></tr>
<tr><td>8</td><td>@ehosick/config-core</td><td>A republish / duplicate of (7)</td><td>✅</td><td>❌</td><td>❌</td></tr>
<tr><td>9</td><td>nconf</td><td>A hierarchical configuration library with 3.5M downloads, last published 1 year ago</td><td>✅</td><td>✅</td><td>✅</td></tr>
<tr><td>10</td><td>fconf</td><td>A hierarchical configuration library with 11 downloads, last published 5 years ago</td><td>✅</td><td>❌</td><td>❌</td></tr>
</table>

Only 8 of the above results are configuration libraries, 6 have fewer than 100 downloads and 7 haven't been published within 3 years. Only [config](https://www.npmjs.com/package/config) and [nconf](https://www.npmjs.com/package/nconf) arguably justify a top 10 spot. More relevant, current and popular libraries like [cosmiconfig](https://www.npmjs.com/package/cosmiconfig) (61M) and [dotenv](https://www.npmjs.com/package/dotenv) (42M) do not even feature on the first page.

#### 'Most Downloaded This Week' search order
<table>
<tr><th>Rank</th><th>Library</th><th>Assessment</th><th>Relevant</th><th>Current</th><th>Popular</th></tr>
<tr><td>1</td><td>commander</td><td>A cli library with 165M downloads, last published 7 months ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>2</td><td>execa</td><td>A process execution library with 87M downloads, last published 1 month ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>3</td><td>schema-utils</td><td>A webpack validation library with 80M downloads, last published 1 year ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>4</td><td>strip-json-comments</td><td>A library to remove comments from JSON files with 62M downloads, last published 1 year ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>5</td><td>cosmiconfig</td><td>A hierarchical configuration library with 61M downloads, last published 1 year ago</td><td>✅</td><td>✅</td><td>✅</td></tr>
<tr><td>6</td><td>eslint</td><td>A linting library with 45M downloads, last published 1 week ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>7</td><td>dotenv</td><td>A hierarchical configuration library with 32M downloads, last published 3 days ago</td><td>✅</td><td>✅</td><td>✅</td></tr>
<tr><td>8</td><td>diff-sequences</td><td>A library for comparing sequences with 41M downloads, last published 1 year ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>9</td><td>@eslint/eslintrc</td><td>A linting library with 36M downloads, last published 1 month ago</td><td>❌</td><td>✅</td><td>✅</td></tr>
<tr><td>10</td><td>css-select</td><td>A CSS selector compiler with 32M downloads, last published 3 years ago</td><td>❌</td><td>❌</td><td>✅</td></tr>
</table>

While all of the results are popular and current, only 2 are configuration libraries.

### Conclusion
The new npm search algorithm returns either

- largely relevant, but unpopular and unmaintained libraries, or
- largely irrelevant, but popular and well maintained libraries

Neither option is useful.

I understand that balancing relevance, popularity, and recency is complex. Furthermore, improving tools like npm search is no small feat, especially given the wide-ranging needs of its user base. I deeply appreciate the work that maintainers to improve npm’s tools. However, the current implementation has clear flaws that undermine its usability.

One of the npm projects authors/maintainers announced the changes via [this discussion](https://github.com/orgs/community/discussions/144952), and invited feedback. I offered the above analysis three weeks ago, but it was initially misunderstood, and after clarification, has still to be adequately acknowledged. If you've encountered similar issues or have ideas for improvement, your feedback could help improve npm's search experience. Please join [the discussion](https://github.com/orgs/community/discussions/144952) on GitHub.

Thank you.
