- `wallet.topup.succeeded.v1`
- `wallet.withdrawal.completed.v1`
- `payroll.disbursement.succeeded.v1`
- `announcement.published.v1`

Events contain identifiers and relevant state snapshots but never credentials or secrets.

## Financial contract implementation status

Fee payment initiation now creates explicit per-invoice `PaymentIntent` reservations with a 15-minute expiry. Active `PENDING`/`PROCESSING` intents reduce available invoice balance; expired intents are excluded from new availability. The existing idempotency ledger remains the request-replay authority.

`PaymentAllocation` is created only after a verified provider-success webhook and must reconcile exactly to the PaymentIntent reservation total. Provider rejection/unknown/OTP paths update PaymentIntent state without treating the reserved amount as collected funds.

Wallet top-ups create the signed CREDIT ledger effect only after verified provider success. Office withdrawals create signed DEBIT effects together with durable withdrawal evidence; posted corrections use compensating reversal entries.

Expiry reconciliation is self-healing during new fee-payment reservation attempts: stale PENDING/PROCESSING intents are marked EXPIRED before availability is recalculated. Real-money Moolre verification remains the production gate.

## Student documents

- GET /api/v1/student-records/students/:studentId/documents — authorized document reads with guardian/teacher object scope plus `students.read`.
- POST /api/v1/student-records/students/:studentId/documents — staff document creation with `students.manage`.
- DELETE /api/v1/student-records/documents/:documentId — staff document removal with `students.manage`.

## Assessment and report-card API

- POST /api/v1/assessments — create an assessment for an assigned subject/term while the term is open.
- GET /api/v1/assessments/assigned?classId=...&termId=...&subjectId=... — return teacher-assigned assessments with results scoped to the selected active class roster.
- POST /api/v1/assessments/:assessmentId/results — enter or correct assessment results; published report-card students require a pending correction request, including after term closure.
- GET /api/v1/assessments/students/:studentId?termId=... — read authorized assessment results.
- GET /api/v1/academic-reports/students/:studentId/current — resolve the current server-authoritative academic report.
- GET /api/v1/academic-reports/students/:studentId/terms/:termId — read the term report.
- POST /api/v1/academic-reports/students/:studentId/terms/:termId/corrections — request a controlled correction for the current published report.
- GET /api/v1/academic-reports/students/:studentId/terms/:termId/corrections — list correction requests.
- POST /api/v1/academic-reports/corrections/:id/approve — approve and publish the freshly recalculated next immutable report version.
- POST /api/v1/academic-reports/corrections/:id/reject — reject a pending correction request.

## Grading policy API

- GET /api/v1/grading-policies — read versioned grading policies; filtered by academic year, level and programme.
- POST /api/v1/grading-policies — create a DRAFT policy with explicit academic-year/level scope and school-supplied grade bands.