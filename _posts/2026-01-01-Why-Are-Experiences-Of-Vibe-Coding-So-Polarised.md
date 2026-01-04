---
title: "Why Are Experiences Of Vibe Coding So Polarised?"
date: 2026-01-01
redirect_from:
  - /2026/01/01/Why-Are-Experiences-of-Vibe-Coding-so-Polarised.html
tags:
- Generative AI
- Claude Code
- Clean Code
- Vibe Coding
---

## TL;DR
This post explores why experiences with Generative AI assisted software development vary so dramatically. While some developers report order of magnitude productivity gains, others encounter architectural drift, excessive code, and serious operational risk. The author argues that this gap is not primarily caused by the tools themselves, but by differences in goals, constraints, and methods of use.

Treating Generative AI as a meta tool that operates on intent rather than code mechanics, the post makes explicit its own optimisation goals: negligible operational debt and highly malleable, clean code. A controlled experiment using a small but realistic URL shortener service tests three ways of working with Claude Code.

Prompt only bootstrapping proved unreliable. A guided approach combining layered templates, explicit implementation notes, rules, and manual review produced a clean, low debt system in just over an hour, delivering an estimated eight to sixteen times improvement over manual development. In contrast, an unattended approach without guidance completed faster but generated substantially more code, significant technical and operational debt, and multiple design regressions.

The conclusion is that Generative AI can deliver dramatic gains, but only when projects are pre-established or properly bootstrapped, and the agent is strongly constrained. Left unattended, it reliably drifts towards verbosity and accidental complexity. The real question for organisations is therefore not whether to adopt Generative AI, but how to use it in a way that aligns with their long term engineering goals.

