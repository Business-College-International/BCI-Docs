# Communication Delivery Operations Milestone

## Completed

- Announcement audience preview for ALL, GUARDIANS, TEACHERS, STAFF and USER audiences.
- Recipient, SMS-preference and push-preference counts before publication.
- Announcement delivery reports grouped by channel and delivery status.
- Staff communication operations summary for drafts, published announcements, pending deliveries and failed deliveries.
- Notification operations queue for pending/failed deliveries.
- Failed notification requeue with an audit record.
- Staff web delivery queue and announcement delivery dashboard.

## Safety boundaries

- No notification mutation endpoint allows arbitrary delivery-state fabrication.
- Requeue is limited to failed deliveries and changes only failed -> pending.
- No retry-attempt count was invented because the current schema has no retry-attempt field.
- Provider transmission remains outside the queue controls.
- Moolre SMS is not activated by this milestone.
- Actual SMS/push success must be reported only by a verified provider adapter/callback.

## Next provider gate

Before live SMS delivery, add a provider-neutral delivery-attempt/reservation contract, verify the Moolre SMS request/signature/response contract, implement provider idempotency, and exercise the PostgreSQL concurrency/retry tests in CI.
