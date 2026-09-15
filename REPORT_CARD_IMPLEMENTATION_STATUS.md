# Report Card Implementation Status

## Implemented foundations

- Configurable grading policy design: `GRADING_POLICY_DESIGN.md`
- Policy-agnostic grading engine with boundary/gap/overlap validation
- Immutable snapshot payload builder with explicit schema version
- Report publication state machine
- Correction request/versioning design: `REPORT_CARD_CORRECTION_DESIGN.md`
- Publication/correction state tests

## Not yet implemented

- Prisma grading-policy tables
- Prisma report-card publication tables
- Prisma correction-request tables
- Publication/correction API endpoints
- Official grade persistence
- Report-card PDF generation
- Guardian publication-history UI

All database-backed publication work remains blocked on successful PostgreSQL contract verification and the canonical initial Prisma migration.
