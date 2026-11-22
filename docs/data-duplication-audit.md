# Data Duplication Audit (Sprint 5, ticket #1)

**Date:** Nov 22, 2026
**Story:** As a developer, I want a list of every value we save in more than one place, so that we know exactly what to remove and which copy is the real one.
**Why:** professor feedback, "Get rid of duplicate places where data is held (documents = transaction_used)".
**Code read:** `main` of budgetly_backend_nodejs (`firestore.rules`, `storage.rules`, `functions/index.js`, `functions/lib/*`, `functions/scripts/*`) and budgetly_web_customer (`src/`), as of the Sprint 4 merges (backend `1ad897a`, web `d02886e`). Line numbers are from those commits. Old docs were not trusted; where `budgetly_contracts/schemas/data-model.md` disagrees with the code, the code wins.

There is no field called `transaction_used` anywhere in the code. The professor's example matches the import record's `transactionCount` (item D1).

## How to read this

- **Source of truth** is the copy we keep. Everything else gets derived from it.
- **Remove** means stop writing the copy, change its readers to use the source of truth, and drop it from the rules and the contract.
- **Keep** only when there's a reason stronger than "it's handy" (a query Firestore can't do otherwise, a security rule, or history that must not change).
- **Ticket:** #2 "Import records stop copying their transactions" (Zach) takes the import-record copies (D1, plus D4 and D5, which were moved to #2 because they touch the same files). #3 "Remove the other duplicates the audit finds" (Tyler) takes the rest.

## Summary

| # | Duplicate | Source of truth | Decision | Ticket |
| --- | --- | --- | --- | --- |
| D1 | Import record `transactionCount` | The transactions with `sourceStatementId == importId` | **Remove** | #2 |
| D2 | A deleted statement's `deletedAt` / `deletedBy` / `purgeAfter`, copied onto every one of its transactions | The import record | Keep for now (needs a rework, deferred past code freeze) | — |
| D3 | `accountId` on the import record and on each of its transactions | Import record (imported), transaction (manual) | Keep (query + manual) | — |
| D4 | Import record `accountType` (the type of the account it's filed under) | The account (`ACCOUNTS[accountId].type`) | **Remove** | #2 (moved from #3) |
| D5 | Import record `fileName` and `fileType`, also inside `storagePath` | `storagePath` | **Remove** | #2 (moved from #3) |
| D6 | Transaction `isUserCorrected` vs `categorySource == "user"` | `categorySource` | **Remove** | #3 |
| D7 | Budget `categoryId` field vs the budget's document ID | The document ID | **Remove** | #3 |
| D8 | `budgetMonths` `month` field vs the document ID | The document ID | **Remove** | #3 |
| D9 | Leftover `users/{uid}/transactions` documents from before the migration | `transactions/{uid}_{oldId}` | **Remove** | #3 |
| D10 | `users/{uid}.email` vs the Firebase Auth email | Firebase Auth | **Remove** | #3 |
| D11 | `displayName` in both Firebase Auth and `users/{uid}` | `users/{uid}.displayName` | **Remove** (the Auth copy) | #3 |
| D12 | `users/{uid}/merchantCache` vs the user's own category choices on transactions | The transactions | Keep (lookup index) | — |
| D13 | Transaction `source` vs `sourceStatementId` | Both (different facts) | Keep (security rule) | — |
| D14 | Import record `sizeBytes`, `uploadedAt` vs the Storage file's own metadata | Import record | Keep (Storage can't be queried or listened to) | — |
| D15 | Storage custom metadata `accountId` / `accountType` on the uploaded file | Import record (after the upload) | Keep (only way to hand the choice to the trigger) | — |
| D16 | Audit log `details.fileName` / `details.transactionCount` | Audit log (history) | Keep (history must not change) | — |
| D17 | Import record `skippedRowCount` vs `rowErrors.length` | Both | Keep (`rowErrors` is capped at 100) | — |

Things we were asked to check that turned out **not** to be duplicates are listed at the end.

---

