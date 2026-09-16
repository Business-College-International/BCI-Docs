# Director Audit and Compliance Milestone

## Completed

The BCI backend now exposes a read-only audit log API at `GET /api/v1/audit/logs`, protected by `audit.read`.

Supported filters:
- actor user ID
- audit action
- entity type
- entity ID
- date range
- page/page size

Page size is bounded at 100 records. Invalid pagination and date values are rejected at the service boundary.

The audit center never exposes a mutation endpoint. Existing audit records remain append-oriented evidence written by domain services.

The BCI web staff portal now includes an Audit Center for users holding `audit.read`. It supports filtering, pagination and inspection of before/after JSON for each audit event.

## Governance boundary

The audit center is evidence, not a workflow engine. It cannot edit or delete audit records.

Actor IDs are shown directly because the current audit read contract does not join a staff/guardian directory. No names are guessed or synthesized from partial data.

## Future compliance work

- retention policy and archival strategy;
- export package generation for authorized investigations;
- immutable database-level append protections;
- actor directory joins once a dedicated compliance reporting contract exists;
- alerting for high-risk financial/security events.
