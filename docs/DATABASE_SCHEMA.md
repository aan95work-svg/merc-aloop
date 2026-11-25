
# MercaLoop — Proposed Database Schema

**Status:** Revised conceptual schema for the prepaid reseller-entitlement model and manual local partner-store fulfilment. This is a proposed database design, not an applied schema or migration.

**Updated:** 26 November 2026

## 1. Database principles

- PostgreSQL is the source of truth.
- Use UUID primary keys for internal entities. Keep the human-readable MercaLoop UID separate and unique.
- Use `NUMERIC`/decimal types for monetary amounts with explicit precision and scale for USDT. Never use floating-point types for money.
- Store timestamps in UTC.
- Add `created_at` and `updated_at` where relevant. Preserve immutable historical snapshots for prices and financial events.
- Use foreign keys, unique constraints, check constraints and indexes for data integrity.
- All financial, reseller-entitlement, partner-stock, reservation and settlement mutations must be transactional and idempotent.
- Use soft deactivation for users, products, partners and configurations when historical references must remain valid.
- Never cascade-delete financial ledger history.
- **Critical separation:** Reseller prepaid selling entitlements are not physical partner-store stock. Never combine these into one quantity or imply that the reseller physically possesses the products.

## 2. Identity and access

### `profiles`

- `id` UUID primary key, references Supabase `auth.users.id`.
- `uid` unique, immutable, human-readable MercaLoop identifier.
- `display_name`, `phone`, `country_code`, `status`.
- `created_at`, `updated_at`.

### `roles`, `permissions`, `role_permissions`, `user_roles`

- Define super administrator, admin/operations staff, reseller and customer roles.
- A partner-store owner/contact is not required to have a MercaLoop platform account in the MVP.
- A user may have multiple roles only if explicitly permitted by the product rules.
- Enforce permissions in the NestJS backend.
- If Supabase Row Level Security (RLS) is used, its policies must complement, not replace, server-side authorization.

### `admin_audit_logs`

- Actor user, action, target type and ID, timestamp, request/correlation ID, safe before/after metadata and reason.
- Never store passwords, tokens, private keys or other secrets in audit metadata.
- Record sensitive administrative operations such as deposit approvals, wallet adjustments, payment verification, delivery confirmations and settlement approvals.

## 3. Shops, countries and partner network

### `shops`

- `id` UUID primary key.
- `owner_user_id` references `profiles.id`.
- Unique normalized `slug`.
- `name`, `status`.
- Storefront configuration and supported settings.
- MVP constraint: one shop per reseller unless multiple shops are explicitly approved later.
- `created_at`, `updated_at`.

### `countries`, `serviceable_destinations`

- Country ISO code, display name and enablement status.
- Optional regions, cities and postal zones.
- Admin-configured destination coverage and delivery availability.
- A destination must not be enabled merely because the website is accessible there.
- Checkout must validate destination serviceability before accepting an order.

### `partner_stores`

- `id` UUID primary key.
- Store/business name, display name, status and internal notes.
- Created and updated timestamps.
- Represents an independent local store owned by a friend, family member or another approved partner.
- This is an admin-managed business record, not necessarily a MercaLoop user account.

### `partner_store_locations`

- `id` UUID primary key.
- `partner_store_id` references `partner_stores.id`.
- Country, region/city, address, postal coverage and status.
- Operating and delivery notes.
- A partner may have multiple stores or locations.
- Store addresses and location information are private and restricted to authorized operations staff.

### `partner_contacts`

- `id` UUID primary key.
- `partner_store_id` references `partner_stores.id`.
- Optional `partner_store_location_id` references `partner_store_locations.id`.
- Contact name, phone, email, preferred contact method, status and internal notes.
- Access restricted to authorized operations/admin staff.

### `partner_product_availability`

- `id` UUID primary key.
- `partner_store_location_id` references `partner_store_locations.id`.
- `product_variant_id` references `product_variants.id`.
- Reported quantity, nullable if the partner only confirms availability.
- Availability status.
- Partner-reported timestamp.
- Last admin-verified timestamp and verifying admin.
- Notes.
- Use either a unique active availability record per location/variant or explicit dated availability snapshots.
- Partner-reported quantities are informational and may become stale.
- They are not live API data and must not be treated as guaranteed physical stock.
- Manual, order-specific confirmation is required before fulfilment is committed.

