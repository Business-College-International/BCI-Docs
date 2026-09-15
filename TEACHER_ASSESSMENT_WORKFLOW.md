# Teacher Assessment Workflow

## Purpose

Provide teachers with a scoped web workflow for creating an assessment and entering results without allowing the client to bypass class, subject, term, enrolment, or score rules.

## Backend contract

`GET /api/v1/assessments/roster?classId=&termId=&subjectId=`

The roster endpoint validates:

- term exists
- class exists
- subject exists
- class and term belong to the same academic year
- subject level matches class level
- teacher is assigned to the exact class/subject/term, unless the actor is a privileged academic role

The response contains only active students enrolled in the requested class and term.

`POST /api/v1/assessments`

Creates an assessment after verifying that the term is open and the actor is assigned to the subject/term. `maxScore` and optional `weight` are validated.

`POST /api/v1/assessments/:assessmentId/results`

The server re-checks assignment scope and student eligibility. Duplicate student IDs are rejected and every score must be between zero and the assessment maximum. Results are upserted transactionally and the assessment change is audited.

## Web workflow

1. Staff session is authenticated through `/auth/me`.
2. The staff workspace loads authoritative teaching assignments from `/staff/me`.
3. The assessment workspace is rendered only when the actor has `assessments.manage`.
4. The teacher selects one assigned class/subject/term combination.
5. The portal creates the assessment using that assignment's term and subject IDs.
6. The portal loads the scoped assessment roster from the backend.
7. The teacher enters scores and submits them as one results request.
8. The backend remains authoritative for all eligibility and score checks.

## Security boundary

No frontend filtering is treated as an authorization mechanism. Every sensitive action re-validates the actor's assignment and student scope on the backend.

## Testing

The backend assessment service tests cover:

- teacher denied without subject assignment
- result rejected for a student outside the teacher's assigned class
- score rejected above maximum
- assigned teacher receives the active assessment roster
- unassigned teacher is denied the roster
- cross-academic-year class/term mismatch is rejected

## Deliberate non-goals

This workflow does not assign official grades. Grading bands, publication rules, report-card locking, approval, and transcript generation require a configurable academic policy and remain separate from raw assessment entry.
