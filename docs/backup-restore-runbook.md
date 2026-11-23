# Runbook: Restoring Budgetly from a Backup

Sprint 5, ticket SCRUM-159. For admins only (Zach, Tyler). Read the whole page before you start, and practice on a throwaway project first. The backup design itself is in [data-framework.md](data-framework.md); the commands live in the backend README under "Backups".

## When to use this

| Situation | Use |
|---|---|
| One user deleted something and it is past the 30-day window | The archive restore script (SCRUM-157), not a backup |
| A bad script or migration changed or removed data | This runbook |
| Data was corrupted or lost and the archive can't help | This runbook |

A restore is a last resort. A backup import writes every document in the backup over the matching document in the live database. It does not delete documents that were created later. Anything saved after the backup was taken, and edited since, is replaced by the older version.

## What we have

* A Firestore export of the whole default database every day, at about 3 AM Central, written to the default Storage bucket under `backups/YYYY-MM-DD/`. The last 7 days are kept.
* Manual backups from `npm run backup`, written to `backups/YYYY-MM-DD-manual-HHMM/`. Take one before any risky script or migration.
* Backups hold Firestore only. Uploaded statement files in Storage are not in them.
* Project: `budgetly-backend-nodejs`. Bucket: `budgetly-backend-nodejs.firebasestorage.app` (check Project settings if it differs).

## Before you start

You need your own Google login with the **Cloud Datastore Import Export Admin** role (project owners have it) and the `gcloud` CLI.

```
gcloud auth login
gcloud auth application-default login
gcloud config set project budgetly-backend-nodejs
```

## Step 1: Stop the damage and decide what to restore

1. Stop whatever caused it (a running script, a bad deploy). Tell the other admin.
2. Write down when the problem started. You want the newest backup from **before** that time.
3. Decide the scope: the whole database, or only some collections (for example only `transactions`). A smaller restore overwrites less.

## Step 2: Find the backup

```
gcloud storage ls gs://budgetly-backend-nodejs.firebasestorage.app/backups/
```

Pick a folder dated before the problem. Backup times are UTC, and a daily folder is taken around 3 AM Central. A folder dated today holds yesterday evening's data plus anything up to 3 AM.

## Step 3: Take a safety backup of the live database first

Even a broken database is worth keeping, because the restore overwrites documents.

```
cd budgetly_backend_nodejs/functions
npm run backup -- --project budgetly-backend-nodejs
```

Note the folder name it prints (`...-manual-HHMM`). If the restore goes wrong, this is what you go back to.

## Step 4: Practice on a throwaway project

Never run a restore for the first time on live.

1. Make a free Firebase project (for example `budgetly-restore-test`) with Firestore in the same region.
2. Give that project's Firestore service agent read access to the bucket holding the backup, so it can read across projects:
   ```
   gcloud storage buckets add-iam-policy-binding gs://budgetly-backend-nodejs.firebasestorage.app \
     --member="serviceAccount:service-<TEST_PROJECT_NUMBER>@gcp-sa-firestore.iam.gserviceaccount.com" \
     --role="roles/storage.objectViewer"
   ```
3. Import the backup there:
   ```
   gcloud firestore import gs://budgetly-backend-nodejs.firebasestorage.app/backups/YYYY-MM-DD/ --project budgetly-restore-test
   ```
4. Check the data in the console (Step 6).

Remove the extra access when you are done.

## Step 5: Restore to live

Only after the practice worked.

Whole database:

```
gcloud firestore import gs://budgetly-backend-nodejs.firebasestorage.app/backups/YYYY-MM-DD/ --project budgetly-backend-nodejs
```

Only some collections (smaller, safer):

```
gcloud firestore import gs://budgetly-backend-nodejs.firebasestorage.app/backups/YYYY-MM-DD/ --collection-ids=transactions --project budgetly-backend-nodejs
```

If you only need a few documents, import into the throwaway project and copy just those back with an admin script. That avoids overwriting everything else.

The import runs as a long operation. Check on it with `gcloud firestore operations list --project budgetly-backend-nodejs`, and wait for it to say it finished.

## Step 6: Check it worked

1. In the Firebase console, open Firestore and compare counts for `transactions`, the `users` documents and the `archive_*` collections against what you expect.
2. Open a few documents you know. Confirm the values (amounts, `deletedAt`, categories) match the backup date.
3. Log in to the web app with a test account and check the dashboard and Transactions page.
4. Confirm the next daily backup still runs (look for the new date folder the next morning).
5. Tell the team what was restored, from which backup, and what was lost (everything saved after the backup).

## Known limits

* A restore brings back old versions of documents, and does not remove documents added later. Duplicates and leftovers are possible, so check.
* Data written between the backup and the restore is lost for any document that was overwritten.
* Statement files in Storage are not backed up. The import record and transactions come back, but the original file only exists if it is still in Storage or the archive.
* Once cents are in place (SCRUM-148 to SCRUM-152), an old backup may hold dollar amounts. Convert again with the migration script after restoring.

## Practice log

Fill this in each time you practice. A real restore should be added here too.

| Date | Who | Backup used | Where restored | Collections | Result | Notes |
|---|---|---|---|---|---|---|
| | | | | | | |
