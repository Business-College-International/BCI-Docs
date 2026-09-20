# BCI Financial Invariants

Money is a safety-critical subsystem. These rules apply to fees, payments, wallets, stationery sales, expenses, payroll and refunds.

## 1. Ledger authority

Never use a mutable `balance` field as the sole source of financial truth. Balances may be cached/derived, but every movement must have an immutable journal/ledger representation.

## 2. Monetary precision

Store monetary values using PostgreSQL `numeric`/Prisma `Decimal`, never binary floating-point. Currency is explicit (initially GHS) and amounts are positive where the transaction type requires it.

## 3. Double-entry direction

The eventual finance ledger should use explicit debit/credit entries or an equally rigorous balanced journal model. A financial transaction is not considered posted unless its journal lines balance.

## 4. Fees are not payments

A fee charge/invoice establishes what is owed. A payment records money received. Allocation links payments to one or more charges. Partial payments are supported. A successful payment must not silently rewrite the original charge amount.

## 5. Wallet is separate from school receivables

Student pocket-money is a distinct liability/ledger from school fees. A wallet top-up cannot reduce a school fee balance. A wallet withdrawal cannot be treated as an expense without the appropriate accounting entry and authorization.

## 6. Idempotency

Every externally initiated or retryable money mutation accepts an idempotency key. Repeating the same logical request must return the existing outcome rather than create a second charge, wallet top-up, refund request, wallet withdrawal/reversal, or payroll payment. Refund request creation uses the same replay authority as payment and wallet mutations.

## 7. Provider ambiguity

A timeout from Moolre or another provider means `status=unknown/pending`, not failure. Provider status is resolved through webhook and/or reconciliation polling. Clients never decide the final payment state.

## 8. Webhooks

Provider callbacks are deduplicated by provider event/reference and processed transactionally. A webhook may advance state only through allowed transitions.

## 9. No destructive edits

Posted financial records are immutable. Corrections use reversal/adjustment records referencing the original transaction. Wallet withdrawals and reversals create explicit signed ledger effects and corresponding balanced double-entry journal transactions. Amounts and references required for audit are never silently overwritten.

## 10. Receipts

A receipt is generated only after the backend has committed a successful payment outcome. Receipt numbering is unique and auditable. Reissued copies retain the original transaction identity.

## 11. Refunds

Refunds are separate transactions linked to the original payment. A refund cannot exceed the refundable amount. Refund approval is distinct from routine fee collection and is audited.

## 12. Physical cash wallet withdrawal

A wallet withdrawal reduces the student's wallet ledger only after the office workflow validates the student and authorized operator. The operation records amount, operator, student verification, timestamp and reference.

## 13. Payroll

Payroll is calculated for a defined pay period from an approved salary structure and controlled adjustments. A staff member cannot be marked paid until a successful payment/disbursement outcome is reconciled or a documented manual payment record is entered by an authorized user.

## 14. Expenses

Expenses have an explicit lifecycle: draft → submitted → approved/rejected → paid/recorded → reconciled where applicable. The same actor should not bypass required approval separation.

## 15. Inventory money

Stationery order totals are calculated server-side from authoritative product prices and accepted quantities. Stock is decremented through transactionally recorded stock movements, not a blind client-side quantity update.

## 16. Daily reconciliation

The finance area now includes an integrity report that checks payment/refund/wallet journal balance and missing postings in addition to allocation, receipt, wallet, invoice, and stationery consistency. The finance area must support reconciliation views for:
- provider collections vs internal payment records;
- provider disbursements vs payroll/refunds;
- wallet top-ups/withdrawals;
- stationery sales and stock movement;
- cash/manual transactions;
- outstanding fee balances.

Any unreconciled item remains visible rather than being silently ignored.
