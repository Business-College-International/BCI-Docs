# Academic Operations Milestone

## Implemented

This milestone extends BCI academic administration without changing the Prisma schema.

### Scoped class rosters

`GET /api/v1/school-classes/:classId/roster?termId=...`

Office/director/principal/accountant roles can inspect the selected roster. Teachers must have a matching `TeacherAssignment` for the exact class and term. The class and term must belong to the same academic year.

The response includes the active enrolment population, admission number, student identity, primary guardian contact summary, class metadata, term metadata and capacity.

### Elective assignment

`POST /api/v1/students/:id/electives`

`DELETE /api/v1/students/:id/electives/:subjectId?termId=...`

The server verifies:

- student is active;
- the selected term has an active enrolment for the student;
- the subject exists, is active and is marked elective;
- subject level matches the enrolment level;
- programme-compatible electives are enforced;
- duplicate elective assignments are rejected;
- create/remove actions are audited transactionally.

### Academic progression

`POST /api/v1/students/:id/progress`

Progression is a term-to-later-term transition. The current active enrolment is completed and a new active enrolment is created atomically.

The server verifies:

- student is active and has an active enrolment;
- target term is later than the current enrolment;
- target class belongs to the target term academic year;
- target class matches requested target level/programme;
- target class capacity is available;
- the student does not already have a target-term enrolment;
- the complete transition is audit logged.

### Web portal

The staff workspace now contains an authoritative class-roster workspace with academic-year, term and class selection. The web API client exposes the new roster, elective and progression contracts.

## Deliberate exclusions

### Intra-term transfer

Not implemented yet. The current schema has `@@unique([studentId, termId])` on `Enrolment`, which is appropriate for one authoritative placement per student per term but does not safely represent multiple intra-term placements while preserving immutable history. A dedicated transfer-history design is required before that workflow is introduced.

### Official report publication

Still migration-gated. Published report cards require immutable persisted snapshots and versioned correction history.

### Timetable

Still migration-gated. The schema currently contains teacher assignments but no timetable/version/period/room entities.

## Verification coverage

Added focused tests for:

- class-roster teacher authorization;
- class-roster successful authorized read;
- progression into a full target class;
- duplicate elective assignment.

The project continues to use PR/manual CI rather than push-triggered verification.
