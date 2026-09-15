# BCI Engineering — START HERE

## Mission
Build one trustworthy school operating system for Business College International across KG, JHS and SHS. The product must support school operations without allowing the mobile or web clients to become competing sources of business truth.

## Repository authority
- `bci-backend-api`: authoritative application services, authorization, workflows, persistence, integrations and jobs.
- `bci-mobile-app`: guardian + staff experience; consumes backend contracts.
- `bci-web-portal`: operational control surface for office, finance, academic leadership and director.
- `bci-website`: public information and admissions entry point.
- `bci-docs`: architecture, contracts, invariants, roadmap and decisions.

## Non-negotiable rules
1. Never put authoritative fee, payroll, wallet, inventory or attendance calculations only in clients.
2. Every privileged mutation has server-side authorization and an audit trail.
3. Every money-moving operation is idempotent, transactional and reconcilable.
4. External provider success/failure is not assumed from a client response; provider webhooks/status checks are reconciled.
5. Important entity lifecycles are modeled as explicit transitions, not arbitrary status edits.
6. Historical school records must remain intelligible after a student is promoted, transferred, withdrawn or graduated.
7. Configuration is data: programmes, classes, streams, fee schedules, grading rules and roles must not be hard-coded into UI components.
8. Personal data is minimized, access-scoped and never exposed merely because an actor has a globally valid ID.
9. Do not silently substitute demo/static data when an authoritative request fails.
10. Changes that affect another repository require a contract/architecture update before or with implementation.

## Build order
Foundation → Identity/RBAC → School structure → People/Admissions → Student lifecycle → Academics/attendance → Finance/payments → Staff/payroll → Inventory/wallet → Communication/notifications → Reporting/audit → production hardening.

## Definition of done for a domain module
- Domain states documented.
- Permissions documented.
- Database constraints and indexes designed.
- Service/use cases implemented server-side.
- API contract documented.
- Idempotency/concurrency behavior tested where relevant.
- Audit requirements implemented.
- Web/mobile consumers use the same server contract.
- Failure and retry behavior tested.
