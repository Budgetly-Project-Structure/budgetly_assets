# Bug Bash: Sprint 4 (SCRUM-140)

**Date:** Nov 2, 2026
**Code tested:** `main` of budgetly_backend_nodejs and budgetly_web_customer (everything through SCRUM-122, 128, 135 and the dashboard fix) plus the SCRUM-137 branches (monthly auto budget), not yet merged at the time.
**Where:** the local emulators (Auth, Firestore, Storage and the real Cloud Functions, project `demo-budgetly`) with the web app running against them. No real data was used.
**How:** a brand-new test user was created through the Sign up page, then every statement file was uploaded through the real Import page, one at a time, and the user went through review, budgets, delete and restore in the browser.

Automated tests on the same code: 322 backend unit tests and 77 rules tests pass, and the web app builds.

## What was tested

| Area | Steps | Result |
| --- | --- | --- |
| Sign up and consent | New account, consent screen, land on the empty dashboard | Works |
| Import (12 files) | Every file in `test-data/` and the backend test fixtures, with "Detect from the file" | Works, except issues 1 and 3 below |
| Account detection (SCRUM-135) | Checking CSVs and PDFs, credit card CSVs | All detected correctly |
| Bad files | `not_a_statement.pdf`, `scanned_statement.pdf` | Clear, friendly "Failed" messages |
| Partly readable file | `messy.csv` | 3 imported, 4 skipped, with "See which ones" |
| Review | Change an uncategorized transaction's category | Saved; the "Needs review" count went down |
| Manual transaction | Add, delete (undo toast), undo, delete again, restore from Recently deleted | Works |
| Delete statement (SCRUM-120/121) | Delete a 13-transaction statement, restore it from Recently deleted | Works; audit log has `import.deleted(13)` and `import.restored(13)` |
| Monthly auto budget (SCRUM-137) | First visit of the month after importing | Banner shown; budgets created; no second run on reload |
| Auto budget | Preview with $200 savings, Apply | Applied; savings saved on the month record for next month |
| Remove budget | Remove, undo; remove, restore from Recently deleted | Works |

## Issues found

| # | Issue | Severity | Status |
| --- | --- | --- | --- |
| 1 | The same statement imported as CSV and PDF counts 11 of 39 transactions twice | High | **Fixed** (backend) |
| 2 | CSV imports keep full account / ID numbers in merchant names | Medium | **Fixed** (backend) |
| 3 | Statements from a different bank account are filed under the one "Checking" account, and real transactions get dropped as duplicates | Medium | Not fixed (bigger change) |
| 4 | Dashboard says "The AI wasn't confident" although AI categorization is off | Low | **Fixed** (web) |
| 5 | Cleaned merchant names keep the state code in title case ("Steam Purchase Wa") | Low | Not fixed |
| 6 | The consent screen says transaction details are sent to an AI model, but AI categorization is switched off | Low | Not fixed (product decision) |
| 7 | "Try again" is offered for a PDF that isn't a statement and will always fail | Low | Not fixed |
| 8 | ATM withdrawals: the CSV masking hides the street number together with the ATM's sequence number | Low | Not fixed |

### 1. Same statement as CSV and PDF counts transactions twice (High, fixed)

* **Steps:** import `test-data/chase_checking_oct_2026.csv`, then `test-data/checking_statement_oct_2026.pdf` (the same October statement).
* **Expected:** "0 transactions imported, 39 already imported".
* **Actual:** "11 transactions imported, 28 already imported". October's money in and money out were inflated (two extra paychecks, rent, autopays).
* **Cause:** the duplicate check (SCRUM-136) compared the exact merchant text, but the PDF masks numbers and leaves out reference codes: `NETFLIX.COM 866-579-7172 CA` vs `NETFLIX.COM ••••7172 CA`, `... MGMT JPM99A1B2C3` vs `... MGMT`.
* **Fix:** `functions/lib/duplicates.js` compares merchant names without masked numbers, numbers of 6+ digits (phone, account and ID numbers) and long reference codes. Short store numbers (`#1187`, `STORE 08812`) still count, so different stores stay different. 10 of the 11 rows now match; the 11th is issue 8. Tests in `functions/test/duplicates.test.js` use the real pairs from these two files.

### 2. CSV imports keep full account / ID numbers (Medium, fixed)