### Optional future: `partner_stock_movements`

- Add only if MercaLoop decides to maintain a formal physical stock ledger for partner locations.
- Keep this separate from reseller entitlement movements.
- Do not automatically decrement partner-store stock in the MVP without a reliable partner confirmation process.

## 4. Central product catalogue

### `categories`

- `id` UUID primary key.
- `name`, `slug`.
- Optional parent category.
- `status`, sort order.
- Created and updated timestamps.

### `products`

- `id` UUID primary key.
- `category_id` references `categories.id`.
- `name`, `slug`, `description`.
- Brand/model fields and product status.
- Created and updated timestamps.

### `product_variants`

- `id` UUID primary key.
- `product_id` references `products.id`.
- Unique SKU where required.
- Variant attributes, status.
- Created and updated timestamps.

### `product_images`

- `id` UUID primary key.
- Product/variant reference.
- Storage path, alt text and sort order.
- Created and updated timestamps.

### `catalogue_prices`

- `id` UUID primary key.
- `product_variant_id` references `product_variants.id`.
- Wholesale price and currency.
- Initial reseller wallet currency: USDT.
- Effective period and status.
- Historical reseller inventory purchase lots must preserve the actual unit wholesale price paid.
- Future catalogue price changes must never rewrite historical purchase costs.

## 5. Reseller prepaid inventory entitlements

The reseller prepays for the commercial right and quantity to list and sell a product through MercaLoop.

The actual physical products remain with local partner stores. These database tables track reseller selling entitlements, not physical custody.

### `reseller_inventory_purchases`

- `id` UUID primary key.
- `reseller_user_id` references `profiles.id`.
- Purchase status.
- Total USDT cost.
- Wallet ledger transaction reference to `ledger_transactions.id`.
- Purchased timestamp.
- Actor/source metadata.
- Created timestamp.
- Represents one prepaid purchase of product quantities.
- Each approved purchase must have a corresponding wallet ledger transaction.

### `reseller_inventory_purchase_items`

- `id` UUID primary key.
- `purchase_id` references `reseller_inventory_purchases.id`.
- `product_id` references `products.id`.
- `product_variant_id` references `product_variants.id`.
- Purchased quantity.
- Wholesale unit price snapshot.
- Line total.
- Catalogue price reference/snapshot.
- Each purchase item creates one separate historical inventory cost lot.
- Historical wholesale price must never be overwritten by later catalogue price changes.

### `reseller_product_entitlements`

- `id` UUID primary key.
- `reseller_user_id` references `profiles.id`.
- `product_variant_id` references `product_variants.id`.
- `status`.
- `created_at`, `updated_at`.
- Unique constraint on `(reseller_user_id, product_variant_id)`.
- Represents the reseller's aggregate commercial entitlement to list and sell a product variant.
- Multiple prepaid inventory purchase lots may contribute to the same entitlement.
- A reseller has one entitlement record per product variant in the MVP.
- Does not represent physical partner-store stock.
- Available, reserved and sold quantities must be calculated from or reconciled against the underlying inventory lots and immutable entitlement movements.
- `shop_listings` must reference this table.

### `reseller_inventory_lots`

- `id` UUID primary key.
- `reseller_product_entitlement_id` references `reseller_product_entitlements.id`.
- `reseller_user_id` references `profiles.id`.
- `purchase_item_id` references `reseller_inventory_purchase_items.id`.
- `product_variant_id` references `product_variants.id`.
- `original_quantity`.
- `available_quantity`.
- `reserved_quantity`.
- `sold_quantity`.
- `wholesale_unit_cost_snapshot`.
- `purchased_at`, `status`.
- Each lot belongs to exactly one reseller product entitlement.
- The reseller and product variant must match the referenced entitlement.
- Preserve separate lots when the same reseller purchases the same product variant at different wholesale prices.
- Available, reserved and sold quantities are transactionally maintained operational counters.
- Every quantity change must update the lot counters and create the corresponding immutable `reseller_entitlement_movements` record in the same database transaction.
- A lot quantity must never be updated without its corresponding movement record.
- Lot counters must be reconcilable against the complete movement history.
- Purchase, reservation, release, consumption, return and authorized adjustment operations must use database transactions and appropriate row locking to prevent concurrent updates and overselling.
- **TBD:** Inventory lot consumption and cost-allocation policy, such as FIFO, before profit reporting is implemented.

