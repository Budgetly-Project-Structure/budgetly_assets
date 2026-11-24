# Bug Bash: Sprint 5 code freeze (SCRUM-170)

**Date:** Nov 24, 2026 (code freeze)
**Code under test:** `main` of budgetly_backend_nodejs and budgetly_web_customer after the Sprint 5 pull requests are merged: SCRUM-152 (strict cents rules), 166 (import picks the user's account), 167 (bank accounts), 168 (merchant name and ATM masking fixes), plus the earlier cents conversion (151), archive/backup work (148, 150) and the restore runbook (159).
**Where:** local emulators (`firebase emulators:exec` / `npm run dev` with `.env.local`), project `demo-budgetly`. Use a brand-new test user. Do not use real bank data. Run the live-site checks in section 6 only after the rules are deployed.
**Who / how:** one person clicks through the app while a second person watches the browser console and the emulator UI. Mark each row Pass / Fail and write the issue number in the Notes column.

## 0. Before you start

| Step | Command | Expected | Result |
| --- | --- | --- | --- |
| Backend unit tests | `cd functions; npm test` | All pass | |
| Rules tests | `firebase emulators:exec --only firestore,storage --project demo-budgetly "npm test"` | All pass (stop any running emulator first) | |
| Web build | `npm run build` in budgetly_web_customer | Builds with no errors | |
| Web lint | `npm run lint` | No new warnings (6 old ones are known) | |
| `main` is clean | `git status` and `git log -1` in each repo | Up to date with origin, no stray files | |

## 1. Sign up and money in cents

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 1.1 | Sign up a new user, accept consent | Empty dashboard, no console errors | |
| 1.2 | Add a manual expense of $12.34 | Shows $12.34 everywhere (list, dashboard total) | |
| 1.3 | Add an expense of $0.10 and $0.20, then look at the total | Total is exactly $0.30 | |
| 1.4 | Try to add an expense of $0 or an empty amount | Friendly error, nothing saved | |
| 1.5 | Edit a transaction's amount, then reload | New amount stays | |
| 1.6 | Set a budget limit of $250, reload, change it to $300 | Saves both times, no "permission denied" | |
| 1.7 | In the Firestore emulator UI open that transaction and budget | Fields are `amountCents` / `limitCents` as whole numbers; no `amount`, `limit` or `categoryId` | |

## 2. Bank accounts (SCRUM-167, 166)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 2.1 | Settings > Accounts: add a checking account (institution, last 4, nickname) | It appears in the list | |
| 2.2 | Add a credit card account | Appears; type shows as credit | |
| 2.3 | Try a last4 with letters or 5 digits | Rejected with a clear message | |
| 2.4 | Edit the nickname, reload | Saved | |
| 2.5 | Delete an account | Disappears from Settings and the Import picker | |
| 2.6 | Second user: sign up and look at Settings and the Import picker | Cannot see the first user's accounts | |
| 2.7 | Import page with two accounts of the same type, leave "Detect from the file" | Picker is offered; nothing is filed under the wrong account | |
| 2.8 | Import with exactly one account of the detected type | Filed under that account | |
| 2.9 | Import with no accounts | Built-in Checking / Credit still work | |
| 2.10 | Transactions page | Each row shows its account name; rows with a deleted account do not crash the page | |

## 3. Import

Use every file in `test-data/` and the backend fixtures, one at a time.

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 3.1 | Import `chase_checking_oct_2026.csv` | Imports; ID numbers in merchant names are masked | |
| 3.2 | Import `checking_statement_oct_2026.pdf` (same month) | "0 imported, 39 already imported" (bug bash 4 issue 1) | |
| 3.3 | Row `ATM WITHDRAWAL 000412 1420 E BATTLEFIELD` | Street number 1420 is still visible and matches the PDF row (SCRUM-168) | |
| 3.4 | Merchant `STEAM PURCHASE ... WA` | Shows "Steam", not "Steam Purchase Wa" (SCRUM-168) | |
| 3.5 | Merchant `Transfer to Me` | Not shortened | |
| 3.6 | Bank A then bank B statements (`bank_a_...`, `bank_b_...`) with two accounts set up | Nothing from bank B is dropped as a duplicate of bank A | |
| 3.7 | `not_a_statement.pdf`, `scanned_statement.pdf` | Clear "Failed" message (known: "Try again" still shown, Sprint 4 issue 7) | |
| 3.8 | `messy.csv` | Imports the good rows and lists the skipped ones | |
| 3.9 | Delete an imported statement, then restore it from Recently deleted | Transactions disappear and come back with the same amounts | |

## 4. Review, budgets and deleting

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 4.1 | Change a category on an uncategorized transaction | Saved; "Needs review" count goes down | |
| 4.2 | First visit of the month after an import | Auto budget banner appears once, not again on reload | |
| 4.3 | Auto budget preview and Apply | Budgets created in cents, amounts add up | |
| 4.4 | Delete a transaction, Undo; delete again, restore from Recently deleted | Works both ways | |
| 4.5 | Remove a budget, restore it | Works | |

## 5. Privacy and safety

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 5.1 | Sign in as user B and open a link or id from user A | Nothing shows; no data from A | |
| 5.2 | Browser console during every step above | No red errors, no `permission-denied` | |
| 5.3 | Postman: write a transaction with old `amount` and no `amountCents` | Rejected by the rules | |
| 5.4 | Search the Firestore emulator for full account numbers (9+ digits) in merchant names | None found | |
| 5.5 | Settings: delete my data / export, if present | Works, nothing left behind | |

## 6. Live site (after `firebase deploy --only firestore:rules`)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 6.1 | Add a transaction | Saves | |
| 6.2 | Change a budget limit | Saves (this was never checked live) | |
| 6.3 | Add an account in Settings, see it in the Import picker | Works | |
| 6.4 | Sign in with a second account | Sees only its own data | |
| 6.5 | `npm run backup` then check `backups/` | New backup file, 7 days kept | |

## 7. Backup and restore (SCRUM-148, 150, 159)

| # | Steps | Expected | Result |
| --- | --- | --- | --- |
| 7.1 | Follow the restore runbook on a test project, not live | Data comes back; fill in the runbook's practice log | |
| 7.2 | Archive job and restore script (SCRUM-155, 156, 157) | Only if Zach's SCRUM-155 is merged; otherwise list as not tested | |

## Known issues going into the freeze

| Issue | Severity | Status |
| --- | --- | --- |
| "Try again" is shown for a PDF that is not a statement (Sprint 4 issue 7) | Low | Open |
| Consent screen mentions an AI model while AI categorization is off (Sprint 4 issue 6) | Low | Product decision |
| Old transactions were not moved to the new per-user accounts | Low | Open |
| SCRUM-156 / 157 depend on SCRUM-155 | Medium | Blocked on Zach |
| SCRUM-164 shared merchant memory | Low | Not started |

## Results

Fill in after the run: date, tester, number of rows Pass / Fail, and for every Fail a line in the form "Bug (severity): what happened. Steps. Expected."

## For Jira

Create one ticket per Fail. Anything High or Medium found today gets fixed or explicitly deferred before the freeze is lifted; Low issues go in the backlog.