## Introduction
Jason Gorman’s recent [post](https://codemanship.wordpress.com/2025/11/25/the-future-of-software-development-is-software-developers/) on the future of software development is causing quite a stir. The comments on the accompanying Hacker News [discussion](https://news.ycombinator.com/item?id=46424233) span an extraordinary range of experience with vibe coding, particularly when using Claude Code. Some describe dramatic productivity gains and rapid delivery of complex systems. Others report dangerously misleading output, architectural drift, and large amounts of unusable code. Both sides speak with confidence, often dismissing the other as naïve, reckless, or simply doing it wrong. The discussion reflects a broader pattern which has been bothering me for some time.

I have been a professional software engineer for thirty years, and strong disagreements are nothing new. Functional versus object-oriented programming, static versus dynamic typing, competing approaches to testing. What is different this time is the nature of the tool itself. Generative AI does not operate at the level of a language, framework, or architectural paradigm. It operates at the level of intent. In that sense it is a meta tool, unconstrained by the trade-offs that normally explain why a tool works well for some people or some problems, and poorly for others.

It does not matter that Generative AI is unusually non-deterministic. For it to be useful to anyone, it still has to produce desired outcomes with sufficient consistency. That makes the scale and intensity of the reported disagreement surprising. Vastly different experiences are unlikely to be explained by an inherent defect in the tool itself. Even allowing for bias and differing [personality temperaments](https://keirsey.com/temperament-overview/) does not adequately explain the gap. The same tool is being described, with equal confidence, as producing dangerous, unmaintainable AI slop on the one hand, and delivering twenty times productivity on the other.

This gap is pushing organisations to make extreme decisions. Some, driven by unrealistic expectations, are rushing too quickly into AI adoption, creating unnecessary anxiety and disruption. Others are avoiding it entirely, missing out on potential benefits and frustrating their internal AI champions. The gap needs to be understood and closed.

## What I Want From Generative AI

One possible explanation for difference in experience is that people want different outcomes. If that is the case, then disagreement about Generative AI performance is inevitable. Before comparing tools or techniques, the goals themselves need to be explicit. I want Generative AI to rapidly create applications with the following characteristics:

### 1. Negligible Operational Debt
Operational debt is distinct from technical debt. Technical debt only incurs a cost when change is required and is often an explicit trade-off. Operational debt creates ongoing risk and [unplanned work](https://blog.while-true-do.io/devops-4-types-of-work/) and, in the worst cases, can consume an entire team’s capacity through incidents and urgent remediation.

### 2. Unparalleled Malleability
[Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/) is malleable. When we write clean code, we make a comparatively small sacrifice now to reserve the ability to change rapidly in the future. The best way to achieve clean code is to write as little code as possible. This is done by creating a good domain model. When the domain model is good, the code vanishes. As Linus Torvalds is [reported](https://read.engineerscodex.com/p/good-programmers-worry-about-data) to have said:

> “Bad programmers worry about the code. Good programmers worry about data structures and their relationships.”

A good domain model requires good encapsulation. Behaviour is contained in one place, colocated with the associated data. Accessors are few, mutators fewer still. Conditional logic is pushed to the boundaries of the application and eliminated internally through polymorphism. The code clearly expresses intent.

I have a nagging suspicion that my attachment to clean code may be what Scott Adams calls [Loserthink](https://en.wikipedia.org/wiki/Loserthink), and no longer needed in the world of Generative AI. Until that suspicion is proven, I am sticking with it though. The risk of a future filled with vast quantities of even less malleable code than we already have is too great.

## My Experience Of Vibe Coding (So Far)

### The Good

It is often stated that Generative AI is effective at mechanical or rote tasks, or comparable to the output of a junior engineer, but this is not where it offers the most value. The area where I have found my Generative AI tool of choice, [Claude Code](https://www.claude.com/product/claude-code), to be most effective is where I have strong high-level judgement about what needs to be achieved, but lack the detailed, low-level knowledge to implement it without time, research, and mistakes. In those cases, the limiting factor is not judgement, but execution.

CI/CD pipelines leveraging Docker provide a good example. I know what I want a pipeline to do, how the stages should fit together, and what correct behaviour looks like. Implementing that by hand usually involves reading documentation, iterating on syntax, and discovering edge cases through failure. Claude is much faster at cycling through that execution loop than I am, especially after installing the [GitHub CLI](https://cli.github.com), which is an absolute game changer for working with [GitHub Actions](https://github.com/features/actions)!

In summary, Generative AI adds the most value when it operates below my level of judgement but above my knowledge or ability to acquire it. It accelerates low-level execution without being asked to make high-level decisions it is poorly suited to make.

### The Bad

At the same time, Claude, has ignored explicit instructions, made false assumptions, implemented changes that were not requested, and drifted away from the intended structure, particularly early in a codebase when there is less existing context. It has also misdiagnosed problems and disappeared down rabbit holes without ever fixing them, or worse, unilaterally decided that they can be safely ignored.

What stands out is how sensitive the results are to relatively small changes in how the tool is used. Claude is not a compiler. The results are not deterministic. Small differences in context, ordering, or phrasing can lead to materially different outcomes, even when the intent appears unchanged. Overall, the outcomes are still positive, but I have yet to achieve the 20x improvement reported by some. Either those reports are grossly overstated, or those making them have found ways to circumvent these issues and make Claude perform consistently well. If the latter is true, I want to learn and adopt their methods, but what are they?

## What We Need Is An Experiment

To move this discussion forward, we need something more concrete than confident anecdote. When the same tool is reported to produce both dangerous, unmaintainable systems and dramatic productivity gains, opinion alone cannot tell us whether the difference lies in the tool itself, the goals being optimised for, or the way it is being used. The only way to separate those factors is to make the goals explicit and then test, in a controlled way, whether a particular method of using Generative AI can reliably produce outcomes aligned with them. What is needed is a repeatable test case against which different methods can be evaluated. It must be small enough to run repeatedly, representative of real world software, and sufficiently complex to expose meaningful trade offs and failure modes. A URL shortening service meets those criteria. Here are the stories:

| Story                                                 | Title                          | Description                                                                                            |
|-------------------------------------------------------|--------------------------------|--------------------------------------------------------------------------------------------------------|
| [EPIC](https://github.com/cressie176/shorty/issues/1) | URL Shortener Service          | Build a complete URL shortener service with persistence, automatic expiry, and scheduled maintenance.  |
| [1](https://github.com/cressie176/shorty/issues/2)    | Project Initialisation         | Create the project structure and infrastructure.                                                       |
| [2](https://github.com/cressie176/shorty/issues/3)    | Shorten URL                    | Shortens the given URL.                                                                                |
| [3](https://github.com/cressie176/shorty/issues/4)    | Get URL                        | Returns a URL for the given short key.                                                                 |
| [4](https://github.com/cressie176/shorty/issues/5)    | URL Redirection                | Redirect requests for a short key to the URL.                                                          |
| [5](https://github.com/cressie176/shorty/issues/6)    | Improve Duplicate Key Handling | Detect and handle the extremely rare case where the same short key is generated for different URLs.    |
| [6](https://github.com/cressie176/shorty/issues/7)    | Expire Redirects               | Automatically expire the redirects when they have not been accessed for a configurable period of time. |
| [7](https://github.com/cressie176/shorty/issues/8)    | Delete Expired Redirects       | Automatically delete expired redirects.                                                                |
| [8](https://github.com/cressie176/shorty/issues/9)    | Schedule Database Maintenance  | Schedule daily database maintenance to maintain PostgreSQL query planner statistics.                   |

### Hypothesis

The vast variation of experience comes from how Generative AI is being used, not from a fundamental weakness in the tool itself.

### Apparatus

* **Machine:** MacBook M1 Pro with 32GB RAM
* **Operating system:** macOS 15.6.1 (24G90)
* **Model:** Claude Sonnet 4.5 via AWS Bedrock
* **Context window:** 1MB
* **Claude version:** 2.0.76
* **Editor:** [Zed](https://zed.dev) (barely used)
* **Generative AI execution:** Separate terminal window running in plan mode, with safe GitHub and Bash commands pre-allowed
* **CLI tooling:** [GitHub CLI](https://cli.github.com) and other [macos-tools](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/macos-tools/commands/install-macos-tools.md)
* **Claude Marketplace**: [cressie176-claude-marketplace](https://github.com/cressie176/cressie176-claude-marketplace)
* **Node.js Templates:** [node-templates](https://github.com/cressie176/node-templates)
* **Stories:** [shorty/issues](https://github.com/cressie176/shorty/issues)

### Method 1: Prompt Bootstrapping, Implementation Notes and Manual Accepts (Abandoned)
The initial approach was to use pre-written stories, marketplace skills and interactive prompts to fully implement the URL Shortening service. The intention was to encode structure, constraints, and best practices entirely through instructions. Unfortunately, this proved unreliable, particularly while the codebase was in infancy. Even when instructions were repeated and made increasingly explicit, Claude would occasionally ignore them or drift away from the intended structure. Continuing in this direction wasted both time and tokens. I needed a way to bootstrap the application without Claude.

### Method 2: Template Bootstrapping, Implementation Notes and Manual Accepts
For my second attempt, I still worked from pre-written stories, marketplace skills and interactive prompts, but my infrastructure story instructed Claude to bootstrap the service from a custom [GitHub template repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository). A base service template establishes the core structure, with additional templates layered on top for concerns such as PostgreSQL or other infrastructure. This allows common practices to be shared while still supporting different combinations.

Traditional automation struggles here. As the number of layers increases, reliably merging templates becomes difficult, particularly where cross-cutting concerns are involved. This is where Generative AI proved useful. Each layer includes a "Wiring.md" file describing how it should be integrated into the base. Claude can read and apply these instructions in a way that would be awkward to achieve with scripts. The bootstrap process also made it possible to provide Claude.md files in the templates, but this introduced merge problems as templates were combined. Using [rules](https://code.claude.com/docs/en/memory) proved more effective, as each template can include its own rules separately.

Even with templates and rules in place, Generative AI still required technical direction at the story level. Each story therefore includes explicit Implementation Notes recommending an approach, along with reminders to review the relevant rules and skills. These could have been entered interactively, but this would have made the experiment less repeatable and transparent. Templates capture structural decisions, rules capture behavioural constraints, and stories capture local trade-offs and more nuanced technical decisions. Together, they allow experience to be shared through the artefact itself, rather than through constant explanation or review.

Claude was instructed to install the required templates before implementing any functional stories. With that foundation in place, apart from the occasional and minor course correction, a single prompt was all that was required.
```
╭─── Claude Code v2.0.76 ────────────────────────────────────────────────────────────────────────────╮
│                                                    │ Tips for getting started                      │
│                    Welcome back!                   │ Run /init to create a CLAUDE.md file          │
│                                                    │ ──────────────────────────────────────────────│
│                    * ▗ ▗   ▖ ▖ *                   │ Recent activity                               │
│                   *             *                  │ No recent activity                            │
│                    *   ▘▘ ▝▝   *                   │                                               │
│                                                    │                                               │
│ arn:aws:bedrock:eu-west-1:808… · API Usage Billing │                                               │
│          ~/Development/cressie176/shorty           │                                               │
╰────────────────────────────────────────────────────────────────────────────────────────────────────╯

  /model to try Opus 4.5. Note: you may need to request access from your cloud provider

──────────────────────────────────────────────────────────────────────────────────────────────────────
> Implement the URL Shortener epic https://github.com/cressie176/shorty/issues/1 one story at a time 
──────────────────────────────────────────────────────────────────────────────────────────────────────
  ? for shortcuts
```
One deliberate adjustment was made around test-driven development. I allowed Claude to generate tests and production code for a story in a single pass, rather than enforcing a strict red, green, refactor cycle. My assumption was that the model does not benefit from incremental test feedback in the same way a human does, although this remains an open question.

Almost all of my interaction during this phase took place using Claude via the terminal. I reviewed diffs and observed Claude’s workflow there, intervening only when necessary. I deliberately avoided continuous editing in an IDE. Instead, I switched to zed only once Claude had completed each story, in order to review the changes as a coherent logical unit and to scan through the tests.

I reran the experiment multiple times from the same starting point and received approximately similar results.

### Results

The code can be found here: [shorty](https://github.com/cressie176/shorty/)

| Story | Description                    | Status | Time  | Interventions | Commit  |
|-------|--------------------------------|--------|-------|---------------|---------|
| 1     | Project Initialisation         | ✅     | 00:08 | 0             | [2f37d74](https://github.com/cressie176/shorty/commit/2f37d746d84039a2f58e239fa5bae90760db194b) |
| 2     | Shorten URL                    | ✅     | 00:21 | 7             | [b83c51f](https://github.com/cressie176/shorty/commit/2f37d746d84039a2f58e239fa5bae90760db194b) |
| 3     | Get URL                        | ✅     | 00:10 | 4             | [ea2f837](https://github.com/cressie176/shorty/commit/ea2f83796b255a7c1130ae4baa92ac097deba1e0) |
| 4     | URL Redirection                | ✅     | 00:07 | 3             | [9ebc8e3](https://github.com/cressie176/shorty/commit/9ebc8e37754125c7f6d5feb7c464643a1d24e232) |
| 5     | Handle Key Collisions          | ✅     | 00:10 | 3             | [9da1d50](https://github.com/cressie176/shorty/commit/9da1d50236d8eae8bd378246d7acd14294859d22) |
| 6     | Expire Redirects               | ✅     | 00:06 | 0             | [aafa1ba](https://github.com/cressie176/shorty/commit/aafa1ba1d980758b303e2749df01f711e73f69eb) |
| 7     | Delete Expired Redirects       | ✅     | 00:03 | 0             | [6419983](https://github.com/cressie176/shorty/commit/64199839424353f7bd002400afbb28631326607d) |
| 8     | Schedule Database Maintenance  | ✅     | 00:02 | 0             | [4ed2837](https://github.com/cressie176/shorty/commit/4ed283719e72528998c7d6cccf3ffdafffae5339) |
|       |                                |        | 01:07 | 17            |         |

#### Tool Rejections (deduped)
1. Don't duplicate config in tests
2. Prefer single-line if statements
3. Don't use try-catch for testing errors, use throws/rejects
4. Pass the full redirect config not just the key
5. Use object parameters in constructors
6. Don't be lazy with assertions (use eq not ok/match when you know the full string)
7. Inject the full error message into the JSX template
8. Suppress expected error logs in tests
9. Destructure { rows } instead of result.rows

Using this approach, Claude correctly implemented the URL shortener service in one hour and seven minutes, with minimal intervention or further prompting. The architectural drift and disobedience seen earlier largely disappeared once the environment was properly bootstrapped. 

There were some minor style problems. Claude is overly fond of blank lines, and despite it being mentioned in the implementation plan, it missed that an object’s toJSON() method will be called automatically by Hono’s JSON serialiser. Slightly more serious was that Claude omitted running database migrations from each integration test. The tests still passed because migrations were run from the template Postgres.test.ts, but this may not have run first, potentially causing intermittent failures. I have since updated the node-pg template to run migrations automatically from within the Postgres class to avoid this in future. Overall, these are minor niggles, and the code satisfied my goals of minimal operational debt and cleanliness.

I estimate it would have taken me 1 to 2 working days to produce an equivalent codebase from the same templates without AI. This would suggest that Claude achieved an 8x to 16x improvement. However, the implementation time does not reflect the full cost. These results required repeated iteration on both the stories and the skills. Significant effort went into refining story structure, clarifying implementation notes, and adjusting skills so that Claude behaved consistently.

### Method 3: Templated Bootstraping, No Implementation Notes and Automatic Accepts

As another experiment, I tried a deliberately unattended approach without the benefit of implementation notes. The code can be found here: [claude-unattended](https://github.com/cressie176/shorty/tree/claude-unattended)
<br/>

```
╭─── Claude Code v2.0.76 ────────────────────────────────────────────────────────────────────────────╮
│                                                    │ Tips for getting started                      │
│                    Welcome back!                   │ Run /init to create a CLAUDE.md file          │
│                                                    │ ──────────────────────────────────────────────│
│                    * ▗ ▗   ▖ ▖ *                   │ Recent activity                               │
│                   *             *                  │ No recent activity                            │
│                    *   ▘▘ ▝▝   *                   │                                               │
│                                                    │                                               │
│ arn:aws:bedrock:eu-west-1:808… · API Usage Billing │                                               │
│          ~/Development/cressie176/shorty           │                                               │
╰────────────────────────────────────────────────────────────────────────────────────────────────────╯

  /model to try Opus 4.5. Note: you may need to request access from your cloud provider

──────────────────────────────────────────────────────────────────────────────────────────────────────
> I want you to implement https://github.com/cressie176/shorty/issues/1 story by story.
  I want you to completely ignore the implementation notes within the GitHub issues for everyting
  EXCEPT the Project Initialisation story. You MUST NOT even read them otherwise they will influence 
  you. To avoid reading them pipe the output from the gh issue command into something to remove 
  everything after the "Implementation Notes" title so that it is completely unavailable to you. 
  When you think a story is done, build, lint, test and commit.
──────────────────────────────────────────────────────────────────────────────────────────────────────
  ? for shortcuts
```

### Results

Claude completed all eight stories in 30 minutes and 34 seconds. On the surface, this looks impressive. In reality the code was a hot mess.
<br/>
<br/>

| Area     | Metric                     | Method 2 (Guided) | Method 3 (Unattended) |
|----------|----------------------------|-------------------|------------------------|
| Domain   | Files                      | 1                 | 1                      |
|          | TypeScript lines            | 55                | 6                      |
|          | SQL lines                   | 0                 | 0                      |
|          | TSX lines                   | 0                 | 0                      |
|          | Comments                    | 0                 | 0                      |
| Services | Files                      | 4                 | 4                      |
|          | TypeScript lines            | 47                | 199                    |
|          | SQL lines                   | 7                 | 0                      |
|          | TSX lines                   | 0                 | 0                      |
|          | Comments                    | 0                 | 7                      |
| Routes   | Files                      | 3                 | 3                      |
|          | TypeScript lines            | 31                | 66                     |
|          | TSX lines                   | 18                | 0                      |
|          | SQL lines                   | 0                 | 0                      |
|          | Comments                    | 0                 | 0                      |
| Overall  | Total files                | 8                 | 8                      |
|          | Total comments              | 0                 | 7                      |
|          | **Total lines (excl blanks)**   | **158**               | **271**                    |


The guided version produced 158 lines of code with no comments. The unattended version produced 271 lines with seven (pointless) comments. **That's 72% more code and 104% more TypeScript for the same behaviour!**.

Furthermore, the unattended implementation introduced substantial technical and operational debt:

* Unnecesary new services for key generation and expiry management.
* Redirects retrieved from the API were not expired.
* Consecutive database queries were performed using separate client calls, so they were not part of the same transaction, in disregard of the [Unit of Work](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/typescript-service-cookbook/skills/typescript-service-cookbook/SKILL.md#unit-of-work-pattern-with-asynclocalstorage) pattern.
* Errors polluted the logs during test runs in disregard of the [Suppress Expected Error in Logs](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/typescript-tdd-cookbook/skills/typescript-tdd-cookbook/SKILL.md#suppressing-expected-error-logs) guidance.
* It implemented an explicit rude word filter rather than removing vowels, increasing maintenance burden.
* setInterval was used for the maintenance tasks without unref(), which can prevent the process from exiting and does not scale horizontally.
* Functions were far larger, with SQL and JSX inlined making the code harder to follow, in disregard of the [Function Design](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/typescript-clean-code-cookbook/skills/typescript-clean-code-cookbook/SKILL.md#function-design) guidance.
* PostgreSQL was used poorly, with application code compensating for weak queries, e.g.
  - Expiry logic was checked in application code rather than expressed directly in a query.
  - Duplicate urls were inserted rather than being upserted.
  - Scheduling was done in application code in disgregard of the [pg_cron](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/postgresql-cookbook/skills/postgresql-cookbook/SKILL.md#scheduled-deletion-of-old-records-with-pg_cron) guidance.
* The codebase was littered with low value comments that narrated rather than clarified, in disregard of the [Comments](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/typescript-clean-code-cookbook/skills/typescript-clean-code-cookbook/SKILL.md#comments) guidance.
* Tests used Monkey patching instead of dependency injection to fake behaviour.
* Fetch was used in tests directly increasing duplication, reducing readability, making them brittle. and in disregard of the [Test Client](https://github.com/cressie176/cressie176-claude-marketplace/blob/main/plugins/typescript-tdd-cookbook/skills/typescript-tdd-cookbook/SKILL.md#test-client-pattern) pattern.

## Conclusion

**Method 1 (Prompt Bootstrapping, Implementation Notes, and Manual Accepts)** showed that Claude is ineffective when asked to bootstrap a non-trivial system from scratch using prompts alone. Even with detailed stories and marketplace skills, progress was slow and fragile, with frequent drift making the process an uphill and unnecessary battle. The results suggest that Claude is far more productive when working against an existing codebase or concrete structure, and that it appears to weight existing artefacts more heavily than abstract guidance from prompts or skills. This limitation motivated the move to a templated bootstrap approach.

**Method 2 (Template Bootstrapping, Implementation Notes, and Manual Accepts)** demonstrates that Claude can produce high-quality code far more rapidly than even an experienced software engineer, but only when projects are already established or properly bootstrapped, and the agent is guided by effective prompts, whether through marketplace skils, embedded directly in user stories or provided interactively. While marketplace skills represent a long-term investment that can be reused, stories do not. Incorporating detailed implementation notes into each story was necessary for this experiment, but it also made those stories brittle. Real world success will therefore depend either on becoming very good at writing implementation notes up front, or on the engineers driving Claude being skilled enough to provide sufficiently strong just-in-time guidance. Where that is the case, stories that previously took days can be delivered in hours, shifting the primary bottleneck from implementation to story writing. It may be that AI-empowered teams will need to be significantly smaller.

It is also important to put these productivity gains into context. Software engineers do not spend anything close to 100% of their time writing code. Design discussions, team ceremonies, research, troubleshooting, supporting others, administrative work, and training all consume significant portions of a typical working day. Even a dramatic improvement in coding effectiveness therefore does not translate directly into an equivalent improvement in overall productivity.

In contrast, **Method 3 (Template Bootstrapping, No Implementation Notes, and Automatic Accepts)** demonstrates that Claude is not yet something that can be left to operate unattended while still producing consistently good outcomes. Without strong constraints, it reliably drifts towards verbosity, duplication, and accidental complexity, even when the resulting system is functionally correct. That gap between apparent success and long-term maintainability likely explains much of the current scepticism and pushback.

After all of this, I am still left pondering the following open questions:

1. Were my goals the right ones (particularly my need for Clean Code)?
2. Was my method the most effective way to use Generative AI, and specifically Claude Code?
3. Do other goals matter more in different contexts?

If you are getting better results from vibe coding, what are you optimising for, and how does your approach support that? If you are getting poor results how does your approach differ from mine? I'd love to know.


