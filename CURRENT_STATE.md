
Attendance sessions validate class/term academic-year consistency, session date inside term dates, term not closed, start/end ordering, and subject/class-level compatibility.

The roster endpoint returns the authoritative active class roster with existing attendance status, so clients do not need to invent or manually enumerate student IDs.

Attendance records can only be written for students with active enrolments in the session class and term. Duplicate student entries in one marking request are rejected. Changes are audited.

Guardians receive the `attendance.read` role capability, but `canViewAcademic` on the specific guardian-student link still controls whether that guardian can see a ward's attendance. Teachers can read attendance for students in their assigned active class/term; privileged school roles can read broader records.

### Assessments and reporting foundation

Assessments are implemented without database changes:

- `POST /api/v1/assessments`
- `POST /api/v1/assessments/:assessmentId/results`
- `GET /api/v1/assessments/students/:studentId?termId=...`
- `GET /api/v1/academic-reports/students/:studentId/current`
- `GET /api/v1/academic-reports/students/:studentId/terms/:termId`

Assessment creation requires teacher assignment to the subject/term and is limited to open terms. Result entry verifies assignment scope, active student enrolment in a class assigned for that subject/term, duplicate-student rejection, and score <= maxScore. Result updates are upserts and audited.

The academic-report endpoints are read-only and derive assessment percentages, subject averages, and an overall percentage when the term uses a consistent weighting policy. Fully weighted terms use weighted contribution; fully unweighted terms use an average percentage. Mixed weighted/unweighted results deliberately return `MIXED_POLICY_REQUIRED` rather than silently choosing an interpretation.

The current-term endpoint resolves the open term inside the current academic year on the server, so guardians do not need administrative term IDs.

The system now resolves official grades from a versioned, configurable grading policy. Active policies are scoped by academic year/level with optional programme override, validated for complete 0–100 coverage, published transactionally, and recorded with their policy version in report output. Published report-card snapshot persistence is now implemented; report-card correction/history beyond void-and-replace remains a later hardening gate.

Guardians have `assessments.read`, but report access still requires `canViewAcademic` on the specific guardian-student relationship. Teachers are scoped to the student's active class/term assignment.

The Flutter guardian app displays current-term academic results using the same server-derived report contract and does not calculate or invent grades locally.

### Database verification gate

Dedicated database-contract, PostgreSQL schema-contract, and Prisma migration-review workflows validate the complete schema and migration history against clean PostgreSQL and review migration deltas.

This workflow is a verification mechanism, not a production migration claim. The generated SQL has not been marked as the canonical production migration until the workflow actually executes successfully against the complete schema.

### Runtime/security foundation

A deterministic Prisma seed establishes role-permission defaults without fake school users or records.

Every HTTP response receives a server-generated `X-Request-Id` correlation identifier.

Startup validates `DATABASE_URL`, `JWT_ACCESS_SECRET`, `NODE_ENV`, `PORT`, and production `CORS_ORIGINS` before listening. HTTP requests emit structured timing/status logs. A dependency-free in-process limiter protects authentication and public application endpoints; distributed rate limiting remains a production gate before horizontal scaling.

The Prisma schema baseline was restored from the last complete Git blob after a reviewed schema-edit attempt was found to have truncated the file. Finance, payment-provider, wallet, inventory, notification, audit, attendance, assessment, staff, and payroll models are confirmed present again. No migration was generated from the truncated version.

### Public/authenticated backend endpoints

Admissions:
- `POST /api/v1/applications`
- `GET /api/v1/applications/track/:trackingCode`
- `GET /api/v1/applications`
- `POST /api/v1/applications/:id/review`
- `POST /api/v1/applications/:id/admit`

Student/guardian:
- `GET /api/v1/students/me/wards`
- `GET /api/v1/students/:id`
- `GET /api/v1/guardians/me/profile`
- `PATCH /api/v1/guardians/me/profile`
- `POST /api/v1/students/:id/guardians`
- `DELETE /api/v1/students/:id/guardians/:guardianId`
- `POST /api/v1/students/:id/withdraw`

Staff:
- `GET /api/v1/staff/me`
- `GET /api/v1/staff/directory`
- `GET /api/v1/staff/:staffPersonId/assignments`
- `POST /api/v1/staff/:staffPersonId/duties`
- `POST /api/v1/staff/:staffPersonId/teacher-assignments`

Payroll:
- `GET /api/v1/payroll/me`
- `GET /api/v1/payroll/periods`
- `GET /api/v1/payroll/periods/:periodId/entries`

Finance:
- `GET /api/v1/finance/fee-schedules?termId=...`
- `POST /api/v1/finance/fee-schedules`
- `POST /api/v1/finance/invoices`
- `GET /api/v1/finance/students/:studentId/invoices`
- `POST /api/v1/finance/students/:studentId/payments`
- `POST /api/v1/finance/students/:studentId/payments/:paymentId/otp`
- `GET /api/v1/finance/students/:studentId/receipts`

