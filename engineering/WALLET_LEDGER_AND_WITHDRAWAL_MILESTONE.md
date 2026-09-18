# BCI Wallet Ledger and Withdrawal Milestone

## Completed

- Wallet balances are calculated from the complete transaction ledger, not the displayed history window.
- Successful provider wallet top-ups are converted into `TOP_UP` credit entries only after the verified payment webhook settles the payment.
- `TOP_UP` transactions increase balance.
- `WITHDRAWAL` transactions decrease balance.
- The current schema's `REVERSAL` transaction does not identify a source transaction, so wallets containing reversal rows are flagged `LEDGER_POLICY_REQUIRED` instead of exposing a potentially incorrect balance.
- Guardian wallet reads remain restricted by the existing `GuardianStudent.canManageWallet` flag.
- Office/director/accountant withdrawals use `wallet.manage` and execute inside a PostgreSQL serializable transaction.
- Withdrawals cannot exceed the current ledger balance.
- Concurrent serialization conflicts return a retryable conflict rather than risking an overdraft.
- Every completed withdrawal writes an audit record.
- Guardian portal shows wallet balance/history but cannot perform cash withdrawals.

## Deliberate gates

- Guardian wallet top-up initiation now creates an idempotent `WALLET_TOP_UP` payment reservation and starts the provider collection; wallet credit is still created only from verified provider settlement.
- Provider OTP continuation is permitted for wallet top-ups only when the guardian relationship has `canManageWallet`.
- Provider references are not invented for cash withdrawals.
- Reversals now identify exactly one original transaction and create the opposite signed effect; an original transaction may be reversed only once.
- A richer withdrawal-request model can be added later if BCI needs pending/approval states before cash is actually handed to a student; the current operation represents an immediate authorized office disbursement.
- Wallet top-up initiation is intentionally separate from fee invoice payment allocation; it creates no invoice allocation.

## Verification requirement

The wallet concurrency behavior must be validated against clean PostgreSQL with the project's PR/manual database workflow before production activation.
