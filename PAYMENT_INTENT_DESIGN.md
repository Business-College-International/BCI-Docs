# BCI Payment Intent Design Gate

## Why payment intents are not live yet

The current `Payment` model represents a financial payment after it exists, while `PaymentAllocation` applies a successful payment to an invoice. The current model does not identify the invoice being targeted while provider payment is pending.

That distinction matters because multiple concurrent pending requests could otherwise attempt to consume the same outstanding invoice balance.

## Required schema capability

Before payment creation is enabled, the database must support these invariants atomically:

1. Every fee payment intent identifies exactly one target invoice for its initial fee-payment flow.
2. The intended amount is fixed when the intent is created; provider callbacks cannot change it.
3. Invoice outstanding amount is derived from immutable invoice lines minus valid successful payment allocations.
4. Outstanding availability also subtracts non-expired `PENDING` / `PROCESSING` reservations against that same invoice.
5. An idempotency key is unique for `(initiating user, operation)` and returns the original payment intent on a retry with the same request hash.
6. A reused idempotency key with a different request hash is rejected as an idempotency conflict.
7. Provider attempts and provider webhook events reference the same payment intent/payment record without creating a second financial record.
8. Only a verified provider success transition can create the final `PaymentAllocation` and `Receipt`.
9. Duplicate provider callbacks are harmless, produce no duplicate allocation/receipt, and remain auditable.
10. Expired pending reservations become unavailable for balance calculations and can be reconciled/marked expired without changing historical successful payments.

## Planned model shape

The exact field names will be finalized with the first PostgreSQL migration, but the conceptual relationship should be:

```text
Guardian/Initiator
      │
      └── Payment Intent ── target Invoice
              │
              ├── fixed amount
              ├── status
              ├── idempotency identity
              ├── expiresAt
              ├── provider
              ├── clientReference
              └── provider attempts
                         │
                         └── webhook events

Verified provider success
      │
      ▼
Payment (financial fact)
      │
      ├── PaymentAllocation → StudentInvoice
      └── Receipt
```

The preferred implementation is a dedicated `PaymentIntent`/reservation record rather than overloading the existing `Payment` row with target-invoice state. A payment intent is an operational reservation; a successful `Payment` is the durable financial fact.

The intent should carry enough identity to safely retry and reconcile, including initiator, invoice target, currency, fixed amount, provider/client reference, idempotency key, created/expiry timestamps, and lifecycle status. The final `Payment` should reference the intent so audit and reconciliation can follow the entire journey.

## Reservation calculation

For invoice `I`:

```text
Outstanding = InvoiceLinesTotal
            - SuccessfulAllocations
            - ActivePaymentReservations
```

`ActivePaymentReservations` includes only intents in `PENDING` or `PROCESSING` whose `expiresAt` is still in the future.

The server must reject a new intent when requested amount exceeds the currently available outstanding amount. This check and reservation creation must happen in the same database transaction.

## State machine

```text
REQUESTED → PENDING → {PROCESSING | CANCELLED | EXPIRED}
PROCESSING → {SUCCEEDED | FAILED | UNKNOWN}
UNKNOWN → {SUCCEEDED | FAILED}
```

A successful transition performs the financial posting exactly once:

```text
provider success
    → verify webhook/lookup
    → lock/reconcile intent
    → create Payment
    → create PaymentAllocation
    → update Invoice status
    → create Receipt
    → audit
```

The sequence must be transactional where the provider semantics permit it. External provider calls are never treated as proof of local financial success until the callback/verification is accepted by the backend.

## Provider boundary

The provider adapter must be isolated behind an internal interface. The domain layer should not know Moolre-specific HTTP payloads.

Because the live Moolre API contract has not been verified in the current environment, this repository must not hard-code endpoint paths, signatures, callback fields, or response semantics until that contract is independently confirmed.

## Current API boundary

The finance API currently supports fee schedule management, invoice issuance, and scoped invoice reading.

It deliberately does **not** expose payment-intent creation, provider callbacks, refunds, wallet top-ups, or disbursements yet.

## Migration rule

When this model is implemented, generate the initial PostgreSQL migration from the **complete** hardened Prisma schema in one controlled change. Do not introduce repeated speculative schema migrations merely to make an intermediate payment prototype compile.
