# BCI Payment Intent Design Gate

## Why payment intents are not live yet

The current `Payment` model can represent a financial payment and the current `PaymentAllocation` model can allocate a successful payment to one or more invoices. It does not, however, identify the invoice being targeted while a provider payment is still pending.

That distinction matters because multiple pending requests can otherwise reserve the same outstanding invoice balance concurrently.

## Required schema capability before payment creation is enabled

The payment design must support all of the following atomically:

1. Identify the intended invoice for a pending payment.
2. Calculate outstanding balance from immutable invoice lines and successful allocations.
3. Include non-expired pending/processing reservations when deciding how much remains available to pay.
4. Make the idempotency key authoritative so retries return the original intent instead of creating a second payment.
5. Allow provider attempts and webhook events to reference the same payment without changing the financial amount after creation.
6. Only a verified provider success transition may create the final payment allocation and receipt.
7. Repeated provider callbacks must be harmless and auditable.

## Planned shape

The next schema revision should introduce an explicit invoice target/reservation relationship for payment intents. The exact field/model should be finalized together with the initial PostgreSQL migration so that the migration is generated once from the complete domain model rather than through repeated speculative schema edits.

## Current API boundary

The finance API currently supports fee schedule management, invoice issuance, and scoped invoice reading. It deliberately does **not** expose a payment-intent creation endpoint yet.

Moolre adapter code must not be introduced until the provider's live API contract is verified and the reservation model is in place.
