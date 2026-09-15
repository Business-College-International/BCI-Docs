# BCI State Machines

Important school workflows use explicit allowed transitions. Generic status PATCH operations should not bypass these rules.

## Application

```text
DRAFT → SUBMITTED → UNDER_REVIEW → {ADMITTED | REJECTED | NEEDS_INFORMATION}
NEEDS_INFORMATION → SUBMITTED
ADMITTED → ENROLLED
```

Only authorized admissions personnel may transition submitted applications. Admission acceptance must create or link the intended enrolment rather than relying on a frontend side effect.

## Student lifecycle

```text
APPLICANT → ADMITTED → ACTIVE → {SUSPENDED | WITHDRAWN | TRANSFERRED | GRADUATED}
SUSPENDED → ACTIVE
```

Historical enrolments remain queryable. A student changing class/programme/term creates a new enrolment or assignment record rather than rewriting historical data.

## Fee charge

```text
ISSUED → PARTIALLY_PAID → PAID
ISSUED → VOID
PARTIALLY_PAID → VOID (only under controlled reversal rules)
```

A charge cannot be marked paid by the client; payment allocation determines the state.

## Payment

```text
INITIATED → PENDING → {SUCCEEDED | FAILED | CANCELLED | UNKNOWN}
UNKNOWN → {SUCCEEDED | FAILED}
```

Only backend/provider reconciliation may establish terminal provider status. Duplicate callbacks do not create duplicate effects.

## Refund

```text
REQUESTED → UNDER_REVIEW → {APPROVED | REJECTED}
APPROVED → PROCESSING → {SUCCEEDED | FAILED | UNKNOWN}
UNKNOWN → {SUCCEEDED | FAILED}
```

## Wallet withdrawal

```text
REQUESTED → VERIFIED → APPROVED → DISPENSED
                           └──────→ REJECTED
```

The wallet ledger is updated exactly once against the controlled dispense operation.

## Payroll

```text
DRAFT → CALCULATED → REVIEWED → APPROVED → DISPATCHING → {PAID | FAILED | UNKNOWN}
UNKNOWN → {PAID | FAILED}
```

A payroll run may not be approved if required employee/pay-period inputs are incomplete.

## Expense

```text
DRAFT → SUBMITTED → {APPROVED | REJECTED}
APPROVED → PAID → RECONCILED
```

Required approval authority depends on amount/category and must be represented by permissions/policy.

## Stationery order

```text
CART → PAYMENT_PENDING → PAID → PREPARING → READY → COLLECTED
                    └──────────────→ PAYMENT_FAILED
PAID → CANCELLED only under controlled cancellation/refund rules
```

Stock is reserved/decremented according to the documented inventory policy; order totals are server-calculated.

## Announcement delivery

```text
DRAFT → PUBLISHED → DELIVERY_QUEUED → DELIVERY_PARTIAL/DELIVERED
```

Publishing the announcement is a durable backend event. Individual SMS/push deliveries have their own status and retries.
