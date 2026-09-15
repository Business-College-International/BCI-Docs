# School Configuration Milestone

Implemented subject and fee-schedule configuration surfaces using the existing Prisma schema.

## Subject catalogue

- List subjects with level/programme/elective/active filters.
- Create subjects for academic managers.
- Update name, elective flag, programme and active state.
- Prevent level changes once teaching assignments exist.
- Keep programme-bound subjects explicitly elective.
- Audit configuration mutations.

## Fee schedules

- List schedules by term for authorized finance/academic users.
- Create fee items for finance managers.
- Validate term existence and non-negative amounts.
- Prevent duplicate term/level/programme/item combinations through the existing uniqueness constraint.
- Audit schedule creation.

## Web portal

`ConfigurationWorkspace` is integrated into the staff portal and provides subject and fee setup for accounts with the corresponding server permissions.

## Safety boundaries

No schema migration was introduced. Grading policies, timetable entities, emergency-contact ownership and payment-reservation structures remain separate design/migration gates.
