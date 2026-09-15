# BCI Grading Policy Design

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

A DRAFT may be edited or deleted before publication. An ACTIVE policy may only be RETIRED by an authorized administrator. Retirement does not alter historical report cards.

## Report-card calculation

Report generation remains derived from assessment results. The calculation pipeline is:

1. validate student enrolment and term scope
2. calculate subject/term percentage from assessment results using the already-defined weighting policy
3. resolve the applicable ACTIVE grading policy for the student's academic year, level and programme
4. map the percentage into exactly one grade band
5. return grade code, descriptor, pass/fail and configured points where applicable

If no ACTIVE policy exists, the report must expose `GRADING_POLICY_REQUIRED` and must not invent a grade.

If the underlying assessment weighting is inconsistent, continue returning `MIXED_POLICY_REQUIRED` before grade resolution.

## Historical stability

Report views should identify the grading-policy version used. Published report-card records, when implemented, must store the policy-version reference so later policy changes cannot rewrite historical results.

## Security and audit

Policy creation, update, publish and retire operations require explicit grading/report permissions and create audit records. Teachers can read the applicable policy but cannot publish or change it unless separately authorized.

## Migration gate

Do not modify `prisma/schema.prisma` for grading until the first migration has been verified against PostgreSQL. The eventual schema change must be introduced as one deliberate revision and included in the migration review.
