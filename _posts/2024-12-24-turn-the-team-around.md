---
title: "Turn The Team Around!"
date: 2024-12-24
tags:
- Team
- Rescue
- Motivation
- Slack
- Waste
- Agile
- Targets
---
Targets can be a double-edged sword. When poorly conceived they encourage behaviours that prioritise metrics over real success. For instance, setting a goal like "close 100 tickets this quarter" might push a team to focus on easy wins rather than meaningful work. However, I once encountered a target that broke this mould and completely shifted my perspective. It was when challenged to "make Portal a destination team".

At the time, the School Portal team had a poor reputation. Engineers were reluctant to join, citing a disjointed codebase and a team culture that, while professional, felt siloed and under immense pressure. David’s target wasn’t about deliverables; it was about creating a team environment where people wanted to be. Success would be measured by whether engineers were eager to join us.

### Taking Stock

When I first joined the Portal team, I took a close look at the challenges we faced. The problems were easy to spot but significant in scale. The four engineers on the team were working in isolation, each handling months-long initiatives. This approach had been the norm for over a year, leading to knowledge silos. There was no slack. If something broke, only the original developer could fix it, and this would derail whatever feature they were currently working on. The product manager was constantly juggling stakeholder expectations amidst these disruptions, and rapidly losing their confidence.

The quality of the work also left much to be desired. Without collaboration or shared accountability, individual coding styles went unchecked. For example, I know my tendency to overcomplicate solutions needs balancing input from others, but here, such feedback loops were nonexistent. The user experience was no better. Customers frequently reported “bugs” that were actually just confusing designs that violated basic UX principles.

An earlier attempt to fix the codebase had backfired. The team had embarked on a large-scale refactor, but without breaking it into manageable, independent deliverables. They didn’t finish a quarter of the work before their three-month grace period elapsed, leaving the code in an even worse state than before. This was compounded by architectural issues which resulted in frequent timeouts and race conditions.

The planning board was another headache. Hundreds of tickets sat there — bugs, technical improvements, features. It was impossible to prioritise effectively. Meanwhile, bugs were piling up, creating a flood of failure demand (work caused by defects or inefficiencies, as opposed to value demand which directly serves customer needs).

One bug in particular highlighted the problem. Over the course of a year, an issue with embedded videos was reported five times by different customers. Each time, it was investigated, deemed low priority, and closed without being fixed. This resulted in a poor customer experience and wasted time not only for our team but also for the company’s helpdesk. Each report required a day of investigation, yet fixing the issue outright would have taken less than two day.

What struck me most, though, was the team’s attitude. They openly disparaged the codebase. It reminded me of a story from "Turn the Ship Around!" by L. David Marquet, where the submarine crew were ashamed of their vessel until they rediscovered pride in their work. The Portal team needed that same shift in mindset.

### Turning the Tide

The first challenge was morale. Without hope for change, there would be no progress. Luckily, I had an ace up my sleeve: John Hustler, a colleague and friend. John’s thoughtful approach and willingness to go where he was needed most made him the perfect addition. When he joined, it gave me the support I needed to align the team.

To foster a sense of pride, I introduced a small but symbolic initiative. We created a "Show Portal Some Love" ticket. Every time someone made an improvement, no matter how small, they earned a ❤️. The team may have thought it a bit childish, but my hope was that it gave them permission to make things better, while celebrating small wins and lightening the mood. Slowly but surely, they started to see the Portal as something worth investing in.

The next step was tackling waste. I reviewed every ticket on our bloated planning board, moving low-priority ones to a separate board and closing those that weren’t worth pursuing. The issues I moved weren’t high-priority from the product perspective, but they generated a lot of noise or failure demand. I broke them down into clear, actionable steps and with the help of a borrowed contractor, chipped away at them. Quick wins, like simple UX improvements reduced customer confusion and lowered the volume of reported issues. We also removed unreliable, nice-to-have features, such as a tool for exporting CVs as a single PDF. This feature only worked one in ten times, often crashing the system or timing out. Fixing it would have taken weeks, but removing it took minutes, eliminating a major source of frustration for both users and the team.

With fewer interruptions, the team could finally focus. The improved flow created a virtuous cycle: more capacity led to more fixes and a higher standard of work, which led to even greater capacity. Over time, we gained enough breathing room to address deeper issues, like architectural flaws.

Meanwhile, I worked with the product manager and Chief Product Officer to reduce the team’s workload. We dropped one of the large deliverables, freeing up an engineer to focus on improvements. This decision, though difficult, paid off as the team regained momentum.

### A New Way of Working

Once the immediate crises were under control, I shifted our approach. Instead of each engineer tackling separate features, we began working together on a single epic. Our first shared effort was integrating social media into the jobs platform. The team was initially skeptical. They worried it would slow them down, but the results spoke for themselves.

Working together meant we could deliver incrementally. Instead of waiting three months to finish three separate features, we completed one feature in the first month, another in the second, and the final one in the third. Not only did this approach deliver value faster, but it also fostered collaboration and shared ownership.

### The Results

Within six months, the transformation was evident. The backlog was cleared, the main board had fewer than 25 tickets, and morale was at an all-time high. Six months later, the team had re-architected the application, eliminating performance bottlenecks and race conditions while continuing to deliver product features. We showcased our work at departmental demos, and, for the first time, other engineers were expressing interest in joining the Portal team.

By focusing on collaboration, reducing waste, and building pride, we had turned the team around. The journey wasn’t about individual heroics but about creating an environment where the team could thrive. It’s proof that with the right focus, any team can become a destination team.

----
<br/>
#### Recommended Reading
- [Turn the Ship Around!](https://davidmarquet.com/turn-the-ship-around-book/)
- [The Design of Everyday Things](https://www.nngroup.com/books/design-everyday-things-revised/)