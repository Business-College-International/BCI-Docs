# Refunds and Financial Journals Milestone

## Completed

- Refund requests are restricted to finance-management roles.
- Only succeeded payments can be refunded.
- Refund amount is capped by the payment amount less prior non-failed/non-cancelled refunds.
- Refund requests require a reason and audit record.
- The requester cannot approve their own refund.
- Refund approval records `approvedBy` but does not execute a provider transaction.
- Refund requests are visible to finance managers in the staff web portal.
- Balanced journal batches are validated using exact Prisma Decimal arithmetic.
- Journal transactions require at least one debit and one credit and must balance exactly.
- All journal lines in one transaction share an `entryNumber` and are persisted atomically.

## Provider boundary

Refund provider execution is intentionally not implemented. An approved refund remains a financial authorization state until the verified provider contract and payment-reservation/provider execution gates are complete.

## Accounting invariants

1. A failed, cancelled, or non-succeeded payment must never be treated as collected cash.
2. Refunds cannot exceed the unrefunded succeeded payment amount.
3. A refund requester cannot approve their own request.
4. Balanced journal creation must fail before persistence when debits and credits do not match.
5. Provider execution must be idempotent and reconciled to durable payment/refund records before live activation.

## Remaining gate

The final live-money flow remains:

`Payment reservation -> verified provider initiation -> verified webhook -> successful payment -> allocation -> receipt -> journal -> refund request -> approval -> verified provider refund -> refund success -> reversal journal`
