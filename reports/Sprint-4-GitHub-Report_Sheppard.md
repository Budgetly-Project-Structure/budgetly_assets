# Sprint 4 GitHub Report — Zachary Sheppard

**Sprint:** SCRUM Sprint 4 (21 Oct – 3 Nov 2026)
**GitHub username:** zsheppard78

**Repository URLs:**
- Web app (customer-facing): https://github.com/Budgetly-Project-Structure/budgetly_web_customer
- Backend (Firebase rules + Cloud Functions): https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs
- Contracts (shared schemas): https://github.com/Budgetly-Project-Structure/budgetly_contracts
- Supporting repo: [budgetly_assets](https://github.com/Budgetly-Project-Structure/budgetly_assets) (docs and this report)
- Admin dashboard: https://github.com/Budgetly-Project-Structure/budgetly_web_dash (no PRs this sprint)

---

## Pull Requests Authored by Me (Merged During Sprint)

Query used: `is:pr is:merged author:zsheppard78 merged:2026-10-21..2026-11-03` (run in each repo; data pulled from the GitHub REST API `pulls?state=closed` and `pulls/{n}/reviews`, merger cross-checked against the merge commits on `main`).
Titles are the PR titles; where a title was the default branch name, the Jira key is added from the commit subject. Reviewers: "Approved" = a formal GitHub approval on the PR; "No formal review (merged by X)" = review was requested but the teammate merged without leaving an approval. No closed-without-merge PRs in the sprint window.

### budgetly_web_customer

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 44 | Feature/scrum 121 restore import (SCRUM-121) | [PR #44](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/44) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 45 | SCRUM-120: Delete statement uses softDeleteImport and hides deleted statements | [PR #45](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/45) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 46 | Fixes money left over in dash | [PR #46](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/46) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 2, 2026 |
| 47 | SCRUM-135: Import page detects the account type and shows it | [PR #47](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/47) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |
| 48 | SCRUM-140: Dashboard hint no longer mentions AI | [PR #48](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/48) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |
| 49 | SCRUM-137: Budget sets itself at the start of each month | [PR #49](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/49) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |

### budgetly_backend_nodejs

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 19 | SCRUM-113: Firestore rules for the top level transactions collection | [PR #19](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/19) | zsheppard78 | tylerpetersen1 (Approved) | Oct 26, 2026 |
| 20 | Feature/soft delete rules (soft delete only for manual transactions and budgets) | [PR #20](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/20) | zsheppard78 | tylerpetersen1 (Approved) | Oct 26, 2026 |
| 21 | Scrum-112: Callables take the user id only from the auth token | [PR #21](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/21) | zsheppard78 | tylerpetersen1 (Approved) | Oct 26, 2026 |
| 27 | SCRUM-116: Migrate old transactions to the top-level collection | [PR #27](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/27) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 28 | Feature/scrum 120 soft delete import (SCRUM-120) | [PR #28](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/28) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 29 | Feature/scrum 121 restore import (SCRUM-121) | [PR #29](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/29) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 30 | Feature/scrum 119 admin scan (SCRUM-119) | [PR #30](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/30) | zsheppard78 | tylerpetersen1 (Approved) | Nov 1, 2026 |
| 32 | SCRUM-128: Emulator walkthrough of the full delete lifecycle | [PR #32](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/32) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 2, 2026 |
| 33 | Feature/scrum 122 purge statements (SCRUM-122) | [PR #33](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/33) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |
| 34 | SCRUM-135: Detect checking vs credit card from the statement file | [PR #34](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/34) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |
| 35 | SCRUM-137: Rules for budget source and the monthly budget record | [PR #35](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/35) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |
| 36 | SCRUM-140: Duplicate check matches CSV and PDF imports | [PR #36](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/36) | zsheppard78 | tylerpetersen1 (Approved) | Nov 2, 2026 |

### budgetly_contracts

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 4 | SCRUM-109: Define top-level transactions schema with userId | [PR #4](https://github.com/Budgetly-Project-Structure/budgetly_contracts/pull/4) | zsheppard78 | No formal review (merged by tylerpetersen1) | Oct 26, 2026 |

### budgetly_assets

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 5 | Feature/scrum 115 postman demo (SCRUM-115) | [PR #5](https://github.com/Budgetly-Project-Structure/budgetly_assets/pull/5) | zsheppard78 | tylerpetersen1 (Approved) | Nov 1, 2026 |
| 6 | added new file (data framework / trust model doc, SCRUM-110) | [PR #6](https://github.com/Budgetly-Project-Structure/budgetly_assets/pull/6) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 1, 2026 |
| 7 | SCRUM-128: Document the delete lifecycle walkthrough and gaps | [PR #7](https://github.com/Budgetly-Project-Structure/budgetly_assets/pull/7) | zsheppard78 | No formal review (merged by tylerpetersen1) | Nov 2, 2026 |

**Totals:** 22 PRs authored and merged (6 web, 12 backend, 1 contracts, 3 assets). All 22 were merged by my teammate and none were self-merged, compared with 13 of 22 self-merged in Sprint 3. 12 of the 22 have a formal GitHub approval; the other 10 had a review requested and were merged by tylerpetersen1 without an approval being recorded.

**PRs I reviewed/merged for my teammate:**
- budgetly_backend_nodejs, by tylerpetersen1, each approved and merged by me: [PR #22](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/22) (SCRUM-111: import pipeline writes to top-level transactions, Oct 29), [PR #23](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/23) (SCRUM-114: rules tests for ownership and impersonation, Oct 31), [PR #24](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/24) (SCRUM-127: daily purge of soft-deleted transactions and budgets, Oct 31), [PR #25](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/25) (SCRUM-136: skip already-imported transactions, Nov 1), [PR #26](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/26) (Categorize bookstores as Shopping, Nov 1), [PR #31](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/31) (Duplicate check ignores transactions from deleted statements, Nov 2).
- budgetly_web_customer [PR #36](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/36) (SCRUM-117: web app reads and writes transactions with userId), by tylerpetersen1, merged by me Oct 31 (no formal approval recorded).
- Total: 7 teammate PRs merged by me, 6 with a formal approval.

---

## Commit Activity

| Repo | My commits (non-merge, on `main`) |
|---|---|
| budgetly_web_customer | 6 |
| budgetly_backend_nodejs | 12 |
| budgetly_contracts | 1 |
| budgetly_assets | 3 |
| **Total** | **22** |

- **Date range:** Oct 26, 2026 (first commit) – Nov 2, 2026 (last commit), with commits on Oct 26, Nov 1 and Nov 2.
- Query: `git log origin/main --author=shepp014 --since=2026-10-21 --until=2026-11-04 --no-merges`.
- Every commit maps to one of the merged PRs above; no direct pushes to `main` this sprint. Commit subjects and branch names carry Jira keys (e.g. `feature/scrum-120-soft-delete-import`).

---

## Individual Contribution Summary

| Area | What I delivered | PRs | Jira |
|---|---|---|---|
| Top-level transactions schema | Contract for a top-level `transactions` collection keyed by `userId` | contracts #4 | SCRUM-109 |
| Data framework / trust model | `data-framework.md`: how user data is stored and deleted, soft-delete plan with retention and audit log | assets #6 | SCRUM-110 |
| Callable auth hardening | Callables take the user id only from the auth token, never the request | backend #21 | SCRUM-112 |
| Transactions rules | Firestore rules for the top-level transactions collection; soft delete allowed only for manual transactions and budgets | backend #19, #20 | SCRUM-113 |
| Impersonation demo | Postman collection and write-up showing the impersonation attempt is rejected | assets #5 | SCRUM-115 |
| Data migration | Migrate old per-user transactions to the top-level collection and close the old path | backend #27 | SCRUM-116 |
| Admin scan | Read-only admin scan for duplicate and unusually large transactions | backend #30 | SCRUM-119 |
| Soft delete statement | `softDeleteImport` callable with audit log entry; Delete button uses it and hides deleted statements | backend #28, web #45 | SCRUM-120 |
| Restore statement | `restoreImport` callable with audit log entry; restore from Recently deleted | backend #29, web #44 | SCRUM-121 |
| Purge statements | Purge deleted statements (record, file, leftovers) after 30 days | backend #33 | SCRUM-122 |
| Delete lifecycle walkthrough | Emulator walkthrough of upload → delete → restore → purge, plus doc of results and gaps | backend #32, assets #7 | SCRUM-128 |
| Account type detection | Detect checking vs credit card from the statement file; Import page shows it | backend #34, web #47 | SCRUM-135 |
| Automatic monthly budget | Rules for budget source and monthly budget record; budget sets itself at the start of each month | backend #35, web #49 | SCRUM-137 |
| Bug bash fixes | Duplicate check matches CSV and PDF imports (and hides account numbers in CSVs); dashboard hint no longer mentions AI | backend #36, web #48 | SCRUM-140 |
| Dashboard fix | "Left over" uses monthly income or shows a dash when no money came in | web #46 | — (bug fix) |

---

## Summary

Sprint 4 was about trust and data safety. I moved transactions to a top-level collection owned by `userId` (schema, rules, migration), locked callables to the auth token's user id, and replaced hard deletes with soft delete, restore and a 30-day purge, each with an audit log entry and verified end to end in an emulator walkthrough. I also added automatic account-type detection, an automatic monthly budget, and fixes from the bug bash. On process, every change went through a PR merged by my teammate (no self-merges, down from 13 in Sprint 3), and I reviewed and merged 7 of Tyler's PRs.
