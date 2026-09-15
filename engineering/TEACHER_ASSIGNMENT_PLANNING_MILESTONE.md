# Teacher Assignment Planning Milestone

## Completed

- Expanded the existing transactional teacher-assignment service with filtered staff assignment reads.
- Added management-level teacher assignment listing with term/class/subject filters.
- Added controlled unassignment for assignments in open terms.
- Closed-term assignments are immutable through the management endpoint.
- Assignment reads remain staff-management gated for school administration and assignment-scoped for ordinary staff reads.
- Added a dedicated management controller to avoid destabilizing the existing staff controller.
- Added a web Teacher Assignment Planning workspace showing teacher, class, subject, term and room and allowing safe unassignment.
- Added focused backend tests for filtered reads, management reads, closed-term immutability and missing-assignment handling.

## Architectural boundary

- Timetable scheduling is still separate. Teacher assignments establish who teaches what, but there is intentionally no day/period/room-conflict engine until the dedicated timetable schema is introduced and verified.
- Assignment creation remains transactional and validates academic-year, level, programme and term status consistency.
- No assignment mutation is allowed against a closed term.

## Next integration opportunities

- Timetable/versioned scheduling can use TeacherAssignment as its authoritative staffing relationship.
- Workload reporting can aggregate assignment counts/hours once timetable periods exist.
- Teacher substitution/coverage can be added without changing the underlying assignment contract.