## D1. Import record `transactionCount`. Remove (#2)

The import record counts the transactions it saved. The transactions themselves already say which import they came from (`sourceStatementId`), so the count is a second copy of "how many transactions belong to this statement". Nothing keeps it up to date after the import finishes.

**Stored in**
- `users/{uid}/imports/{importId}.transactionCount`
  - written on success: `budgetly_backend_nodejs/functions/lib/importPipeline.js:140-149` (`transactionCount: docs.length`)
  - written as `0` on failure: `importPipeline.js:73` (`fail`), `importPipeline.js:214` (timeout), `functions/index.js:157-161` (download failed)
  - all through `functions/lib/firestoreStore.js:54` (`update`)
- Source of truth: `transactions` where `userId == uid` and `sourceStatementId == importId` (`firestoreStore.js:51`)

**Read by**
- `budgetly_web_customer/src/data/FinanceProvider.jsx:79` (`statementFromDoc`), then shown at
  - `src/pages/ImportPage.jsx:84` (delete dialog: "This removes N transactions")
  - `src/pages/ImportPage.jsx:194` (Recently deleted row)
  - `src/pages/ImportPage.jsx:515` (Import history row)
- `src/pages/ImportPage.jsx:342` sets `transactionCount: 0` on the browser-only "uploading" row (not stored)
- `budgetly_backend_nodejs/functions/lib/purgeJob.js:163` (the `import.purged` audit entry copies it)

**How to remove** (done in #2)
- The Import page counts each finished import's transactions with a `count()` query: `where("userId", "==", uid)` and `where("sourceStatementId", "==", importId)`. That's one small read per import instead of a stored copy.
- `purgeJob.js:163`: use the number of transactions the purge actually removes. Step 2 already counts them (`leftover`, `purgeJob.js:132-144`), but today that count is usually 0 because `purgeQuery` on transactions (`purgeJob.js:194`) removes them first (D2 gives them the same `purgeAfter`). The fix is to purge statements **before** the transaction sweep, so step 2 removes and counts all of them.
- Stop writing it in `importPipeline.js` and `index.js`. Keep it in the function's **return value** (that's just a log line, `index.js:171`, not stored).
- Update the Statement table in `budgetly_contracts/schemas/data-model.md`, which still lists `transactionCount`.

## D2. Statement delete fields copied onto its transactions. Keep for now

**Decision (Nov 22):** keep for now. The Transactions page and the server queries filter transactions on their own `deletedAt == null`. Removing the copy means reworking all of those readers, so it's deferred past code freeze. The rest of this section is the plan for when it's picked up.

When a statement is deleted, `deletedAt`, `deletedBy` and `purgeAfter` are written on the import record **and** on every one of its transactions, in batches of 400. Restore clears them in both places. That's the same "this statement is deleted" fact stored N+1 times. It also isn't atomic: if the function fails halfway, some transactions are deleted and others aren't.

This is the reverse direction of D1 (the transactions copy the import), and it touches the same files (`softDeleteImport.js`, `restoreImport.js`, `purgeJob.js`, `firestoreStore.js`).

**Stored in**
- Source of truth: `users/{uid}/imports/{importId}.deletedAt / deletedBy / purgeAfter`, written by `firestoreStore.js:113` (`softDeleteImportDoc`) and cleared by `firestoreStore.js:133` (`restoreImportDoc`)
- Copy: every `transactions/{id}` of that import, written by `firestoreStore.js:102-110` (`softDeleteTransactionBatch`, called from `functions/lib/softDeleteImport.js:63`) and cleared by `firestoreStore.js:122-130` (`restoreTransactionBatch`, called from `functions/lib/restoreImport.js:64`)

**Read by (the copy on transactions)**
- `src/data/FinanceProvider.jsx:466`: hides deleted transactions (`!t.deletedAt`)
- `functions/lib/firestoreStore.js:154`: the duplicate check skips transactions of deleted statements
- `functions/lib/firestoreStore.js:103` and `:123`: soft delete and restore find the transactions by these fields
- `functions/lib/purgeJob.js:194`: the daily purge removes transactions whose `purgeAfter` has passed
- `functions/scripts/scanTransactions.js` (skips soft-deleted transactions) and `functions/scripts/walkthroughDeleteLifecycle.js:139`

