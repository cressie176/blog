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

In the fast-paced world of software engineering, missing a critical renewal or update can cause significant disruptions. Consider a scenario I experienced recently: the secret key for a third-party payment provider expired after three years. Since the provider did not send reminder emails, and the individual who created the key forgot to set a reminder, the company lost its ability to process payments, leading to a scramble to restore operations.

Another example involves Active Directory authentication tokens. These tokens have a maximum two-year expiry and provide only a small orange triangle as a warning. Such subtle cues can easily be overlooked, leading to avoidable downtime. These incidents reveal the necessity of a robust and reliable reminder system.

Many organisations attempt to manage reminders through calendar applications like Outlook. However, these platforms are not designed for managing events scheduled far into the future, and if the creator of a reminder leaves the organisation, their reminders may be lost or become read-only.

Hosted systems that provide automated reminders are another option, but they come with their own challenges. Often, reminders are not a first class feature of such systems and the automation required is complex. Furthermore, they may have a per user licensing model, making them prohibively expensive for use by the entire engineering team. Finally, these services may not be available years into the future, and an export feature may not be provided.

Ultimately, you cannot rely on third parties to notify you about critical events, on engineers to set reminders, or on organisations to provide systems that make it convenient to do so.

### The Case for Reminders as Code

Reminders as code offer an open, convenient and long term approach solution to this problem. By storing reminders in a text file and committing them to a source code repository, you gain the following advantages:

- **Transparency**: The reminders are transparent, versioned, and provided good source control management practices are in place, will be accompanied by a meaningful commit message.

- **Automation**: All good issue tracking systems expose an API for programmatically creating issues, so with a suitable script, reminders can automatically create tickets in an issue tracking system at the appropriate time.

- **Notification**: Issue tracking systems typically have a wide range of user configurable notification mechanisms, ensuring the reminders do not go unmissed.

- **Longevity**: Because reminders are stored as code, they can easily be migrated if you migrate to a new issue tracking system.

- **Scalability**: This approach scales across teams and projects without relying on individual ownership or third-party platforms.

- **Accessibility**: Anyone with access to the source control system and issue tracking system can manage and receive notification of reminders.

### Introducing Knuff

[Knuff](github.com/cressie176/knuff) is an open-source tool designed to implement reminders as code. Reminders are specified as JSON or YAML and stored in plain text. A script that processes the reminders file, identifies due reminders, and posts them to an issue tracking system (e.g., GitHub) via a 'driver'. The script must be run by a scheduler, such as that provided by GitHub Actions.

#### Example Reminders File

```yaml
  # A unique id used to duplicate checking
- id: 'update-contentful-api-key'

  # Optional description that can be useful to track the background behind the reminder
  description: |
    The Contentful API key expires yearly.

  #  Schedule a single reminder for 1st July 2025 (See RRULE / RFC5545 format)
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
Reliable reminders are often a critical gap in the engineering process and IT operations. By leveraging tools like Knuff to managing reminders as code, organisations will enjoy greater stability and suffer fewer nasty surprises.
