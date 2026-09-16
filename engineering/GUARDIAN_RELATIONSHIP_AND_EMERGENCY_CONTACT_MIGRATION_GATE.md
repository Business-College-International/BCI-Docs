# Guardian Relationship and Emergency Contact Migration Gate

## Current schema findings

The current Prisma schema stores `GuardianStudent` links with `isPrimaryContact`, `canViewAcademic`, `canPayFees`, and `canManageWallet`.

The database does not currently enforce one primary guardian per student.

`EmergencyContact` is attached to `Person`; `Student` has no direct emergency-contact relation. This prevents the application from representing a student's emergency contacts as a first-class student record.

## Required migration

### 1. Student emergency contacts

Add a nullable/required student foreign key to the emergency-contact entity and a `Student.emergencyContacts` relation. Preserve existing `Person`-level emergency contacts during migration; do not reinterpret existing rows without an explicit ownership mapping.

The migration must include a backfill strategy for any existing emergency contacts before making the new student relationship mandatory.

### 2. One primary guardian per student

The application currently audits this invariant but the database does not enforce it.

The final PostgreSQL migration should enforce at most one `GuardianStudent.isPrimaryContact = true` row per `studentId`, using a partial unique index.

Before adding the index, the migration must identify and resolve duplicate primary rows deterministically through a controlled data-cleanup process rather than silently choosing a guardian.

### 3. Permission semantics

Guardian portal permissions remain independent flags:

- `canViewAcademic`
- `canPayFees`
- `canManageWallet`

The application must continue to honor these independently. No migration should infer one permission from another.

## Promotion gate

Do not hand-edit production SQL as a substitute for a generated Prisma migration.

The migration must be generated from the canonical Prisma schema, applied to a clean PostgreSQL database, checked for drift, and covered by regression tests before promotion.

Required regression cases:

- exactly one primary guardian;
- no primary guardian;
- duplicate primary guardians rejected after migration;
- guardian can pay fees without academic visibility;
- guardian can view academics without wallet management;
- emergency contact belongs to exactly one student after migration;
- legacy person-level emergency contacts remain accounted for.
