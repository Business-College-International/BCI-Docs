# BCI Wallet Ledger and Withdrawal Milestone

## Completed

- Wallet balances are calculated from the complete transaction ledger, not the displayed history window.
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

- Wallet top-up remains behind the payment provider/reservation gate.
- Provider references are not invented for cash withdrawals.
- A future reversal model should identify the transaction being reversed before reversal amounts participate in balance calculation.
- A richer withdrawal-request model can be added later if BCI needs pending/approval states before cash is actually handed to a student; the current operation represents an immediate authorized office disbursement.

## Verification requirement

The wallet concurrency behavior must be validated against clean PostgreSQL with the project's PR/manual database workflow before production activation.
