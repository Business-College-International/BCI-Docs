# BCI Security Boundaries

## 1. Identity

One identity system is used across mobile, web portal and public admissions flows. A `User` authenticates; one user may have several roles. A person record (guardian, staff member, etc.) is a domain object, not an authentication record.

Phone numbers are first-class identifiers in the Ghanaian operating context, but email may also be used. Credentials are never returned by APIs.

## 2. Authorization

Authorization is evaluated on every protected request using:

`actor identity + active permission + organization/school scope + object ownership/relationship + lifecycle state`

UI hiding is not authorization.

The backend now has a reusable server-side permission-code guard backed by `UserPermission`. This is the foundation for moving away from coarse job-title checks. Permission provisioning and object/scope evaluation are separate work and must be completed before production access control is considered complete.

Examples:
- A guardian may read only students linked to that guardian.
- A teacher may mark attendance only for sessions/classes they are assigned to, during an allowed state window.
- Office staff may enter expenses but may not approve their own expense when approval separation is required.
- Finance staff may prepare payroll but payroll disbursement requires the configured approval authority.
- Principal authority is below Director authority and must be represented by permissions, not frontend assumptions.
- Director-level operations are explicitly audited.

## 3. Request correlation

Every HTTP request receives a server-generated `X-Request-Id` response header. The identifier must be propagated into application logs and privileged audit records so one business operation can be traced across API, worker and provider events.

## 4. Data sensitivity

### Highly sensitive
Student medical notes, identity documents, guardian contact data, staff salary data, payroll records, financial transactions and authentication/security records.

### Sensitive
Academic results, attendance, disciplinary records, application data, internal messages and operational reports.

### Public
Approved public website content, public admissions requirements, school contact information and intentionally published announcements.

APIs must return only fields necessary for the actor's use case.

## 5. Files

Every upload has an owner and purpose. Access to private files is authorized server-side and uses short-lived signed URLs where appropriate. Public website assets are explicitly public; student/staff documents are not.

## 6. Audit

Audit events are append-only. At minimum record actor, action, target type/id, request id, timestamp, relevant scope, outcome and enough metadata to reconstruct the business operation without storing credentials or unnecessary personal data.

Audit particularly privileged changes: role/permission changes, student status changes, admission decisions, fee changes, payment/refund operations, wallet withdrawals, payroll approval/disbursement, expense approval and inventory adjustments.

## 7. Sessions and tokens

Prefer short-lived access tokens plus revocable refresh/session records. Passwords are hashed using a modern password hashing algorithm. Sensitive actions may require step-up authentication or recent-authentication checks.

Device/session revocation is supported.

## 8. Integration security

Provider webhooks are authenticated and deduplicated. Never trust a webhook body alone to identify an internal object; map through provider reference + expected transaction context.

Secrets are environment-managed and never committed. Production credentials are separate from test/mock credentials.

## 9. Privacy

BCI is expected to handle personal data in Ghana. Data collection must have a defined operational purpose, retention should be deliberate, and privileged access should be minimized. Legal/compliance decisions require formal review before production policy is finalized.
