# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
`bci-docs` is the canonical engineering source of truth for architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release criteria.

### Backend
`bci-backend-api` has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, JWT authentication/session rotation, admissions, academic-structure, student lifecycle, and initial finance modules.

Authorization has a canonical permission catalog plus `RolePermission` for default capabilities and `UserPermission` for explicit exceptions. The permission guard resolves both role defaults and direct grants.

`GET /api/v1/auth/me` returns the server-authoritative current-user profile, roles, effective permissions, direct scoped grants, and linked guardian/staff profile where applicable.

Admissions endpoints use explicit permission checks. Admission placement validates academic-year/term/class consistency, term dates/status, application level/programme compatibility, class capacity, and admission-number uniqueness. Focused tests cover rejected and successful paths, including atomic student/guardian/enrolment/decision/audit creation.

Student reads require `students.read` and apply object scope. Guardians may read only their wards; privileged office/leadership roles may read broader student records; teachers may read only students in assigned classes/terms. Student documents remain restricted to privileged staff.

Guardian relationship permissions are enforced: `canViewAcademic` controls academic visibility, while `canPayFees` and `canManageWallet` are explicit link-level capabilities for finance/wallet workflows.

Staff with `students.manage` can create/remove guardian links. Linking can designate a primary contact and set guardian link capabilities. Link creation/removal is transactional and audited; duplicate links are rejected.

A controlled withdrawal transition uses `students.manage`, closes the active enrolment, marks the student withdrawn, records the reason, and writes an audit record atomically.

Guardian self-profile endpoints allow updates to non-login profile fields and notification preferences. Phone/email login identifiers remain read-only until a verified change flow exists.

Academic endpoints use explicit `academics.read` / `academics.manage` permissions. Teachers receive only assigned classes; privileged academic roles can browse broader class structure.

A deterministic Prisma seed establishes role-permission defaults without creating fake school users or records.

Every HTTP response receives a server-generated `X-Request-Id` correlation identifier.

Startup validates `DATABASE_URL`, `JWT_ACCESS_SECRET`, `NODE_ENV`, `PORT`, and production `CORS_ORIGINS` before listening. HTTP requests emit structured timing/status logs. A conservative dependency-free in-process limiter protects authentication and public application endpoints; distributed rate limiting remains a production gate before horizontal scaling.

### Finance foundation

The first finance slice is implemented without unsafe provider behavior:

- `GET /api/v1/finance/fee-schedules?termId=...` for authorized finance staff.
- `POST /api/v1/finance/fee-schedules` for authorized finance managers.
- `POST /api/v1/finance/invoices` to issue a student invoice from validated term/level/programme-matching fee schedules.
- `GET /api/v1/finance/students/:studentId/invoices` for finance staff or guardians whose ward link has `canPayFees=true`.

Invoice issuance is transactional and validates active student/enrolment, open term, matching fee schedule level/programme, unique active fee IDs, and absence of another open/partially-paid invoice for that student/term.

Invoice balances are derived from invoice lines and payment allocations; there is no mutable `amountPaid` field.

Focused finance tests cover fee/enrolment mismatch, duplicate open-invoice prevention, guardian `canPayFees` denial, and privileged finance access.

Payment-intent creation is intentionally **not** exposed yet. The current schema needs an explicit invoice-target/reservation relationship so concurrent pending payment attempts cannot over-commit an invoice balance. This gate is documented in `PAYMENT_INTENT_DESIGN.md`.

Moolre adapter/callback code has not been introduced. Provider behavior must be verified before integration is implemented.

The Prisma schema baseline was restored from the last complete Git blob after a reviewed schema-edit attempt was found to have truncated the file. Finance, payment-provider, wallet, inventory, notification, and audit models are confirmed present again. No migration was generated from the truncated version.

### Public/authenticated backend endpoints

Current public admissions endpoints:

- `POST /api/v1/applications`
- `GET /api/v1/applications/track/:trackingCode`

Authenticated staff admissions endpoints include:

