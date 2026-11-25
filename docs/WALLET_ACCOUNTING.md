
# MercaLoop — Wallet and Inventory Entitlement Accounting

**Status:** Prepay / Option A confirmed. This document defines reseller wallet treatment, prepaid product entitlements, customer sale settlement and related accounting controls. Partner-store compensation remains a separate TBD operational/accounting decision.

**Updated:** 26 November 2026

## 1. Core business and accounting rule

MercaLoop uses an internal USDT-denominated reseller wallet.

A reseller prepays the wholesale entitlement amount from their available wallet balance when purchasing a quantity of product entitlements.

These entitlements give the reseller the commercial right and quantity to list and sell products through their MercaLoop shop.

They do not represent physical goods held by the reseller. Actual products remain at independent local partner stores owned by friends, family members or other approved partners.

When a customer places an order, MercaLoop administrators manually contact an appropriate local partner store and coordinate delivery to the customer's address.

After the relevant customer order quantity has been:

1. Delivered by the local partner.
2. Confirmed as received by the customer.
3. Independently verified as paid for by MercaLoop.

The system credits the reseller with the full eligible customer sale proceeds, less only separately configured and disclosed reseller fees or adjustments.

**Never deduct the original wholesale entitlement purchase cost again at sale.**

## 2. Example: 10 mobile phones

Assume no additional fees or reseller adjustments.

| Description | Amount |
|---|---:|
| Opening reseller wallet | 1,000 USDT |
| Wholesale entitlement price per unit | 80 USDT |
| Quantity purchased | 10 |
| Total prepaid entitlement cost | 800 USDT |
| Wallet after purchase | 200 USDT |
| Reseller retail price per unit | 100 USDT |
| Customer purchases 10 units | 1,000 USDT |
| Reseller sale proceeds credited after confirmation | +1,000 USDT |
| Final reseller wallet | 1,200 USDT |
| Reseller gross spread | 200 USDT |

Calculation:

- Prepaid entitlement purchase: 10 × 80 = 800 USDT.
- Customer sale proceeds: 10 × 100 = 1,000 USDT.
- Gross spread: 1,000 − 800 = 200 USDT.
- Final wallet: 200 + 1,000 = 1,200 USDT.

The original 800 USDT purchase debit has already happened.

Do not debit it again when the order is delivered or settled.

The 200 USDT represents the reseller's gross spread before separately configured and disclosed fees or adjustments.

This example does not establish the local partner store's compensation. Partner compensation must be handled separately under the approved partner agreement.

## 3. Required distinction between quantity records

MercaLoop must maintain two independent quantity domains.

### 3.1 Reseller prepaid entitlement quantity

This records the commercial quantity a reseller has purchased and may list or sell.

It has the following quantity states:

- Available: prepaid quantity that can be listed or reserved.
- Reserved: quantity temporarily locked for customer orders.
- Sold: quantity consumed after successful delivery, customer receipt confirmation and payment verification.

Multiple prepaid purchases of the same product variant may contribute to one aggregate reseller product entitlement.

Each purchase must retain its own historical inventory lot and wholesale unit cost.

### 3.2 Partner physical stock and availability

This records products physically held by independent partner stores.

It is maintained separately by:

- Partner store.
- Store location.
- Country/region.
- Product variant.
- Partner-reported availability.
- Date of the latest verification.

Partner stock information may become outdated.

A reseller purchasing 10 entitlement units does not create 10 physical units at a partner store.

A customer order reserves reseller entitlements first. The administrator then contacts an appropriate local partner store and records the partner's product availability and fulfilment confirmation.

Never represent reseller entitlements as physical stock owned or held by the reseller.

## 4. Internal reseller wallet

The MercaLoop wallet is an internal USDT-denominated accounting balance.

It is not itself a blockchain wallet and does not imply that MercaLoop controls users' private keys.

The wallet must support:

- Available balance.
- Pending deposit requests.
- Immutable transaction history.
- Inventory entitlement purchases.
- Customer sale proceeds.
- Authorized adjustments.
- Separate, approved fees and refunds.

### 4.1 Available balance

