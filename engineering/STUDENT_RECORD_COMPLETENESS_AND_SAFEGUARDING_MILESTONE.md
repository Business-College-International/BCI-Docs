# Student Record Completeness & Safeguarding Milestone

## Completed

BCI now exposes a read-only student record completeness audit through `students.read` staff scope.

The audit checks the authoritative current schema for:

- student name
- date of birth
- admission number
- passport photo
- previous school
- hometown
- region
- primary guardian relationship
- guardian phone availability
- active enrolment
- stored student documents
- guardian access permissions

It returns a completeness score, current placement, linked guardians, stored documents, and explicit findings grouped as `BLOCKING`, `WARNING`, or `INFO`.

## Safeguarding boundary

The current Prisma schema contains `EmergencyContact`, but the model is attached to `Person` and `Student` has no `Person` relation. The completeness service therefore reports `EMERGENCY_CONTACT_SCHEMA_GAP` rather than falsely claiming that emergency-contact coverage can be verified.

## Next migration requirement

The eventual student emergency-contact model should link contacts directly to `Student` or introduce a deliberate Student-to-Person identity relation. The migration must define:

1. one or more contacts per student;
2. primary-contact uniqueness semantics;
3. phone/contact validation;
4. relationship classification;
5. staff read/write permissions;
6. guardian visibility rules;
7. audit history for sensitive changes.

That migration should be generated and verified through the PostgreSQL schema-contract workflow before becoming canonical.
