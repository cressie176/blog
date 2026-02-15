---
title: "More Experiences of Vibe Coding"
date: 2026-02-15
excerpt: "Further experience with Claude has strengthened my view that code quality matters more, not less, in the age of vibe coding. Left unguided, it amplifies duplication and weak abstractions; tightly constrained, it can be impressively precise. Generative AI is an amplifier. It accelerates coherence, and it accelerates mess."
tags:
- Generative AI
- Claude Code
- Clean Code
- Vibe Coding
---

One of the outstanding questions from my [previous post](/2026/01/01/Why-Are-Experiences-of-Vibe-Coding-so-Polarised.html) was whether code quality mattered to AI. I am now more convinced that it does. What I have observed in extended use is this: unless guided carefully, Claude produces more code than necessary, with weaker abstractions and noticeable duplication. As the codebase grows, the problem compounds. A bug fixed in one place introduces a bug somewhere else. Fix that, and you either recreate the original defect or produce a new one. It becomes a kind of Dr. Strange vs Dormammu time loop, where you are trapped in an endless cycle of regression; or, alternatively, a maddening game of whack-a-mole.

<figure style="float: right; margin: 0 0 1em 2em; max-width: 400px;">
  <img src="/images/dormammu.png" alt="Dormammu Time Loop" style="width: 100%;" />
  <figcaption style="text-align: center;">Trapped in an endless cycle of regression</figcaption>
</figure>

The pattern is not surprising. Claude is highly influenced by the existing codebase. If the surface area is large, inconsistent, or structurally weak, the model will amplify those characteristics. Conversely, if the surface area is small, cohesive and internally consistent, the model's output improves markedly.

For clean code, three principles dominate.

**Strong Domain Models**<br/>
The core concepts of the system should be explicit, named, and represented directly in code. Behaviour should live with the concepts it belongs to. When the model is coherent, both humans and AI can extend it predictably. When the domain is implicit or smeared across utilities and controllers, every change becomes guesswork.

**Encapsulation**<br/>
Keep data and behaviour together and expose only meaningful operations that reflect the language of the domain. Do not provide mutators (setters) and avoid casual accessors (getters); every accessor that leaks state invites distributed behaviour, breaking the domain model. AI systems are particularly sensitive to porous boundaries, because they will happily reimplement logic.

**Minimal Conditional Logic**<br/>
All but the briefest branching structures are a code smell. They signal decisions being made too late, missing domain concepts, missing polymorphism, or collapsed responsibilities. Nested conditionals increase cognitive load and expand the probability space the model must reason over. Move decisions back to the caller, which already knows the intent, or replace conditionals with richer, polymorphic domain types. This reduces complexity at the point of execution and keeps behaviour aligned with explicit intent rather than inferred state.

<h2>When Claude got everything right</h2>

That said, it would be misleading to suggest the experience is uniformly negative. Recently I built a small utility application for testing single sign on in a single prompt, and Claude produced exactly what I wanted.

The prompt was:

```
Create a very simple web application for testing authentication with azure.
It should let me sign in, and when I click a button, makes a request to a 
backend service using the bearer token. The backend service is 
../azure-ad-jwt-debugger. Use the msal library as appropriate.
```

And Claude delivered precisely that. Below is the generated interface...

<figure style="text-align: center; margin: 2em 0;">
  <img src="/images/azure-ad-jwt-debugger.png" alt="Azure AD JWT Debugger" style="display: block; margin: 0 auto;" />
  <figcaption>The Azure AD JWT debugger interface, generated in a single prompt</figcaption>
</figure>

It signs in with MSAL, retrieves the token, and calls the backend with the bearer token exactly as requested. No over-engineering. No unnecessary abstractions. Just a simple, correct implementation. You can review the source code [here](https://github.com/cressie176/azure-ad-jwt-debugger-web)

Interestingly, it did not just implement the minimum described in the prompt. It also added:

- A sign out button
- A structured JSON response display panel

Neither of these were explicitly requested. Both were appropriate. Sign out is operationally useful when testing authentication flows. A formatted JSON viewer makes token inspection and backend responses materially easier to reason about. These are pragmatic affordances, not speculative over-engineering.

In conclusion, when the intent is crisp and the domain small, vibe coding works extremely well. When the system grows and architectural trade-offs become material, discipline becomes essential.That is not an argument against generative AI. It is an argument for treating it as a powerful amplifier. Code quality matters more, not less, in the age of generative AI. Until models evolve in ways that reason more structurally about long-term design consequences, sustainable AI-assisted development depends on disciplined architecture. If the underlying design is coherent, it accelerates you. If it is messy, it accelerates the mess.
