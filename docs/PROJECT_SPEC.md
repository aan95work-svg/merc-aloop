# MercaLoop — Product and Functional Specification

**Status:** Updated baseline requirements after confirming the prepay model and local partner-store fulfilment model. Items marked **TBD** require a decision before the related production feature is enabled.

**Updated:** 26 November 2026

## 1. Product summary

MercaLoop is a centrally managed wholesale and reseller marketplace supported by a network of independent local partner stores in different countries. The platform operator manages the catalogue, wholesale prices, reseller prepaid selling entitlements, wallet approvals, orders, partner coordination and settlement records.

Approved resellers use an internal USDT-denominated wallet to prepay for quantities of products at wholesale prices. The purchase creates a **reseller inventory entitlement** in MercaLoop: a recorded quantity the reseller is allowed to list and sell through their MercaLoop shop. It does **not** mean that the physical goods are shipped to, stored by, or held by the reseller.

Physical products remain at independent partner stores operated by friends, family members or other approved local contacts. When a customer orders from a reseller's shop, MercaLoop administrators manually contact a suitable partner store in the customer's delivery country, confirm that the store can fulfil the item, and share the delivery details through the approved operational process. The partner store delivers directly to the customer.

The reseller's country does not determine the delivery source. For example, a reseller based in India may list a mobile phone in their MercaLoop shop and a customer in the UK may order it. MercaLoop operations then coordinate with an approved UK partner store that can supply and deliver the phone to the customer's UK address.

MercaLoop is **not** an automated supplier/dropshipping integration in the initial operating model. Do not implement automatic supplier API forwarding, automatic partner purchase orders, or automatic courier integrations as assumed default behaviour. Partner contacts and fulfilment coordination are admin-operated.

## 2. Roles and permissions

- **Super administrator:** Full platform configuration and oversight.
- **Admin / operations staff:** Permissions assigned by the super administrator. Depending on permissions, staff may manage products, partner stores, partner availability, deposits, orders, fulfilment assignments, payment confirmations, chats and support.
- **Reseller / shop owner:** Manages their shop, retail prices, eligible prepaid inventory entitlements, orders and customer conversations; can view their own wallet and ledger.
- **Customer:** Browses a reseller's shop, places orders, provides delivery details and communicates with that reseller.
- **Local partner store / partner contact:** An independent business/contact that physically holds and delivers products. In the initial MVP, partners do not need platform login accounts or a self-service portal. Their store, location, contact details and product availability are maintained by authorized MercaLoop staff in an admin-only area.

Every privileged action must be enforced by the backend. Hiding a button in the frontend is not authorization. Partner contact information, customer addresses and internal fulfilment notes must be restricted to authorized operations staff.

## 3. Account registration and identity

- Users register and authenticate through the platform authentication system.
- Each user receives a unique, immutable MercaLoop UID for support and manual deposit-credit workflows.
- A user profile stores contact and country information needed for support and orders.
- A user may become a reseller only after the platform's configured approval process, if approval is required.
- User roles, shop ownership and record access must be enforced server-side.

## 4. Central catalogue, reseller entitlements and partner-store stock

These are two separate concepts and must never be represented as one inventory quantity.

### 4.1 Central catalogue

- Platform administrators create and manage categories, products, variants, images, descriptions, wholesale prices and catalogue availability.
- The catalogue is the central product list resellers can browse.
- Products may be associated with one or more partner stores/locations that report they can supply them.
- Product coverage and delivery availability are country/location-specific. A product being listed in the central catalogue does not guarantee that it can be delivered to every country.

### 4.2 Reseller prepaid inventory entitlement (not physical possession)

