- `wallet.topup.succeeded.v1`
- `wallet.withdrawal.completed.v1`
- `payroll.disbursement.succeeded.v1`
- `announcement.published.v1`

Events contain identifiers and relevant state snapshots but never credentials or secrets.

## Financial contract implementation status

Fee payment initiation now creates explicit per-invoice `PaymentIntent` reservations with a 15-minute expiry. Active `PENDING`/`PROCESSING` intents reduce available invoice balance; expired intents are excluded from new availability. The existing idempotency ledger remains the request-replay authority.

`PaymentAllocation` is created only after a verified provider-success webhook and must reconcile exactly to the PaymentIntent reservation total. Provider rejection/unknown/OTP paths update PaymentIntent state without treating the reserved amount as collected funds.

Wallet top-ups create the signed CREDIT ledger effect only after verified provider success. Office withdrawals create signed DEBIT effects together with durable withdrawal evidence; posted corrections use compensating reversal entries.

Expiry reconciliation is self-healing during new fee-payment reservation attempts: stale PENDING/PROCESSING intents are marked EXPIRED before availability is recalculated. Real-money Moolre verification remains the production gate.

## Grading policy API

- GET /api/v1/grading-policies — read versioned grading policies; filtered by academic year, level and programme.
- POST /api/v1/grading-policies — create a DRAFT policy with explicit academic-year/level scope and school-supplied grade bands.