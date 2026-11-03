# Sprint 4 GitHub Report — Tyler Petersen

**Sprint:** SCRUM Sprint 4 (21 Oct – 3 Nov 2026)
**GitHub username:** tylerpetersen1

**Repository URLs:**
- Backend (Firebase rules + Cloud Functions): https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs
- Web app (customer-facing): https://github.com/Budgetly-Project-Structure/budgetly_web_customer
- Contracts and assets (PRs reviewed and merged): https://github.com/Budgetly-Project-Structure/budgetly_contracts, https://github.com/Budgetly-Project-Structure/budgetly_assets

---

## Pull Requests Authored by Me (Merged During Sprint)

Query used: `is:pr is:merged author:tylerpetersen1 merged:2026-10-21..2026-11-03` (run in each repo; data from the GitHub REST API `pulls?state=closed` and `pulls/{n}/reviews`, merger cross-checked against the merge commits on `main`).
Reviewers: "Approved" = a formal GitHub approval on the PR; "No formal review (merged by X)" = the teammate merged without leaving an approval; "None (self-merged)" = I merged my own PR. No closed-without-merge PRs in the sprint window.

### budgetly_web_customer

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 36 | SCRUM-117: web app reads and writes transactions with userId | [PR #36](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/36) | tylerpetersen1 | No formal review (merged by zsheppard78) | Oct 31, 2026 |
| 37 | SCRUM-125: recently deleted section with restore on the Import page | [PR #37](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/37) | tylerpetersen1 | None (self-merged) | Oct 31, 2026 |
| 38 | SCRUM-126: undo toast after deleting a transaction or removing a budget | [PR #38](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/38) | tylerpetersen1 | None (self-merged) | Oct 31, 2026 |
| 39 | Show cleaned-up merchant names for imported transactions | [PR #39](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/39) | tylerpetersen1 | None (self-merged) | Oct 31, 2026 |
| 40 | SCRUM-139: consistent loading, success and error feedback | [PR #40](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/40) | tylerpetersen1 | None (self-merged) | Nov 1, 2026 |
| 41 | Hide Try again when every row was already imported | [PR #41](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/41) | tylerpetersen1 | None (self-merged) | Nov 1, 2026 |
| 42 | Month filter lists the months that have transactions | [PR #42](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/42) | tylerpetersen1 | None (self-merged) | Nov 1, 2026 |
| 43 | Insights uses the newest month with data | [PR #43](https://github.com/Budgetly-Project-Structure/budgetly_web_customer/pull/43) | tylerpetersen1 | None (self-merged) | Nov 1, 2026 |

### budgetly_backend_nodejs

| # | PR Title | Link | Author | Reviewers | Merge Date |
|---|---|---|---|---|---|
| 22 | SCRUM-111: import pipeline writes to the top-level transactions collection | [PR #22](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/22) | tylerpetersen1 | zsheppard78 (Approved) | Oct 29, 2026 |
| 23 | SCRUM-114: rules tests for ownership and impersonation | [PR #23](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/23) | tylerpetersen1 | zsheppard78 (Approved) | Oct 31, 2026 |
| 24 | SCRUM-127: daily purge of soft-deleted transactions and budgets, with tests | [PR #24](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/24) | tylerpetersen1 | zsheppard78 (Approved) | Oct 31, 2026 |
| 25 | SCRUM-136: skip transactions already imported from an overlapping statement | [PR #25](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/25) | tylerpetersen1 | zsheppard78 (Approved) | Nov 1, 2026 |
| 26 | Categorize bookstores as Shopping | [PR #26](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/26) | tylerpetersen1 | zsheppard78 (Approved) | Nov 1, 2026 |
| 31 | Duplicate check ignores transactions from deleted statements | [PR #31](https://github.com/Budgetly-Project-Structure/budgetly_backend_nodejs/pull/31) | tylerpetersen1 | zsheppard78 (Approved) | Nov 2, 2026 |

**Totals:** 14 PRs authored and merged (8 web, 6 backend). All 6 backend PRs were approved and merged by my teammate. On the web side, PR #36 was merged by my teammate without a formal approval, and PRs #37–43 were self-merged.

**PRs I reviewed/merged for my teammate (zsheppard78):** all 22 of his merged Sprint 4 PRs, 12 of them with a formal approval:
- budgetly_backend_nodejs: approved and merged #19, #20, #21, #30, #33, #34, #35, #36; merged #27, #28, #29, #32.
- budgetly_web_customer: approved and merged #47, #48, #49; merged #44, #45, #46.
- budgetly_contracts: merged #4.
- budgetly_assets: approved and merged #5; merged #6, #7.

---

## Commit Activity

| Repo | My commits (non-merge, on `main`) |
|---|---|
| budgetly_backend_nodejs | 6 |
| budgetly_web_customer | 8 |
| **Total** | **14** (all through merged PRs) |

- **Date range:** Oct 29, 2026 (first commit) – Nov 1, 2026 (last commit), with commits on Oct 29, 30, 31 and Nov 1.
- Query: `git log origin/main --author=tylerpetersen --since=2026-10-21 --until=2026-11-04 --no-merges`.
- Every commit maps to one of the merged PRs above; no direct pushes to `main` this sprint. Ticket work references its Jira key in the commit subject and branch name (e.g. `feature/scrum-127-purge-job`).

---

## Individual Contribution Summary

| Area | What I delivered | PRs | Jira |
|---|---|---|---|
| Top-level transactions (backend) | Import pipeline writes to the top-level `transactions` collection with `userId` | backend #22 | SCRUM-111 |
| Top-level transactions (web) | Web app reads and writes transactions with `userId` | web #36 | SCRUM-117 |
| Rules tests | Emulator tests for ownership and impersonation on the new collection | backend #23 | SCRUM-114 |
| Recently deleted | Recently deleted section with restore on the Import page | web #37 | SCRUM-125 |
| Undo | Undo toast after deleting a transaction or removing a budget | web #38 | SCRUM-126 |
| Purge job | Daily purge of soft-deleted transactions and budgets, with tests | backend #24 | SCRUM-127 |
| Duplicate imports | Skip transactions already imported from an overlapping statement; duplicate check ignores deleted statements; hide "Try again" when every row was a duplicate | backend #25, #31, web #41 | SCRUM-136 |
| Feedback | Consistent loading, success and error feedback across pages | web #40 | SCRUM-139 |
| Polish and fixes | Cleaned-up merchant names; bookstores categorized as Shopping; month filter lists months with data; Insights uses the newest month with data | web #39, #42, #43, backend #26 | — (fixes) |
| Code review | Reviewed and merged all 22 of my teammate's Sprint 4 PRs (12 formal approvals) | — | — |

---

## Summary

My Sprint 4 work moved the import pipeline and web app onto the top-level `transactions` collection, backed by ownership and impersonation tests, and built the user side of soft delete: a Recently deleted section with restore, an undo toast, and a daily purge job. I also stopped overlapping statements from creating duplicate transactions and made loading, success and error feedback consistent. Every commit went through a PR, all of my backend PRs were approved by my teammate, and I reviewed and merged all 22 of my teammate's PRs.
