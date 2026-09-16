# Academic Report Draft Milestone

## Delivered
- Extended the existing academic-report term summary with active placement context.
- Added term attendance metrics from real attendance records.
- Kept assessment calculation modes explicit: weighted, unweighted average, mixed-policy-required, or no results.
- Added subject-level assessment summaries.
- Added explicit grading state: no configurable grading-band policy is applied yet.
- Added explicit publication state: `DRAFT_VIEW`, not a persisted official report.
- Added backend regression coverage for placement and attendance calculations.
- Added a staff portal read-only academic report draft workspace.

## Scope/security
- Existing `assessments.read` authorization remains the endpoint boundary.
- Guardians remain subject to `canViewAcademic`.
- Teachers remain assignment/class/term scoped.
- Leadership/office roles retain privileged access.

## Deliberate non-goals
- No official report-card publication.
- No persisted report snapshot migration in this milestone.
- No grading-policy persistence or hard-coded BCI grade bands.
- No report PDF generation yet.

## Next gates
1. Persist configurable grading-policy versions through verified PostgreSQL migration.
2. Generate immutable report snapshots from this draft payload.
3. Add publication/reviewer controls and correction/versioning persistence.
4. Add official guardian report-card access only after publication state exists.
