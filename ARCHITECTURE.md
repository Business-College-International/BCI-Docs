# BCI System Architecture

## 1. System shape

BCI is one logical platform with four user-facing applications and one authoritative backend:

```text
                         ┌──────────────────────────┐
                         │       PostgreSQL         │
                         │ persisted school truth   │
                         └────────────▲─────────────┘
                                      │
                         ┌────────────┴────────────┐
                         │      bci-backend-api    │
                         │ NestJS/TypeScript       │
                         │ auth • domain • finance │
                         │ integrations • jobs     │
                         └───┬────────┬────────┬───┘
                             │        │        │
                    ┌────────▼──┐ ┌──▼────────▼──┐ ┌─────────────┐
                    │ Flutter   │ │ Web Portal   │ │ Public Web  │
                    │ mobile    │ │ Vite/React   │ │ Vite/React  │
                    └───────────┘ └──────────────┘ └─────────────┘
```

External providers (payments, disbursement, SMS, push and object storage) are adapters behind backend-owned interfaces. Provider SDKs do not become domain logic.

## 2. Backend bounded modules

### Identity & Access
Users, credentials, sessions/tokens, roles, permissions, scoped grants, device registration and audit context.

### People
Canonical people records, guardian relationships, emergency contacts and addresses. Authentication identity is separate from person identity.

### Admissions
Applications, required documents, review, admission decision, acceptance and conversion into enrolment.

### School Structure
Academic years, terms, levels, programmes, subjects, elective groups, classes/streams, rooms, houses and timetable periods.

### Student Lifecycle
Student profile, enrolments, progression, transfers, withdrawal, graduation, documents and status history.

### Academics
Teacher assignments, timetables, attendance sessions, assessments, marks, grade calculations and report-card publication.

### Finance
Fee structures, charges/invoices, payment intents, payment transactions, allocations, receipts, refunds, financial journal and reconciliation.

### Student Wallet
A ring-fenced spending-money ledger separate from school-fee receivables. Supports electronic top-up and controlled physical-cash withdrawal.

### Inventory & Stationery
Catalog, stock movement, purchase/sale orders, order lines, fulfilment and stock reconciliation.

### Staff & Payroll
Employment records, duties, teaching assignments, leave/attendance where enabled, pay structures, payroll periods, approvals, payslips and disbursement attempts.

### Communications
Announcements, audience resolution, in-app messages, channels, delivery records, push/SMS jobs and provider delivery status.

### Audit & Reporting
Immutable audit events, operational reporting, finance reporting, dashboards and export jobs.

## 3. Authority flow

For every important mutation:

`request → authenticate → authorize actor/scope → validate state → idempotency/claim → DB transaction → persist authoritative outcome → commit event/outbox → downstream delivery`

Clients may optimistically display local state, but only the committed backend response is authoritative.

## 4. Eventing

Use a durable outbox pattern for events that must cause downstream work after a successful database transaction. Examples: payment confirmed, fee receipt issued, payroll disbursement completed, announcement published and student status changed.

Events are at-least-once. Consumers must be idempotent. A socket, push notification or SMS is never the system of record.

## 5. Storage

Relational data lives in PostgreSQL. User uploads and generated PDFs live in S3-compatible object storage. The database stores object keys, metadata, ownership and retention information rather than raw large files.

## 6. Application boundaries

### Flutter mobile
Guardian-first experience plus staff/teacher workflows. Mobile must not calculate authoritative financial values or mutate restricted state outside backend APIs.

### Web portal
Operational control surface for office, finance, admissions, academic leadership, principal and director. It may expose more capabilities than mobile but is still constrained by backend authorization.

### Public website
Public content and admissions entry point. No privileged school operational data is exposed from this application.

## 7. Deployment boundaries

Start with one backend deployment, one worker process and managed PostgreSQL. Split workers or modules operationally only when traffic or reliability evidence requires it.