- `GET /api/v1/applications`
- `POST /api/v1/applications/:id/review`
- `POST /api/v1/applications/:id/admit`

Authenticated student/guardian reads include:

- `GET /api/v1/students/me/wards`
- `GET /api/v1/students/:id`

Guardian profile endpoints include:

- `GET /api/v1/guardians/me/profile`
- `PATCH /api/v1/guardians/me/profile`

Student lifecycle includes:

- `POST /api/v1/students/:id/guardians`
- `DELETE /api/v1/students/:id/guardians/:guardianId`
- `POST /api/v1/students/:id/withdraw`

Finance endpoints include:

- `GET /api/v1/finance/fee-schedules?termId=...`
- `POST /api/v1/finance/fee-schedules`
- `POST /api/v1/finance/invoices`
- `GET /api/v1/finance/students/:studentId/invoices`

Admission identity linking is hardened: an existing guardian account may be linked by phone, but a non-guardian account cannot be silently attached. Pre-created guardian Person records are reused during later guardian registration instead of duplicated.

The Prisma model uses a dedicated application tracking code rather than exposing the application UUID as the public lookup credential. Provider payment attempts and webhook events have durable models for future reconciliation.

### Web portal
`bci-web-portal` has a staff sign-in surface, authenticated admissions workspace, application list/review controls, public tracking-code lookup, and a server-driven admission placement workflow.

The portal verifies the access token through `/auth/me`. Application list/review/admit controls use server-returned permission codes. Admission placement loads academic years/classes from the backend, filters to open terms and matching classes, and sends authoritative IDs back to the admission endpoint.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell, secure token storage, shared auth login/refresh/logout, session restoration through `/auth/me`, and an authenticated guardian dashboard loading `/students/me/wards`. Guardian relationship permissions are surfaced per ward.

### Public website
`bci-website` has a public BCI shell and admissions form concept wired to the shared backend application endpoint and authoritative tracking-code response.

## Engineering hygiene

GitHub Issues are not the default implementation journal during foundation work. Active work is tracked by canonical docs and repository commits; issues are opened only for bounded reviewable tasks when useful.

Backend, web portal, mobile, and public website CI run only on pull requests or intentional manual dispatch. Direct pushes to `main` do not trigger these workflows.

Backend CI validates Prisma, generates the client, compiles, and runs Jest. Web CI builds the portal. Flutter CI runs analysis/tests. Website CI builds the public site.

Organization-wide workflow scanning found no remaining `push:` trigger in the BCI repositories.

There are currently no open pull requests or open issues in the BCI organization.

## Open blockers before production

1. Generate and verify the initial Prisma migration from the complete hardened schema and establish a repeatable PostgreSQL verification path.
2. Complete remaining object/scope authorization across attendance, assessments, finance, inventory, messaging, and staff workflows.
3. Replace the bootstrap in-process rate limiter with distributed protection before running multiple API instances.
4. Complete remaining guardian/student lifecycle mutations, including verified login-identifier changes and transfer/progression workflows. Intra-term transfer history still needs a dedicated relational history model; the current `Enrolment` uniqueness model has intentionally not been weakened.
5. Finalize the payment-intent invoice-target/reservation schema and only then expose payment creation.
6. Verify the live Moolre API contract before implementing provider adapters and reconciliation workers.
7. Build successful payment allocation, receipts, refunds, immutable journal posting, and reconciliation before finance goes live.
8. Build attendance, staff/payroll, wallet, inventory, messaging, and notification workflows.
9. Expand integration and cross-repository journey tests, including admissions, guardian mobile session, and finance invoice workflows.
10. Establish deployment, secrets, backups, restore drills, and production monitoring.

## Current next execution order

1. Prisma migration + database verification.
2. Remaining object/scope authorization.
3. Guardian/student lifecycle mutations and profile management.
4. Finalize payment reservation schema and reconciliation design.
5. Finance payment/receipt foundation after schema verification.
6. Attendance.
7. Staff/payroll.
8. Wallet/inventory.
9. Communication/notifications.
10. Reporting and production hardening.
