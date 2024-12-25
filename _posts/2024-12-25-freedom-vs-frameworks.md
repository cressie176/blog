---
title: "Freedom vs. Frameworks: Striking the Balance"
date: 2024-12-25
tags:
- Autonomy
- Control
- Engineering
- Leadership
- Management
- Tech Radar
---
In his book *Drive*, Dan Pink highlights autonomy as one of the critical motivators for high-performing teams. Autonomy empowers engineers to innovate, solve problems creatively, and own their work. But with great autonomy comes great responsibility — and sometimes, significant challenges. One such challenge is the tendency for engineers to gravitate towards shiny new toys, often prioritising popularity over suitability. I explored this phenomenon in my blog post ["Prisma and the Naivety of Crowds"](https://www.stephen-cresswell.com/2024/04/17/prisma-and-the-naivety-of-crowds.html), where I discussed how crowdsourcing trends can sometimes overshadow thoughtful, context-specific decision-making.

Guidelines are necessary to strike a balance between autonomy and consistency, ensuring that choices align with organisational goals. But guidelines alone are insufficient. As Jeff Bezos famously said, "Good intentions don't work. Good mechanisms do." To address this, I'm excited to introduce **eslint-plugin-tech-radar**, a robust mechanism to help engineering teams enforce dependency guidelines at scale.

### Introducing eslint-plugin-tech-radar?

A traditional [Tech Radar](https://www.thoughtworks.com/radar/byor) provides a visual framework for evaluating tools and technologies based on maturity and strategic alignment. However, it doesn’t prevent engineers from inadvertently or deliberately installing prohibited modules. Mechanisms like private npm registries or post-installation scans have significant drawbacks—blocking transitive dependencies or acting too late in the pipeline.

**eslint-plugin-tech-radar** solves these problems by integrating directly into your development workflow. It provides:

- **Proactive validation:** Dependencies are checked against your organisation’s Tech Radar during development, pre-commit, or CI/CD stages.
- **Shared configuration:** Centralised rules ensure consistent enforcement across repositories.
- **Flexibility:** Teams can override or adapt rules on a per-repository basis using familiar ESLint escape hatches.
- **Version tracking:** A built-in “latest” rule ensures your shared configuration stays up to date.

By leveraging eslint-plugin-tech-radar, teams can adhere to established guidelines without sacrificing agility.

### Getting Started

#### Step 1: Define Your Tech Radar

Start by building your Tech Radar. For example:

```csv
name,ring,quadrant,isNew,description
prisma,hold,backend,FALSE,Persistence
winston,hold,backend,FALSE,Logging
bunyan,hold,backend,FALSE,Logging
@pgtyped/query,assess,TRUE,Persistence
orchid-orm,trial,backend,FALSE,Persistence
pino,adopt,backend,FALSE,Logging
sequelize,adopt,backend,FALSE,Persistence
```

Export this CSV as a JSON configuration:

```bash
npx --package eslint-plugin-tech-radar -- export-tech-radar \
  --input radar.csv \
  --documentation https://github.com/your-organisation/tech-radar \
  --output radar.json
```

#### Step 2: Create a Shared ESLint Configuration

Create a shared ESLint configuration that uses your Tech Radar:

```json
{
  "extends": ["eslint:recommended"],
  "plugins": ["tech-radar"],
  "rules": {
    "tech-radar/adherence": [
      "error",
      {
        "hold": ["prisma", "winston", "bunyan"],
        "assess": ["@pgtyped/query"],
        "trial": ["orchid-orm"],
        "adopt": ["pino", "sequelize"],
        "ignore": [],
        "documentation": "https://github.com/your-organisation/tech-radar"
      }
    ],
    "tech-radar/latest": [
      "error",
      {
        "packages": ["eslint-config-your-organisation"]
      }
    ]
  }
}
```

#### Step 3: Enforce Rules Across Repositories

Include the shared configuration in your application’s ESLint setup:

```json
{
  "extends": ["eslint-config-your-organisation"]
}
```

Run ESLint as part of your CI/CD pipeline or pre-commit hooks to ensure compliance.

### Real-World Examples

If your `package.json` includes a dependency on `prisma` (a "hold" package), ESLint will flag it:

```bash
> eslint .

~/your-application/package.json
  1:1  error  Package 'prisma' is discouraged. See https://github.com/your-organisation/tech-radar for more details  tech-radar/adherence

✖ 1 problem (1 error, 0 warnings)
```

If your shared configuration is outdated, the “latest” rule will catch it:

```bash
> eslint .

~/your-application/package.json
  1:1  error  Package 'eslint-config-your-organisation' must be version 1.0.2.  tech-radar/latest

✖ 1 problem (1 error, 0 warnings)
```

### Additional Tips

#### Block Installations with npm Scripts

To prevent undesirable dependencies from being installed, run ESLint during `npm install`:

```json
{
  "scripts": {
    "preinstall": "eslint ."
  }
}
```

#### Encourage Healthy Discussion

Changes to the Tech Radar should be accompanied by documented discussions in pull requests or issues. This fosters transparency and ensures alignment.

### Conclusion

Autonomy is essential, but so is alignment. eslint-plugin-tech-radar bridges the gap, providing a scalable mechanism for enforcing dependency guidelines. By embedding this tool in your workflow, you can maintain the freedom engineers need to innovate while ensuring their choices align with organisational objectives.

Ready to give it a try? Check out the [GitHub repository](https://github.com/acuminous/eslint-plugin-tech-radar) for more details and start building a better, more consistent codebase today.

----
<br/>

#### Recommended Reading
- [Drive](https://www.danpink.com/books/drive/)
