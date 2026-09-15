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
- `GET /api/v1/students/:id/charges`
- `POST /api/v1/payment-intents`
- `GET /api/v1/payments/:id`
- `POST /api/v1/payments/:id/reconcile`
- `GET /api/v1/students/:id/receipts`
- `POST /api/v1/refunds`

### Wallet
- `POST /api/v1/students/:id/wallet/top-up-intents`
- `GET /api/v1/students/:id/wallet`
- `POST /api/v1/students/:id/wallet/withdrawals`

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
