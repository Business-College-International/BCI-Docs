# BCI Engineering Batch — 2026-09-16

## Guardian relationship integrity hardening

The guardian/student relationship path received a concurrency hardening pass.

- Primary-guardian replacement is serialized with a PostgreSQL transaction-scoped advisory lock keyed by student ID.
- The API transaction still clears the previous primary and creates the replacement atomically.
- Regression tests verify the primary path acquires the lock and non-primary links do not.
- The database remains the final integrity boundary: the canonical initial migration must create a partial unique index enforcing at most one primary guardian per student.

Backend commits:
- `72116d2` — serialize primary guardian mutations per student.
- `8cc169d` — regression coverage for the serialization behavior.

## Database migration gate

The backend release gate now documents that there is no canonical production Prisma migration history yet. The repository's migration-review workflow can generate a baseline artifact, but that artifact is review-only until a controlled migration is committed and verified against clean PostgreSQL.

A draft PR `#6` was opened solely to trigger the existing migration-review workflow against the current schema. No production migration is being claimed by that PR, and no live database has been modified.

## Guardian data invariant

`GUARDIAN_DATA_INVARIANTS.md` defines the required duplicate scan and PostgreSQL partial unique index for `GuardianStudent.isPrimaryContact`.

## Current blocker

The initial production migration still requires a successful clean PostgreSQL generation/apply/reconciliation cycle before BCI should operate against a real school database. Payment-intent schema changes remain coupled to this first controlled migration so the system does not accumulate speculative intermediate migrations.
