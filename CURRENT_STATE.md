# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
`bci-docs` is the canonical engineering source of truth for architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release criteria.

### Backend
`bci-backend-api` has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, JWT authentication/session rotation, admissions, academic-structure, student lifecycle, finance, attendance, assessments, derived academic-report, staff-operation, read-only payroll, and read-only wallet modules.

Authorization has a canonical permission catalog plus `RolePermission` for default capabilities and `UserPermission` for explicit exceptions. The permission guard resolves both role defaults and direct grants. Staff now have explicit `staff.read` and `staff.manage` permissions; wallet access has explicit `wallet.read` and `wallet.manage` permission codes.

`GET /api/v1/auth/me` returns the server-authoritative current-user profile, roles, effective permissions, direct scoped grants, and linked guardian/staff profile where applicable.

Admissions endpoints use explicit permission checks. Admission placement validates academic-year/term/class consistency, term dates/status, application level/programme compatibility, class capacity, and admission-number uniqueness. Focused tests cover rejected and successful paths, including atomic student/guardian/enrolment/decision/audit creation.

Student reads require `students.read` and apply object scope. Guardians may read only their wards; privileged office/leadership roles may read broader student records; teachers may read only students in assigned classes/terms. Student documents remain restricted to privileged staff.

Guardian relationship permissions are enforced: `canViewAcademic` controls academic visibility, while `canPayFees` and `canManageWallet` are explicit link-level capabilities for finance/wallet workflows.

Staff with `students.manage` can create/remove guardian links. Linking can designate a primary contact and set guardian link capabilities. Link creation/removal is transactional and audited; duplicate links are rejected.

A controlled withdrawal transition uses `students.manage`, closes the active enrolment, marks the student withdrawn, records the reason, and writes an audit record atomically.

Guardian self-profile endpoints allow updates to non-login profile fields and notification preferences. Phone/email login identifiers remain read-only until a verified change flow exists.

Academic endpoints use explicit `academics.read` / `academics.manage` permissions. Teachers receive only assigned classes; privileged academic roles can browse broader class structure.

### Staff/teacher operations

The staff module provides:

- `GET /api/v1/staff/me` for an authenticated staff member's profile, active duties, and teaching assignments.
- `GET /api/v1/staff/directory` for authorized internal staff directory access.
- `GET /api/v1/staff/:staffPersonId/assignments` for appropriately scoped assignment reads.
- `POST /api/v1/staff/:staffPersonId/duties` for staff-duty assignment.
- `POST /api/v1/staff/:staffPersonId/teacher-assignments` for class/subject/term assignment.

Teacher-assignment creation validates the class and term share the same academic year, the subject level matches the class level, SHS programme compatibility, open-term status, duplicate assignment protection, and writes an audit record. Focused tests cover academic-year mismatch.

The staff web workspace displays the authenticated employee's duties and teaching assignments from the authoritative backend.

A timetable database entity does not yet exist in the restored Prisma schema. `TIMETABLE_DESIGN.md` defines the proposed timetable/version/period model and conflict invariants as an explicit schema gate rather than improvising timetable columns on teacher assignments.

### Payroll read foundation

The backend exposes read-only payroll information:

- `GET /api/v1/payroll/me` for the authenticated staff member's current salary structure, payroll entries, and disbursement status history.
- `GET /api/v1/payroll/periods` for authorized director/principal/accountant payroll-period summaries.
- `GET /api/v1/payroll/periods/:periodId/entries` for authorized payroll-period entry detail.

Payroll writes, salary changes, approvals, and staff disbursements are deliberately not exposed through these endpoints. No provider transfer is initiated by the payroll module.

### Finance foundation

The first finance slice is implemented without unsafe provider behavior:

- `GET /api/v1/finance/fee-schedules?termId=...`
- `POST /api/v1/finance/fee-schedules`
- `POST /api/v1/finance/invoices`
- `GET /api/v1/finance/students/:studentId/invoices`

Invoices are issued transactionally from active fee schedules that match the student's active enrolment, term, level, and programme. A second open/partially-paid invoice for the same student/term is rejected. Invoice balances are derived from invoice lines and payment allocations; there is no mutable `amountPaid` field.

Guardians can read invoices only when their guardian-student link has `canPayFees=true`. Finance staff use server-side finance permissions.

Payment-intent creation is intentionally **not** exposed yet. `PAYMENT_INTENT_DESIGN.md` defines the required reservation, expiry, idempotency, concurrency, callback, allocation and receipt invariants. The preferred design is a dedicated payment-intent/reservation record linked to the target invoice and eventual financial payment.

Moolre adapter/callback code has not been introduced. Provider behavior must be verified before integration is implemented. The live contract is not being guessed from static assumptions.

The Flutter guardian app exposes a read-only fee view for wards whose relationship has `canPayFees=true`. It shows server-derived invoice totals, allocated amounts, outstanding balance, due date and invoice lines. It deliberately has no payment button until the reservation/provider flow is production-safe.

### Wallet foundation

The backend exposes a read-only wallet statement endpoint:

- `GET /api/v1/wallets/students/:studentId`

Access requires `wallet.read` and is then constrained by guardian-student `canManageWallet` for guardians. Director/principal/office/accountant roles have broader read access.

The endpoint intentionally does **not** calculate or report an authoritative wallet balance yet. The current `WalletTransaction.REVERSAL` model lacks an original-transaction reference/direction, so deriving a balance from type alone could be financially incorrect. `WALLET_LEDGER_DESIGN.md` defines the required signed-effect, reversal-reference, provider-linkage, concurrency, and office-dispense invariants.