### `reseller_inventory_balances`

- Optional aggregate/cache keyed by reseller and product variant.
- May store available, reserved and sold entitlement quantities for faster display.
- If implemented, it must be transactionally updated and reconcilable against inventory lots and immutable entitlement movements.
- If omitted, derive quantities from lot and movement records.
- Never use this table to represent partner-store physical stock.
- A cached balance is never the sole source of truth for inventory availability.

### `reseller_entitlement_movements`

- `id` UUID primary key.
- `reseller_product_entitlement_id` references `reseller_product_entitlements.id`.
- `reseller_inventory_lot_id` references `reseller_inventory_lots.id`.
- `reseller_user_id` references `profiles.id`.
- `product_variant_id` references `product_variants.id`.
- Immutable movement events:
  - Purchase.
  - Reserve.
  - Release.
  - Consume/sale.
  - Authorized adjustment.
  - Return/reinstate.
- Movement type.
- Quantity.
- From/to quantity bucket.
- Source event type and source ID.
- Unique idempotency key.
- Actor, reason and timestamp.
- The referenced inventory lot and entitlement must belong to the same reseller and product variant.
- Use non-negative quantity fields with explicit movement/bucket semantics.
- A purchase movement increases available entitlement.
- A reservation movement transfers quantity from available to reserved.
- A release movement transfers quantity from reserved back to available.
- A sale/consumption movement transfers quantity from reserved to sold.
- Returns and authorized adjustments must use explicit movement types and reasons.
- Enforce non-negative available and reserved quantities transactionally.
- Each movement and its corresponding lot counter update must happen in the same database transaction.
- Movement records are append-only. Completed movement records must never be silently edited or deleted.

### `shop_listings`

- `id` UUID primary key.
- `shop_id` references `shops.id`.
- `reseller_product_entitlement_id` references `reseller_product_entitlements.id`.
- `retail_price` using NUMERIC.
- Currency.
- Visibility/status.
- `created_at`, `updated_at`.
- Unique constraint on `(shop_id, reseller_product_entitlement_id)`.
- Backend must verify that the entitlement belongs to the owner of the shop.
- Listing quantity must be calculated from or constrained by the reseller's available entitlement.
- An independently editable listing quantity must never exceed available entitlement.
- A listing does not guarantee partner stock in every destination country.
- Checkout must validate destination coverage, and partner confirmation follows manually.
- Never expose wholesale unit cost or internal partner information publicly.

## 6. Wallet, deposits and accounting

### `wallet_accounts`

- `id` UUID primary key.
- Owner user references `profiles.id`.
- Asset/currency, initially USDT.
- Status.
- Created and updated timestamps.
- Unique constraint on `(owner_user_id, asset)` unless multiple wallets per asset are deliberately supported.
- Optional cached available/pending balance.
- If cached, reconcile against posted ledger entries.
- A cached balance must never be the sole source of financial truth.

### `financial_accounts`

- `id` UUID primary key.
- Internal accounting accounts such as:
  - User wallet liability.
  - Platform payment/crypto clearing.
  - Deposit clearing.
  - Order proceeds clearing.
  - Wholesale/partner clearing.
  - Fee revenue.
- Final account mapping depends on the approved accounting design and partner compensation terms.
- Do not assume the partner receives a particular amount merely because the reseller paid a wholesale price.

### `ledger_transactions`

- `id` UUID primary key.
- Wallet/accounting event type.
- Status.
- Unique idempotency key.
- Source type and source ID.
- Description.
- Actor.
- Timestamp.
- Add appropriate unique idempotency/source constraints.
- Posted journal transactions must be immutable.

### `ledger_entries`

- `id` UUID primary key.
- `ledger_transaction_id` references `ledger_transactions.id`.
- `financial_account_id` references `financial_accounts.id`.
- Debit/credit direction.
- Amount using NUMERIC.
- Currency.
- Amount must be positive; direction represents debit or credit.
- Append-only.
- Posted entries cannot be edited or deleted.
- Posted journal transactions must balance for each currency according to the approved accounting design.

### `deposit_methods`