Available balance is the amount the reseller can spend on eligible entitlement purchases.

Only posted and eligible ledger credits contribute to spendable balance.

Pending deposits must not increase the available balance.

### 4.2 Pending deposits

Pending deposits are requests awaiting admin verification.

They must not be treated as spendable funds.

A deposit becomes available only after authorized approval and successful creation of the corresponding posted ledger transaction.

### 4.3 Balance calculation

The financial ledger is the source of truth.

Available and other balance displays may be derived from posted ledger entries or maintained as transactionally updated and regularly reconciled cached balances.

A frontend-submitted balance must never be accepted as authority for a financial operation.

The backend calculates every wallet balance and validates available funds before approving a purchase.

## 5. Expected transaction sequence

### A. Reseller deposit

The reseller may fund their wallet using either of the supported initial workflows.

#### Crypto deposit

- Reseller selects an admin-enabled deposit method.
- The platform displays the configured asset, blockchain network, receiving address, QR code, minimum/maximum limits and instructions.
- Initial examples include USDT on Tron/TRC20, BNB Smart Chain/BEP20 and Ethereum/ERC20.
- Reseller submits the deposit amount and TXID.
- Deposit request remains pending.
- An authorized admin independently verifies the transaction on the relevant blockchain/network or through the approved provider.
- Verification must confirm asset, network, receiving address, actual amount, transaction status and required confirmations.
- A screenshot or unverified TXID is not proof that funds were received.
- Approval creates exactly one posted wallet ledger credit.

#### Bank-transfer deposit through support

- Reseller contacts customer support.
- Support provides appropriate approved bank/payment instructions for the reseller's country.
- Reseller transfers fiat and submits the requested reference/details.
- Authorized admin independently verifies that funds actually arrived.
- Admin records the received fiat amount, currency, exchange rate, fees and approved USDT credit.
- The system records the reseller UID, verifier, timestamps and resulting ledger transaction.
- Only authorized approval can create the wallet credit.

**TBD before production:**

- Supported crypto networks and assets.
- Crypto verification provider and confirmation policy.
- Supported bank-transfer countries and partners.
- Exchange-rate source and rate-lock policy.
- Bank conversion spread and fees.
- Deposit limits, refunds and compliance requirements.

### B. Prepaid entitlement purchase

The reseller selects a product variant and desired quantity from the central MercaLoop catalogue.

The backend must:

1. Validate that the reseller account is eligible to purchase.
2. Validate product and variant availability in the central catalogue.
3. Read the current approved wholesale entitlement price.
4. Calculate the total using decimal arithmetic.
5. Verify sufficient available USDT wallet balance.
6. Prevent duplicate purchase processing using an idempotency key.
7. Create the inventory purchase and purchase-item records.
8. Create or retrieve the reseller's product entitlement.
9. Create a new historical inventory cost lot.
10. Create an immutable purchase entitlement movement.
11. Post the corresponding balanced wallet ledger transaction.
12. Update the operational inventory quantity counters.

All related operations must happen inside one database transaction.

If any required operation fails, the entire transaction must roll back.

The transaction must:

- Debit the reseller wallet exactly once.
- Create the purchase record and its item.
- Preserve the wholesale unit price snapshot.
- Create the corresponding inventory lot.
- Increase available reseller entitlement quantity.
- Insert the immutable movement record.

The same purchase retry must never create a second wallet debit or duplicate entitlement quantity.

**Important:** The purchase creates a prepaid selling entitlement. It does not physically purchase, allocate or reserve goods at a partner store.

### C. Customer order and entitlement reservation

A customer visits a reseller storefront and places an order.

The backend must:

1. Validate the shop and listing status.
2. Validate that the listing belongs to the shop owner.
3. Validate the reseller's available entitlement quantity.
4. Validate destination serviceability.
5. Snapshot the product and retail price.
6. Create the order and order items.
7. Reserve the required reseller entitlement quantity from one or more inventory lots.
8. Insert corresponding immutable entitlement movement records.

All order creation and reservation operations must happen transactionally.

Reservation transfers quantity from available to reserved.

Reservation does not:

