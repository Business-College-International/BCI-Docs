# BCI Current State

**Last reconciled:** 2026-09-15

## What is true now

### Documentation
The docs repository contains the canonical engineering entrypoint, architecture, domain model, security boundaries, financial invariants, state machines, API contracts, roadmap and release checklist.

### Backend
`bci-backend-api` has a NestJS bootstrap, strict TypeScript configuration, Prisma service, global request validation, `/api/v1/health`, and the first admissions application module.

Current public admissions endpoints:

- `POST /api/v1/applications`
- `GET /api/v1/applications/track/:trackingCode`

The Prisma application model now issues a dedicated tracking code rather than exposing the application UUID as the public lookup credential.

### Web portal
`bci-web-portal` has a Vite/React/TypeScript shell, TanStack Query, an API client and an initial admissions-status surface wired to the backend contract. Authentication/RBAC is not yet implemented and therefore must not be treated as production-ready.

### Mobile
`bci-mobile-app` has a Flutter/Riverpod shell and an initial admissions-status screen using the same backend tracking-code contract. Authentication, guardian identity, secure session lifecycle and push registration are still pending.

### Public website
`bci-website` has a responsive React/Vite public shell covering KG/JHS/SHS, the four SHS programmes and the unified admissions concept. The application form itself is still pending.

## Known blockers before production

1. Replace the starter Prisma schema with the hardened domain model and generate real migrations.
2. Implement authentication, refresh-token rotation, MFA/step-up controls where appropriate, RBAC and scoped object authorization.
3. Implement authenticated admissions review and application-to-student/enrolment transaction.
4. Add request correlation IDs, structured audit logs and rate limiting.
5. Add idempotency persistence before accepting retryable public or financial mutations.
6. Build immutable financial journal/payment/reconciliation structures before fees, wallet or payroll go live.
7. Add provider integrations only behind explicit adapters and reconciliation workers.
8. Add automated tests at unit, integration and cross-workflow levels.
9. Establish deployment, secret management, backups, restore drills and monitoring.

## Current next execution order

1. Identity/RBAC foundation.
2. Hardened school/academic schema.
3. Admissions review → admit → enrolment.
4. Guardian linking and student profile.
5. Fee charge/payment/reconciliation foundation.
6. Attendance.
7. Staff/payroll.
8. Wallet/inventory.
9. Communication/notifications.
10. Reporting and production hardening.