- `id` UUID primary key.
- Asset.
- Network/chain.
- Token identifier/contract where relevant.
- Display label.
- Receiving address.
- QR/storage reference.
- Instructions.
- Minimum and maximum deposit limits.
- Enabled/disabled status.
- Sort order.
- Asset and network must be stored separately.
- Examples:
  - USDT + Tron/TRC20.
  - USDT + BNB Smart Chain/BEP20.
  - USDT + Ethereum/ERC20.
- Never store private keys or seed phrases.

### `crypto_deposit_requests`

- `id` UUID primary key.
- User references `profiles.id`.
- Deposit method reference.
- Requested amount.
- Submitted TXID.
- Observed amount, network, receiving address and confirmations when independently verified.
- Status.
- Reviewer and reviewed timestamp.
- Ledger transaction reference.
- Enforce duplicate TXID prevention for the same network where appropriate.
- User submission or screenshot alone must never trigger wallet credit.

### `bank_deposit_requests`

- `id` UUID primary key.
- User references `profiles.id`.
- Fiat currency and amount.
- Bank reference.
- Submitted details.
- Verified received amount.
- Applied exchange rate.
- Fee.
- Approved USDT amount.
- Status.
- Reviewer and review timestamps.
- Ledger transaction reference.
- Receipt upload is supporting information only, not proof that funds arrived.
- Only authorized admin approval after actual receipt verification may create a wallet credit.

### `wallet_adjustments`

- `id` UUID primary key.
- Wallet reference.
- Authorized adjustment reason.
- Amount and currency.
- Actor.
- Approval if required.
- Linked ledger transaction.
- Never silently overwrite a wallet balance.

## 7. Commerce, reservations and partner fulfilment

### `carts`, `cart_items`

#### `carts`

- `id` UUID primary key.
- Customer/session reference.
- `shop_id` references `shops.id`.
- Status and timestamps.

#### `cart_items`

- `id` UUID primary key.
- `cart_id` references `carts.id`.
- `shop_listing_id` references `shop_listings.id`.
- Product variant.
- Quantity.
- Current retail price snapshot.
- Currency.

### `orders`, `order_items`

#### `orders`

- `id` UUID primary key.
- Unique order number.
- Customer references `profiles.id` where customer accounts are enabled.
- `shop_id` references `shops.id`.
- Reseller reference.
- Destination country, address and required delivery details.
- Status.
- Subtotal, fees and total.
- Currency.
- Created and updated timestamps.

#### `order_items`

- `id` UUID primary key.
- `order_id` references `orders.id`.
- Product/variant.
- `reseller_product_entitlement_id` references `reseller_product_entitlements.id`.
- Product name and SKU snapshots.
- Quantity.
- Retail unit price snapshot.
- Line total.
- Currency.
- Prevent sale quantities exceeding available reseller entitlements.

### `inventory_reservations`

- `id` UUID primary key.
- `order_item_id` references `order_items.id`.
- `reseller_inventory_lot_id` references `reseller_inventory_lots.id`.
- Quantity.
- Status.
- Expiry/release details.
- Unique idempotency key.
- One order item may reserve from multiple reseller inventory lots.
- These reservations lock reseller entitlements.
- They do not reserve a partner store's physical stock automatically.
- Creating, releasing or consuming a reservation must create the corresponding entitlement movement in the same database transaction.

### `partner_order_assignments`

- `id` UUID primary key.
- `order_item_id` references `order_items.id`.
- `partner_store_id` references `partner_stores.id`.
- `partner_store_location_id` references `partner_store_locations.id`.
- Assigned quantity.
- Assignment status.
- Assigned-by admin.
- Contacted timestamp.
- Partner stock/fulfilment confirmation timestamp and method.
- Notes/reference.
- Allow multiple assignments for one order item to support split fulfilment across partner stores.
- Assignment is created/updated by authorized admin after manually contacting the partner.
- No automatic supplier API or partner portal is assumed in the MVP.
- Enforce that total active assigned quantity cannot exceed the remaining order-item quantity.

### `customer_payments`

- `id` UUID primary key.
- `order_id` references `orders.id`.
- Payment method.
- Asset/currency.
- Amount.
- Provider/reference/TXID.
- Status.
- Independently verified receipt details.
- Reviewer and timestamps.
- Submitted, pending, verified/received, failed and refunded states must be distinguishable.

### `payment_allocations`