**How to remove (later)**
- Soft delete and restore of a statement become a single write to the import record (plus the audit entry). The batch loops go away.
- Web: the provider already has the import records (`FinanceProvider.jsx:653-656`). Build the set of deleted import IDs and hide transactions whose `sourceStatementId` is in it, as well as manual ones with their own `deletedAt`.
- Duplicate check (`firestoreStore.js:143-158`): read the user's deleted import IDs once (`users/{uid}/imports` where `deletedAt != null`) and skip transactions from those.
- Purge: `purgeImports` step 2 (`purgeJob.js:131-144`) already deletes every transaction of a purged statement by `sourceStatementId`. The transaction purge (`purgeJob.js:192-200`) then only finds manual ones.
- Manual transactions **keep** their own `deletedAt` / `deletedBy` / `purgeAfter`. For them it's the only copy (they have no import record).
- New imported transactions can still be created with `deletedAt: null` (`transactions.js:73`) so `where("deletedAt", "==", null)` queries keep working, but nothing should set it on them afterwards.
- Existing data: statements deleted before this change have the copies on their transactions. Either let them age out (they purge within 30 days) or clear them in a one-time script.

## D3. `accountId` on the import and on its transactions. Keep

**Stored in:** `users/{uid}/imports/{importId}.accountId` (`index.js:141`, `importPipeline.js:90`) and `transactions/{id}.accountId` (`transactions.js:70` for imports, `FinanceProvider.jsx:565` and `:592` for manual ones).
**Read by:** `src/pages/ImportPage.jsx:487` (import row), `src/pages/TransactionsPage.jsx:484` and `:540` (transaction rows), `functions/lib/firestoreStore.js:147` (duplicate check query).
**Keep because:** for a manual transaction the field on the transaction is the only copy. For imported ones, the duplicate check has to *query* transactions by account (`firestoreStore.js:147`). Firestore can't join to the import record, so without the field it would have to read every statement and then all their transactions. Both copies are written once and can't change for imported transactions (the rules only let `accountId` change on manual ones, `firestore.rules:184-188`), so they can't drift apart.

## D4. Import record `accountType`. Remove (#2, moved from #3)

The statement's account type is stored next to its `accountId`, but the account already has a type (`acct-checking` is checking, `acct-credit` is credit). The pipeline even makes the `accountId` *from* the type (`importPipeline.js:88-89`).

**Stored in:** `users/{uid}/imports/{importId}.accountType`, written at `index.js:142` and `importPipeline.js:90`. Source of truth: the account's type, today in code (`src/data/mockData.js:37-40` `ACCOUNTS`, and `functions/lib/accountType.js:24` `ACCOUNT_ID_FOR_TYPE`).
**Read by:** only `src/data/FinanceProvider.jsx:76`, which maps it onto the statement. No page uses `s.accountType` (`ImportPage.jsx:487-494` uses `acct.type` and `detectedAccountType`).
**How to remove:** stop writing it in `index.js` and `importPipeline.js:90`, and drop it from `statementFromDoc`. The pipeline still *uses* the type while it runs (it comes in through the Storage metadata, D15). **Keep `detectedAccountType`**: it's what the file looked like, which nothing else records.
**Related, not stored data:** the two accounts are hard-coded in two places (web `mockData.js:37-40`, backend `accountType.js:24`). That's a code copy, not a Firestore copy. It goes away when accounts move to Firestore.

## D5. Import record `fileName` and `fileType`. Remove (#2, moved from #3)

`storagePath` is `users/{uid}/imports/{fileName}`, so `fileName` is just its last part and `fileType` is that name's extension.

