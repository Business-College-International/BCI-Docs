# Payment webhook processing milestone

## Completed

- Added a provider-neutral normalized payment webhook contract.
- Added exact amount/currency validation helpers.
- Added a webhook processor that can update an existing payment only after a provider adapter has verified and normalized an event.
- Payment lookup supports provider reference or client reference.
- Provider attempt status is synchronized with normalized payment status.
- Terminal payment states cannot regress to a different status through a later event.
- Missing payments, amount mismatches and currency mismatches are recorded on the durable webhook event instead of being silently discarded.
- Moolre verification, normalization and initiation remain explicitly disabled until the live provider contract and credentials are independently verified.

## Deliberate boundaries

The processor does not create allocations, receipts, reservations or journal entries. Successful provider callbacks therefore cannot create financial records that have not passed the reservation/allocation gate.

The durable reservation schema is still migration-gated. A payment may not be initiated merely because the preflight layer says an amount could be allocated.