- A reseller browses the central catalogue and prepays for a chosen quantity at the current wholesale price using their available USDT wallet balance.
- A successful purchase atomically debits the reseller wallet and creates a prepaid reseller inventory entitlement and purchase-lot record.
- The purchase record preserves the actual unit wholesale price, quantity, line total, product/variant and purchase time. Later catalogue price changes must not rewrite historical cost records.
- The purchased quantity becomes available for that reseller to list in their own shop. The reseller may set a retail price subject to platform rules.
- Entitlement quantities are tracked as available, reserved and sold (or an equivalent ledger-derived model). Orders reserve these quantities so the same entitlement cannot be sold twice.
- The entitlement is a platform accounting/commercial record. It is not proof that the reseller physically owns or possesses the product, and the UI must not claim otherwise.
- Multiple purchases of the same variant at different wholesale prices must retain separate cost lots. The allocation method used when units are sold (for example, FIFO) is **TBD** and must be selected before cost/profit reporting is enabled.

### 4.3 Physical stock at partner stores

- Physical goods remain at partner stores, not with the reseller.
- The admin area records partner stores, store locations, contacts, products/variants they can supply, reported quantities (where available), last-reported/last-verified timestamps and operational status.
- Partner-reported availability is an operational indication, not a real-time guarantee. There is no automatic inventory API in the MVP.
- After an order arrives, an administrator contacts the appropriate local partner and confirms that the partner can fulfil the requested product and quantity. The administrator records the confirmation and assignment.
- The physical partner stock record and reseller entitlement record are separate. A reseller may have a prepaid entitlement while a particular destination's partner store is temporarily out of stock.
- If the first partner cannot fulfil, operations may contact another suitable partner. If no partner can fulfil, the order follows an explicit cancellation/refund process and the reseller entitlement reservation is released or otherwise resolved according to the order state.

### 4.4 Reseller inventory listing

- A reseller may list only product variants for which they have eligible available prepaid entitlement.
- Shop listing quantity must be derived from or safely constrained by the reseller's available entitlement; it must not be an independent editable stock count that can exceed entitlement.
- A public listing must not expose wholesale purchase cost, partner contacts, partner stock notes, wallet ledger entries or admin-only information.
- At checkout, the destination country/region must be checked against platform serviceability and known partner coverage. The final partner stock confirmation remains a manual operations step after the order is placed; the checkout must not promise delivery based solely on a stale partner availability record.

## 5. Reseller storefronts

- Each reseller has an individual shop with a unique slug and a subdomain such as `shopname.mercaloop.com`.
- A shop displays only products and quantities made available through that reseller's eligible prepaid entitlements, with the reseller's retail prices.
- Wholesale purchase cost, other resellers' data, internal wallet ledger details and admin-only partner information must never appear in a public shop.
- The platform needs wildcard DNS and SSL for subdomains at production launch. Local development can use a shop route or test host mapping before the real domain is purchased.
- Resellers can manage supported shop details, product visibility and retail pricing within platform rules.

## 6. Wallet funding

The reseller wallet is an **internal USDT-denominated accounting balance**. It is not itself a blockchain wallet and must not imply that MercaLoop controls user private keys.

### 6.1 Crypto deposits

- Admin configures supported assets/networks, receiving addresses, QR codes, minimums, maximums, instructions and enabled/disabled status.
- Initial examples include USDT on Tron (TRC20), BNB Smart Chain (BEP20) and Ethereum (ERC20). Asset and network are separate fields.
- The reseller selects an available deposit method, sees its configured address and instructions, submits the amount and transaction hash (TXID), and receives a pending deposit request.
- An admin independently verifies the transaction on the relevant network/provider, including asset, network, receiving address, amount, confirmations and transaction status, before approval.
- A screenshot or user-submitted TXID alone is not proof of funds received. Do not automatically credit a wallet just because a request was submitted.
- Approval creates one immutable wallet ledger transaction. Repeated clicks or retries must not create duplicate credits.
- **TBD:** Crypto provider or explorer process, confirmation policy, supported assets/networks, deposit fees and limits.

### 6.2 Bank-transfer deposits via support