**Stored in:** `users/{uid}/imports/{importId}` at `index.js:138` (`fileName`), `:139` (`storagePath`), `:140` (`fileType`). Source of truth: `storagePath`.
**Read by:** `src/data/FinanceProvider.jsx:72-73` (then `ImportPage.jsx:100`, `:192`, `:201`, `:204`, `:510`, `:534` and the type label at `:498`), and the audit entries at `functions/lib/softDeleteImport.js:77`, `functions/lib/restoreImport.js:78`, `functions/lib/purgeJob.js:163`.
**How to remove:** add a small helper (backend and web) that takes the last part of `storagePath` and its extension, and use it in the readers above. The browser's own "uploading" row (`ImportPage.jsx:337-338`) is never stored, so it isn't affected.
**Moved to #2:** D4 and D5 touch the same import record and files as D1 (`index.js`, `importPipeline.js`, `statementFromDoc`), so they're done in #2. Tyler's #3 doesn't do them.

## D6. Transaction `isUserCorrected`. Remove (#3)

`isUserCorrected` means the same thing as `categorySource == "user"`. It's already **wrong** in the database: when a user re-categorizes an imported transaction, the rules only allow `category`, `categorySource` and `updatedAt` to change (`firestore.rules:178-182`). So the stored `isUserCorrected` stays `false` while `categorySource` becomes `"user"`.

**Stored in:** `transactions/{id}.isUserCorrected`, written at `functions/lib/transactions.js:69` (`false`) and `src/data/FinanceProvider.jsx:564` (`true`). Allowed by `firestore.rules:168`. Source of truth: `categorySource`.
**Read by:** nothing reads the stored field. The web app works it out from `categorySource` (`FinanceProvider.jsx:104`, `:116`), then uses that at `src/lib/finance.js:30` and `src/pages/TransactionsPage.jsx:58`, `:96`.
**How to remove:** stop writing it in both places, remove it from the create rule's field list (`firestore.rules:168`) and from `data-model.md`. Old documents can keep the field; nothing reads it.
**Related (keep):** `categoryConfidence` also goes stale when a user re-categorizes (same rule), but it's the AI's or rule's own confidence, which isn't stored anywhere else. Every reader checks `isUserCorrected` first (`finance.js:30`). For manual transactions it's always `1` (`FinanceProvider.jsx:563`); dropping that write is optional.

## D7. Budget `categoryId` field. Remove (#3)

Budgets live at `users/{uid}/budgets/{categoryId}`, and each one also has a `categoryId` field. The rules force the two to match (`firestore.rules:113`).

**Stored in:** the field, written at `src/data/FinanceProvider.jsx:211`. Source of truth: the document ID.
**Read by:** `src/data/FinanceProvider.jsx:275` and `:286`, which already fall back to `d.id`.
**How to remove:** stop writing it, read `d.id`, and take it out of the rule's field list and equality check (`firestore.rules:112-113`) and `data-model.md`. The purge job finds budgets by `purgeAfter`, not by this field (`purgeJob.js:204`).

## D8. `budgetMonths` `month` field. Remove (#3)

**Stored in:** `users/{uid}/budgetMonths/{YYYY-MM}.month`, written at `src/data/FinanceProvider.jsx:373` and `:498-499`. The rules force it to equal the document ID (`firestore.rules:129-131`). Source of truth: the document ID.
**Read by:** nothing. The listener uses `d.id` (`FinanceProvider.jsx:352`).
**How to remove:** stop writing it and change the create rule so it no longer requires `month` (keep the `^[0-9]{4}-[0-9]{2}$` check on the document ID).
**Checked:** budget *limits* are **not** copied into `budgetMonths`. It only stores `auto` and `savings`, which exist nowhere else.

## D9. Leftover `users/{uid}/transactions` documents. Remove (#3)

SCRUM-116 moved transactions to the top-level `transactions` collection. The migration script copies the old documents and only deletes the originals when run with `--delete-old` (`functions/scripts/migrateTransactions.js:18`, `:24`, `:61`). We don't know if that was ever run against the real project, so old copies may still be there.