- Deduct the reseller's wholesale cost again.
- Create another wallet purchase debit.
- Automatically reserve partner-store physical stock.
- Guarantee that a particular local partner can fulfil the order.

Use row locking or equivalent concurrency control to prevent simultaneous orders from reserving the same entitlement quantity.

### D. Manual partner-store coordination

After an order is created:

- The order appears in the reseller dashboard.
- The order appears in the admin dashboard.
- Admin reviews the delivery destination and ordered products.
- Admin identifies a suitable local partner store.
- Admin contacts the store owner manually.
- Admin shares the required delivery address, product details, variants and quantities using an approved operational process.
- Partner confirms whether the products can be delivered.
- Admin records the partner store, location, assigned quantity, confirmation method and operational notes.
- If necessary, admin may cancel an unsuitable assignment and contact another partner.

Multiple partner stores may be used for split fulfilment.

The total active assigned quantity must never exceed the outstanding quantity of the order item.

If no suitable partner can fulfil the order, the order follows its explicit cancellation/refund process and reseller entitlement reservation is released or otherwise resolved according to the order state.

No automatic supplier API, purchase order or courier integration is assumed in the MVP.

### E. Delivery and customer payment verification

The local partner delivers the products directly to the customer's delivery address.

For each eligible order quantity:

- Partner delivery details and evidence/reference are recorded.
- Customer confirms receipt through the platform.
- MercaLoop independently verifies actual receipt of the customer's payment.
- Admin records or approves the relevant delivery and payment verification according to permissions.

Delivery and payment are separate conditions.

A delivered but unpaid/unverified order is not eligible for reseller settlement.

A paid but undelivered order is also not eligible under the agreed settlement model.

For partial deliveries, evaluate eligibility separately for each delivered quantity.

### F. Option A settlement

When all required conditions are satisfied for an eligible quantity:

1. Identify the delivered fulfilment item and quantity.
2. Confirm customer receipt.
3. Verify that sufficient customer payment has actually been received.
4. Confirm the relevant payment allocation.
5. Calculate sale proceeds from the saved retail-price snapshot.
6. Deduct only separately configured and disclosed reseller fees or adjustments.
7. Create the settlement record and settlement-item record.
8. Consume the corresponding reserved reseller entitlement quantity.
9. Create the corresponding immutable inventory movement.
10. Post one wallet credit for the full eligible sale proceeds, less authorized fees/adjustments.
11. Mark the eligible entitlement quantity as sold.
12. Record the settlement actor, source events and timestamps.

All settlement operations must occur in one database transaction.

The settlement command must be idempotent. Retrying an approved settlement must not create another wallet credit or consume the same entitlement quantity again.

**Never deduct the original wholesale purchase cost again.**

## 6. Partial orders and settlement

Example:

A customer orders 10 mobile phones, but only 6 are delivered and paid for.

The system must:

- Settle only the 6 eligible delivered-and-paid units.
- Calculate proceeds from the saved retail unit-price snapshots.
- Credit only the eligible proceeds, less approved fees/adjustments.
- Consume the corresponding 6 reserved entitlement units.
- Preserve the remaining 4 units as pending, reassigned or released according to the order outcome.
- Keep the remaining units from being settled prematurely.
- Prevent the same delivered and paid units from being credited twice.

Settlement eligibility must be based on the intersection of:

1. Delivered quantity.
2. Customer-confirmed receipt quantity.
3. Quantity supported by verified and allocated payment.

The system must never settle more than the eligible quantity or amount.

Where one payment covers multiple order items, payment allocations must identify how much verified payment is attributable to each eligible item/quantity.

Any policy for handling a partially paid unit, multiple currencies, taxes or shipping charges must be explicitly defined before the corresponding production functionality is activated.

## 7. Cancellations, returns and refunds

### 7.1 Order cancellation before fulfilment

- Release reserved reseller entitlement quantity when the units remain eligible for resale.
- Record a release movement.
- Update the corresponding inventory lot counters in the same database transaction.
- Do not automatically refund the original wholesale entitlement purchase debit.
- The prepaid entitlement may return to available status.

### 7.2 Prepaid entitlement cancellation or invalidation

