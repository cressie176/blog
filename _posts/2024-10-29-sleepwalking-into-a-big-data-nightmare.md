---
title: "Sleepwalking into a Big Data Nightmare"
date: 2024-10-29
tags:
- Big Data
- Snowflake
- AI
- Domain Modelling
- ETL
---
Sourcing big data within organisations has become routine, with operational data stores (or their replicas) feeding data lakes like Snowflake or Redshift via expensive Change Data Capture (CDC) tools. These lakes become the foundation for reporting and increasingly, Artificial Intelligence. At first glance, this appears efficient and scalable. But there is a lurking problem: tight coupling between the operational data store and downstream analytics.

#### The Hidden Danger of Tight Coupling

Operational data stores are not static; they evolve rapidly under active development. Features are added, models are refined, and, in Agile environments, changes are iterative rather than pre-planned. By sourcing data directly from these stores, you tether your reporting and analytics to a system in flux. This creates several critical issues:

1. **Constraints on Engineering**: Changes to the operational data store must accommodate downstream dependencies. Engineers may become bogged down by these constraints, slowing down the development of new features.

2. **Broken Integrations**: Without automated tests, changes to the operational store often break models and reports. The resulting chaos is compounded when consumers, often financial and commercial teams, rely on these reports to make decisions.

3. **Corporate Fallout**: When reports fail, the fallout is rarely trivial. Commercial stakeholders typically wield significant organisational power. Engineering teams find themselves under pressure to implement ad-hoc fixes and facing new layers of process bureaucracy, introducing waste and blocking progress.

#### A Cautionary Tale: Job Adverts at Tes Global

A clear example of this dynamic comes from my experience with Tes Global, the world's largest teacher network. At Tes, I led the team developing a jobs board where the lifecycle of a job advert was anything but simple, transitioning between implicit and explicit states of 'draft', 'published', 'live', 'new', 'ending soon', 'ended', 'closed' and 'deleted'. Here’s how they worked:

- A job advert began as a 'draft', and was 'published' by the advert's creator.
- Published job adverts automatically becoame 'live' after the advert start date.
- Job adverts could be soft 'deleted' before going live, but after that, they could only be cancelled since there were associated viewing stats, applications, and potentially charges.
- Job adverts were 'new' for the first two days, and 'ending soon' for the final two, after which they transitioned to 'ended'.
- Finally, the job advert was 'closed' once the application close date had passed, indicating that no further applications could be submitted.

The complexity was compounded by the absence of an explicitly persisted state for job adverts. This decision was partly because some state change triggers were time-based, making explicit persistence challenging. Instead, the state was derived in application code from various flags and dates. However, we later faced the somewhat unenviable task of reverse engineering the rules and duplicating them in the data lake, with all the reliability issues that entailed.

#### A Better Approach

A better solution would have been to explicitly model the job advert’s state and export that refined model to the data lake. Real-time events could have fed the data lake while simultaneously unlocking the ability to coordinate and propagate changes across our microservice architecture. Alternatively, data changes (deltas) could have been exported from the database at regular intervals and uploaded to a cloud-based storage service in a text-based, machine-readable format, such as [Apache Avro](https://avro.apache.org/). Both these approaches would also have allowed us to write automated tests to catch breaking schema changes. Although this demands more upfront thought, patience, and effort, it avoids the pitfalls of tight coupling and fragile integrations.

#### Why Fast and Loose Fails in Big Data

Organisations often opt for the fast-and-loose approach because they believe it offers agility. Every Data team I've worked with has request we "just give them everything", because they are unsure of which reports will be required in the future. This mindset is pervasive but flawed. Poorly modelled data leads to brittle systems, wasted engineering hours, and unreliable analytics.

In contrast, slow and steady wins the race when it comes to data modelling, not least because we are on the brink of a Machine Learning (ML) and Generative AI revolution. However, these technologies remain impotent without high-quality, well-structured data to train on. The upfront investment in thoughtful modelling pays exponential dividends in long-term reliability and usability.

#### Final Thoughts

The allure of big data has led many organisations to sleepwalk into an architectural nightmare. Directly coupling reporting to operational data stores may seem expedient but ultimately causes more harm than good. By prioritising deliberate schema design, testing, and simple integration, organisations can avoid these pitfalls and unlock the full potential of their data. The choice is simple: sleepwalk into chaos or step deliberately into clarity.

