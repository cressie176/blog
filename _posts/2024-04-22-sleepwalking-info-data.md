---
title: "sleepwalking-into-data"
date: 2024-04-22
tags:
- Data
- ETL
---

Big Data is arguably been one of the most successful developments in the last ten years.
What has it enabled?
What will it enable? AI / Machine Learning
Competitive Advantage

There is a trend to extract data directly from source (or a replica) using tools like HVR.
It's an attractive option
It's quick to setup, and efficient to perform. It handles the syncing issue
It means the data analysists get everything, and so can build models, reports and respond to ad-hoc queries not previously requested with minimum delay.

There is a catch. The first is obvious. When building models directly onto an application database, when that application changes, the models and reports will break. If those models and reports become business critical, the reaction will be swift, and reputation damaged. Leading to demands for change control, and bureaucracy. All of which will damage product development. Depending on data directly from an application is integration by database - in stealth mode. Application developers will forget and no tests will warn them. We have learned slowly, and through many years of mistakes that integration by database is an anti-pattern.

The second problem is worse, because it will take longer to become apparent, and when it does, will be harder to rectify. When you integrate by database, you break encapsulation, and consequently distribute logic across multiple systems. Take the example of a jobs board.

A job advert has a start date and and end date. To work out which job adverts are live, the application runs a query `SELECT * FROM job_advert WHERE start_date >= $1 AND end_date <= $1`. Note, that both dates are inclusive. Adverts may also be cancelled. The system in question is poorly designed, and cancels them by setting the end date to 1970-01-01. Our original query can stay the same, but to find out which adverts were cancelled the application runs the query `SELECT * FROM job_advert WHERE end_date = '1970-01-01'`. Adverts may also be curtailed, but since we want to retain the end_date, a second date is added, confusing named 'cancelation_date', which is not inclusive. Now the original query must be `SELECT * FROM job_advert WHERE start_date >= $1 AND end_date <= $1 AND cancelation_date < $1`

When we integrate our analytics systems by database, this logic must be explained replicated exactly. As the application grows, and becomes more complicated, we couple the two systems more and more tightly, making the application harder and harder to change.

Applications aren't perfect, and can't always be easily refactored (legacy systems)

Instead

Export the conclusions, deliberately and consistently

advert_id, start_date, end_date, cancelation_date, curtailment_date

whether as a snapshot or set of deltas.

Mechanisms
Export script maintained and tested by the application developers, in a format agreed by the data engineers. Possibly via a restful endpoint, with a cron uploading avro files to s3,


