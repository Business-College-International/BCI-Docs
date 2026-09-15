# BCI Report Card Publication Design

## Purpose

A live academic report is a calculated view over assessment results. An official report card is a historical school record and must remain stable even if an assessment is corrected later.

## Required lifecycle

`DRAFT_VIEW -> READY_FOR_PUBLICATION -> PUBLISHED -> VOIDED`

Only authorized academic/office leadership may publish. Voiding must require a reason and an audit record; published snapshots are never updated in place.

## Snapshot principle

Publication creates an immutable snapshot containing:

- student identity and admission number
- academic year and term identity
- programme/level/class at publication time
- grading-policy version used
- subject rows
- assessment/result-derived percentages used for each subject
- subject grade, descriptor, points and pass/fail where configured
- overall percentage/grade/points where policy permits
- attendance summary included in the official report where the school policy requires it
- class position/rank only if a separately approved ranking policy is active
- publication timestamp and publishing actor
- deterministic snapshot/document hash

The snapshot must not depend on later reads of mutable `AssessmentResult`, `Enrolment`, `TeacherAssignment`, or grading-policy rows.

## Grading policy linkage

Every published report must identify the exact grading-policy version used. Changing the active grading policy does not retroactively change previously published reports.

## Corrections

If an approved correction is required after publication:

1. void the existing report snapshot with a reason;
2. retain the old snapshot and audit trail;
3. correct the underlying academic record through the normal audited workflow;
4. generate and publish a new snapshot with a new publication identity.

## Security

Guardians may view published reports only for linked wards with `canViewAcademic=true`. Teachers may view reports only for students/classes in their active assignment scope. Leadership/authorized office roles may manage publication.

## Schema gate

Do not add publication tables until the complete Prisma schema has passed the PostgreSQL database-contract workflow and the canonical initial migration has been established.
