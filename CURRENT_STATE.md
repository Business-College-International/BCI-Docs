# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
`bci-docs` is the canonical engineering source of truth for architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release criteria.

### Backend
`bci-backend-api` has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, JWT authentication/session rotation, admissions, academic-structure and student-read modules.

Authorization now has a canonical permission catalog plus a `RolePermission` model for default capabilities and `UserPermission` for explicit exceptions. The permission guard resolves both role defaults and direct grants. Existing role checks remain transitional until object/scope authorization is complete.

A deterministic Prisma seed establishes role-permission defaults without creating fake school users or records.

`GET /api/v1/auth/me` now returns the server-authoritative current-user profile, roles, effective permissions, direct scoped permission assignments, and linked guardian/staff profile where applicable.

Every HTTP response receives a server-generated `X-Request-Id` correlation identifier.

Internal academic-structure reads are authenticated; academic years and class streams are no longer public endpoints.

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

Guardian access is relationship-scoped: a guardian can read only a linked ward. Privileged office/leadership roles can read student records; teacher-specific class/object scoping remains a separate authorization task.

Admission identity linking is hardened: an existing guardian account may be linked by phone, but a non-guardian account cannot be silently attached to a student. Pre-created guardian Person records are reused during later guardian registration instead of duplicated.

The Prisma model uses a dedicated application tracking code rather than exposing the application UUID as the public lookup credential. Provider payment attempts and webhook events now have durable models for future Moolre reconciliation.

### Web portal
`bci-web-portal` has a staff sign-in surface, authenticated admissions workspace, application list/review controls, and public tracking-code lookup aligned to the backend contract.

The portal now verifies the access token by calling `/auth/me` before showing the authenticated workspace. Application list and review controls use server-returned permission codes rather than treating token presence as sufficient authorization.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell and an admissions-status screen using the same tracking-code contract as web and backend. Full guardian authentication and authenticated staff experiences remain in progress.

### Public website
`bci-website` has a public BCI shell and admissions form concept wired to the shared backend application endpoint and authoritative tracking-code response.

## Engineering hygiene

GitHub Issues are not being used as the default implementation journal during foundation work. The previous bootstrap/design issues were closed as planning artifacts. Active work is tracked by the canonical docs roadmap and repository commits; issues will only be opened for a bounded, reviewable task when that becomes useful.

Backend CI runs on pull requests or intentional manual dispatch only. Direct pushes to `main` do not trigger CI, preventing noisy failure notifications while the repository is still being bootstrapped.

CI now validates the Prisma schema, generates the Prisma client, compiles the backend and runs the Jest unit suite during those controlled verification runs.

There are currently no open pull requests or open issues in the BCI organization.

## Open blockers before production

1. Generate and verify the initial Prisma migration from the hardened schema and establish a repeatable PostgreSQL verification path.
2. Complete object/scope authorization and replace remaining coarse role checks where appropriate.
3. Add structured logging, rate limiting and centralized environment/secret validation.
4. Complete guardian/student lifecycle mutations and profile management.
5. Verify the live Moolre API contract before implementing provider adapters and reconciliation workers.
6. Build fee charges, payment intents, reconciliation, receipts and immutable journal posting before finance goes live.
7. Build attendance, staff/payroll, wallet, inventory, messaging and notification workflows.
8. Expand unit, integration and cross-repository journey tests.
9. Establish deployment, secrets, backups, restore drills and production monitoring.

## Current next execution order

1. Prisma migration + database verification.
2. Object/scope authorization hardening.
3. Guardian/student lifecycle mutations and profile management.
4. Complete admissions admission/enrolment UI using academic-year/term/class APIs.
5. Finance/payment foundation.
6. Attendance.
7. Staff/payroll.
8. Wallet/inventory.
9. Communication/notifications.
10. Reporting and production hardening.
