## Why Are Experiences Of Vibe Coding So Polarised?

Jason Gorman’s recent [post](https://codemanship.wordpress.com/2025/11/25/the-future-of-software-development-is-software-developers/) on the future of software development is causing quite a stir. The comments on the accompanying Hacker News [discussion](https://news.ycombinator.com/item?id=46424233) span an extraordinary range of experience with vibe coding, particularly when using Claude Code. Some describe dramatic productivity gains and rapid delivery of complex systems. Others report dangerously misleading output, architectural drift, and large amounts of unusable code. Both sides speak with confidence, often dismissing the other as naïve, reckless, or simply doing it wrong. The discussion reflects a broader pattern which has been bothering me for some time.

I have been a professional software engineer for thirty years, and strong disagreements are nothing new. Functional versus object-oriented programming, static versus dynamic typing, competing approaches to testing. What is different this time is the nature of the tool itself. Generative AI does not operate at the level of a language, framework, or architectural paradigm. It operates at the level of intent. In that sense it is a meta tool, unconstrained by the trade-offs that normally explain why a tool works well for some people or some problems, and poorly for others.

It does not matter that Generative AI is unusually non-deterministic. For it to be useful to anyone, it still has to produce desired outcomes with sufficient consistency. That makes the scale and intensity of the reported disagreement surprising. Vastly different experiences are unlikely to be explained by an inherent defect in the tool itself.

Even allowing for bias and differing [personality temperaments](https://keirsey.com/temperament-overview/) (Artisans derive greater pleasure from novelty than Rationals or Guardians) does not adequately explain the gap. The same tool is being described, with equal confidence, as producing dangerous, unmaintainable AI slop on the one hand, and delivering twenty times productivity on the other.

This gap is pushing organisations to make extreme decisions. Some, driven by unrealistic expectations, are rushing to quickly into AI adoption, creating unnecessary anxiety and disruption. Others are avoiding it entirely, missing out on potential benefits and frustrating their internal AI advocates. The gap needs to be understood and closed.

## **What I Want From Generative AI**

One possible explanation for difference in reported experience is that people want different outcomes. If that is the case, then disagreement about Generative AI performance is inevitable. Before comparing tools or techniques, the goals themselves need to be explicit. I want Generative AI to rapidly create applications with the following characteristics:

### 1. Negligible Operational Debt
Operational debt is distinct from technical debt. Technical debt only incurs a cost when change is required and is often an explicit trade-off. Operational debt creates ongoing risk and [unplanned work](https://blog.while-true-do.io/devops-4-types-of-work/) and, in the worst cases, can consume an entire team’s capacity through incidents and urgent remediation. A concrete example of this is GitLab’s 2017 data loss [incident](https://about.gitlab.com/blog/postmortem-of-database-outage-of-january-31/) where operational failures led to the permanent deletion of customer repositories.

### 2. Unparalleled Malleability
[Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/) is malleable. When we write clean code, we make a comparatively small sacrifice now to reserve the ability to change rapidly in the future.
   
The best way to achieve clean code is to write as little code as possible. This is done by creating a good domain model. When the domain model is good, the code vanishes. As Linus Torvalds is [reported](https://read.engineerscodex.com/p/good-programmers-worry-about-data) to have said:

> “Bad programmers worry about the code. Good programmers worry about data structures and their relationships.”

A good domain model requires good encapsulation. Behaviour is contained in one place, colocated with the associated data. Accessors are few, mutators fewer still. Conditional logic is pushed to the boundaries of the application and eliminated internally through polymorphism. The code clearly expresses intent.

I have a nagging suspicion that my attachment to clean code, framed by Scott Adams as [Loserthink](https://en.wikipedia.org/wiki/Loserthink) may be unnecessary in a world of Generative AI. Until that suspicion is proven, I am sticking with it. The risk of a future filled with vast quantities of even less malleable code than we already have is too great.

To achieve these objectives in teams, the AI tooling must behave consistently well. Outcomes cannot depend on individuals repeatedly crafting perfect prompts. Until that point is reached, tooling must support and encourage the development and adoption of best practices. Without this, AI will always be a [damp squib](https://en.wiktionary.org/wiki/damp_squib).

## My Experience Of Vibe Coding (So Far)

### The Good

It is often stated that Generative AI is effective at mechanical or rote tasks, or comparable to the output of a junior engineer, but this is not where it offers the most value. The area where I have found my Generative AI tool of choice, [Claude Code](https://www.claude.com/product/claude-code), to be most effective is where I have strong high-level judgement about what needs to be achieved, but lack the detailed, low-level knowledge to implement it quickly without time, research, and mistakes. In those cases, the limiting factor is not judgement, but execution.

CI/CD pipelines and Docker-related configuration provide good examples. I know what I want a pipeline to do, how the stages should fit together, and what correct behaviour looks like. Implementing that by hand usually involves reading documentation, iterating on syntax, and discovering edge cases through failure. Claude is much faster at cycling through that execution loop than I am, especially after installing the [GitHub CLI](https://cli.github.com) - a genuine game changer!

In summary, Generative AI adds the most value when it operates below my level of judgement but above my knowledge or ability to acquire it. It accelerates low-level execution without being asked to make high-level decisions it is poorly suited to make.

### The Bad

At the same time, Claude, has ignored explicit instructions, made false assumptions, implemented changes that were not requested, and drifted away from the intended structure, particularly early in a codebase when there is less existing context. It has also misdiagnosed problems and disappeared down rabbit holes without ever fixing them, or worse, unilaterally decided that they can be safely ignored.

What stands out is how sensitive the results are to relatively small changes in how the tool is used. Claude is not a compiler. The results are not deterministic. Small differences in context, ordering, or phrasing can lead to materially different outcomes, even when the intent appears unchanged. Overall, the outcomes are still positive, but I have yet to achieve the reported 20x improvement. Either those reports are grossly overstated, or those making them have found ways to circumvent these issues and make Claude perform consistently well. If the latter is true, I want to learn and adopt their methods, but what are they?

## What We Need Is An Experiment

To move this discussion forward, we need something more concrete than confident anecdote. When the same tool is reported to produce both dangerous, unmaintainable systems and dramatic productivity gains, opinion alone cannot tell us whether the difference lies in the tool, the goals, or the way it is being used. The only way to separate those factors is to make the goals explicit and then test, in a controlled way, whether a particular method of using Generative AI can reliably produce outcomes aligned with them.

### Hypothesis

The vast variation of experience comes from how Generative AI is being used, not from a fundamental weakness in the tool itself.

### Aparatus

* **Machine:** MacBook M1 Pro with 32GB RAM
* **Operating system:** macOS 15.6.1 (24G90)
* **Model:** Claude Sonnet 4.5 via AWS Bedrock
* **Context window:** 1MB
* **Claude version:** 2.0.76
* **Editor:** [Zed](https://zed.dev)
* **Generative AI execution:** Separate terminal window running in plan mode, with safe GitHub client and Bash commands allowed
* **CLI tooling:** GitHub CLI and faster command line alternatives
* **Claude Marketplace**: [cressie176-claude-marketplace](https://github.com/cressie176/cressie176-claude-marketplace)
* **Node.JS Templates:** [node-templates](https://github.com/cressie176/node-templates)
* **Stories:** [shorty/issues](https://github.com/cressie176/shorty/issues)

## Method 1: Prompt Bootstrapping (Abandoned)

The initial approach was to use interactive prompts and marketplace skills to bootstrap the project from an empty repository. The intention was to encode structure, constraints, and best practices entirely through instructions. In practice this proved unreliable, particularly while the codebase was in infancy. Even when instructions were repeated and made increasingly explicit, Claude would occasionally ignore them or drift away from the intended structure. Early architectural. Continuing in this direction wasted both time and tokens.

I briefly considered developing a reference repository, which would have provided concrete examples of structure and conventions rather than relying on abstract descriptions. While viable in principle, this approach does not scale in a microservice environment. The number of permutations of infrastructure components (databases, message brokers, etc.) grows rapidly, and maintaining reference repositories for each combination would simply relocate the complexity.

## Method 2: Template Bootstrapping

Instead of prompting, I adopted a templating approach to bootstramp the project. A base service template establishes the core structure, with additional templates layered on top for concerns such as PostgreSQL or other infrastructure. This allows common practices to be shared while still supporting different combinations.

Traditional automation struggles here. As the number of layers increases, reliably merging templates becomes difficult, particularly where cross-cutting concerns are involved. This is where Generative AI proved useful. Each layer includes a "Wiring.md" file describing how it should be integrated into the base. Claude can read and apply these instructions in a way that would be awkward to achieve with scripts. The bootstrap process also made it possible to provide Claude.md files in the templates, but this introduced merge problems as templates were combined. Using [rules](https://code.claude.com/docs/en/memory) proved more effective, as each template can include its own rules separately.

Even with templates and rules in place, Generative AI still required technical direction at the story level. Each story therefore includes explicit Implementation Notes describing constraints, expectations, and trade-offs, along with reminders to review the relevant rules and skills.

The intent is to make best practices explicit, inspectable, and repeatable. Templates capture structural decisions, rules capture behavioural constraints, and stories capture local trade-offs. Together, they allow experience to be shared through the artefact itself, rather than through constant explanation or review.

Claude was instructed to install the required templates before implementing any functional stories. With that foundation in place, a single prompt, "Implement the [URL Shortener epic](https://github.com/cressie176/shorty/issues/1) one story at a time", apart from the occasional and minor course correction, was all that was required.

One deliberate adjustment was made around test-driven development. I allowed Claude to generate tests and production code for a story in a single pass, rather than enforcing a strict red, green, refactor cycle. My assumption was that the model does not benefit from incremental test feedback in the same way a human does, although this remains an open question.

I reran the experiment multiple times from the same starting point and received similar results.

### Results

**Generated Codebase:** [shorty](https://github.com/cressie176/shorty)

Using this approach, Generative AI correctly implemented the URL shortener end to end in under two hours, with minimal intervention or further prompting. The architectural drift and disobedience seen earlier largely disappeared once the environment was properly bootstrapped. The resulting code met the goals I was optimising for and is of a high standard. I estimate it would have taken me 2-3 working days to produce an equivalent codebase working without AI.

However, that elapsed time does not reflect the full cost. Achieving these results required repeated iteration on both the stories and the skills. Significant effort went into refining story structure, clarifying implementation notes, and adjusting skills so that Claude behaved consistently.

### Conclusion

This experiment demonstrates that Claude can produce high quality code more rapidly than even an experienced software engineer, but only when projects are properly bootstrapped and guided by effective prompts in the form of skills and detailed stories. 

While the skills represent a long-term investment that can be reused, the stories do not. My method necessitates embedding implementation notes into each story, making them brittle. The success of the approach depends either on becoming very good at writing stories up front, or on the engineers driving Claude being experienced enough to recognise when stories are flawed.

When stories are wrong or incomplete, someone must stop and rewrite them, along with any dependent stories, some of which may already be in development. In effect, this approach shifts the primary bottleneck from implementation to story writing and maintenance. It may be that my method is only suitable for small teams of experienced engineers.

I still have the following open questions.

1. Were my goals the right ones?
2. Was my method the most effective way to use Generative AI, and specifically Claude Code?
3. Do other goals matter more in different contexts?

If you are getting good results from vibe coding, what are you optimising for, and how does your approach support that? If you are getting poor results how does your approach differ from mine? I'd love to know.
