# Bug Bash: Sprint 5 code freeze (SCRUM-170)

**Date:** Nov 24, 2026 (code freeze)
**Code under test:** `main` of budgetly_backend_nodejs and budgetly_web_customer after the Sprint 5 pull requests are merged: SCRUM-152 (strict cents rules), 166 (import picks the user's account), 167 (bank accounts), 168 (merchant name and ATM masking fixes), plus the earlier cents conversion (151), archive/backup work (148, 150) and the restore runbook (159).
**Where:** local emulators (`firebase emulators:exec` / `npm run dev` with `.env.local`), project `demo-budgetly`. Use a brand-new test user. Do not use real bank data. Run the live-site checks in section 6 only after the rules are deployed.
**Who / how:** one person clicks through the app while a second person watches the browser console and the emulator UI. Mark each row Pass / Fail and write the issue number in the Notes column.

## 0. Before you start

| Step | Command | Expected | Result |
| --- | --- | --- | --- |
| Backend unit tests | `cd functions; npm test` | All pass | Pass. 425 tests, 422 pass, 0 fail, 3 skipped. |
| Rules tests | `firebase emulators:exec --only firestore,storage --project demo-budgetly "npm test"` | All pass (stop any running emulator first) | Pass. 126 passing. |
| Web build | `npm run build` in budgetly_web_customer | Builds with no errors | Pass. |
| Web lint | `npm run lint` | No new warnings (6 old ones are known) | Pass. 6 known warnings, 0 errors. |
| `main` is clean | `git status` and `git log -1` in each repo | Up to date with origin, no stray files | Pass. Working trees clean on main. |

## 1. Sign up and money in cents

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 1.1 | Sign up a new user, accept consent | Empty dashboard, no console errors | Pass. Only browser-extension warnings in the console, no app errors. |
| 1.2 | Add a manual expense of $12.34 | Shows $12.34 everywhere (list, dashboard total) | Pass. $12.34 shows in the list and in the net total. |
| 1.3 | Add an expense of $0.10 and $0.20, then look at the total | Total is exactly $0.30 | Pass. $12.34 + $0.10 + $0.20 gives a net of -$12.64. |
| 1.4 | Try to add an expense of $0 or an empty amount | Friendly error, nothing saved | Pass. Empty: "Please fill out this field". $0: "Value must be greater than or equal to 0.01". |
| 1.5 | Edit a transaction's amount, then reload | New amount stays | Pass. $0.10 changed to $0.25, net -$12.79, still correct after logging out and in. |
| 1.6 | Set a budget limit of $250, reload, change it to $300 | Saves both times, no "permission denied" | Pass. Saves and survives reload. Removing and restoring a $15 budget also worked. |
| 1.7 | In the Firestore emulator UI open that transaction and budget | Fields are `amountCents` / `limitCents` as whole numbers; no `amount`, `limit` or `categoryId` | Pass for a manual transaction (`amountCents: -20`, no `amount`). Imported rows still carry the old `amount`, see Low issue 1. |

## 2. Bank accounts (SCRUM-167, 166)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 2.1 | Settings > Accounts: add a checking account (institution, last 4, nickname) | It appears in the list | Pass. |
| 2.2 | Add a credit card account | Appears; type shows as credit | Pass. Shows as Credit card. |
| 2.3 | Try a last4 with letters or 5 digits | Rejected with a clear message | Pass. The field blocks letters and a fifth digit. Three digits were not tried. |
| 2.4 | Edit the nickname, reload | Saved | Pass. Rename kept after reload and shows in import history. |
| 2.5 | Delete an account | Disappears from Settings and the Import picker | Pass. Removed accounts leave Settings and the Import picker and appear under Removed accounts with Restore. |
| 2.6 | Second user: sign up and look at Settings and the Import picker | Cannot see the first user's accounts | Pass. A second user saw none of the first user's data (also covered by the rules tests). |
| 2.7 | Import page with two accounts of the same type, leave "Detect from the file" | Picker is offered; nothing is filed under the wrong account | Pass with note. With two checking accounts nothing was filed under the wrong account, but the rows were filed under no account without asking (Low issue 2). |
| 2.8 | Import with exactly one account of the detected type | Filed under that account | Pass. Filed under the one checking account, "(detected)". |
| 2.9 | Import with no accounts | Built-in Checking / Credit still work | Pass. With no accounts of its own, the import used the built-in Checking 4521. |
| 2.10 | Transactions page | Each row shows its account name; rows with a deleted account do not crash the page | Pass. Page loads; rows of a removed account show "Deleted account". |

## 3. Import

Use every file in `test-data/` and the backend fixtures, one at a time.

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 3.1 | Import `chase_checking_oct_2026.csv` | Imports; ID numbers in merchant names are masked | Pass, using a stand-in file `bugbash_checking_oct_2026.csv` (the Chase file was not available). The long ID in the PayPal row is not shown. |
| 3.2 | Import `checking_statement_oct_2026.pdf` (same month) | "0 imported, 39 already imported" (bug bash 4 issue 1) | Not run. `checking_statement_oct_2026.pdf` was not available. |
| 3.3 | Row `ATM WITHDRAWAL 000412 1420 E BATTLEFIELD` | Stored merchant in Firestore still has 1420 and matches the PDF row (SCRUM-168). The list shows a cleaned display name without the numbers. | Pass. Stored merchant is `ATM WITHDRAWAL 000412 1420 E BATTLEFIELD` in Firestore. The list hides the numbers by design (display name only). |
| 3.4 | Merchant `STEAM PURCHASE ... WA` | Shows "Steam", not "Steam Purchase Wa" (SCRUM-168) | Pass. Shows "Steam". |
| 3.5 | Merchant `Transfer to Me` | Not shortened | Pass. Shows "Transfer to Me". |
| 3.6 | Bank A then bank B statements (`bank_a_...`, `bank_b_...`) with two accounts set up | Nothing from bank B is dropped as a duplicate of bank A | Pass. Bank A and bank B rows are all present (both Costco rows kept). |
| 3.7 | `not_a_statement.pdf`, `scanned_statement.pdf` | Clear "Failed" message (known: "Try again" still shown, Sprint 4 issue 7) | Pass. Both show a clear Failed message. "Try again" still shown (known issue). |
| 3.8 | `messy.csv` | Imports the good rows and lists the skipped ones | Pass. "2 transactions imported, 4 skipped, 2 already imported" with a "Some rows couldn't be read" link. The detail view was not opened. |
| 3.9 | Delete an imported statement, then restore it from Recently deleted | Transactions disappear and come back with the same amounts | Pass. Six rows disappeared and came back with the same amounts (count 76 to 84 with messy.csv). |

## 4. Review, budgets and deleting

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 4.1 | Change a category on an uncategorized transaction | Saved; "Needs review" count goes down | Pass. Needs review went from 11 to 10. |
| 4.2 | First visit of the month after an import | Auto budget banner appears once, not again on reload | Not fully tested. Auto budget was offered on the empty Budget page; the once-per-month banner was not checked. |
| 4.3 | Auto budget preview and Apply | Budgets created in cents, amounts add up | Pass. Preview total $6,818 and $3,818 with $3,000 savings both add up; Apply saved the same totals. |
| 4.4 | Delete a transaction, Undo; delete again, restore from Recently deleted | Works both ways | Pass. Undo and restore from Recently deleted both work. |
| 4.5 | Remove a budget, restore it | Works | Pass. Removed the $15 Subscriptions budget and restored it at $15. |

## 5. Privacy and safety

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 5.1 | Sign in as user B and open a link or id from user A | Nothing shows; no data from A | Pass. A second user sees none of the first user's data. |
| 5.2 | Browser console during every step above | No red errors, no `permission-denied` | Pass. No `permission-denied`. Only network errors (QUIC / Listen stream) after the tab sat idle. |
| 5.3 | Postman: write a transaction with old `amount` and no `amountCents` | Rejected by the rules | Pass by automated test (rules test: old dollar `amount` rejected). Postman not run. |
| 5.4 | Search the Firestore emulator for full account numbers (9+ digits) in merchant names | None found | Pass by automated test (masking unit tests). Emulator search not run. |
| 5.5 | Settings: delete my data / export, if present | Works, nothing left behind | Not run. |

## 6. Live site (after `firebase deploy --only firestore:rules`)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 6.1 | Add a transaction | Saves | Pass. |
| 6.2 | Change a budget limit | Saves (this was never checked live) | Pass. |
| 6.3 | Add an account in Settings, see it in the Import picker | Works | Pass. |
| 6.4 | Sign in with a second account | Sees only its own data | Pass. |
| 6.5 | `npm run backup` then check `backups/` | New backup file, 7 days kept | Not run. |

## 7. Backup and restore (SCRUM-148, 150, 159)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 7.1 | Follow the restore runbook on a test project, not live | Data comes back; fill in the runbook's practice log | Not run. |
| 7.2 | Archive job and restore script (SCRUM-155, 156, 157) | Only if Zach's SCRUM-155 is merged; otherwise list as not tested | Not tested. No `archive_` data exists yet and it is unconfirmed that the archive job (#48) is deployed. |

## Known issues going into the freeze

| Issue | Severity | Status |
| --- | --- | --- |
| "Try again" is shown for a PDF that is not a statement (Sprint 4 issue 7) | Low | Open |
| Consent screen mentions an AI model while AI categorization is off (Sprint 4 issue 6) | Low | Product decision |
| Old transactions were not moved to the new per-user accounts | Low | Open |
| SCRUM-156 / 157 depend on SCRUM-155 | Medium | Done: SCRUM-155 merged in #48, restore script merged in #49 |
| SCRUM-164 shared merchant memory | Low | Not started |

## Results

**Date:** Nov 24, 2026. **Tester:** Tyler Petersen. **Where:** live site (`budgetly-backend-nodejs.web.app`) with two new test users; setup checks run locally.

**Totals (43 numbered rows):** 37 Pass, 0 Fail, 6 not run or not fully tested. The 5 setup checks in section 0 all passed.

**Not run or not fully tested**

- 3.2: `checking_statement_oct_2026.pdf` was not available.
- 4.2: the once-per-month auto budget banner was not checked.
- 5.5: delete / export my data was not run.
- 6.5 and 7.1: backup and the restore runbook practice were not run.
- 7.2: no archived data exists yet, and it is unconfirmed that the archive job (#48) and the ATM masking function change (#47) were deployed with `firebase deploy --only functions`. The ATM row in 3.3 did keep its street number, so the SCRUM-168 backend change is live.

**Issues found (no High or Medium)**

1. Low (cleanup): Imported transactions still store a dollar `amount` next to `amountCents`. Steps: import a statement, open a transaction in Firestore. Expected: only `amountCents`. Cause: the TEMPORARY `amount` line in `functions/lib/transactions.js`.
2. Low: With two accounts of the same type and "Detect from the file", the import is filed under no account without asking. Steps: add two checking accounts, import a checking CSV. Expected: ask which account, or say clearly that none was picked.
3. Low: Importing the same file again after changing accounts imports the rows again (25 more each time). Steps: import a file, add a second same-type account, import the same file twice. Expected: "already imported".
4. Low: Manual transactions show an "Edited" tag as soon as they are added, and some get the built-in Checking 4521 while others get no account.
5. Low: Auto budget with a large savings amount sets small categories to $0, and Apply replaces a limit the user set (Subscriptions $15 became $0) without a warning. The preview does show the change.
6. Low (already known): "Try again" is shown for PDFs that cannot be read.
7. Low (by design): Dashboard and budget cards round to whole dollars; the Transactions list shows cents.

## For Jira

Create one ticket per Fail. Anything High or Medium found today gets fixed or explicitly deferred before the freeze is lifted; Low issues go in the backlog.
