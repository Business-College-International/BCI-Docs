# BCI Student Lifecycle — Terminal States Milestone

## Completed

- Preserved the existing transactional withdrawal flow.
- Added lifecycle history for authorized school management users.
- Added graduation control: only active SHS3 students with a closed final term can graduate.
- Graduation completes the active enrolment and changes the student status to `GRADUATED` atomically.
- Added transfer control requiring a destination school and reason.
- Transfer completes the active enrolment with `TRANSFERRED` status and changes the student status to `TRANSFERRED` atomically.
- Every terminal transition is audited.
- Added staff web lifecycle-history workspace.

## Existing progression relationship

The existing lifecycle service already supports cross-term progression. Terminal transitions remain separate so progression cannot accidentally reactivate a graduated, transferred, or withdrawn student.

## Deliberate boundaries

- Student status is not changed by merely completing a normal term progression.
- Graduation is not available before the final term closes.
- No automatic guardian/account deletion is performed by withdrawal, transfer, or graduation.
- Re-enrolment of terminal-state students remains a dedicated policy decision and is not silently enabled.
