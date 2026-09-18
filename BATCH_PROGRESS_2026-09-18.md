# BCI Engineering Batch — 2026-09-18

## Backend integrity continuation

The backend integrity pass continued with another attendance boundary repair.

- Attendance-session creation now locks the selected class row inside the transaction before checking for an existing session.
- Period labels are normalized before duplicate detection/storage.
- A duplicate session for the same term, class, subject, date, and period is rejected.
- Regression coverage verifies the PostgreSQL row-lock path and duplicate-session rejection.

Merged backend PR:
- #27 — `fix(attendance): prevent duplicate session creation races`

Backend CI and the PostgreSQL Database Contract workflow both passed for the repair before merge.

## Mobile staff workflow

The Flutter app now has a teacher attendance workflow against the authoritative backend contract.

- Teaching assignments expose their class, subject, term, room, and term-date scope.
- Teachers can open an attendance session from an assigned open-term class.
- The mobile client loads the server-authoritative roster and submits attendance in one bulk request.
- Closed terms disable attendance actions in the mobile UI.
- Expired access tokens now trigger a single-flight refresh for authenticated GET/POST/PATCH operations instead of failing only on non-GET writes.

Merged mobile PR:
- #3 — `feat(staff): add teacher attendance workflow and resilient auth refresh`

Flutter analysis and the full Flutter test suite passed before merge.

## Web portal reliability/parity

The internal portal now:
- Refreshes expired authenticated sessions once and retries the original request safely.
- Passes the subject scope required by the authoritative assessment-roster API.

Merged web portal PR:
- #3 — `fix(auth): refresh portal sessions and repair assessment roster scope`

Web Portal CI passed before merge.

## Database migration gate

The complete Prisma schema has already passed the clean PostgreSQL contract workflow and produced the review artifact `bci-initial-postgres-schema`.

The canonical initial Prisma migration is still **not** claimed as committed. The remaining step is to commit the verified generated SQL as the controlled initial migration and run a fresh `prisma migrate deploy` reconciliation gate. No live-school database should be treated as migrated until that step succeeds.

## Current execution focus

1. Commit and verify the canonical initial Prisma migration.
2. Complete remaining object/scope authorization review across finance, inventory, messaging, staff, payroll, wallet, and remaining academic mutations.
3. Complete mobile teacher assessment entry and academic-report parity.
4. Finalize configurable grading and report-card publication rules.
5. Continue timetable, payroll disbursement, wallet ledger, inventory, messaging/notifications, deployment, backups, monitoring, and restore-drill gates.