- A reseller who cannot use crypto contacts customer support.
- Support provides the appropriate local bank/payment instructions for that user's country through an approved support process.
- The reseller transfers fiat and supplies the requested reference/details.
- An authorized admin verifies that funds actually arrived in the platform's bank account, applies the configured exchange rate and fees, then manually enters the reseller UID and approved USDT credit amount in the admin panel.
- The system records the fiat amount, fiat currency, reference, exchange rate, fees, USDT amount, verifier and timestamps. An uploaded receipt image is supporting information, not proof by itself.
- **TBD:** Supported countries, bank accounts/payment partners, exchange-rate source, rate-lock policy, conversion spread, fees, refund handling and applicable compliance process.

## 7. Customer orders and manual partner fulfilment

- A customer visits a reseller's storefront, views retail prices and submits an order with a delivery address.
- The order appears in both the reseller dashboard and admin dashboard.
- The system atomically reserves the required reseller entitlement quantity when the order is accepted, so that entitlement cannot be sold twice.
- The order destination determines which partner stores may be suitable; the reseller's own country does not determine the delivery source.
- Admin reviews the order and manually contacts a suitable local partner store in the destination country. Admin shares the product, variant, quantity and delivery address through an approved operational process.
- Admin records which partner/location was contacted or assigned, the partner's stock/fulfilment confirmation, assignment status, delivery reference/evidence and customer-payment status.
- One order item may be split across more than one partner store if operations permits it; assignments must record the quantity handled by each partner.
- Partner store confirmation is a separate operational checkpoint. If a partner cannot fulfil, operations may reassign or cancel the affected quantity and release its reseller entitlement reservation as appropriate.
- The partner delivers directly to the customer. The customer confirms receipt through the platform. Admin records/validates the delivery outcome and independently verifies that the platform received the customer's payment.
- Delivery and payment are separate conditions. A delivered order with unpaid/unverified customer payment is not eligible for reseller wallet settlement. A payment without completed delivery and customer receipt confirmation is also not eligible under the agreed settlement rule.
- Sensitive representative contact information, partner purchase details and internal notes remain admin-only.
- Orders should support partial quantities, cancellations, failed delivery, returns, disputes and refunds through explicit states and audited actions.
- Customer-to-reseller chat is required for product questions and order-related communication. Admin support/oversight may be added with permission controls.

### Customer checkout payment method

Crypto was previously selected as the initial customer-payment direction. Keep checkout/payment methods configurable; do not hard-code a provider or mark a payment received automatically until the provider and independent verification process are selected. Admin confirmation of actual funds received is required for settlement.

## 8. Finalized reseller settlement rule — Prepay / Option A

The reseller prepays the wholesale amount when purchasing inventory entitlement. After the relevant quantity is delivered to the customer, the customer confirms receipt, and MercaLoop independently verifies receipt of the customer's payment, MercaLoop credits the reseller with the full confirmed customer sale proceeds, less only separately configured and disclosed fees/adjustments.

Example with no fees:

1. Opening reseller wallet: 1,000 USDT.
2. Purchase entitlement for 10 units at 80 USDT each: debit 800 USDT; available wallet becomes 200 USDT.
3. Reseller lists the 10 units at 100 USDT each.
4. Customer orders all 10 units; the system reserves 10 entitlement units.
5. Admin coordinates with a suitable local partner store; partner delivers; customer confirms receipt; MercaLoop verifies receipt of 1,000 USDT from the customer.
6. System credits 1,000 USDT once.
7. Final reseller wallet: 1,200 USDT. Reseller's gross spread is 200 USDT before any separately configured/disclosed fees or adjustments.

Do not deduct the 800 USDT wholesale amount again at sale. Settlement must be tied to the exact order quantities and reseller entitlement lots consumed. On partial delivery/payment, settle only the eligible delivered and paid quantity. Cancellations release reservations where appropriate. Refunds and fees are separate ledger events, never silent edits to completed entries.

