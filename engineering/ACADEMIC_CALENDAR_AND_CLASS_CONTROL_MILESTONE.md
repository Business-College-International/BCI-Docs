# Academic Calendar and Class Control Milestone

## Completed

- Academic years can be created with validated date ranges and an audited `isCurrent` marker.
- Administrators can switch the single current academic year transactionally.
- Terms are constrained to their academic year's date range.
- Term transitions are explicit: `DRAFT -> OPEN` and `OPEN -> CLOSED` only.
- Opening a term closes other terms in the same academic year.
- Reopening a closed term is rejected.
- Academic years and terms are shown in deterministic chronological order.
- Classes can be created with level/programme consistency checks.
- Class capacity must be positive when provided.
- Administrative class edits are audited.
- Class capacity cannot be reduced below its current active enrolment population.
- Office portal now has an Academic Control Center for year selection/current-year control, term transitions, and safe class edits.
- Focused backend tests cover term opening, illegal reopening, and class-capacity protection.

## Security and integrity boundaries

- All mutations require `academics.manage`.
- Existing teacher class read scoping is unchanged.
- Calendar state changes remain server-authoritative; the web portal does not locally infer whether a transition is legal.
- Class level/programme remain immutable through the edit endpoint so existing academic relationships are not silently reclassified.

## Deferred deliberately

- Bulk calendar cloning from a previous academic year.
- Timetable entries and publishing, which require their own schema.
- Intra-term class transfer history, which requires the historical enrolment design.
- Fully automated term rollover, which must be introduced only after verified PostgreSQL migration and operational policy review.
