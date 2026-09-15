# Finance Read/Reconciliation Milestone

Status: implemented at application-contract level; runtime CI/database verification still pending.

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

- No live payment creation endpoint has been enabled.
- No Moolre request/collection flow is enabled.
- No payment reservation/invoice-target schema has been persisted yet.
- No wallet mutation has been enabled.
- These remain gated by the verified PostgreSQL migration and payment-reservation design.

## Verification status

The latest direct commits have no recorded status checks because the BCI repositories use PR/manual CI rather than direct-main push workflows. Do not interpret an empty status collection as a passed build. The PostgreSQL contract workflow remains the authoritative migration verification path.
