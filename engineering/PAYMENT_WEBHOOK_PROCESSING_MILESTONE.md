# Payment webhook processing milestone

## Completed

- Added a provider-neutral normalized payment webhook contract.
- Added exact amount/currency validation helpers.
- Added a webhook processor that can update an existing payment only after a provider adapter has verified and normalized an event.
- Payment lookup supports provider reference or client reference.
- Provider attempt status is synchronized with normalized payment status.
- Terminal payment states cannot regress to a different status through a later event.
- Missing payments, amount mismatches and currency mismatches are recorded on the durable webhook event instead of being silently discarded.
- Moolre verification, normalization, initiation and OTP continuation are implemented behind an explicit live-money configuration gate; default CI/local configuration remains MOCK.

## Settlement boundaries

The processor updates durable payment state only after provider verification. Verified success can issue a receipt and settle purpose-specific downstream effects, including fee allocation, wallet top-up ledger credit and stationery-order `PAID` transition. These effects remain inside the same database transaction as payment settlement.

Duplicate webhook events and repeated wallet/stationery settlement are idempotent against durable payment/domain identifiers.
