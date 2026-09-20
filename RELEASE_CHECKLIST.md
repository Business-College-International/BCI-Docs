# BCI Production Release Checklist

## Backend
- [ ] TypeScript build succeeds.
- [ ] Lint succeeds.
- [ ] Unit and integration tests pass against a real PostgreSQL test database.
- [ ] Database migrations apply cleanly from an empty database.
- [ ] No destructive migration without an explicit reviewed plan.
- [ ] Environment/configuration validation fails safely on missing secrets.
- [x] HTTP liveness/readiness endpoints exist; readiness verifies PostgreSQL and returns 503 when the database is unavailable.
- [ ] Readiness additionally reports all required background workers/dependencies once those production workers exist.
- [ ] Background jobs are implemented, retryable, and observable; the current backend has no scheduler/worker runtime yet.
- [ ] Provider webhooks are authenticated and idempotent.

## Authorization
- [ ] Every protected endpoint has server-side authorization.
- [ ] Guardian object-scope tests exist.
- [ ] Teacher class/subject scope tests exist.
- [ ] Principal vs Director privilege boundaries are tested.
- [ ] Finance approval separation is tested.
- [ ] Privileged operations are audited.

## Finance
- [ ] Payment/ledger concurrency tests pass.
- [ ] Duplicate request tests pass.
- [ ] Duplicate webhook tests pass.
- [ ] Timeout/unknown provider state is recoverable.
- [ ] Refund limits and approval rules are tested.
- [ ] Wallet cannot cross-contaminate fee balances.
- [ ] Payroll disbursement reconciliation is tested.
- [ ] Daily reconciliation report is usable by finance.

## Mobile/Web
- [ ] Clients are generated/validated against the current API contract.
- [ ] Auth expiry/refresh is tested.
- [ ] Offline/read-cache behavior never claims uncommitted financial truth.
- [ ] Critical errors show actionable state, not fake success.
- [ ] Android production build is signed and tested.
- [ ] Web portal access is scoped by role.
- [ ] Public website does not expose private records.

## Operations

### Current automated operational baseline
The backend exposes `/api/v1/health` and `/api/v1/health/live` for liveness and `/api/v1/health/ready` for database-backed readiness. Health probes bypass the database-backed request limiter so outage detection remains usable when PostgreSQL is unavailable.
- [ ] Backups verified.
- [ ] Restore drill completed.
- [ ] Error monitoring active.
- [ ] Audit log retention policy defined.
- [ ] Provider credentials separated by environment.
- [ ] Incident contact/ownership documented.
- [ ] Rollback plan tested.
