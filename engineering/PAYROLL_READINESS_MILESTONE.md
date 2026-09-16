# Payroll readiness milestone

## Completed

BCI now has a read-only payroll readiness report at `GET /api/v1/payroll/readiness/periods/:periodId` for director, accountant and principal roles with payroll-read permission.

The report verifies:

- payroll lifecycle state;
- presence of payroll entries;
- exact `grossPay - totalDeductions = netPay` arithmetic using Prisma Decimal values;
- non-negative gross pay and deductions;
- deductions not exceeding gross pay;
- active staff employment status;
- approval state consistency between period and entries;
- successful disbursements not exceeding net pay;
- more than one processing disbursement on an entry;
- a period marked `PAID` while entries remain unpaid.

The web staff portal exposes the same report with period selection, totals, blocking findings and per-staff payment state.

## Intentional boundary

This milestone is read-only. It does not initiate, retry or approve provider disbursements. Provider execution remains behind the existing disbursement/payment-provider verification gates.

## Financial correctness

All readiness monetary calculations use `Prisma.Decimal`. JavaScript floating-point arithmetic is intentionally not used for payroll totals or comparisons.

## Next gates

The remaining payroll financial gates are provider contract verification, idempotent disbursement reservation, verified provider callbacks, reconciliation, reversal handling and final period closure.