**TBD:** The separate commercial arrangement between MercaLoop and each partner store, including whether/how a partner is compensated, the amount or pricing basis, timing and evidence of partner payment. Do not infer or automate partner payouts from the reseller's 80 USDT wholesale price until this arrangement is defined. Partner compensation/reconciliation must be tracked separately from the reseller wallet unless an explicitly approved reseller fee/adjustment applies.

## 9. Dashboards

### Reseller dashboard

- Overview and shop performance.
- Shop profile and storefront settings.
- Central wholesale product marketplace.
- My prepaid inventory entitlements: available/reserved/sold quantities and historical unit costs.
- Retail pricing and shop product visibility.
- Orders and order details.
- USDT wallet balance and immutable transaction history.
- Deposit requests and status.
- Customer conversations/messages.
- Notifications and account settings.

The reseller dashboard must describe the reseller's quantity as platform inventory entitlement or available selling quantity, not physical stock at the reseller's premises.

### Admin dashboard

- Overview and operational queue.
- User, reseller and shop management.
- Categories, products, variants, images and wholesale prices.
- Country and destination serviceability configuration.
- Partner store, partner location, partner contact and partner product-availability management.
- Deposit networks/addresses/QR/configuration.
- Crypto deposit review and manual bank deposit verification/crediting.
- Wallet ledger lookup and audited adjustments.
- Reseller prepaid entitlement purchases, lots and movements.
- Orders, entitlement reservations, partner assignments and fulfilment records.
- Customer payment verification, customer receipt confirmation and reseller settlement review.
- Customer/reseller support and chat oversight according to permissions.
- Fees, exchange-rate settings, reports, staff roles and audit logs.

### Customer storefront

- Reseller shop home and product catalogue.
- Product details and destination availability information.
- Cart and checkout.
- Order confirmation and order status.
- Customer-to-reseller chat.
- Account/profile and order history where accounts are enabled.

## 10. Countries and serviceability

The brand may be presented as global, but delivery must only be offered to destinations where the admin has configured an operational fulfilment path and partner-store coverage. Checkout must validate destination support and the reseller's available entitlement before accepting an order. Partner-reported availability is not a final guarantee; manual partner confirmation is required after order placement. Do not promise worldwide delivery merely because the site is accessible worldwide.

## 11. Non-functional requirements

- Responsive, accessible UI for desktop and mobile.
- Secure authentication, role-based access control and server-side authorization.
- Validate all amounts, quantities, prices, ownership and order states on the server.
- Use database transactions for wallet debits/credits, entitlement changes, reservations and settlement.
- Use idempotency keys and unique constraints to prevent duplicate financial or inventory operations.
- Append-only financial ledger and audit trail; never edit/delete completed ledger rows to correct an error.
- Store timestamps in UTC and render in the user's locale.
- Protect customer personal data and restrict partner contacts, customer addresses and internal notes.
- Backups, monitoring, error reporting, rate limiting, abuse controls and incident procedures are required before production.
- No real funds or production financial balances during local development.
- Do not represent reseller entitlements as physical stock or partner-reported stock as real-time guaranteed stock.

## 12. Decisions still required before production

- Legal entity, operating jurisdictions, licensing/compliance, KYC/AML and sanctions obligations.
- Crypto custody/payment provider and exact supported assets/networks.
- Customer checkout provider and independent confirmation method.
- Supported countries, partner stores, product coverage, service levels, delivery fees, returns and failed-delivery process.
- Partner stock verification/update process and how often partner availability becomes stale.
- Partner compensation, partner payables and partner payment/reconciliation policy.
- Bank conversion rates, fees, cut-off times and reconciliation process.
- Platform fees/commissions, reseller withdrawal policy and withdrawal availability.
- Whether reseller accounts require manual approval and what verification is required.
- Tax, invoices, consumer protection, privacy policy and terms.
- Inventory entitlement cost allocation method when the same reseller buys the same variant at different wholesale prices (for example, FIFO).
