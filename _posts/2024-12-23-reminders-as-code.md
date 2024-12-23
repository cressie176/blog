---
title: "Reminders as Code"
date: 2024-04-22
tags:
- Reminders
- Calendar
- RRule
- rfc5545
---
Reminders as Code: An open, convenient and long term approach to managing critical notifications

In the fast-paced world of software engineering, missing a critical renewal or update can cause significant disruption. Consider a scenario I experienced recently: the secret key for a third-party payment provider expired after three years. Since the provider did not send a warning, and the individual who created the key forgot to set a reminder, the company lost its ability to process payments, leading to a brief but avoidable outage and a scramble to restore operations. Another example involves Active Directory client secrets. These tokens have a maximum two-year expiry and provide only a small orange triangle as a warning. Such subtle cues can easily be overlooked, leading to avoidable downtime. These incidents demonstrate the necessity of a robust and reliable reminder system.

Many organisations attempt to manage reminders through software calendars like Outlook. However, the user interfaces for such applications are not designed for managing distant events, and if the creator of a reminder leaves the organisation, it may be lost or become read-only.

SAAS platforms that provide automated reminders are another option, but they come with their own challenges. Often, reminders are not a first class features and the automation required to set them up is complex. Furthermore, they may have a per user licensing model or fall within the domain of an IT operations team, making them prohibively expensive or inaccessible. Finally, these services may not be available years into the future, and may lack an export feature.

Ultimately, you cannot rely on third parties to notify you about critical events, on engineers to set reminders, or on organisations to provide systems that make it convenient to do so.

### Reminders as Code

Reminders as Code offers an open, convenient and long term solution to this problem. Namely, storing reminders in a text file, committed into to a source code repository. By doing so you gain the following advantages:

- **Transparency**: The reminders are transparent, versioned, and provided good source control management practices are in place, will be accompanied by a meaningful commit message, so their purpose can be understood after the original committer has left.

- **Automation**: All good issue tracking systems expose a programmatic API, so with a suitable script, reminders can be created as issues at the appropriate time.

- **Notification**: Issue tracking systems typically have a wide range of user configurable notification mechanisms, ensuring the reminders do not go unmissed.

- **Longevity**: Because reminders are stored in an open format, they can easily be migrated if you migrate to a new issue tracking system.

- **Scalability**: The approach scales across teams and projects without relying on individual ownership or third-party platforms.

- **Accessibility**: Anyone with access to the source control system and issue tracking system can manage and receive notification of reminders.

### Introducing Knuff

[Knuff](https://github.com/acuminous/knuff) is an open-source reminders as code implementation. Reminders are specified as JSON or YAML and stored in plain text. A script processes the reminders file, identifying those that are due, posting them to the issue tracking system of choice (e.g., GitHub) via a 'driver'. The script must be run by a scheduler (such as that provided by GitHub Actions) on at least a daily basis.

#### Example Reminders File

```yaml
  # A unique id used for duplicate checking
- id: 'update-contentful-api-key'

  # Optional description that can be useful to track the background behind the reminder
  description: |
    The Contentful API key expires yearly.

  # Schedule a single reminder for 1st July 2025 (See RRULE / RFC5545 format)
  schedule: |
    DTSTART;TZID=Europe/London:20250701T080000
    RRULE:FREQ=DAILY;COUNT=1

  # The reminder title
  title: 'Update Contentful API Key'

  # The reminder body
  body: |
    The Contentful API Key expires on the 14th of July 2025.
    - [ ] Regenerate the Contentful API Key
    - [ ] Update AWS Secrets Manager
    - [ ] Redeploy the website
    - [ ] Update the reminder for next year

  # A set of labels / tags
  labels:
    - 'Automated Reminder'

  # A list of repositories to post the reminders to
  repositories:
    - 'acuminous/reminders'
```

#### Example Script

```js
import fs from 'node:fs';
import yaml from 'yaml';
import { Octokit } from '@octokit/rest';
import GitHubDriver from 'knuff-github-driver';
import Knuff from 'knuff/index.js';

const pathToReminders = process.argv[2] || 'reminders.yaml';

const config = {
  repositories: {
    'acuminous/reminders': {
      owner: 'acuminous',
      name: 'reminders',
      driver: 'github',
    },
  },
};

const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });
const drivers = { github: new GitHubDriver(octokit) };
const knuff = new Knuff(config, drivers)
  .on('error', console.error)
  .on('progress', console.log);
const reminders = yaml.parse(fs.readFileSync(pathToReminders, 'utf8'));

knuff.process(reminders).then((stats) => {
  console.log(`Successfully processed ${stats.reminders} reminders`);
}).catch((error) => {
  console.error(error);
  process.exit(1);
});
```

### Example GitHub Action
```
name: Check Reminders

on:
  workflow_dispatch: # Allows manual triggering of the workflow
  schedule:
    - cron: "*/60 * * * *" # Runs every 60 minutes

jobs:
  run-reminder:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install dependencies with npm ci
        run: npm ci

      - name: Run Knuff
        run: node index.js reminders.yaml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```


### Limitations

Knuff’s primary limitation is that it can't force engineers to use it. However, drawing on inspiration from the Kevin Costner movie Field of Dreams: "If you build it, they will come"; Knuff encourages adoption through its simplicity and utility.

A second limitation exists for implementations involving many reminders or repositories. A GitHub Action or Personal Access Token like the one used in the above example may be rate limited, forcing you to register the script as a GitHub App and to implement a more convoluted authorisation flow.

### Conclusion
Reliable reminders are not just a convenience; they are a necessity in ensuring the smooth operation of modern software systems. The consequences of missed notifications, ranging from expired keys to overlooked updates, can lead to downtime, financial loss, and unnecessary fire-fighting. Traditional approaches, such as relying on individual engineers or third-party solutions, have proven inadequate due to their inherent limitations, including ill-suited UI, lack of transparency, dependence on specific platforms, and the risk of obsolescence or inaccessibility.

By adopting the Reminders as Code approach, organisations can address these shortcomings. This method integrates reminders into the development process itself, ensuring they are version-controlled, transparent, and long-lasting. Tools like [Knuff](https://github.com/acuminous/knuff) exemplify how this approach can be operationalised, providing a practical, open-source framework that enables automation, scalability, and accessibility.
