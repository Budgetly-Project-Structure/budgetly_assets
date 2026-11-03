# Sprint 4 Jira Report

**Jira Project:** Software Engineering (Financial Planner)
**Jira Link:** https://team3csci675.atlassian.net/jira/software/projects/SCRUM/boards/1
**Sprint:** SCRUM Sprint 4 (26 Oct – 3 Nov 2026 for the class; the Jira sprint is set to 26 Oct – 9 Nov)
**Sprint Goal:** Every user's transactions live in one secure top-level collection that nobody can read or write as another user, and deleting data becomes reversible and recorded.

---

## Sprint Commitment vs Delivery

| Metric | Count |
|---|---|
| Issues committed at sprint start | 32 (SCRUM-109 – 140) |
| Issues completed | 27 |
| Issues not completed | 5 (SCRUM-128, 129, 130, 131, 133) |
| Issues added mid-sprint | 0 |

---

## Issue Breakdown by Type

All work items in the SCRUM project are created as Tasks, the same as in Sprints 2 and 3.

| Type | To Do | In Progress | Done |
|---|---|---|---|
| Story | 0 | 0 | 0 |
| Task | 5 | 0 | 27 |
| Bug | 0 | 0 | 0 |

| Epic | Issues | Done |
|---|---|---|
| Secure Transactions | 11 (SCRUM-109 – 119) | 11 |
| Soft Delete and Audit Log | 9 (SCRUM-120 – 128) | 8 |
| More Automation | 12 (SCRUM-129 – 140) | 8 |

---

## Per-Student Work Allocation

| Student | Issues Assigned | Issues Completed | Story Points Completed |
|---|---|---|---|
| ZS (Zachary Sheppard) | 18 | 14 | 32 of 43 |
| TP (Tyler Petersen) | 14 | 13 | 33 of 35 |

- **ZS:** transactions schema with userId (SCRUM-109), trust model doc answering "where does userId come from" (110), callables that take the uid only from the auth token (112), Firestore rules for the top-level collection (113), Postman impersonation demo (115), migration of existing transactions (116), cross-user admin scan (119), softDeleteImport and restoreImport (120, 121), purging soft-deleted statements after 30 days (122), audit log entries (123), soft-delete rules (124), account type detection (135) and the self-setting monthly budget (137).
- **TP:** import pipeline writing to the top-level collection (SCRUM-111), ownership and impersonation rules tests (114), web app reads and writes with userId (117), merchant cache trigger move (118), Recently deleted with Restore (125), Undo toast (126), soft-delete unit tests (127), mocked AI categorization tests (132), Needs review list (134), duplicate row skipping (136), search and filter (138) and consistent loading and error feedback (139).
- **Note:** SCRUM-140 (bug bash) is assigned to TP in Jira, but ZS ran it and fixed the issues it found.

---

## Estimation & Accuracy

| Metric | Value |
|---|---|
| Total story points committed | 78 (ZS 43, TP 35) |
| Total story points completed | 65 |
| Completion % | 83% (65 / 78) |

Ticket sizes stayed small: every issue was estimated at 1–3 points.

---

## Workflow Discipline

- [ ] Issues moved through workflow states (To Do → In Progress → Done). Not consistently: most issues went straight from To Do to Done when their PRs merged, and some (SCRUM-123, 124) were only moved to Done at the end of the sprint.
- [x] Issues closed only after acceptance criteria met. Each issue was moved to Done only after its PR was merged and the change was deployed and checked on the live site.
- [ ] Sprint completed/closed in Jira (to be closed after the Sprint 4 demo; the 5 unfinished issues move to Sprint 5).

---

## Blockers & Scope Changes

- **Major blockers:**
  - The Gemini AI chain (SCRUM-129 – 131) needs a free API key from a separate Google project with no billing, so it can never charge us. That project wasn't set up in time, so the free keyword rules and merchant memory still do the categorizing.
  - The new rules for soft delete would have broken "Remove budget" in the live web app, so deploying them had to wait until the web app switched to soft delete (SCRUM-126). We then deployed in order: indexes, rules, functions, then hosting.
  - The new callable functions needed the Cloud Run invoker permission before they worked on the live project.
- **Scope changes:**
  - After the professor's feedback, the sprint was built around one top-level `transactions` collection with a `userId` field instead of per-user subcollections, plus soft delete instead of hard delete.
  - The plan had 34 issues. Two were folded into other tickets before the sprint started (the Firestore indexes ticket and the soft-delete schema ticket), so 32 were committed.
  - The bug bash (SCRUM-140) found one medium issue, bank accounts that should be per user, which was written up as a new story for Sprint 5.
- **Why work spilled over:** SCRUM-129 – 131 were blocked on the free Gemini key, and SCRUM-128 (emulator walkthrough) and SCRUM-133 (shared merchant memory) ran out of time. All 5 move to Sprint 5.

---

## Jira Evidence Links

- **Sprint report link:** https://team3csci675.atlassian.net/jira/software/projects/SCRUM/boards/1/reports/sprint-retrospective
- **Backlog link:** https://team3csci675.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog
- **Board link (filtered to sprint):** https://team3csci675.atlassian.net/jira/software/projects/SCRUM/boards/1