- `id` UUID primary key.
- `payment_id` references `customer_payments.id`.
- Order reference.
- Optional order item reference.
- Allocated amount/currency.
- Eligible quantity where relevant.
- Status.
- Reviewer/source.
- Supports multiple/partial customer payments.
- Prevents the same verified payment value being allocated more than once.
- Allocations must reconcile to independently verified received payment amounts.

### `fulfilments`

- `id` UUID primary key.
- `order_id` references `orders.id`.
- Destination.
- Status.
- Delivery evidence/reference.
- Delivered timestamp.
- Customer receipt confirmation timestamp and source.
- Confirming actor and admin verification.
- A fulfilment may cover one or more order items.

### `fulfilment_items`

- `id` UUID primary key.
- `fulfilment_id` references `fulfilments.id`.
- `order_item_id` references `order_items.id`.
- `partner_order_assignment_id` references `partner_order_assignments.id`.
- Product variant.
- Delivered quantity.
- Item delivery status.
- Evidence/reference.
- Supports partial delivery and split partner fulfilment.

### `order_settlements`

- `id` UUID primary key.
- `order_id` references `orders.id`.
- Reseller reference.
- Gross proceeds.
- Separately configured fees/adjustments.
- Reseller credit amount.
- Status.
- Ledger transaction reference.
- Settlement actor and timestamp.
- Settlement eligibility requires eligible delivered quantity, customer receipt confirmation and sufficient verified/allocated customer payment.

### `order_settlement_items`

- `id` UUID primary key.
- `order_settlement_id` references `order_settlements.id`.
- `order_item_id` references `order_items.id`.
- `fulfilment_item_id` references `fulfilment_items.id`.
- `inventory_reservation_id` references `inventory_reservations.id`.
- Eligible delivered-and-paid quantity.
- Retail unit price snapshot.
- Gross proceeds.
- Separately configured fees/adjustments.
- Reseller credit.
- Ledger transaction linkage.
- Unique/idempotent settlement allocation for each eligible quantity/event.
- Prevent repeated credits for the same sold units.
- Partial settlement must be supported.
- Settled quantity must not exceed delivered quantity or the quantity supported by verified and allocated customer payments.

### `order_settlement_payment_allocations`

- `id` UUID primary key.
- `order_settlement_item_id` references `order_settlement_items.id`.
- `payment_allocation_id` references `payment_allocations.id`.
- Amount applied to this settlement item.
- Currency.
- Created timestamp.
- Connects eligible customer payments to the exact reseller settlement amount.
- Prevents the same payment allocation amount being used more than once.
- All allocated amounts must reconcile to verified customer payments.

### `refunds`, `disputes`, `order_status_events`

- Explicit records for refunds, disputes, cancellations, returns and status transitions.
- Refunds use new compensating ledger entries.
- Never edit historical wallet credits to process refunds.

## 8. Chat and notifications

### `conversations`, `conversation_participants`, `messages`

- Customer/reseller conversation.
- Optional order reference.
- Participant membership.
- Message content.
- Timestamps and read state.
- Enforce participant-level access.
- Admin access must be role-restricted and audited.
- Do not use customer/reseller chat as an uncontrolled channel for exposing private partner contacts or internal fulfilment notes.

### `notifications`

- `id` UUID primary key.
- User reference.
- Event type.
- Payload.
- Read/delivery status.
- Timestamp.
- Do not include secrets or unnecessary personal information in notification payloads.

## 9. Future partner accounting — Not activated until terms are agreed

Partner compensation is not yet defined.

Before implementing partner payouts, agree whether a partner receives:
- A fixed per-unit amount.
- An agreed invoice amount.
- A share of the wholesale amount.
- Another approved compensation arrangement.

Also determine when payment becomes due.

After those decisions, add/review:

### `partner_compensation_terms`

- Approved terms by partner/product/location.
- Amount or agreed calculation method.
- Effective dates.
- Approval and status.

### `partner_payables`

- Amount MercaLoop owes a partner for confirmed fulfilment.
- Source order/fulfilment references.
- Status and due date.

### `partner_payments`

- Actual payment.
- Method and reference.
- Approver.
- Payment date.
- Reconciliation state.

Do not infer a partner payment amount from the reseller's wholesale price.

Do not mix partner payable records with reseller wallet credits.

## 10. Relationships and integrity rules

