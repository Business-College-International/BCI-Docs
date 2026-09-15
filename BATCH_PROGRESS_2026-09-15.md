# BCI Engineering Batch — 2026-09-15

This batch intentionally groups related work rather than treating every file edit as a separate project checkpoint.

## Academic record integrity

- Teacher assessment entry now has a scoped roster contract and teacher web workflow.
- Assessment creation/result entry remains assignment-scoped and audited.
- Grading is policy-driven through a pure, tested engine; no official grade bands are hard-coded.
- Report-card publication is modeled as immutable versioned snapshots with correction/republication rules.

## Communications

The backend now has a durable announcement flow using the existing `Announcement` and `NotificationDelivery` models:

- authenticated audience-filtered announcement reads
- manager-only announcement creation
- draft announcements
- publish transition
- audience types: `ALL`, `GUARDIANS`, `STAFF`, `TEACHERS`, `USER`
- durable in-app delivery records on publish
- manager draft retrieval
- audit records for creation and publication

External SMS is intentionally still adapter-gated. No Moolre/SMS contract has been guessed.

The web staff workspace now includes an announcement center. Guardians can read the same server-filtered announcement feed through the Flutter app.

## Stationery

The existing stationery database models now have an application boundary:

- active catalog read
- inventory-manager item creation
- positive stock receipt/count-correction adjustments
- audit records for stock changes
- guardian-only draft order creation
- guardian-student `canPayFees` gate on draft orders
- duplicate line/item validation
- guardian order history

Draft orders do not reserve stock and do not initiate payment. Payment/reservation integration remains intentionally blocked until the financial reservation schema is verified.

The Flutter guardian app now exposes the read-only stationery catalog. The web client has typed stationery/announcement contracts for the next staff inventory UI pass.

## Safety corrections caught during this batch

- Announcement drafts were initially not visible to their managers; the listing query now includes the manager's own unpublished drafts.
- Stock movement initially attempted to write a nonexistent `note` field; that was corrected to use existing reference fields plus audit JSON.
- Stationery order tests initially used a fake decimal object; tests now use Prisma's actual Decimal behavior.
- Announcement read access intentionally does not depend on the still-racing permission-catalog edit; it is authenticated and audience-filtered at the service boundary.

## Still blocked

- Canonical initial Prisma migration and real PostgreSQL execution remain unverified.
- Payment reservation/invoice targeting and live Moolre verification remain blocked.
- Wallet balance/reversal semantics remain blocked.
- Timetable schema and report-publication persistence still require deliberate Prisma schema work.

## Engineering discipline

No new GitHub issues or PRs were created for this batch. Existing CI remains PR/manual only; direct pushes to `main` are not used as a verification trigger.
