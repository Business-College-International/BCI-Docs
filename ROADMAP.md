# BCI Implementation Roadmap

The existing `MASTER_PLAN.md` is the historical/product plan. This roadmap is the engineering execution order. New implementation should follow this dependency order.

## Phase 0 — Foundation
- Lock architecture, domain model, security and financial invariants.
- Bootstrap CI for all repositories.
- Establish environment/config conventions.
- Set up backend health/readiness, structured logs, API versioning and error format.
- Establish PostgreSQL migration workflow and test database.
- Define OpenAPI contract generation strategy.
- Establish object-storage abstraction, notification abstraction and payment-provider abstraction.

## Phase 1 — Identity and school structure
- User/session/authentication lifecycle.
- RBAC with roles + permissions + scopes.
- People, guardian and staff profiles.
- Academic year/term/level/programme/subject/class/stream configuration.
- Initial director/office/principal administration.

## Phase 2 — Admissions and student lifecycle
- Public/application intake.
- Document upload.
- Admissions review and decisions.
- Acceptance/enrolment conversion.
- Student profile and guardian relationships.
- Promotion/transfer/withdrawal/graduation state transitions.

## Phase 3 — Academics and attendance
- Teacher assignments.
- Timetable creation/publishing.
- Attendance sessions and marking.
- Guardian attendance view.
- Attendance dashboards.
- Assessment/result foundation and report-card model.

## Phase 4 — Fees and payments
- Fee plan configuration.
- Charge/invoice generation.
- Payment intents and Moolre integration adapter.
- Webhooks + reconciliation worker.
- Payment allocation and receipts.
- Outstanding balances and finance reporting.
- Refund workflow.

## Phase 5 — Wallet, stationery and expenses
- Student wallet ledger.
- Guardian top-up.
- Controlled office withdrawal workflow.
- Stationery catalogue, stock movement and order fulfilment.
- Expenses and approval flow.
- Daily financial dashboard.

## Phase 6 — Staff, payroll and communication
- Staff duties.
- Staff attendance if adopted.
- Compensation structures.
- Payroll calculation, approval and disbursement.
- Payslips.
- Announcements + SMS delivery.
- Push notifications.
- Staff messaging/chat.

## Phase 7 — Reporting and production hardening
- Director/principal dashboards.
- Audit search/export.
- Operational reports.
- Data exports/backups.
- Security review.
- Performance testing.
- Disaster recovery test.
- Provider reconciliation drills.
- App/web release readiness.

## Phase gates

A phase is not complete because screens exist. It is complete only when the backend workflow, authorization, database constraints, API contract, client flow, tests, audit/reconciliation requirements and failure paths are implemented.

## Priority rule

Build the smallest useful vertical slice first:

`guardian/applicant → application → office review → admission → student record`

Then:

`student → fee charge → Moolre payment → webhook → allocation → receipt → guardian/office reconciliation`

These vertical slices prove the core system architecture before broader feature expansion.
