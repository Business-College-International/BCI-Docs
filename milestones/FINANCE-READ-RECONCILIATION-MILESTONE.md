# Finance Read/Reconciliation Milestone

Status: implemented at application-contract level; backend PR CI and PostgreSQL contract verification are passing on the current implementation branch.

## Backend

- Fee schedules and invoice issuance remain active.
- Guardian invoice reads remain scoped by guardian-student `canPayFees`.
- Successful payment receipt history is available per ward.
- Finance summary reports invoice counts, invoiced amount, allocated amount, outstanding amount, collected payment amount, and pending/processing amount separately.
- Pending and processing payments are never counted as collected.
- Summary supports optional term/date filters.
- Receipt access is guarded by the same guardian finance scope.

## Web

- Accountant/director/office users with `finance.read` see a read-only reconciliation workspace.
- The workspace distinguishes collected from pending/processing amounts.

## Mobile

- Guardians with `canPayFees` can open payment history for a ward.
- Receipt records include receipt number when available, completed date, payment provider/reference, and invoice allocations.

## Deliberate boundary

- Payment initiation, wallet top-up and stationery payment reservation paths exist in the backend.
- The Moolre adapter remains MOCK unless the explicit live-money configuration gate is satisfied.
- Successful verified provider settlement is the only path that mutates collected-payment/wallet/stationery financial state.
- Fee allocation remains separate from wallet and stationery ledgers.

## Verification status

The latest direct commits have no recorded status checks because the BCI repositories use PR/manual CI rather than direct-main push workflows. Do not interpret an empty status collection as a passed build. The PostgreSQL contract workflow remains the authoritative migration verification path.