* **Steps:** import `chase_checking_oct_2026.csv`, look at the merchant names in Firestore.
* **Expected:** numbers masked like everywhere else (ui-constraints.md), as the PDF importer already does: `ACME CORP PAYROLL PPD ID: ••••6411`.
* **Actual:** `ACME CORP PAYROLL PPD ID: 9100236411` (4 such transactions in this file).
* **Fix:** the CSV importer now masks 8+ digit numbers like the PDF importer (`maskAccountNumbers` moved into `functions/lib/csvParser.js`, shared by both). Test in `functions/test/csvParser.test.js`.

### 3. Different bank accounts collapse into one (Medium, not fixed)

* **Steps:** import `bank_a_checking_sept_2026.pdf` (account ••3788), then `bank_b_credit_union_sept_2026.pdf` (a different bank, account ••7732).
* **Expected:** two separate accounts; nothing from the credit union treated as a duplicate of the other bank.
* **Actual:** both are filed under "Checking ••4521" (the app's built-in accounts), and 3 of the credit union's 11 transactions were skipped as "already imported" because they had the same date, amount and merchant as the other bank's rows.
* **Why not fixed:** the app has two fixed sample accounts (`mockData.js` ACCOUNTS). Fixing this needs real per-user accounts (for example keyed by the statement's last 4 digits) in the data model, rules and Import page. Suggested as its own story.

### 4. "The AI wasn't confident about these" (Low, fixed)

* **Where:** dashboard, "10 transactions need a category check".
* **Problem:** AI categorization is switched off (the free keyword rules categorize imports), so the wording is wrong.
* **Fix:** now reads "We couldn't pick a category for these with confidence." (`src/pages/HomePage.jsx`).

### 5. State codes in merchant names (Low, not fixed)

Imported names like `STEAM PURCHASE 425-889-9642 WA` show as "Steam Purchase Wa" and `STARBUCKS STORE 08812 SPRINGFIELD MO` as "Starbucks Store Springfield Mo". Cosmetic; `cleanMerchant` could drop a trailing 2-letter state code.

### 6. Consent screen mentions an AI model (Low, not fixed)

The consent screen and Settings say merchant, amount and date are sent to an AI model, but no AI call is made while AI categorization is off. Leave as is if SCRUM-129 to 131 (Gemini) will turn it on; otherwise reword.

### 7. "Try again" on a file that can't work (Low, not fixed)

`not_a_statement.pdf` fails with "This statement's layout isn't supported yet. Please export a CSV..." and still offers "Try again", which will fail the same way. Hide "Try again" for this kind of failure.

### 8. ATM sequence and street number masked together (Low, not fixed)

`ATM WITHDRAWAL 000412 1420 E BATTLEFIELD` reads `000412 1420` as one 10-digit number and masks it to `••••1420`, so this CSV row still doesn't match the PDF's `ATM WITHDRAWAL 1420 E BATTLEFIELD`. Harmless (one possible double count of an ATM withdrawal when the same statement is imported in both formats).

## Notes

* The two October checking files in `test-data` (`chase_checking_oct_2026.csv` and `checking_within_budget_oct_2026.csv`) are different statements for the same account and month, so importing both doubles October. That's the test data, not a bug.
* The undo toast closes after a few seconds; Undo after that is done from Recently deleted. Working as designed.

## For Jira

1. Bug (High, fixed in SCRUM-140): Same statement imported as CSV and PDF counts 11 of 39 transactions twice. Duplicate check now ignores masked and long numbers.
2. Bug (Medium, fixed in SCRUM-140): CSV imports stored full account/ID numbers in merchant names. Now masked like PDFs.
3. Story (Medium, open): Real per-user bank accounts. Statements from different banks are filed under one "Checking" account, and real transactions are skipped as duplicates.
4. Bug (Low, fixed in SCRUM-140): Dashboard said "The AI wasn't confident" while AI categorization is off.
5. Bug (Low, open): Cleaned merchant names keep state codes ("Steam Purchase Wa").
6. Task (Low, open): Reword the consent screen's AI line, or enable AI categorization (SCRUM-129 to 131).
7. Bug (Low, open): Hide "Try again" for statements whose layout isn't supported.
8. Bug (Low, open): CSV masking merges an ATM sequence number with the street number.