**Stored in:** `users/{uid}/transactions/{oldId}` (old) and `transactions/{uid}_{oldId}` (new, source of truth).
**Read by:** nothing in the app. `firestore.rules:98-100` closed the old path (anything not matched is denied), and `test/firestore.rules.test.js:72` checks that.
**How to remove:** run `node scripts/migrateTransactions.js --dry-run --project budgetly-backend-nodejs` to see what's left, then `--delete-old`. After that, the rules comment can stay as history. The script and its test can stay as the record of the migration.
**Leftover rules:** none. No rule matches the old path any more.

## D10. `users/{uid}.email`. Remove (#3)

**Stored in:** Firebase Auth (source of truth, used to sign in) and `users/{uid}.email`, written once at signup (`src/lib/auth.js:57`), allowed by `firestore.rules:87`.
**Read by:** nothing. The email on Settings comes from Firebase Auth (`FinanceProvider.jsx:434`, shown at `src/pages/SettingsPage.jsx:110`). The Firestore copy would also go stale if the email were ever changed in Auth.
**How to remove:** stop writing it at signup, remove `email` from the create rule (`firestore.rules:87`) and from `data-model.md`.

## D11. `displayName` in Auth and in `users/{uid}`. Remove the Auth copy (#3)

**Stored in:** `users/{uid}.displayName` (source of truth: the rules check it, `firestore.rules:40-45`, and the app listens to it live), and the Firebase Auth profile, written at `src/lib/auth.js:53` (signup) and `src/data/FinanceProvider.jsx:330-332` (Settings).
**Read by:** the Firestore copy at `FinanceProvider.jsx:310`. The Auth copy only fills in before the profile loads (`FinanceProvider.jsx:433`). Shown at `src/pages/HomePage.jsx:124` and `src/components/Layout.jsx:173`, `:267`.
**How to remove:** stop writing the Auth copy in `saveProfile`. For the moment before the profile loads, fall back to the email's first part, which is already the second fallback (`FinanceProvider.jsx:433`). Signup can stop calling `updateProfile` too.

## D12. `users/{uid}/merchantCache`. Keep

The cache stores "this merchant is this category" for every category a user picked. That's derived from the user's transactions (where `categorySource == "user"`).

**Stored in:** `users/{uid}/merchantCache/{docId}`, written only by the `onTransactionCategoryChanged` trigger (`functions/index.js:216-229`, `functions/lib/merchantCache.js:153-163`). Source of truth: the transactions.
**Read by:** the import categorizer (`functions/lib/importCategorization.js:60-61` via `merchantCache.js:136-150`).
**Keep because:** it's a lookup index, not a second editable copy. It's keyed by the cleaned-up merchant (`merchantKey`, which strips store numbers), so an import can look up each merchant directly instead of reading and grouping every transaction the user has ever categorized. Only the trigger writes it, and nobody can edit it (no Firestore rule allows the browser in). So the transactions stay the source of truth, and the cache can be rebuilt from them at any time. It also deliberately keeps a choice after the transaction is deleted, so a user's preference isn't forgotten.
**Shared merchant memory:** there isn't one in Firestore. The shared merchant knowledge is the keyword list in code (`functions/lib/merchantRules.js`), so it doesn't overlap with the per-user cache in the database.

## D13. Transaction `source` vs `sourceStatementId`. Keep

Today `source == "import"` exactly when `sourceStatementId` is set, so one could be worked out from the other.
**Read by:** `firestore.rules:185`, `:192` (only `manual` transactions can be edited or deleted by the user), `functions/lib/firestoreStore.js:146`, `src/data/FinanceProvider.jsx:111`, `:119`, `:520`.
**Keep because:** it's what the security rules check. If the rule said "no `sourceStatementId` means manual" instead, any document missing the link would become editable by the user. Migrated documents without a statement link have no `source` (`migrateTransactions.js:37-41` only sets it when the link exists), so today they're locked. With a derived rule they would be open. `source` is an explicit "who created this", and `sourceStatementId` is the link to the statement. They're two facts that happen to line up today.

