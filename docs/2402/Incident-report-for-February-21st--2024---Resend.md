<!--yml
category: 未分类
date: 2024-05-29 13:19:49
-->

# Incident report for February 21st, 2024 · Resend

> 来源：[https://resend.com/blog/incident-report-for-february-21-2024](https://resend.com/blog/incident-report-for-february-21-2024)

## Summary (TL;DR)

On February 21st, 2024, Resend experienced an [outage](https://resend-status.com/incident/329826) that affected all users due to a database migration that went wrong. This prevented users from using the API (including sending emails) and accessing the dashboard from 05:01 to 17:05 UTC (about 12 hours).

The database migration accidentally deleted data from production servers. We immediately began the restoration process from a backup, which completed 6 hours later. Unfortunately, once it finished, we found that it failed to restore the data, so we had to start the restoration process a second time with a different backup.

During this time, no API requests were being accepted and no data being stored. For data created prior to the migration, there was 5 minute of data loss from when the migration started and the database went offline from 04:50:00 to 04:56:27 UTC. We are currently working on re-populating the data from this 5-minute window.

We sincerely apologize for the impact and inconvenience caused by this outage. We place immense importance on reliability, but this week, we fell short of our commitment to you all. It is clear that we have a long way to go in becoming an industry-leading infrastructure provider, but in learning from this incident, we will improve our operations and tooling to avoid outages like this in the future, whatever the cause.

## Timeline

*All times are in Coordinated Universal Time (UTC)*

**February 21st, 2024**

*   04:56: Database migration started
*   04:57: Noticed tables being dropped from the production database
*   05:01: Began restoring the database from a backup
*   05:02: Posted on [status page](https://resend-status.com/incident/329826), updating every 30-60 minutes until resolution
*   11:02: First restoration process completed
*   11:03: Realized the first backup failed and started to investigate
*   11:33: Found that the backup failed due to a wrong selection of the backup timestamp
*   11:48: Increased compute to speed up the restoration process - updated database memory from 128GB to 256GB and CPU from 32-core ARM to 64-core ARM
*   12:05: Began restoring the database from an older backup
*   17:01: Second restoration process completed
*   17:02: API began receiving requests
*   17:05: Dashboard was accessible again, and incident was resolved

## What happened

While building a feature, we performed a database migration command locally, but it incorrectly pointed to the production environment instead, which dropped all tables in production.

The first attempt to restore the database took 6 hours but failed due to a wrong selection of the backup timestamp. The second attempt to restore took an extra 5 hours and succeeded, bringing all data back besides a 5-minute window of data loss.

## Impact

All users were unable to send emails, use the API, or access the Resend dashboard for 12 hours from 05:01 to 17:05 UTC.

For data created prior to the migration, there was 5 minutes of data loss from when the migration started and the database went offline from 04:50:00 to 04:56:27 UTC.

## Next steps and improvements

*   Re-populate data from the 5-minute window of data loss.

*   No accessible user role should have write privileges on the production database.

*   Improve local development to reduce risks related to database migrations.

*   Create redundancy to preserve sending function even during a database outage.

*   Increase cadence for disaster recovery tests.

*   Implement incident banner on Resend dashboard to inform users quickly.

To our customers, we are deeply sorry that this incident occurred and that it prevented you from delivering your mission-critical emails. We know that actions speak louder than words, so we will continue to learn, grow, and improve, starting by implementing the improvements listed above.