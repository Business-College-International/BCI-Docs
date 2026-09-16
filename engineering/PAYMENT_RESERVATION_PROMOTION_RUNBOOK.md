# BCI Payment Reservation Promotion Runbook

## Current state

The backend has a provider-neutral payment preflight layer. It validates guardian fee authority, computes deterministic invoice allocation, and detects existing pending/processing payment attempts.

The current Prisma schema does **not** yet contain durable payment-reservation entities, and the backend repository does not expose a committed `prisma/migrations` history on `main`. Therefore live payment initiation must remain disabled.

## Required schema change

The next schema change must introduce a durable reservation model and reservation-to-invoice allocation model.

Required invariants:

1. A reservation belongs to exactly one student and initiating guardian context.
2. A reservation has an explicit lifecycle: ACTIVE, EXPIRED, CONSUMED, CANCELLED.
3. Every reservation allocation belongs to exactly one reservation and one invoice.
4. Allocated reservation value may not exceed invoice outstanding value.
5. A reservation may not exceed the requested payment amount.
6. Reservation creation must be idempotent at the API boundary.
7. Concurrent reservation attempts for the same student must serialize safely.
8. Provider initiation must require an ACTIVE reservation.
9. A verified provider success event must consume the reservation atomically with payment creation/allocation.
10. Failed/cancelled/expired provider flows must release the reservation without mutating collected totals.

## Concurrency strategy

Do not rely on an application-only `findFirst` followed by `create` check.

Use a PostgreSQL-safe transaction strategy with serializable isolation and bounded retry of serialization failures. The reservation transaction must re-read current active reservations and current invoice outstanding values after obtaining the transaction serialization boundary.

The implementation must prove that two concurrent attempts cannot both reserve the same outstanding balance.

## Verification run

The canonical migration must be generated from the Prisma schema by Prisma itself.

The controlled verification job must:

1. Start a clean PostgreSQL service.
2. Run `prisma validate`.
3. Run `prisma generate`.
4. Generate the migration SQL from the complete schema.
5. Apply the generated migration to the clean database.
6. Run Prisma schema drift checks against that database.
7. Run the reservation concurrency integration suite.
8. Run backend build/lint/tests.
9. Publish the generated SQL as an inspection artifact.

No hand-authored migration SQL should be promoted directly to `main`.

## Required concurrency tests

At minimum:

- two simultaneous reservations for the same student and same invoices;
- two simultaneous reservations for the same student with partially overlapping invoices;
- reservation expiry racing with provider initiation;
- verified provider success racing with a repeated webhook;
- duplicate client idempotency key with identical request;
- duplicate client idempotency key with different request hash;
- retry after PostgreSQL serialization failure.

Expected result: at most one reservation can consume a given invoice balance at a time, payment records remain idempotent, and collected totals never count pending provider attempts.

## Promotion gate

The reservation migration is **not ready for promotion** until the controlled PostgreSQL job passes. The absence of a workflow result is not equivalent to success.

Until promotion is verified, guardian clients may prepare payment review/preflight data but must not initiate a live provider transaction.
