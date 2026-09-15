# BCI Payroll, Expenses, and Staff Mobile Milestone

## Completed

### Expenses
- Draft expense creation for authorized finance/staff roles.
- Explicit `DRAFT -> SUBMITTED -> APPROVED/REJECTED` lifecycle.
- Submitter cannot approve or reject their own expense.
- Mutations are transactional and audited.
- Monetary values are serialized with fixed decimal precision.
- Actual payment/disbursement remains separate from expense approval.

### Payroll
- Deterministic compensation calculator using fixed amounts and base-pay percentages.
- Negative net-pay calculations are rejected.
- Malformed allowance/deduction JSON fails closed.
- Draft payroll periods can be calculated into payroll entries.
- Calculated periods can be approved by authorized payroll management roles.
- Payroll entries capture the calculation result and status transition.
- Payroll read APIs remain available to staff for their own payroll history.
- No payroll disbursement endpoint has been enabled.

### Staff mobile parity
- Authenticated staff users are now recognized by the Flutter app.
- Staff mobile dashboard shows staff identity, employment status, duties and teaching assignments.
- Staff mobile dashboard shows read-only payroll history and current base pay when authorized.
- Ordinary teachers/support staff receive `payroll.read` in the seed defaults; principal/office receive payroll read without payroll management.

## Explicit limitations / gates

The current payroll schema has `approvedBy` but does not persist `calculatedBy`. Therefore true maker/checker separation between payroll calculation and payroll approval cannot yet be enforced at the database level. The application currently enforces role access and approval-state transitions, but a future migration should add the calculation actor and approval audit fields before production payroll approval is considered fully segregated.

The current schema also does not include a durable conversation/thread/message model. Staff chat is therefore not being fabricated in application code; it remains a deliberate schema-design gate.

The PostgreSQL migration contract remains unverified until the manual/PR database workflow executes successfully. No production migration is claimed from static inspection alone.
