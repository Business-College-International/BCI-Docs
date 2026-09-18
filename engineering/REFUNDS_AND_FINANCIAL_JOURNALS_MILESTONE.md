# Refunds and Financial Journals Milestone

## Completed

- Refund requests are restricted to finance-management roles.
- Only succeeded payments can be refunded.
- Refund amount is capped by the payment amount less prior non-failed/non-cancelled refunds.
- Refund requests require a reason and audit record.
- The requester cannot approve their own refund.
- Refund approval records `approvedBy` and remains separate from provider execution; execution reserves refund capacity before disbursement.
- Refund requests are visible to finance managers in the staff web portal.
- Balanced journal batches are validated using exact Prisma Decimal arithmetic.
- Journal transactions require at least one debit and one credit and must balance exactly.
- All journal lines in one transaction share an `entryNumber` and are persisted atomically.

## Provider boundary

Application-level refund execution is implemented against the provider-neutral disbursement abstraction. It verifies remaining refund capacity under a serializable transaction, moves the refund to `PROCESSING`, sends the provider request, and reconciles ambiguous provider outcomes. Live Moolre money movement remains configuration-gated and requires explicit live confirmation.

## Accounting invariants

1. A failed, cancelled, or non-succeeded payment must never be treated as collected cash.
2. Refunds cannot exceed the unrefunded succeeded payment amount.
3. A refund requester cannot approve their own request.
4. Balanced journal creation must fail before persistence when debits and credits do not match.
5. Provider execution must be idempotent and reconciled to durable payment/refund records before live activation.
6. Journal replay with the same reference must match the existing accounting lines exactly; same-reference/different-line mutations are rejected.

## Remaining gate

The remaining production gate is provider activation and end-to-end controlled-money verification:

`reserved payment -> provider initiation -> verified webhook -> settled payment -> allocation/receipt -> journal -> refund request -> approval -> provider disbursement -> refund settlement -> reversal journal`
