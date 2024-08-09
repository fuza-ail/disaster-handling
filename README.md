# Study Case ( Disaster Handling )

On February 21st, 2024, experienced an outage that affected all users due to a database migration that went wrong. This prevented users from using the API (including sending emails) and accessing the dashboard from 05:01 to 17:05 UTC (about 12 hours).

The database migration accidentally deleted data from production servers. We immediately began the restoration process from a backup, which completed 6 hours later. Unfortunately, once it finished, we found that it failed to restore the data, so we had to start the restoration process a second time with a different backup.

During this time, no API requests were being accepted and no data being stored. For data created prior to the migration, there was 5 minute of data loss from when the migration started and the database went offline from 04:50:00 to 04:56:27 UTC. We are currently working on re-populating the data from this 5-minute window.

Timeline:
04:56: Database migration started
04:57: Noticed tables being dropped from the production database
05:01: Began restoring the database from a backup
05:02: Posted on status page, updating every 30-60 minutes until resolution
11:02: First restoration process completed
11:03: Realized the first backup failed and started to investigate
11:33: Found that the backup failed due to a wrong selection of the backup timestamp

What happened:
While building a feature, we performed a database migration command locally, but it incorrectly pointed to the production environment instead, which dropped all tables in production.

Impact:
All users were unable to send emails, use the API, or access the dashboard for 12 hours from 05:01 to 17:05 UTC.

For data created prior to the migration, there was 5 minutes of data loss from when the migration started and the database went offline from 04:50:00 to 04:56:27 UTC.

---

## Incident Sumamry

Between the 05:01 to 17:05 UTC (about 12 hours) all users were not able to use API, accessing dashboard, as well as sending email.

The event was triggerred by accidental deletion of production database during database migration process.

## Timeline Events

- **04:56 UTC**: Database migration started.
- **04:57 UTC**: Noticed tables being dropped from the production database.
- **05:01 UTC**: Initiated the restoration process from a backup.
- **05:02 UTC**: Posted the first update on the status page, updating every 30-60 minutes.
- **11:02 UTC**: First restoration process completed.
- **11:03 UTC**: Realized the first backup failed and started to investigate
- **11:33 UTC**: Found that the backup failed due to a wrong selection of the backup timestamp
- **17:05 UTC**: Successfully restored the database.

## Fault

During the development of a new feature, a database migration command was executed locally. However, the command pointed to the production environment instead of the local development environment. This error resulted in the deletion of all tables in the production database.

## Impact

All users were unable to send emails, use the API, or access the dashboard for 12 hours from 05:01 to 17:05 UTC.

For data created prior to the migration, there was 5 minutes of data loss from when the migration started and the database went offline from 04:50:00 to 04:56:27 UTC.

## Detection

## Response

The immediate restoration process was conducted. This took 6 hours to complete but unfortunately the process was failed. Investigation of the failed process is conducted and found out that the backup timestamp was incorrectly selected. Continue restore the database using the correct timestamp.

## Recovery

The database was restored using the correct timestamp.

## Root Cause Identification

- The users can access API, database, email because the production database was deleted.
- The production database was deleted because the local migration process was pointed to the production environment.
- \*The migration mistakenly pointed to production because lacking of centralization and config checking as well as lacking guard of production env.
- \*The centralization problem because lacking of planning and review.
- \*Lacking of planning could be because the tight deadline window

## Corrective Action

- Need to add a guard so that production env can’t be access freely from local
- Add more step or review process if the changes is in production env
- Need to implement more automation to migration and database back up so we can minimize human error.
- Create SOP for all the crucial process
