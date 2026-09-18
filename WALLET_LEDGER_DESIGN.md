# BCI Student Wallet Ledger Design Gate

## Why wallet balance is not yet calculated

The current `WalletTransaction` model has `TOP_UP`, `WITHDRAWAL`, and `REVERSAL` transaction types, but `REVERSAL` does not identify which prior transaction it reverses or whether the reversal is a credit or debit.

A balance derived from transaction type alone could therefore be financially wrong.

## Required ledger invariants

Before wallet top-up, withdrawal, reversal, or balance display is treated as authoritative, the ledger model must support:

1. Every wallet transaction has a clear signed effect on available balance.
2. Every reversal references exactly one original wallet transaction.
3. A transaction cannot be reversed more than once unless an explicit correction workflow exists.
4. Provider-backed top-ups reference the verified payment fact, not merely a provider request.
5. Office withdrawals record operator identity, student identity, amount, reason, approval/verification state, and dispense timestamp.
6. Guardian access remains controlled by `GuardianStudent.canManageWallet`.
7. Successful wallet mutations are append-only; corrections are compensating transactions rather than edits.
8. The displayed balance is derived from the complete valid ledger, not from a cached mutable balance field.
9. Concurrent top-up/withdrawal operations cannot spend the same wallet balance twice.

## Preferred relationship

```text
StudentWallet
   │
   └── WalletTransaction
          ├── TOP_UP       → verified Payment
          ├── WITHDRAWAL   → approved OfficeDispense
          └── REVERSAL     → original WalletTransaction
```

## Current API boundary

The backend exposes a read-only wallet statement endpoint for guardians and authorized school roles.

The endpoint returns `balance: null` with `balanceStatus: LEDGER_POLICY_REQUIRED` when legacy or malformed ledger rows lack complete signed reversal semantics.

Guardian wallet top-up initiation and controlled office withdrawals are exposed. Verified provider settlement creates the wallet credit; physical withdrawals and reversals are compensating append-only transactions.
