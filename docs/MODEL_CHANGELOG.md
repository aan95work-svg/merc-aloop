
# MercaLoop — Business Model Update Log

**Date:** 26 November 2026

**Status:** Business model decisions confirmed. This document records approved requirements and their architectural consequences. It does not mean that the corresponding software features have been implemented.

## 1. Confirmed business decisions

1. **Prepay model retained:** Resellers purchase product quantities at the platform wholesale price using their internal USDT wallet before listing or selling those quantities.

2. **Option A settlement retained:** The wholesale entitlement cost is debited when the reseller purchases the entitlement. After successful delivery, customer receipt confirmation and independent verification of customer payment, the reseller receives the full eligible customer sale proceeds, less separately configured and disclosed fees or adjustments. The original wholesale cost is never deducted again during settlement.

3. **Physical goods remain with local partner stores:** Friends, family members and other approved local contacts operate independent stores in different countries and hold the actual physical products.

4. **No reseller warehousing requirement:** A reseller's purchased quantity represents a prepaid MercaLoop selling entitlement. It does not mean physical goods are held by the reseller or allocated to the reseller's own warehouse.

5. **Manual fulfilment coordination:** After a customer order is placed, MercaLoop administrators contact a suitable partner store in the customer's delivery destination, confirm product availability and quantity, and coordinate direct delivery to the customer's address.

6. **Reseller country does not choose the delivery source:** A reseller in India may sell to a customer in the UK. An appropriate UK partner store may fulfil that order. The customer's delivery destination and available partner coverage determine the fulfilment source.

7. **No automatic supplier or courier integrations in the MVP:** Partner communication, fulfilment assignment, availability confirmation and delivery coordination are manually operated by authorized MercaLoop administrators.

8. **Partner compensation remains TBD:** The amount, calculation basis, payment timing and process for compensating local partner stores must be agreed separately. Do not infer partner payment amounts from the reseller's wholesale entitlement price.

## 2. Inventory model decisions

MercaLoop maintains two independent inventory domains.

### 2.1 Reseller prepaid selling entitlements

- A reseller prepays for a product quantity using their USDT wallet.
- The platform creates a purchase record and historical inventory cost lot.
- The purchase increases the reseller's available entitlement quantity.
- Entitlements represent the commercial quantity the reseller may list and sell.
- Available, reserved and sold quantities are tracked separately.
- Multiple prepaid purchases of the same product variant may contribute to one aggregate reseller product entitlement.
- Each purchase lot preserves its original wholesale unit cost.
- An entitlement purchase does not create or reserve physical partner-store stock.

### 2.2 Partner-store physical stock and availability

- Actual products remain with independent local partner stores.
- Partner stores and locations are maintained in the admin system.
- Partner product availability may be recorded by location and product variant.
- Partner-reported availability is informational and may become outdated.
- Manual order-specific confirmation is required before fulfilment is committed.
- Physical partner stock must never be combined with reseller entitlement quantities.

## 3. Architecture consequences

The following architectural rules result from the confirmed business decisions:

1. Maintain separate database records for reseller product entitlements, inventory lots and entitlement movements.

2. Maintain separate database records for partner stores, locations, contacts and partner-reported product availability.

3. Connect each reseller product entitlement to its underlying prepaid inventory lots.

4. Connect each inventory movement to the relevant reseller entitlement and inventory lot.

5. Update inventory lot counters and insert their corresponding immutable movement records in the same database transaction.

6. Connect reseller shop listings to the correct reseller product entitlement, not to physical partner-store stock.

7. Reserve reseller entitlement quantities when eligible customer orders are accepted.

8. Do not automatically reserve or decrement physical partner-store stock when a reseller entitlement is reserved.

9. After order placement, authorized administrators manually contact a suitable partner store, confirm availability and record the partner assignment.

10. Support multiple partner-store assignments for one order item when split fulfilment is required.

11. Support partner reassignment or order cancellation when a partner cannot fulfil the requested products or quantities.

12. Record delivery evidence, customer receipt confirmation and independently verified customer payment separately.

13. Link reseller settlement to the exact delivered quantity, customer receipt confirmation, verified and allocated payment, order item and corresponding reseller inventory reservation/lot.

14. Support partial delivery and partial settlement without duplicate wallet credits or duplicate entitlement consumption.

15. Apply the confirmed Option A settlement rule: credit full eligible customer sale proceeds less only separately configured and disclosed reseller fees or adjustments.

16. Keep partner compensation and partner payment records separate from reseller wallet settlement.

## 4. Financial example — Confirmed Option A

Assume no additional fees or adjustments.

| Description | Amount |
|---|---:|
| Opening reseller wallet | 1,000 USDT |
| Wholesale price per unit | 80 USDT |
| Quantity purchased | 10 |
| Initial wholesale entitlement debit | -800 USDT |
| Wallet after purchase | 200 USDT |
| Reseller retail price per unit | 100 USDT |
| Customer order total | 1,000 USDT |
| Eligible customer sale proceeds credited | +1,000 USDT |
| Final reseller wallet | 1,200 USDT |
| Reseller gross spread | 200 USDT |

The original 800 USDT wholesale entitlement cost has already been deducted at purchase.

It must never be deducted again at sale.

The example does not establish the amount payable to the local partner store. Partner compensation remains a separate unresolved business decision.

## 5. Partial fulfilment and settlement rules

Example: a customer orders 10 mobile phones, but only 6 are delivered and paid for.

- Only 6 units are eligible for reseller settlement.
- The other 4 units remain pending, may be reassigned to another partner, or may be released/cancelled according to the order outcome.
- The 6 eligible units must be linked to their delivered fulfilment records and verified customer payment allocations.
- The system must not settle the same eligible quantity twice.
- Any released entitlement quantity must have a corresponding immutable movement record.
- Any completed sale settlement must have its corresponding wallet ledger transaction.
- Partial delivery, payment and settlement must remain independently traceable.

Settlement must never exceed the intersection of delivered quantity, customer-confirmed receipt quantity and quantity supported by verified and allocated customer payment.

## 6. Explicitly unresolved business decisions

The following matters are not approved by this update and must not be implemented through assumptions:

- Partner compensation amount and calculation method.
- Partner payment timing and approval process.
- Partner availability confirmation and freshness policy.
- Customer payment provider and verification process.
- Customer payment currency conversion into USDT.
- Refund loss-allocation policy.
- Reseller withdrawal availability and processing policy.
- Final supported countries, delivery coverage and operational service levels.
- Applicable legal, licensing, KYC/AML, sanctions, tax and consumer-protection requirements.

These decisions must be documented and approved before the corresponding production features are enabled.

## 7. Implementation status

This update records confirmed requirements and architectural decisions.

It does not mean the following features have been implemented:

- Reseller wallet transactions.
- Prepaid entitlement purchases.
- Inventory lot management.
- Partner availability management.
- Customer orders.
- Partner assignments.
- Delivery confirmation.
- Customer payment verification.
- Option A wallet settlement.
- Partner compensation and payouts.

All these features remain subject to their respective development, security, testing and approval phases.

**Core decision:** MercaLoop operates on prepaid reseller selling entitlements, physically independent local partner-store fulfilment, manually coordinated delivery and Option A reseller wallet settlement.
