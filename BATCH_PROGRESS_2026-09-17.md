# BCI Engineering Batch — 2026-09-17

## Backend integrity hardening

The backend integrity pass continued from the finance/academic/student lifecycle work.

- Payroll period calculation and approval now serialize concurrent state transitions with PostgreSQL `SERIALIZABLE` transactions and translate `P2034` serialization conflicts into retryable 409 conflicts.
- Student graduation/transfer terminal mutations now serialize with PostgreSQL `SERIALIZABLE` transactions and translate `P2034` conflicts into retryable lifecycle conflicts.
- Application review now uses a conditional state transition inside its transaction, preventing a concurrent reviewer from overwriting a newer application state.
- Capacity-sensitive admissions now run in a `SERIALIZABLE` transaction so concurrent admissions cannot both rely on the same remaining class capacity; serialization conflicts are surfaced as retryable conflicts.

Merged backend PRs:
- `#25` — `fix(payroll): serialize period state transitions`
- `#26` — `fix(admissions): serialize review and capacity-sensitive admissions`

## Database contract verification

PR `#26` completed both backend verification gates successfully:

- Backend CI: Prisma validation/generation, build, and Jest test suite all passed.
- Backend Database Contract: PostgreSQL 16 was initialized; the complete Prisma schema was converted to SQL, applied to a clean database, compared back to the schema, then the backend was built and tested successfully.
- Generated initial SQL artifact: `bci-initial-postgres-schema`, workflow run `35257174367`.

This proves the current complete Prisma schema can generate and reconcile cleanly against PostgreSQL. It does **not** by itself establish a production migration history.

## Remaining migration gate

The next database step is to commit the verified initial Prisma migration from the clean-schema artifact, then run migration deployment/reconciliation against a fresh PostgreSQL instance and preserve that migration as the canonical production baseline. No live-school database should be treated as migrated until that gate is explicitly completed.

## Next execution focus

After the migration baseline is established, continue the remaining object/scope authorization audit and web/mobile academic parity, then proceed through the payment reservation/reconciliation, grading/report-card, payroll disbursement, wallet ledger, inventory, communication, and production-hardening gates defined by the master plan.