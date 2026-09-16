# Payment Reservation Migration Specification

## Purpose

Introduce a durable reservation entity so BCI can prevent concurrent payment attempts from consuming the same invoice balance before a provider transaction is initiated.

## Required lifecycle

`ACTIVE -> EXPIRED`

`ACTIVE -> CONSUMED`

`ACTIVE -> CANCELLED`

A reservation must never become money collected. It only reserves invoice balance until a provider attempt resolves.

## Required fields

The migration should add a `PaymentReservation` model with:

- `id` UUID primary key
- `studentId` foreign key
- `guardianId` nullable foreign key
- `amount` decimal(12,2)
- `currency` string default `GHS`
- `status` enum: `ACTIVE`, `EXPIRED`, `CONSUMED`, `CANCELLED`
- `clientReference` unique/idempotent reference
- `expiresAt`
- `createdAt`
- `consumedAt` nullable
- `cancelledAt` nullable
- `cancelReason` nullable

A related `PaymentReservationAllocation` model should associate reservation amounts to invoice IDs, with a unique `(reservationId, invoiceId)` constraint.

## Concurrency rules

Reservation creation must run in one PostgreSQL transaction with the selected invoice balances locked for update.

For every invoice:

`available = invoice total - successful allocations - active reservations`

The transaction must reject any reservation that exceeds the combined available balance.

No provider call may occur until the reservation transaction commits successfully.

## Idempotency

The application must reuse the existing idempotency infrastructure for payment-start operations. A repeated request with the same authenticated user, operation and idempotency key must return the original reservation/provider-start response instead of creating a second reservation.

## Expiry

Reservations must expire automatically through a scheduled cleanup job or on every read/write boundary that evaluates availability. Expiry is not payment failure and must never create an allocation or financial journal entry.

## Provider reconciliation

The provider attempt references the reservation. Only a verified provider success event may:

1. mark the reservation `CONSUMED`;
2. create the durable `Payment` as `SUCCEEDED`;
3. create `PaymentAllocation` rows from the reservation allocation plan;
4. create the receipt;
5. create the financial journal entries;
6. audit the reconciliation.

These actions must occur in one application transaction after the webhook is verified.

Provider failure, cancellation or timeout must release the reservation without allocating money.

## Required tests before activation

- two concurrent reservations cannot consume the same invoice balance;
- duplicate idempotency keys return the original result;
- expired reservations no longer reduce availability;
- successful verified webhook consumes exactly one reservation;
- duplicate webhook events do not allocate twice;
- provider failure releases the reservation;
- a partial payment cannot allocate more than the reserved amount;
- a guardian cannot reserve another guardian's ward invoices;
- privileged finance users cannot bypass reservation accounting.

## Activation gate

This migration must not be hand-written into production migration SQL. Generate it from the canonical Prisma schema, apply it to a clean PostgreSQL instance, run Prisma drift verification, build the backend, and execute the payment/reconciliation test suite before promoting the migration to the canonical migration directory.
