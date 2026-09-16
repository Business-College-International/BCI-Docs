# Report Publication Readiness Milestone

## Completed

- Added class/term report-publication readiness analysis.
- Requires the term to be CLOSED before a student can be ready.
- Detects missing assessment results for active students.
- Detects mixed weighted/unweighted assessment policy.
- Explicitly blocks publication while persistent grading policy is not configured.
- Enforces teacher access through actual teacher assignment to the class and term.
- Added backend regression coverage for term-state and missing-result blockers.
- Added an office web workspace for class/term readiness and student-level exceptions.

## Deliberate boundary

This capability is a read-only readiness gate. It does not create an official report, change grades, publish results, or expose a mutable report as authoritative.

## Remaining persistence gate

The Prisma schema still lacks persistent grading-policy and report-publication models. The next schema milestone must introduce those through the controlled PostgreSQL schema-contract workflow rather than hand-written unverified SQL.

## Future publication flow

DRAFT VIEW -> READY_FOR_PUBLICATION -> immutable snapshot -> approval -> PUBLISHED -> guardian delivery

Corrections after publication must create a new version/revision and never mutate the original published snapshot.
