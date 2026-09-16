# Timetable Draft Validation Milestone

## Completed

BCI now has a provider/database-independent timetable draft validator in `bci-backend-api`.

Endpoint:

`POST /api/v1/timetable/validate-draft`

Protected by `academics.manage`.

The request identifies a term and a list of proposed slots. Each slot references an existing `TeacherAssignment` and contains:

- day of week (1-7)
- start time
- end time
- room

## Validations

The validator currently rejects:

- missing or unknown assignments;
- assignments from a different term;
- invalid days;
- invalid time windows;
- missing rooms;
- duplicate placement of the same teaching assignment;
- teacher overlap;
- class overlap;
- room double-booking.

A maximum of 500 draft slots is enforced.

## Important boundary

This is a validation/preflight capability only. The repository currently has no persisted timetable model or migration history that can safely be extended. No timetable data is written by this endpoint.

The eventual timetable migration should add durable schedule entries plus publication/version state and then reuse these conflict rules inside the transactional write path.

## Future persisted model requirements

The production timetable schema should support:

1. term and academic-year ownership;
2. teaching-assignment linkage;
3. day/period or exact start/end time;
4. room linkage or normalized room entity;
5. draft/published/voided state;
6. timetable versioning;
7. substitution/override records;
8. database-backed conflict protection for teacher/class/room collisions;
9. immutable published versions;
10. a link from attendance-session creation to the published timetable entry.

## Tests

Regression coverage verifies teacher/class/room conflicts and duplicate/invalid placement conditions.

The timetable persistence migration must still pass the controlled PostgreSQL schema contract workflow before becoming canonical.
