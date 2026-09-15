# Family Portal and Admissions Milestone

## Completed

- Guardian profile API is exposed through `/guardians/me/profile` with audited updates.
- Guardian SMS/push preferences are persisted on the existing `Guardian` model.
- Web portal authentication now accepts guardians and routes them to a dedicated guardian workspace.
- Web guardian workspace shows linked wards and relationship-level academic, fee and wallet permissions.
- Web guardian workspace supports profile/contact edits and notification preference changes.
- Web guardian workspace includes notification history and read-state controls.
- Flutter guardian app exposes the same profile/preferences workflow.
- Flutter guardian home now links directly to guardian profile settings.
- Public application tracking now returns a safe status timeline consisting only of submission/admission/rejection events and timestamps.
- Public website renders the application timeline.
- Backend tests cover public tracking data minimization and timeline ordering.

## Security boundary

- Guardian tracking is public only by tracking code and does not expose guardian contact information, internal review notes, or decision reasons.
- Guardian portal data is authenticated and relationship-scoped by server endpoints.
- Ward permissions remain object-level controls; the client never grants access locally.
- Guardian phone/email remain read-only in profile editing and are managed by school identity workflows.

## Deferred deliberately

- Self-service password change/reset is still an authentication-workflow gate and is not implemented by the profile screen.
- SMS/push transport remains provider-neutral; preference storage exists, but live external provider delivery is separate.
- Guardian-to-ward unlinking remains a school-management operation.
- Application decisions do not expose internal rejection reasons through the public tracker.