If an entitlement purchase itself is cancelled or invalidated:

- Require authorized administrative action.
- Create a separate compensating wallet ledger transaction where a refund is approved.
- Record a corresponding entitlement adjustment.
- Preserve the original purchase and ledger entries.
- Never edit or delete original posted financial entries.

### 7.3 Customer refunds after settlement

A refund after reseller settlement must create a separate approved compensating financial transaction.

The following policies remain TBD before production:

- Which party bears the refund loss.
- Whether an unsettled reseller amount can be withheld.
- Whether a reseller wallet can temporarily become restricted or negative.
- How refunds are handled when the reseller no longer has sufficient available balance.
- How partially returned products affect reseller entitlement quantities.

Never silently reverse a completed sale settlement or modify the original posted ledger records.

### 7.4 Disputes and failed deliveries

- Disputes require explicit status and audit records.
- Failed deliveries must not qualify for settlement.
- Returned products require explicit return/reinstatement decisions.
- A delivery marked completed must not automatically override an unresolved customer dispute.

## 8. Double-entry ledger requirements

Use a double-entry accounting model with balanced posted journal transactions.

All monetary values must use PostgreSQL `NUMERIC` or equivalent decimal arithmetic.

Do not use JavaScript floating-point values as the authority for money calculations. Use appropriate decimal arithmetic in the backend and validate amounts again in PostgreSQL transactions.

### 8.1 Financial event categories

Keep the following financial events separately identifiable:

- Verified reseller deposit.
- Approved bank-transfer credit.
- Prepaid entitlement purchase.
- Customer payment received.
- Reseller sale settlement.
- Approved reseller fee.
- Wallet adjustment.
- Customer refund.
- Entitlement purchase refund.
- Other authorized compensating transaction.

### 8.2 Ledger transaction requirements

Every financial command must have:

- Unique ledger transaction ID.
- Event type.
- Source type and source ID.
- Unique idempotency key.
- Amount and currency where applicable.
- Actor.
- Status.
- Timestamp.
- Balanced debit/credit entries.

Posted ledger entries must be append-only.

Corrections must use new compensating entries, not editing or deleting historical transactions.

### 8.3 Wallet balance rules

- Available balance is based on posted eligible wallet ledger credits and debits.
- Pending deposit requests do not increase available balance.
- A purchase cannot spend pending deposit funds.
- An inventory reservation does not change wallet balance.
- A completed eligible sale creates the full reseller proceeds credit.
- Separately configured fees or adjustments must appear as their own identifiable entries.
- Wallet operations must be transactionally protected against concurrent spending and duplicate processing.

### 8.4 Accounting mapping

The exact debit/credit financial-account mapping must be reviewed before implementation.

Potential accounting accounts include:

- Reseller wallet liability.
- Platform deposit/crypto clearing.
- Customer payment clearing.
- Order proceeds clearing.
- Entitlement purchase clearing/revenue.
- Platform fee revenue.
- Approved refund/adjustment accounts.

The accounting team must approve the account mapping and revenue recognition treatment before real-money transactions are enabled.

## 9. Currency and conversion rules

The initial internal reseller wallet denomination is USDT.

- Wallet balances are maintained in USDT.
- All entitlement purchase costs and reseller settlement credits must have an explicit currency.
- Customer payment currency and asset must be stored separately from the reseller wallet currency.
- Payment verification must confirm actual receipt of the relevant asset/currency.
- If a customer pays using a different currency or crypto asset, the approved conversion process must determine the USDT-equivalent settlement amount.
- Never assume that USD, USDT and other crypto assets are interchangeable at a one-to-one rate.
- Store original received amount, asset/currency, approved conversion rate, conversion fees and resulting USDT amount where conversion is used.

**TBD before production:**

- Supported customer payment assets and networks.
- Conversion provider/source.
- Rate-lock timing.
- Spread and conversion fees.
- Handling of exchange-rate changes between order and settlement.

Do not credit a USDT wallet using an unverified or arbitrarily calculated currency conversion.

## 10. Partner-store compensation is separate and unresolved

The reseller's wholesale entitlement purchase price and partner-store compensation are not automatically the same amount.

