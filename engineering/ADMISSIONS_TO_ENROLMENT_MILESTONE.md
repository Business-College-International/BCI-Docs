# Admissions → Enrolment Operational Milestone

## Completed

- Staff admission queue for pending and under-review applications.
- Current academic year and currently open terms surfaced alongside applications.
- Placement options match application level and programme.
- Class occupancy is calculated per open term and class using active enrolments.
- Capacity is evaluated using the same term-scoped population relevant to admission.
- Queue reports whether an application currently has a valid placement option.
- Staff web workspace supports starting review, rejecting with a reason, and admitting into a selected open term/class.
- Admission-number entry remains optional and is validated by the authoritative admission transaction.
- Existing admission transaction remains atomic for application status, student creation, guardian linking, enrolment creation, decision record and audit record.

## Safety boundaries

The queue is advisory; the existing admission transaction remains the final authority and repeats its academic-year, term, class, programme, level and capacity validation.

The queue does not manufacture guardian accounts, invent emergency contacts, or bypass the existing guardian/person model.

## Remaining gate

The current admission transaction checks class capacity inside a transaction, but true concurrent-admission serialization has not been promoted into a committed PostgreSQL migration/constraint workflow. The PostgreSQL schema contract workflow must be used before treating high-concurrency admissions as production-verified.
