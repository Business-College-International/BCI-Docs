# Attendance Write Integrity Milestone

## Scope

Attendance sessions can be created and read while a term is operational. Attendance records become read-only when the selected term is closed or when the session has been finalized (`AttendanceSession.publishedAt` is set).

## Write boundary

`POST /api/v1/attendance/sessions/:sessionId/records` now passes through `AttendanceWriteGuard`, which loads the target session and applies `AttendanceWritePolicyService` before the attendance mutation executes.

Rules:

- unknown session: existing attendance service returns not found;
- closed term: attendance write rejected;
- finalized/published session: attendance write rejected;
- open, unfinalized session: write may continue to the existing teacher/class/term authorization and enrolment validation.

## Existing protections retained

- teacher assignment scope;
- class/term enrolment validation;
- duplicate student IDs rejected per request;
- unique `(sessionId, studentId)` upsert semantics;
- audit record for attendance updates;
- session dates must fall inside the selected term;
- closed terms cannot create new sessions.

## Follow-up

The persisted timetable/substitution layer should eventually provide the authoritative source for session period ownership. Attendance finalization/publication should then be connected to that scheduling state, and a dedicated administrative correction workflow should be introduced for post-finalization amendments rather than reopening ordinary teacher write access.
