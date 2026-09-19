# BCI Grading Policy Design

Implementation status: **backend policy lifecycle and report-card publication are implemented; published snapshots record the exact grading-policy version and a deterministic snapshot hash.**

## Purpose

Official grades must be configurable school policy, not hard-coded application behavior. Assessment scores and derived percentages remain the source facts; grades are a versioned interpretation of those facts.

## Policy scope

A grading policy belongs to an academic scope. The first supported scope should be academic year + level, with optional programme override when BCI requires programme-specific rules.

A policy contains:

- immutable policy version identifier
- human-readable name
- academic year scope
- level scope
- optional programme scope
- status: DRAFT, ACTIVE, RETIRED
- created/published timestamps
- published-by actor
- ordered grade bands

## Grade band

Each grade band contains:

- label/code, e.g. A1, B2, C4 or another BCI-approved code
- inclusive lower percentage bound
- exclusive upper percentage bound, except the final band may include 100
- pass/fail flag
- official remark/descriptor
- optional points/value for aggregate calculations
- explicit ordering

The server must reject overlapping, gapped or reversed ranges. Every percentage from 0 through 100 must map to exactly one band before a policy can become ACTIVE.

## Publishing rules

Only an authorized academic administrator may publish a policy.

Publishing is transactional and auditable. Once a policy is ACTIVE for a scope, its grade bands are immutable. Changes create a new DRAFT version; they do not mutate the published version.

There must be at most one ACTIVE policy for an identical academic-year/level/programme scope.

A DRAFT may be edited before publication. The current API does not expose draft deletion. An ACTIVE policy may only be RETIRED by an authorized administrator. Retirement does not alter published report-card snapshots.

## Report-card calculation

Report generation remains derived from assessment results. The calculation pipeline is:

1. validate student enrolment and term scope
2. calculate subject/term percentage from assessment results using the already-defined weighting policy
3. resolve the applicable ACTIVE grading policy for the student's academic year, level and programme
4. map the percentage into exactly one grade band
5. return grade code, descriptor, pass/fail and configured points where applicable

If no ACTIVE policy exists, the report returns an unassigned grade with an explicit policy-required reason and must not invent a grade.

If the underlying assessment weighting is inconsistent, continue returning `MIXED_POLICY_REQUIRED` before grade resolution.

## Historical stability

Report views should identify the grading-policy version used. Published report-card records, when implemented, must store the policy-version reference so later policy changes cannot rewrite historical results.

## Security and audit

Policy creation, update, publish and retire operations require explicit grading/report permissions and create audit records. Teachers can read the applicable policy but cannot publish or change it unless separately authorized.

## Implemented backend contract

The backend now provides `GET /grading-policies`, `POST /grading-policies`, `PATCH /grading-policies/:id`, `POST /grading-policies/:id/publish`, and `POST /grading-policies/:id/retire`. Policy creation/update validates contiguous 0–100 coverage through the existing grading engine. Publication is serializable, advisory-lock protected, audited, and guarded by a database-level partial unique index for one ACTIVE policy per scope.

`AcademicReportsService` resolves the ACTIVE exact programme policy first and falls back to a generic policy. Report output includes the applied policy version and resolved grade when a valid policy exists. Report-card publication snapshots persist that exact output, including the policy version and snapshot hash.

The Prisma grading schema is introduced by `20260919070000_grading_policy_foundation`; the active-scope database guard is introduced by `20260919073000_grading_policy_active_scope_guard`. The publication model and its migration are subject to the repository's PostgreSQL and migration-review workflows; the final publication PR was merged only after the backend CI test suite passed.