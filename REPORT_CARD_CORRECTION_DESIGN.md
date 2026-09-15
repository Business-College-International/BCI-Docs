# BCI Report-Card Correction Design

## Purpose

Published report cards are official records. Their snapshots must never be edited in place.

## Version model

Each report-card publication has a monotonically increasing `publicationVersion` for the same student + term.

- Version 1 is the first published snapshot.
- A correction creates version 2+ from a newly calculated snapshot.
- Previous versions remain immutable and readable to authorized staff.
- Exactly one version may be the current published version.

## Correction lifecycle

`PUBLISHED -> CORRECTION_REQUESTED -> APPROVED -> PUBLISHED`

Rejected requests do not change the current published version.

## Required correction data

A correction request must capture:

- student
- term
- current published version
- proposed replacement snapshot
- reason
- requested by
- requested timestamp
- reviewed by
- reviewed timestamp
- approval/rejection decision
- decision note

The replacement snapshot must include the grading-policy version used for the recalculation.

## Safety rules

1. Published snapshots are append-only.
2. A correction cannot target a non-current publication version.
3. The replacement snapshot must be recalculated from authoritative assessment data.
4. Manual grade edits are not allowed in the snapshot itself.
5. Approval is separate from request creation.
6. Every request, approval, rejection, publication and void action is audited.
7. Guardians see only the current published version; privileged staff may inspect publication history.
8. A correction may not silently change the historical grading-policy version attached to an already published snapshot.
9. A voided publication remains immutable and is never reused.

## Recommended database shape

A future migration should introduce two append-oriented entities:

### `ReportCardPublication`

- id
- studentId
- termId
- publicationVersion
- status
- snapshotJson
- gradingPolicyVersionId
- publishedAt
- publishedBy
- voidedAt
- voidedBy
- createdAt

Unique constraint: `(studentId, termId, publicationVersion)`.

A separate uniqueness rule should enforce at most one current published version per student + term through application transaction logic plus a database-safe strategy during migration design.

### `ReportCardCorrectionRequest`

- id
- studentId
- termId
- targetPublicationId
- replacementSnapshotJson
- reason
- requestedBy
- requestedAt
- decision
- decidedBy
- decidedAt
- decisionNote

## Approval boundary

Teachers may request a correction for their assigned subject/class context.

Office/principal/director users may review and approve corrections according to explicit permission codes.

A correction approval must create the replacement publication atomically with the correction decision.

## Current status

This is design-only. No Prisma model has been added yet. Database implementation remains behind the verified PostgreSQL migration gate.
