# BCI Database Migration Protocol

The initial PostgreSQL migration is a production foundation and must not be created by hand or inferred from a partial schema.

## Verification sequence

1. Run the `Backend Database Contract` workflow manually from GitHub Actions or through a pull request.
2. Confirm Prisma validation and client generation succeed.
3. Confirm `prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script` produces SQL without errors.
4. Confirm the generated SQL applies successfully to a clean PostgreSQL 16 database.
5. Confirm `prisma migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-database ... --exit-code` returns no drift after application.
6. Confirm the backend build and Jest suite pass in the same environment.
7. Download and inspect the generated SQL artifact for unexpected destructive statements or missing relations/indexes.
8. Only after all checks pass, create the canonical `prisma/migrations/<timestamp>_initial_foundation/migration.sql` from the verified generated SQL in one reviewed change.
9. Run `prisma migrate deploy` against a fresh staging database from that committed migration.
10. Run an application smoke test against staging before production promotion.

## Rules

- Never hand-edit the initial migration to “make it pass”. Fix the Prisma schema and regenerate instead.
- Never generate the migration from a truncated or partially reviewed schema.
- Never point migration generation at a production database.
- Do not mark the migration production-ready solely because `prisma validate` passes; database application and schema equivalence must also pass.
- Do not add payment reservation, wallet ledger, timetable, or other schema changes to the initial migration speculatively after verification has started. Either include a fully reviewed domain model before generation or create a subsequent reviewed migration.

## Current status

The repository contains the complete Prisma schema baseline and a PR/manual-only PostgreSQL contract workflow. A canonical migration has intentionally not yet been committed because the workflow has not been executed and reviewed in the available session.
