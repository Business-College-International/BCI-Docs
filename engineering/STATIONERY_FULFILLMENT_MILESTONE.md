# BCI Stationery Fulfillment Milestone

## Completed

- Stationery catalogue remains provider-neutral and separate from school-fee invoices.
- Guardian ordering is permission-scoped to wards with `canPayFees`.
- Guardian orders are created as `DRAFT` records only.
- Staff operational order reads are permission-scoped to inventory management roles.
- Canonical operational lifecycle is `DRAFT -> PAID -> READY_FOR_COLLECTION -> COLLECTED`; draft orders may also become `CANCELLED`.
- Fulfillment requires a linked payment with status `SUCCEEDED`, purpose `STATIONERY`, matching student and exact order amount.
- Stock is decremented only during fulfillment.
- Stock decrement uses conditional updates inside a PostgreSQL serializable transaction.
- Every fulfillment stock movement is recorded as a negative `StockMovement` tied to the order.
- Repeat fulfillment is rejected because only `PAID` orders can enter fulfillment.
- Collection is separately recorded and audited.
- No stationery operation fabricates payment success.
- Guardian payment initiation creates the `STATIONERY` payment before the provider call and links that attempt to the draft order.
- Explicit provider rejection returns the order to an unpaid draft; ambiguous provider outcomes remain processing for reconciliation.
- Verified successful payment webhooks transition the linked order to `PAID` atomically with the payment settlement.

## Deliberate gates

- The payment provider must create/link a real stationery payment before an order can be fulfilled.
- Moolre remains disabled until its verified adapter, reservation path and reconciliation tests are promoted.
- No cash/payment allocation is added to the stationery module itself; finance remains the source of payment truth.
- Legacy lowercase or older order statuses are not silently rewritten by this milestone; any historical migration will be a separate verified database change.

## Next integration

The core `StationeryOrder -> provider payment -> verified webhook -> PAID` path is now implemented. The next layer is guardian payment history/receipt presentation and operational stock/reporting views, followed by the client-app flows.
