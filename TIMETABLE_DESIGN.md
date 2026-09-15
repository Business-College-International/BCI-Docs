# BCI Timetable Design Gate

## Status

Design only. No timetable tables or endpoints should be implemented until this model is reviewed against the school’s actual timetable process.

## Why this is a schema gate

The current BCI database has teacher assignments (`Staff + Class + Subject + Term`) but no persisted timetable entity. A timetable cannot safely be represented by reusing `TeacherAssignment`; one teacher assignment may meet on multiple days/periods, and a timetable entry needs its own historical/effective context.

## Proposed domain

```text
Timetable
  ├─ AcademicYear
  ├─ Term
  ├─ Name / version
  ├─ EffectiveFrom
  ├─ EffectiveTo
  └─ Status
       │
       └──< TimetableEntry
              ├─ TeacherAssignment
              ├─ DayOfWeek
              ├─ Period
              ├─ StartTime
              ├─ EndTime
              ├─ Room
              └─ optional notes
```

## Required invariants

1. A timetable entry must reference a teacher assignment from the same term.
2. The assignment's class, subject and teacher remain the authoritative identities; the timetable entry must not duplicate them as independent editable facts.
3. An entry may not overlap another entry for the same teacher, class, or room within the same timetable version/day.
4. Timetable versions must be immutable once published. Changes create a new draft/version rather than rewriting the published historical timetable.
5. Only one timetable version may be published for a given school/term context unless an explicit transition window is modeled.
6. A teacher-facing “my timetable” query must be scoped through the authenticated staff identity, never through a client-supplied teacher ID alone.
7. A class-facing timetable query must be scoped to a class and term the actor is authorized to view.
8. Room conflicts must be checked transactionally when publishing or activating a timetable.
9. Public/student/guardian clients receive only the published timetable version; drafts remain staff-only.

## Period model

Do not hard-code `Period 1`, `Period 2`, etc. as application constants. A school-level `TimetablePeriod` configuration should eventually define:

- label
- ordinal
- start time
- end time
- active days
- optional break/lunch classification

The entry can reference a period configuration while retaining explicit start/end snapshots where historical rendering requires them.

## Publishing workflow

```text
DRAFT → VALIDATING → READY → PUBLISHED → RETIRED
             └──────→ INVALID
```

Validation must detect teacher/class/room overlaps and assignment mismatches before publishing.

## Implementation order

1. Confirm BCI's actual timetable period/day process with school administration.
2. Add timetable period/configuration schema.
3. Add timetable version + entry schema.
4. Add conflict-validation service and tests.
5. Add staff timetable read API.
6. Add class timetable read API.
7. Add web timetable editor for authorized staff.
8. Add teacher mobile timetable view.
9. Add guardian/student published timetable view where appropriate.

## Explicit non-goals for the current schema

Do not add timetable columns to `TeacherAssignment`.
Do not store a mutable “current timetable” JSON blob on `Staff` or `SchoolClass`.
Do not allow clients to overwrite published timetable entries in place.
