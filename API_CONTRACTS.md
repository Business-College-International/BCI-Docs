# BCI API and Cross-Repository Contracts

## Contract principles

- APIs are versioned under `/api/v1`.
- Request/response schemas are explicit; do not return raw ORM objects.
- Every mutation returns the committed authoritative state or a durable operation reference.
- Error responses have stable machine-readable codes.
- Pagination/filtering/sorting semantics are documented per collection.
- Dates are ISO 8601 with timezone; monetary values are decimal strings in API payloads to avoid floating-point ambiguity.
- Sensitive fields are omitted unless the caller has permission to view them.

## Mutation envelope

Every retryable mutation may use:

```http
Idempotency-Key: <client-generated-stable-key>
X-Request-Id: <request-correlation-id>
```

The backend persists idempotency state for operations where duplicate side effects would be harmful.

## Example error shape

```json
{
  "error": {
    "code": "PAYMENT_ALREADY_FINAL",
    "message": "The payment has already reached a terminal state.",
    "requestId": "...",
    "details": {}
  }
}
```

Clients branch on `code`, not localized message text.

## Core endpoint families

### Auth
- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh`
- `POST /api/v1/auth/logout`
- `GET /api/v1/me`

### Admissions
- `POST /api/v1/applications`
- `GET /api/v1/applications/:id`
- `POST /api/v1/applications/:id/submit`
- `POST /api/v1/applications/:id/review`
- `POST /api/v1/applications/:id/admit`
- `POST /api/v1/applications/:id/reject`

### Students
- `GET /api/v1/students`
- `GET /api/v1/students/:id`
- `POST /api/v1/students/:id/enrolments`
- `POST /api/v1/students/:id/transfer`
- `POST /api/v1/students/:id/withdraw`
- `POST /api/v1/students/:id/promote`

### Academics
- `GET /api/v1/academic-years`
- `GET /api/v1/terms`
- `GET /api/v1/classes`
- `GET /api/v1/teacher-assignments`
- `POST /api/v1/attendance/sessions`
- `POST /api/v1/attendance/sessions/:id/records`
- `GET /api/v1/students/:id/attendance`

### Finance
- `GET /api/v1/finance/fee-schedules?termId=...`
- `POST /api/v1/finance/fee-schedules`
- `POST /api/v1/finance/invoices`
- `GET /api/v1/finance/students/:studentId/invoices`
- `POST /api/v1/finance/students/:studentId/payments`
- `POST /api/v1/finance/students/:studentId/payments/:paymentId/otp`
- `GET /api/v1/finance/students/:studentId/receipts`
- `POST /api/v1/refunds`

### Wallet
- `GET /api/v1/wallets/students/:studentId`
- wallet top-up initiation for guardians with `canManageWallet`
- wallet top-up OTP continuation through the payment OTP contract
- controlled office withdrawal/reversal operations

### Payroll
- `GET /api/v1/payroll/periods`
- `POST /api/v1/payroll/periods`
- `POST /api/v1/payroll/periods/:id/calculate`
- `POST /api/v1/payroll/periods/:id/approve`
- `POST /api/v1/payroll/entries/:id/disburse`
- `GET /api/v1/me/payslips`

## Webhooks

Provider webhooks are backend-only endpoints, not exposed through mobile/web app clients. Each integration defines authentication, deduplication key, mapping rules, allowed state transitions and reconciliation fallback.

## Event contracts

Important committed events use stable event names and versioned payload schemas, for example:

- `student.admission.accepted.v1`
- `student.enrolment.created.v1`
- `attendance.session.published.v1`
- `payment.succeeded.v1`
- `refund.succeeded.v1`
- `wallet.topup.succeeded.v1`
- `wallet.withdrawal.completed.v1`
- `payroll.disbursement.succeeded.v1`
- `announcement.published.v1`

Events contain identifiers and relevant state snapshots but never credentials or secrets.

## Financial contract implementation status

Fee payment initiation now creates explicit per-invoice `PaymentIntent` reservations with a 15-minute expiry. Active `PENDING`/`PROCESSING` intents reduce available invoice balance; expired intents are excluded from new availability. The existing idempotency ledger remains the request-replay authority.

`PaymentAllocation` is created only after a verified provider-success webhook and must reconcile exactly to the PaymentIntent reservation total. Provider rejection/unknown/OTP paths update PaymentIntent state without treating the reserved amount as collected funds.

Wallet top-ups create the signed CREDIT ledger effect only after verified provider success. Office withdrawals create signed DEBIT effects together with durable withdrawal evidence; posted corrections use compensating reversal entries.

Automated expiry reconciliation and real-money Moolre verification remain production gates.

## Grading policy API

- GET /api/v1/grading-policies — read versioned grading policies; filtered by academic year, level and programme.
- POST /api/v1/grading-policies — create a DRAFT policy with explicit academic-year/level scope and school-supplied grade bands.
- PATCH /api/v1/grading-policies/:id — edit a DRAFT policy only.
- POST /api/v1/grading-policies/:id/publish — publish an otherwise valid DRAFT transactionally; only one ACTIVE policy is permitted for an identical scope.
- POST /api/v1/grading-policies/:id/retire — retire an ACTIVE policy.

The report service resolves an ACTIVE programme-specific policy first and falls back to a generic policy for the same academic year/level. The returned report identifies the policy version used for grade resolution. No grade is assigned when the relevant policy is absent.

## Report-card publication API

- `POST /api/v1/academic-reports/students/:studentId/terms/:termId/publications` — prepare an immutable snapshot after term closure and readiness checks.
- `POST /api/v1/academic-reports/publications/:id/publish` — publish a prepared snapshot; the backend rejects stale snapshots and duplicate current publications.
- `POST /api/v1/academic-reports/publications/:id/void` — void a published snapshot; a reason is mandatory and the original snapshot remains immutable.
- `GET /api/v1/academic-reports/students/:studentId/terms/:termId/publications/current` — return the current published snapshot subject to normal report-read access scope.