## D14. Import `sizeBytes` and `uploadedAt` vs Storage metadata. Keep

**Stored in:** `index.js:143-145`, and the Storage object's own size and creation time.
**Read by:** `FinanceProvider.jsx:84`, `:87`, and the import list is ordered by `uploadedAt` (`FinanceProvider.jsx:653`).
**Keep because:** the Import page lists statements with one live Firestore query, ordered by `uploadedAt`. Storage can't be queried, sorted or listened to, so reading this from the files would take one request per file and the list wouldn't update live.

## D15. Storage custom metadata on the uploaded file. Keep

**Stored in:** `customMetadata: { accountId, accountType }` on the file (`src/lib/uploads.js:50`), then on the import record (`index.js:115-119`, `:141-142`).
**Keep because:** it's the only way to pass the user's choice from the browser to the `onStatementUploaded` trigger. The import record doesn't exist yet when the file is uploaded. It's read once and never again. (Once D4 is done, only `accountId` ends up on the record.)

## D16. Audit log details. Keep

**Stored in:** `auditLog/{id}.details.fileName` and `.transactionCount` (`softDeleteImport.js:77`, `restoreImport.js:78`, `purgeJob.js:163`).
**Keep because:** an audit entry is a record of what happened at that moment. After a purge, the import record and the transactions are gone, so the entry is the only place left that says what was removed. It must not change when the source changes.

## D17. `skippedRowCount` vs `rowErrors`. Keep

**Stored in:** `importPipeline.js:143` and `:148`. Also `duplicateRowCount` (`:144`).
**Read by:** `ImportPage.jsx:516`, `:586-600`.
**Keep because:** these describe rows that were **not** saved, so they aren't a copy of any transaction. `skippedRowCount` can look like `rowErrors.length`, but `rowErrors` is capped at 100 by the parser (`csvParser.js:28`, `MAX_REPORTED_ERRORS`), so the page needs the real total to say "…and N more" (`ImportPage.jsx:599-600`).

---

## Checked and not a duplicate

- **Category names next to category IDs:** transactions store only `category` (an ID) and budgets only the category ID. No label is saved in Firestore. Labels come from the category list at render time (`FinanceProvider.jsx:387-395`).
  - Related code copy (not stored data): the built-in category list is written out three times, in `budgetly_contracts/schemas/categories.md`, `functions/lib/categories.js:12-24` and `src/data/mockData.js`. That's by design ("change the contract first, then both copies", `categories.js:5`), because the backend and the web app don't share code.
  - Custom categories exist only in one browser's `localStorage` (`FinanceProvider.jsx:148-157`, `:411-415`). That's a single copy, so it's not a duplicate, but it's a gap: another device only sees the ID.
- **Merchant name:** stored once, raw. The cleaned-up name is worked out when it's shown (`FinanceProvider.jsx:111-112`).
- **Account name or type on transactions:** not stored. Transactions carry only `accountId`, and the name and type come from `ACCOUNTS`.
- **Budget "spent" totals:** not stored. Spending is always added up from the transactions (`data-model.md`, Budget notes).
- **Import record `status`, `error`, `detectedAccountType`, `storagePath`:** each is the only copy.
- **Browser `localStorage` "dismissed imports"** (`FinanceProvider.jsx:145`): a per-device display preference, not a copy of anything.

## Order of work

1. **#2 (Zach):** D1, D4 and D5. Existing import records lose the old fields through a one-time script (`functions/scripts/removeImportCopies.js`). Update the Statement table in budgetly_contracts `data-model.md`. **Deploy order:** the backend and the web app ship together, and the cleanup script runs only after both are live (the old web app still reads the fields).
2. **#3 (Tyler):** D6, D7, D8, D10, D11 are small (stop writing, update rules and tests). D9 is a one-time script run against the real project, and needs someone with admin credentials.
3. **Later (after code freeze):** D2.
3. Every rule change needs `test/firestore.rules.test.js` updated. Existing documents can keep the removed fields; nothing will read them.
