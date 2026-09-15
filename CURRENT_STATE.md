# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
The docs repository contains the canonical engineering entrypoint, architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release checklist. fileciteturn93file0

### Backend
`bci-backend-api` now has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, JWT authentication/session rotation, and admissions plus academic-structure modules.

Current public admissions endpoints:

- `POST /api/v1/applications`
- `GET /api/v1/applications/track/:trackingCode`

Authenticated staff admissions endpoints now include:

- `GET /api/v1/applications`
- `POST /api/v1/applications/:id/review`
- `POST /api/v1/applications/:id/admit`

The Prisma model uses a dedicated application tracking code rather than exposing the application UUID as the public lookup credential.

The hardened schema now separates Person/User identity, academic years/terms/enrolments, attendance sessions, assessments, invoices/payment allocations/receipts/refunds, wallet transactions, payroll periods/entries/disbursements, inventory movements, notifications and audit records.

### Web portal
`bci-web-portal` now has a staff sign-in surface, authenticated admissions workspace, application list/review controls, and public tracking-code lookup aligned to the backend contract.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell and an admissions-status screen using the same tracking-code contract as web and backend. Full guardian authentication and authenticated staff experiences remain in progress.

### Public website
`bci-website` now contains a public BCI shell and a real admissions form that submits to the shared backend application endpoint and returns an authoritative tracking code.

## Open blockers before production

1. Generate and verify the initial Prisma migration from the hardened schema and add PostgreSQL migration CI. Issue `bci-backend-api#5` tracks this gate. fileciteturn170file0
2. Replace coarse role checks with complete permission + object/scope authorization.
3. Add request correlation IDs, structured logging, rate limiting and centralized environment/secret validation.
4. Add guardian/student profile APIs and complete the student lifecycle.
5. Implement Moolre through explicit provider adapters and reconciliation workers only after its live API contract is verified.
6. Build fee charges, payment intents, reconciliation, receipts and immutable journal posting before finance goes live.
7. Build attendance, staff/payroll, wallet, inventory, messaging and notification workflows.
8. Add unit, integration and cross-repository journey tests.
9. Establish deployment, secrets, backups, restore drills and production monitoring.

## Current next execution order

1. Prisma migration + database CI.
2. Permission/scoping hardening.
3. Guardian/student identity linking and student lifecycle.
4. Complete admissions admission/enrolment UI using academic-year/term/class APIs.
5. Finance/payment foundation.
6. Attendance.
7. Staff/payroll.
8. Wallet/inventory.
9. Communication/notifications.
10. Reporting and production hardening.
