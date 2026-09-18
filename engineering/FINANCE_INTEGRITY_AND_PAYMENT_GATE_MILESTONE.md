# Finance Integrity and Payment Gate Milestone

## Completed

- Invoice balance views now count only allocations whose linked payment is `SUCCEEDED`.
- Finance summaries use successful allocations for collected invoice allocation totals.
- Added a privileged read-only finance integrity report.
- Integrity checks detect orphan allocations, allocations attached to non-succeeded payments, over-allocated payments, invoice status mismatches, and succeeded payments without receipts.
- Added backend regression tests using real `Prisma.Decimal` values.
- Added the finance integrity workspace to the staff web portal.
- Corrected the PostgreSQL schema contract workflow so drift comparison uses the Prisma datamodel against the actual clean PostgreSQL URL rather than comparing the database datasource to itself.
- Wallet ledger integrity checks now surface malformed signed effects, invalid payment links, succeeded wallet top-ups without ledger credits, invalid reversals, and negative wallet balances.

## Payment gate still not active

The current schema has no durable payment-reservation entity. The guardian web/mobile payment flows therefore stop at payment preflight/allocation review and must not initiate Moolre/provider transactions.

Before live payment initiation is enabled, the reservation schema must be added, generated through Prisma, applied to a clean PostgreSQL service, checked for drift, and tested for concurrent reservation attempts and idempotent retries.

## Accounting invariants

1. Failed, cancelled, pending, processing, or refunded payments do not increase an invoice's paid allocation balance.
2. A payment allocation must never exceed its parent payment amount.
3. A payment allocation without a valid payment is an integrity finding.
4. A succeeded payment should have a receipt before finance considers the payment fully reconciled.
5. Stored invoice status must agree with successful allocated amount unless the invoice is explicitly void.
6. Provider settlement is not the same thing as an invoice allocation; allocation occurs only after verified successful payment state.

## Verification status

GitHub status checks are still required on the affected commits. An empty status collection is not treated as a pass.