MercaLoop must decide whether a local partner is paid:

- A fixed per-unit amount.
- An agreed invoice amount.
- The wholesale amount.
- A delivery fee.
- Another approved commercial arrangement.

The business must also define when partner compensation becomes due and who approves the payment.

Until those terms are approved:

- Do not automatically create partner payouts from the reseller wholesale price.
- Do not assume the partner receives 80 USDT merely because the reseller purchased an entitlement for 80 USDT.
- Do not deduct partner compensation from reseller wallet settlement unless it is a separately approved and disclosed reseller fee/adjustment.
- Keep partner operational assignment/delivery records separate from reseller wallet entries.
- Add partner payable and payment records only as a separately reviewed accounting module.

Partner compensation must not be mixed with reseller wallet credit calculations.

## 11. Withdrawals

A reseller withdrawal policy has not yet been approved.

Before implementing withdrawals, decide:

- Whether withdrawals are supported at all.
- Whether withdrawal is permitted to an external crypto wallet, bank account or both.
- Supported assets and blockchain networks.
- Minimum and maximum withdrawal limits.
- Fees and processing times.
- Identity verification requirements.
- Manual admin approval or payment-provider processing.
- Reversal, failed withdrawal and dispute handling.

Until these terms are approved, do not enable production withdrawals.

Internal reseller wallet accounting and prepaid entitlement purchases can be developed without enabling withdrawals.

## 12. Example wallet activity

For the example involving 10 mobile phones and no fees:

| Event | Reseller wallet effect |
|---|---:|
| Verified deposit | +1,000 USDT |
| Purchase 10 entitlements at 80 USDT each | -800 USDT |
| Reserve 10 entitlements for customer order | 0 USDT |
| Partner delivers 10 units; customer confirms receipt; payment verified | +1,000 USDT |
| Final wallet balance | 1,200 USDT |

The 200 USDT gross spread is customer sale proceeds less the wholesale entitlement cost already prepaid.

The original 800 USDT must never be deducted again at sale.

Any separately configured and disclosed reseller fees or adjustments must appear as individual identifiable wallet transactions.

## 13. Reconciliation and operational controls

Before enabling production money flows, MercaLoop must implement reconciliation procedures.

### Wallet reconciliation

- Compare wallet cached balances with posted ledger entries.
- Detect duplicate or missing ledger transactions.
- Identify wallet entries without valid source events.
- Ensure posted journal transactions balance for each currency.

### Entitlement reconciliation

- Compare each inventory lot's available, reserved and sold counters with its immutable movement history.
- Detect lot quantities updated without corresponding movements.
- Detect duplicate purchase, reservation, release or consumption operations.
- Confirm entitlement totals reconcile with their individual inventory lots.

### Order and settlement reconciliation

- Compare delivered fulfilment quantities with customer receipt confirmations.
- Compare verified customer payments with payment allocations.
- Compare eligible delivered-and-paid quantities with settlement items.
- Confirm each settlement has its corresponding wallet ledger credit.
- Detect repeated settlements and over-credited quantities.

### Administrative controls

- Restrict financial approvals to authorized roles.
- Log deposit approvals, inventory purchases, payment verification, delivery verification, settlement approvals, adjustments and refunds.
- Keep sensitive financial credentials out of audit logs.
- Provide reports for unresolved deposits, unverified payments, pending deliveries, failed fulfilments, unsettled orders and reconciliation discrepancies.

## 14. Development and production restrictions

During local development:

- Use test data only.
- Do not process real customer deposits.
- Do not create real customer payment obligations.
- Do not enable real reseller withdrawals.
- Do not treat test transactions as actual financial balances.
- Do not deploy production financial operations until database integrity, permissions, transaction handling and reconciliation tests pass.

Before live-money launch, MercaLoop must also complete applicable legal, licensing, KYC/AML, sanctions, privacy, consumer protection, tax and operational reviews for its actual operating jurisdictions.

This document defines the agreed wallet and reseller entitlement accounting model. Partner-store compensation, withdrawals, customer payment conversion and certain refund policies remain unresolved and must not be implemented through assumptions.
