# Payment Provider Infrastructure Milestone

## Completed

- Provider-neutral `PaymentProviderPort` defined for payment initiation and webhook verification.
- Durable webhook ingestion uses the existing `ProviderWebhookEvent` model.
- Provider/event identity is idempotent through the existing unique constraint.
- Verified webhook events are persisted before downstream financial handling.
- Provider mismatch is rejected before persistence.
- Unverified webhook processing is explicitly rejected.
- A provider-neutral Moolre adapter now implements initiation, OTP continuation and webhook verification behind configuration gates; repository defaults remain MOCK and no live network call is made unless LIVE mode, complete credentials, webhook secret and explicit live-money confirmation are all present.
- Payment provider infrastructure is registered in the backend application graph.
- Regression tests cover duplicate events, provider mismatch, and signature-verification enforcement.

## Activation gates

The live Moolre adapter must not be enabled until all of the following are independently verified:

1. Provider API endpoint, authentication, request and response schemas are confirmed.
2. Webhook signature/canonicalization rules are confirmed from the provider documentation or controlled integration environment.
3. Durable fee-payment reservation and invoice-row locking exist so concurrent payment attempts cannot over-commit an invoice.
4. Idempotency mapping between BCI client reference and provider reference is tested end-to-end.
5. Success/failure/refund callback state transitions are defined against the durable Payment and PaymentProviderAttempt models.
6. Financial allocation and receipt issuance run only after a provider-confirmed success event.
7. Reconciliation and replay handling are tested against duplicate and out-of-order events.
8. A real PostgreSQL migration/contract verification run passes before production deployment.

## Deliberate non-goals

- No repository configuration enables live Moolre by default.
- Live-money activation requires `MOOLRE_PROVIDER=LIVE`, complete provider credentials, a webhook secret, and `MOOLRE_LIVE_CONFIRMED=true`.
- No production secrets or provider credentials are stored in the repository.
- Fee, wallet and stationery business flows remain dependent on verified provider settlement; MOCK mode is intended for CI/local contract testing.