Wallet:
- `GET /api/v1/wallets/students/:studentId`
- wallet top-up initiation and OTP continuation
- controlled withdrawal/reversal operations

Attendance:
- `POST /api/v1/attendance/sessions`
- `GET /api/v1/attendance/sessions/:sessionId/roster`
- `POST /api/v1/attendance/sessions/:sessionId/records`
- `GET /api/v1/attendance/students/:studentId?termId=...`

Assessments/reports:
- `POST /api/v1/assessments`
- `POST /api/v1/assessments/:assessmentId/results`
- `GET /api/v1/assessments/students/:studentId?termId=...`
- `GET /api/v1/academic-reports/students/:studentId/current`
- `GET /api/v1/academic-reports/students/:studentId/terms/:termId`

Grading policies:
- `GET /api/v1/grading-policies`
- `POST /api/v1/grading-policies`
- `PATCH /api/v1/grading-policies/:id`
- `POST /api/v1/grading-policies/:id/publish`
- `POST /api/v1/grading-policies/:id/retire`

The Prisma model uses a dedicated application tracking code rather than exposing application UUIDs as public lookup credentials. Provider payment attempts, webhook events, signed wallet effects and withdrawal evidence have durable models for reconciliation.

### Web portal
`bci-web-portal` now has the staff sign-in surface, server-authoritative session verification, a staff workspace showing the authenticated employee's active duties/teaching assignments, authenticated admissions workspace, application list/review controls, public tracking-code lookup, and a server-driven admission placement workflow.

The portal uses server-returned permission codes. The staff workspace only loads for accounts with `staff.read` and is sourced from `GET /api/v1/staff/me`. The portal also has progression, grading-policy administration, and report-card publication controls for authorized staff.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell, secure token storage, shared auth login/refresh/logout, session restoration through `/auth/me`, an authenticated guardian dashboard loading `/students/me/wards`, a read-only current-term academic-results page for wards whose relationship has `canViewAcademic=true`, a read-only fee/invoice review page for wards whose relationship has `canPayFees=true`, and a wallet page for wards whose relationship has `canManageWallet=true` that supports mobile-money top-up initiation, OTP continuation, and signed transaction display.

### Public website
`bci-website` has a public BCI shell and admissions form concept wired to the shared backend application endpoint and authoritative tracking-code response.

## Engineering hygiene

GitHub Issues are not the default implementation journal during foundation work. Active work is tracked by canonical docs and repository commits; issues are opened only for bounded reviewable tasks when useful.

Backend, web portal, mobile, and public website CI run only on pull requests or intentional manual dispatch. Direct pushes to `main` do not trigger these workflows. Backend, Flutter and Web workflows also cancel superseded validations so rapid changes do not leave obsolete runs competing for attention.

Backend CI validates Prisma, generates the client, compiles, and runs Jest. The dedicated database-contract workflow additionally boots PostgreSQL 16 and checks schema-to-database equivalence. Web CI builds the portal. Flutter CI runs analysis/tests. Website CI builds the public site.

Organization-wide workflow scanning found no remaining `push:` trigger in the BCI repositories.

The wallet, grading-policy, progression, report-card publication, and PaymentIntent reservation slices have been validated and merged into their respective mainlines. Remaining work is tracked below.

## Open blockers before production

1. Verify the live Moolre API contract before enabling real-money provider operation.
2. Complete remaining object/scope authorization and cross-channel parity across finance, inventory, messaging, staff, payroll, wallet, attendance, assessments and report-card history.
3. Replace the bootstrap in-process rate limiter with distributed protection before running multiple API instances.
4. Complete remaining guardian/student lifecycle mutations, including verified login-identifier changes and transfer/progression workflows. Intra-term transfer history still needs a dedicated relational history model; the current `Enrolment` uniqueness model has intentionally not been weakened.
7. Complete successful payment allocation, receipts, refunds, immutable journal posting, and reconciliation before finance goes live.
8. Complete wallet reconciliation/operational review and preserve the signed ledger/withdrawal evidence model.
9. Expand attendance roster/teacher/mobile UX and reporting.
10. Complete report-card correction/history workflows and durable delivery/exports around the published snapshot.
11. Establish timetable schema/versioning and conflict validation.
12. Build payroll write/approval/disbursement workflows only after financial verification.
13. Build inventory, messaging, notification, deployment, secrets, backups, restore drills, and production monitoring.

## Current next execution order

1. Verify the live Moolre API contract and final finance reconciliation/journal requirements.
2. Complete remaining object/scope authorization audit and web/mobile academic parity for attendance, assessments, academic reports, and grading.
3. Build remaining staff web/mobile operational workflows.
4. Complete report-card correction/history and durable delivery/exports.
5. Expand attendance roster/teacher/mobile UX and reporting.
6. Report-card correction/history and publishing operations.
7. Payroll approval/disbursement.
8. Finance production hardening after provider verification, expiry reconciliation, receipts/refunds and journal/reconciliation completion.
9. Inventory.
10. Communication/notifications.
11. Reporting and production hardening.