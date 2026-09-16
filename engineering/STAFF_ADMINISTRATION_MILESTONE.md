# Staff Administration Milestone

## Completed

- Staff record read endpoint for authorized staff managers.
- Employment record updates for department, contract type and employment status.
- Termination is blocked while unresolved payroll entries remain.
- Staff duties can be explicitly completed/inactivated with audit records.
- Existing authentication roles are intentionally not modified by employment-status changes.
- Staff web portal now exposes the controlled administration workflow.
- Backend tests cover unresolved-payroll termination protection, duty completion auditing, repeated completion rejection and unknown-staff handling.

## Boundaries

- Account suspension/revocation remains a separate authentication operation.
- Salary structure changes remain within payroll/finance controls.
- Staff deletion is not exposed; records remain auditable.
- Full HR history (contracts, leave, performance, documents) requires additional domain modeling.
