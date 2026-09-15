# Attendance Analytics & Student Welfare Milestone

Status: implemented at application level; database migration not required for this slice.

## Backend

The attendance analytics module adds class/term reporting without changing the existing attendance write model.

Endpoints:

- `GET /api/v1/attendance/analytics/classes/:classId/summary?termId=...`
- `GET /api/v1/attendance/analytics/classes/:classId/chronic-absence?termId=...`

The analytics service enforces the same scope boundary as attendance marking: directors, principals and office can inspect authorized school data; teachers must have an assignment for the selected class and term.

The class summary derives present, absent, late, excused and total marked counts per student and for the class. Attendance rate treats present + late as attended and is derived from recorded statuses only.

Chronic-absence monitoring is policy-driven at request time. The default is at least 5 marked sessions and an absence rate of at least 20%. The threshold is not hard-coded into the financial or academic record models.

## Web

The staff portal now includes an Attendance Analytics workspace with academic-year, class and term selection, class attendance rate, per-student attendance table and a chronic-absence watchlist.

## Mobile

Guardian academic view now consumes the existing student attendance endpoint and shows attendance rate, status counts and a monitoring indicator when the ward has at least five marked sessions and an absence rate of 20% or more.

This does not introduce a new guardian authorization path; it reuses the existing student attendance access control.

## Deliberate boundaries

- No attendance records are stored or recalculated differently by the analytics layer.
- No automatic disciplinary action is triggered by the watchlist.
- SMS escalation remains provider-neutral and belongs in the notification delivery layer.
- No timetable, room or availability schema is introduced by this milestone.
