# BCI Student Welfare, Records & Notifications Milestone

## Completed

- Authenticated notification center APIs for the current user.
- Notification delivery filtering by supported lifecycle status.
- Single-notification mark-read and mark-all-read operations.
- Recipient isolation enforced by `recipientUserId`.
- Web staff notification center with refresh and read controls.
- Flutter notification center reachable from the guardian home.
- Student document metadata APIs for list/add/remove.
- Student document reads are scoped to leadership/office roles, linked guardians with `canViewAcademic`, or teachers assigned to the student's active class/term.
- Document mutations are restricted to authorized school-management roles and audited.
- Actual binary file storage/upload is intentionally separate from metadata persistence.

## Deliberate schema gate

The current `EmergencyContact` entity is linked to `Person`, not `Student`. There is no safe student-specific emergency-contact relation in the existing schema. Do not attach student emergency contacts by overloading guardian/person fields. A future schema change should introduce an explicit student emergency-contact relation with primary-contact semantics and audit history.

## Provider boundary

Notifications currently represent durable in-app delivery state. SMS, push and other external channels remain adapters behind the notification/provider layer and must not be represented as successfully sent until provider acknowledgement is received.