- One authentication user has one profile and one immutable unique MercaLoop UID.
- One reseller owns one shop in the MVP unless multiple shops are approved later.
- A shop belongs to one reseller owner.
- Products have variants.
- Partner locations may report availability for product variants independently.
- Prepaid reseller inventory purchases create historical cost lots and entitlement movements.
- Reseller purchases do not create or decrement physical partner-store stock.
- One reseller product entitlement represents the aggregate right to list and sell one product variant.
- Multiple purchase lots may contribute to the same reseller product entitlement.
- Each inventory lot belongs to exactly one reseller product entitlement.
- The reseller and product variant must match between entitlement, lot and movement references.
- Shop listings refer to reseller product entitlements and store retail pricing, not physical stock quantities.
- Each shop listing must belong to the reseller who owns the entitlement.
- Orders belong to one customer and one reseller shop.
- Order items reference reseller product entitlements and reserve reseller inventory lots.
- Partner order assignments connect order items to one or more partner locations.
- Partner assignments are manually recorded by authorized admins.
- Partner physical availability and reseller selling entitlements are separate data domains.
- Wallet ledger transactions may contain multiple balanced ledger entries.
- Every approved deposit, inventory purchase, sale settlement, fee, refund or adjustment must link to a unique ledger transaction.
- Order settlements link to delivered quantities and verified/allocated customer payments.
- Partial settlements must never pay the same quantity twice.

## 11. Recommended indexes, constraints and security rules

- Unique `profiles.uid`.
- Unique normalized shop slug and hostname.
- Unique product and variant SKUs where required.
- Unique `(reseller_user_id, product_variant_id)` on `reseller_product_entitlements`.
- Unique `(shop_id, reseller_product_entitlement_id)` on `shop_listings`.
- Unique `purchase_item_id` on `reseller_inventory_lots` if one purchase item always creates exactly one lot.
- Unique idempotency keys for inventory movements, wallet ledger operations and settlement allocations.
- Unique or appropriately scoped source-event constraints to prevent repeated financial and inventory operations.
- Index orders by shop/status/created date and customer/status/created date.
- Index deposit requests by user/status/created date and TXID/network.
- Check that all monetary request and ledger amounts are valid positive NUMERIC values where applicable.
- Represent debit/credit direction explicitly.
- Enforce `reserved_quantity <= original_quantity - sold_quantity` for ordinary lot operations, with explicit handling for authorized returns and adjustments.
- Enforce non-negative available and reserved quantities using database constraints and transactional logic.
- Ensure the quantity relationship between original, available, reserved and sold counters is consistent.
- Index partner product availability by location/variant/status/last verified time.
- Index partner assignments by location/status/created date and order item.
- Index messages by conversation and creation time.
- Use row locks or equivalent concurrency control when purchasing, reserving, releasing or consuming reseller entitlement lots.
- Every lot quantity update and its immutable movement insertion must occur in the same transaction.
- Reconciliation must detect any discrepancy between lot counters and entitlement movement history.
- Enforce backend authorization for all private resources.
- Restrict fulfilment partner contact information to authorized operations staff.
- Never expose private wallet, wholesale pricing, partner contacts or internal fulfilment notes through public storefront APIs.

## 12. Migration strategy

Do not generate all tables in one unreviewed migration.

Implement in small, reviewed migrations:

1. Identity, roles and shops.
2. Countries, serviceability and partner stores/locations/contacts/product-availability records.
3. Catalogue, products and variants.
4. Wallet accounts and immutable ledger primitives.
5. Deposit configuration and manual deposit requests.
6. Reseller prepaid purchases, product entitlements, inventory lots and entitlement movements.
7. Shop listings, carts, orders and reseller entitlement reservations.
8. Partner assignments, customer payments, payment allocations, fulfilments and fulfilment items.
9. Order settlements, settlement items, settlement payment allocations, refunds and disputes.
10. Chat, notifications, reporting and audit extensions.
11. Partner compensation, payables and payments only after the business terms are agreed.

Each migration must:

- Be reviewed before execution.
- Be tested against a non-production database before deployment.
- Be checked for foreign-key and unique-constraint correctness.
- Be checked for transaction and idempotency behaviour.
- Confirm that the selected Supabase project is MercaLoop before applying changes.

This document describes the proposed model. It is not an applied database schema. Actual SQL/Drizzle table definitions and migrations must be created after the schema and authorization design are reviewed and approved.