Top-ups, physical withdrawals, reversals, and wallet adjustments are not exposed yet.

The Flutter guardian app now has a read-only wallet statement view for wards whose relationship has `canManageWallet=true`. It shows transaction history and explicitly reports that balance calculation is pending the ledger policy; it does not fabricate a zero balance.

### Attendance foundation

The attendance slice is implemented and permission/scope constrained:

- `POST /api/v1/attendance/sessions`
- `GET /api/v1/attendance/sessions/:sessionId/roster`
- `POST /api/v1/attendance/sessions/:sessionId/records`
- `GET /api/v1/attendance/students/:studentId?termId=...`

Teacher session creation, roster access, and marking are checked against the teacher's class/term/subject assignment. Privileged school roles can manage attendance more broadly.

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

The system does not yet assign official grades. Grade bands/cutoffs are intentionally not hard-coded; a configurable school grading policy must be established before report cards can publish grades.

Guardians have `assessments.read`, but report access still requires `canViewAcademic` on the specific guardian-student relationship. Teachers are scoped to the student's active class/term assignment.

The Flutter guardian app displays current-term academic results using the same server-derived report contract and does not calculate or invent grades locally.

### Database verification gate

A dedicated `.github/workflows/database-contract.yml` now runs only on pull requests or deliberate manual dispatch. It starts PostgreSQL 16, validates and generates Prisma, generates the initial SQL from the complete Prisma schema with `prisma migrate diff --from-empty`, applies that SQL to the temporary database, compares the live database back against the schema, then builds/tests the backend and uploads the generated SQL as a seven-day artifact.

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

Wallet:
- `GET /api/v1/wallets/students/:studentId`

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

The Prisma model uses a dedicated application tracking code rather than exposing application UUIDs as public lookup credentials. Provider payment attempts and webhook events have durable models for future reconciliation.

### Web portal
`bci-web-portal` now has the staff sign-in surface, server-authoritative session verification, a staff workspace showing the authenticated employee's active duties/teaching assignments, authenticated admissions workspace, application list/review controls, public tracking-code lookup, and a server-driven admission placement workflow.

The portal uses server-returned permission codes. The staff workspace only loads for accounts with `staff.read` and is sourced from `GET /api/v1/staff/me`.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell, secure token storage, shared auth login/refresh/logout, session restoration through `/auth/me`, an authenticated guardian dashboard loading `/students/me/wards`, a read-only current-term academic-results page for wards whose relationship has `canViewAcademic=true`, a read-only fee/invoice page for wards whose relationship has `canPayFees=true`, and a read-only wallet statement page for wards whose relationship has `canManageWallet=true`.

### Public website
`bci-website` has a public BCI shell and admissions form concept wired to the shared backend application endpoint and authoritative tracking-code response.

## Engineering hygiene

GitHub Issues are not the default implementation journal during foundation work. Active work is tracked by canonical docs and repository commits; issues are opened only for bounded reviewable tasks when useful.

Backend, web portal, mobile, and public website CI run only on pull requests or intentional manual dispatch. Direct pushes to `main` do not trigger these workflows.

Backend CI validates Prisma, generates the client, compiles, and runs Jest. The dedicated database-contract workflow additionally boots PostgreSQL 16 and checks schema-to-database equivalence. Web CI builds the portal. Flutter CI runs analysis/tests. Website CI builds the public site.

Organization-wide workflow scanning found no remaining `push:` trigger in the BCI repositories.

There are currently no open pull requests or open issues in the BCI organization.

## Open blockers before production

1. Execute and review the database-contract workflow successfully, then establish the canonical initial Prisma migration from the verified complete schema.
2. Complete remaining object/scope authorization across finance, inventory, messaging, staff, payroll, wallet, and remaining academic workflows.
3. Replace the bootstrap in-process rate limiter with distributed protection before running multiple API instances.
4. Complete remaining guardian/student lifecycle mutations, including verified login-identifier changes and transfer/progression workflows. Intra-term transfer history still needs a dedicated relational history model; the current `Enrolment` uniqueness model has intentionally not been weakened.
5. Finalize the payment-intent invoice-target/reservation schema and only then expose payment creation.
6. Verify the live Moolre API contract before implementing provider adapters and reconciliation workers.
7. Build successful payment allocation, receipts, refunds, immutable journal posting, and reconciliation before finance goes live.
8. Establish definitive wallet ledger/reversal semantics before calculating or mutating wallet balances.
9. Expand attendance roster/teacher/mobile UX and reporting.
10. Establish configurable grading rules and full report-card publication workflows.
11. Establish timetable schema/versioning and conflict validation.
12. Build payroll write/approval/disbursement workflows only after financial verification.
13. Build inventory, messaging, notification, deployment, secrets, backups, restore drills, and production monitoring.

## Current next execution order

1. Execute/review database contract verification and establish the canonical Prisma migration.
2. Complete web/mobile academic parity for attendance, assessments, and academic reports.
3. Build remaining staff web/mobile operational workflows.
4. Finalize payment reservation schema and reconciliation design.
5. Finance payment/receipt foundation after schema verification and provider-contract verification.
6. Configurable grading/report-card policy.
7. Payroll approval/disbursement.
8. Wallet ledger schema and controlled top-up/withdrawal workflows.
9. Inventory.
10. Communication/notifications.
11. Reporting and production hardening.
