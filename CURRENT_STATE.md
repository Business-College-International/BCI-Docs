# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
`bci-docs` is the canonical engineering source of truth for architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release criteria.

### Backend
`bci-backend-api` has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, JWT authentication/session rotation, admissions, academic-structure and student lifecycle modules.

Authorization has a canonical permission catalog plus `RolePermission` for default capabilities and `UserPermission` for explicit exceptions. The permission guard resolves both role defaults and direct grants.

`GET /api/v1/auth/me` returns the server-authoritative current-user profile, roles, effective permissions, direct scoped permission assignments, and linked guardian/staff profile where applicable.

Admissions endpoints use explicit permission checks for list/review/admit operations rather than coarse role checks.

Admission placement validates that the selected academic year, term and class are mutually consistent, that the term is inside the academic year and open for enrolment, that the class matches the application's level/programme, that a configured class capacity has not been reached, and that an explicit admission number is not already in use.

Student reads require `students.read` and then apply object scope: linked guardians may read only their wards; privileged office/leadership roles may read broader student records; teachers may read only students belonging to classes and terms covered by their `TeacherAssignment`. Student documents remain restricted to privileged staff.

Guardian relationship permissions are enforced: `canViewAcademic` controls academic visibility for guardians, while `canPayFees` and `canManageWallet` are carried as explicit link-level permissions for the finance/wallet layers.

School staff with `students.manage` can create and remove guardian/student links. Linking can designate a single primary contact and set the three guardian link permissions. Link creation/removal is transactional and audited; duplicate links are rejected.

A controlled withdrawal transition now uses `students.manage`, closes the active enrolment, marks the student withdrawn, records the reason and writes an audit record atomically. Repeated withdrawal is rejected.

Guardian self-profile endpoints allow updates to non-login profile fields and notification preferences. Phone/email login identifiers remain read-only until a separate verified change flow exists.

Academic endpoints use explicit `academics.read` / `academics.manage` permissions. Teachers receive only classes covered by their assignments; privileged academic roles can browse the broader class structure.

A deterministic Prisma seed establishes role-permission defaults without creating fake school users or records.

Every HTTP response receives a server-generated `X-Request-Id` correlation identifier.

Startup now validates `DATABASE_URL`, `JWT_ACCESS_SECRET`, `NODE_ENV` and `PORT` before the application listens. JWT access secrets must meet a minimum length requirement. Production also requires explicit `CORS_ORIGINS`; development/test environments use a small local-origin default unless explicitly configured.

HTTP requests now emit structured JSON timing/status logs containing the request correlation ID, method, path, status and duration.

A conservative dependency-free in-process request limiter protects authentication and public application endpoints during foundation work. Distributed rate limiting remains a production infrastructure requirement before horizontal scaling.

The Prisma schema baseline was restored from the last complete Git blob after a reviewed schema-edit attempt was found to have truncated the file. Finance, payment-provider, wallet, inventory, notification and audit models are confirmed present again. No migration was generated from the truncated version.

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

Admission identity linking is hardened: an existing guardian account may be linked by phone, but a non-guardian account cannot be silently attached to a student. Pre-created guardian Person records are reused during later guardian registration instead of duplicated.

The Prisma model uses a dedicated application tracking code rather than exposing the application UUID as the public lookup credential. Provider payment attempts and webhook events now have durable models for future Moolre reconciliation.

### Web portal
`bci-web-portal` has a staff sign-in surface, authenticated admissions workspace, application list/review controls, public tracking-code lookup, and a server-driven admission placement workflow.

The portal verifies the access token by calling `/auth/me` before showing the authenticated workspace. Application list/review/admit controls use server-returned permission codes. Admission placement loads academic years and classes from the backend, filters to open terms and application-matching classes, and sends the selected authoritative IDs back to the admission endpoint for final validation.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell, secure token storage, login/refresh/logout against the shared `/auth` contract, session restoration through `/auth/me`, and an authenticated guardian dashboard loading `/students/me/wards`. Guardian-level relationship permissions are surfaced per ward so the client can respect server-authoritative academic/payment/wallet access. The public admissions-status screen remains available in the underlying feature code, while staff mobile roles remain intentionally disabled until their scoped workflows are implemented.

### Public website
`bci-website` has a public BCI shell and admissions form concept wired to the shared backend application endpoint and authoritative tracking-code response.

## Engineering hygiene

GitHub Issues are not being used as the default implementation journal during foundation work. The previous bootstrap/design issues were closed as planning artifacts. Active work is tracked by the canonical docs roadmap and repository commits; issues will only be opened for a bounded, reviewable task when useful.

Backend CI runs on pull requests or intentional manual dispatch only. Direct pushes to `main` do not trigger CI, preventing noisy failure notifications while the repository is still being bootstrapped.

CI validates the Prisma schema, generates the Prisma client, compiles the backend and runs the Jest unit suite during those controlled verification runs.

There are currently no open pull requests or open issues in the BCI organization.

## Open blockers before production

1. Generate and verify the initial Prisma migration from the complete hardened schema and establish a repeatable PostgreSQL verification path.
2. Complete remaining object/scope authorization across attendance, assessments, finance, inventory, messaging and staff workflows.
3. Replace the bootstrap in-process rate limiter with distributed protection before running multiple API instances.
4. Complete remaining guardian/student lifecycle mutations, including verified login-identifier changes and transfer/progression workflows. Intra-term transfer history still needs a dedicated relational history model before implementation; the current `Enrolment` uniqueness model has intentionally not been weakened yet.
5. Verify the live Moolre API contract before implementing provider adapters and reconciliation workers.
6. Build fee charges, payment intents, reconciliation, receipts and immutable journal posting before finance goes live.
7. Build attendance, staff/payroll, wallet, inventory, messaging and notification workflows.
8. Expand integration and cross-repository journey tests, including the new admission placement contract and guardian mobile session.
9. Establish deployment, secrets, backups, restore drills and production monitoring.

## Current next execution order

1. Prisma migration + database verification.
2. Remaining object/scope authorization.
3. Guardian/student lifecycle mutations and profile management.
4. Complete admissions journey tests and mobile/web parity.
5. Finance/payment foundation.
6. Attendance.
7. Staff/payroll.
8. Wallet/inventory.
9. Communication/notifications.
10. Reporting and production hardening.
