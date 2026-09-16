# Guardian Data Invariants

This document defines the invariants for student/guardian relationships that must hold across the BCI backend and PostgreSQL database.

## Primary guardian invariant

For every student, there may be **zero or one** `GuardianStudent` rows where `isPrimaryContact = true`.

The existing application path serializes primary-guardian mutations with a PostgreSQL transaction-scoped advisory lock keyed by the student ID. This is a concurrency defense for the API path, not the final integrity boundary.

The canonical Prisma migration series must additionally create a PostgreSQL partial unique index:

```sql
CREATE UNIQUE INDEX "GuardianStudent_one_primary_per_student"
ON "GuardianStudent" ("studentId")
WHERE "isPrimaryContact" = true;
```

The migration must be preceded by a duplicate scan. It must fail review if existing data contains more than one primary guardian for the same student rather than silently deleting or choosing a winner.

Recommended duplicate audit query:

```sql
SELECT "studentId", COUNT(*) AS "primaryCount"
FROM "GuardianStudent"
WHERE "isPrimaryContact" = true
GROUP BY "studentId"
HAVING COUNT(*) > 1;
```

The expected result is zero rows before the unique index is created.

## Guardian-link uniqueness

The existing Prisma schema already requires one link per `(guardianId, studentId)` pair. This prevents duplicate relationship rows for the same guardian/student pair and remains distinct from the primary-contact invariant.

## Relationship capability invariants

`canViewAcademic`, `canPayFees`, and `canManageWallet` are relationship-level capabilities. They must be evaluated from the specific authenticated guardian-to-student link, not inferred from the guardian role alone.

A guardian may have multiple wards with different capability combinations.

## Mutation requirements

Guardian-link creation/removal must remain transactional and audited. A primary replacement must clear the previous primary and create the replacement inside the same transaction.

All future write paths that can create or update `GuardianStudent.isPrimaryContact` must either use the same service transaction or rely on the database constraint. Direct administrative scripts must never bypass the invariant.

## Release gate

This invariant is considered release-ready only when:

1. the duplicate audit returns zero rows;
2. the canonical Prisma migration contains the partial unique index;
3. a clean PostgreSQL schema apply succeeds;
4. the backend test suite covers primary replacement and duplicate-link behavior; and
5. the migration is reviewed before any live school database is migrated.
