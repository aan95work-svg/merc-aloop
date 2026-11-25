
# MercaLoop — Technical Architecture

**Status:** Updated architecture for the confirmed prepaid reseller-entitlement and manual local partner-store fulfilment model.

**Updated:** 26 November 2026

This document describes the technical architecture, application boundaries, technology choices, database connectivity, security requirements and development status of MercaLoop.

It is a technical design document, not a deployed production system.

## 1. Product and system overview

MercaLoop is a centrally managed wholesale and reseller marketplace.

The platform operator manages:
- Central product catalogue and wholesale prices.
- Reseller accounts and shops.
- Internal USDT wallet approvals.
- Prepaid reseller product entitlements.
- Local partner stores and their reported product availability.
- Customer orders and delivery coordination.
- Customer payment verification.
- Reseller sale settlements.
- Financial records and administrative audit logs.

Resellers purchase product entitlements using their internal USDT wallet, publish eligible products in their own MercaLoop shops, set retail prices and receive eligible customer sale proceeds after successful delivery, customer receipt confirmation and verified payment.

The physical products remain with independent local partner stores.

MercaLoop administrators manually contact suitable local store owners and coordinate direct delivery to customers.

The initial system does not depend on:
- Automated supplier API integrations.
- Automatic supplier purchase orders.
- Automatic partner stock synchronization.
- An automated partner self-service portal.
- Automatic courier integrations.

## 2. Application components

### 2.1 Frontend — Next.js

Application location:

`apps/web`

Technology:
- Next.js 16.
- React.
- TypeScript.
- Tailwind CSS.
- App Router.
- ESLint.

Responsibilities:
- Public MercaLoop marketplace.
- Reseller storefronts.
- Reseller registration and login experiences.
- Reseller dashboard.
- Customer storefront and checkout experiences.
- Customer order history.
- Customer-to-reseller messaging interface.
- Admin dashboard interface.
- Responsive desktop and mobile layouts.

Frontend responsibilities are limited to presentation, user interaction and calling authorized backend APIs.

The frontend must never be the authority for wallet balances, wholesale prices, inventory entitlement quantities, payment verification or settlement eligibility.

Development server:

`http://localhost:3000`

**Current status:** Next.js application created and its default development page successfully loaded.

### 2.2 Backend — NestJS

Application location:

`apps/api`

Technology:
- NestJS 12 CLI-generated project.
- Node.js 24.
- TypeScript.
- ESM modules.
- Vitest testing.
- Drizzle ORM.
- Drizzle Kit.
- Postgres.js.
- dotenv.

Responsibilities:
- API endpoints.
- Authentication token validation.
- Role-based authorization.
- User and reseller management.
- Product catalogue operations.
- Wholesale price validation.
- Wallet calculations and transactions.
- Crypto and bank deposit approval workflows.
- Prepaid reseller inventory purchases.
- Reseller entitlement tracking.
- Shop listing authorization.
- Customer orders and entitlement reservations.
- Partner-store management.
- Manual partner assignments.
- Customer payment verification records.
- Delivery and receipt confirmation.
- Option A wallet settlement.
- Chat and notification operations.
- Administrative audit logging.

All privileged business operations must be verified by the NestJS backend.

Development server:

`http://localhost:4000`

**Current status:**
- NestJS project created.
- ESM module system selected.
- Initial `Hello World!` endpoint tested successfully.
- Backend build successfully tested after dependency security cleanup.
- Drizzle ORM and database tooling installed.
- Database configuration file created.
- Database provider/module integration is not yet implemented.

### 2.3 Database — Supabase PostgreSQL

Provider:

Supabase

Database technology:

PostgreSQL

MercaLoop has its own independent Supabase project.

Confirmed project region:

`ap-northeast-1` — Tokyo, Japan.

PostgreSQL is the primary source of truth for:
- User profiles.
- Roles and permissions.
- Shops.
- Products and variants.
- Partner stores.
- Reseller prepaid entitlements.
- Inventory purchase lots.
- Wallets and financial ledgers.
- Deposit requests.
- Orders.
- Payment allocations.
- Fulfilment records.
- Settlement records.
- Messages and notifications.
- Audit logs.

The database is separate from all existing projects, including SGECORE.

**Current status:**
- Supabase project created.
- Project region confirmed.
- Database connection string configured locally.
- PostgreSQL connectivity successfully tested.
- No business database tables or migrations have been applied.

### 2.4 Authentication — Supabase Auth

Supabase Auth is the planned authentication provider.

Responsibilities:
- User registration.
- User login.
- Session and access-token management.
- Password recovery.
- Authentication identity.

The NestJS API must validate authentication tokens server-side.

User roles and permissions must be managed and enforced by the backend.

Supabase Auth integration and server-side token validation have not yet been implemented.

## 3. Confirmed technical choices

The following technology decisions have been made for the initial development.

| Component | Technology | Status |
|---|---|---|
| Frontend | Next.js | Installed |
| UI framework | React | Installed |
| Programming language | TypeScript | Installed |
| CSS | Tailwind CSS | Installed |
| Backend | NestJS | Installed |
| Runtime | Node.js 24 | Verified |
| Database | Supabase PostgreSQL | Created |
| ORM | Drizzle ORM | Installed |
| Migration tooling | Drizzle Kit | Installed |
| PostgreSQL driver | Postgres.js | Installed |
| Environment configuration | dotenv | Installed |
| Testing | Vitest | Installed |
| Source control | GitHub | Connected |
| Code editor | Visual Studio Code | Installed |

All selected tools are free or open-source for the current development requirements, subject to their individual service and usage terms.

### 3.1 Database connection method

The selected Supabase connection method is:

**Session Pooler**

Reasons:
- Compatible with IPv4-only client networks.
- Supports the selected local development connection method.
- Suitable as an alternative when a direct IPv6 database connection is unavailable.

The connection is configured through the NestJS backend's private environment file.

The current connection was successfully tested using Postgres.js with a read-only SQL query.

Verified result:

- Database: `postgres`
- Role: `postgres`
- Connection: Successful.

The standalone connection test does not mean the NestJS application has already been integrated with Drizzle.

### 3.2 Drizzle ORM configuration

Configuration file:

`apps/api/drizzle.config.ts`

Current configuration:

- PostgreSQL dialect.
- Schema source: `apps/api/src/db/schema.ts`.
- Migration output: root `database/migrations`.
- Database URL read from the local environment.

The schema TypeScript file and actual Drizzle table definitions have not yet been created.

Drizzle Kit must generate reviewed migrations. Do not apply unreviewed migrations directly to Supabase.

### 3.3 npm project structure

MercaLoop currently maintains independent package configurations:

- `apps/web/package.json`
- `apps/api/package.json`

npm workspaces are not configured.

Do not migrate package files or introduce workspace configuration without a deliberate architecture decision.

## 4. Core business architecture

MercaLoop maintains two independent inventory domains.

### 4.1 Reseller prepaid selling entitlements

Resellers purchase commercial selling entitlements using their USDT wallet.

For example:

- Reseller purchases 10 mobile-phone entitlements.
- Wholesale entitlement price is 80 USDT per unit.
- Total wallet deduction is 800 USDT.
- Reseller receives 10 available entitlement units.

The system creates:
- Inventory purchase record.
- Purchase item.
- Reseller product entitlement.
- Historical inventory cost lot.
- Immutable entitlement movement.
- Corresponding wallet ledger transaction.

The entitlement quantity determines how many units the reseller can list and sell.

It does not mean the reseller physically owns or holds the products.

### 4.2 Partner-store physical stock and availability

Independent local partner stores hold the actual products.

MercaLoop administrators maintain:
- Partner store records.
- Physical store locations.
- Partner contacts.
- Countries and delivery coverage.
- Partner-reported product availability.
- Last availability verification.
- Manual order assignments.
- Fulfilment confirmations.

Partner availability is informational and may become outdated.

The customer's delivery destination determines which local partner store is considered.

The reseller's home country does not determine the fulfilment source.

### 4.3 Reseller storefronts

Each reseller has a branded shop.

Production subdomain example:

`shopname.mercaloop.com`

The reseller can:
- Publish products covered by their prepaid entitlements.
- Set retail prices.
- Control product visibility.
- View orders and customers.
- Communicate with customers.
- See wallet and inventory entitlement records.

Public shop pages must never expose:
- Wholesale unit costs.
- Wallet ledger records.
- Other resellers' information.
- Private partner contacts.
- Internal delivery notes.
- Admin-only payment information.

## 5. Main application data flows

### 5.1 Reseller registration

1. User registers through the frontend.
2. Supabase Auth manages authentication.
3. NestJS validates the authentication token.
4. Backend creates or retrieves the corresponding profile.
5. A unique MercaLoop UID is assigned.
6. User role and account status are checked.
7. A reseller must satisfy any configured approval requirements before using reseller functionality.

### 5.2 Reseller prepaid entitlement purchase

1. Reseller browses the central product catalogue.
2. Reseller selects a product variant and quantity.
3. Frontend sends the request to NestJS.
4. API authenticates the reseller.
5. Backend validates eligibility and the current wholesale price.
6. Backend checks available USDT wallet balance.
7. Backend calculates the total using decimal arithmetic.
8. A PostgreSQL transaction performs the wallet debit and creates the purchase records.
9. A reseller product entitlement is created or retrieved.
10. A new historical inventory lot is created.
11. Available quantity is updated.
12. An immutable entitlement movement is inserted.
13. The API returns the resulting wallet and entitlement information.

All financial and entitlement changes must happen in one database transaction.

A retry must never create duplicate debits or quantities.

### 5.3 Customer order

1. Customer visits a reseller storefront.
2. Customer selects products and quantities.
3. Customer enters their delivery destination and address.
4. API validates listing ownership and visibility.
5. API validates the reseller's available entitlement quantity.
6. API validates destination serviceability.
7. Database transaction creates the order and order items.
8. Available reseller entitlements are reserved.
9. Corresponding inventory movements are inserted.
10. Order appears in reseller and admin dashboards.

Reserving a reseller entitlement does not automatically reserve partner physical stock.

### 5.4 Manual partner-store assignment

1. Admin reviews the order.
2. Admin identifies a suitable partner store based on the customer's delivery destination.
3. Admin contacts the local store owner manually.
4. Admin communicates product details, variants, quantities and delivery address.
5. Partner confirms actual availability and ability to fulfil.
6. Admin records the assignment and partner confirmation.
7. If necessary, admin contacts another partner store.
8. Partner prepares and delivers the order directly to the customer.

Multiple partner stores may fulfil different quantities of one order.

### 5.5 Delivery, customer receipt and payment verification

1. Partner delivers products to the customer.
2. Fulfilment evidence/reference is recorded.
3. Customer confirms receipt through MercaLoop.
4. Admin independently verifies actual receipt of customer payment.
5. Verified payments are allocated to the relevant order/item quantities.
6. Backend determines eligible delivered-and-paid quantities.

Delivery confirmation alone is insufficient.

Payment verification alone is also insufficient.

### 5.6 Finalized Option A settlement

1. Backend identifies eligible delivered-and-paid quantities.
2. Settlement uses saved retail-price snapshots.
3. Separately configured and disclosed reseller fees or adjustments are calculated.
4. Corresponding entitlement reservations are consumed.
5. Immutable inventory movement records are created.
6. Full eligible customer sale proceeds are credited to the reseller wallet, less approved fees/adjustments.
7. Settlement and ledger records are created.
8. The reseller dashboard displays updated wallet balance and transaction history.

**The wholesale entitlement purchase cost must never be deducted again during settlement.**

The settlement operation must be transactional and idempotent.

## 6. Security and authorization boundaries

### 6.1 Frontend security

Browser clients must never receive:
- Database passwords.
- Supabase service-role keys.
- Private API credentials.
- Wallet private keys.
- Internal partner credentials.
- Other privileged secrets.

The frontend must not calculate or decide final wallet balances, settlement eligibility, available inventory quantities or financial approvals.

### 6.2 Backend authorization

Every protected operation must validate:
- Authenticated user.
- Account status.
- User role.
- Resource ownership.
- Requested operation permission.
- Relevant order and financial state.

A reseller can only manage their own:
- Shop.
- Listings.
- Entitlements.
- Orders permitted by their ownership.
- Wallet.
- Deposit requests.
- Customer conversations.

A customer can only access:
- Their permitted account information.
- Their own orders.
- Conversations in which they are an authorized participant.

Administrative privileges must be enforced server-side.

Hiding a frontend button is not sufficient authorization.

### 6.3 Financial security

- Use PostgreSQL NUMERIC for monetary amounts.
- Use decimal arithmetic in backend financial calculations.
- Use database transactions for financial operations.
- Use idempotency keys.
- Lock relevant wallet and entitlement records against concurrent spending or reservations.
- Keep posted financial ledger records immutable.
- Use compensating entries for corrections.
- Maintain administrative audit trails.
- Reconcile cached balances and entitlement counters against their authoritative records.

### 6.4 Supabase security

Supabase service-role credentials must remain backend-only.

If Supabase Row Level Security is enabled, policies must be reviewed and tested.

RLS complements NestJS authorization; it does not replace it.

Never expose admin database credentials through public frontend environment variables.

### 6.5 Environment protection

Real connection strings and secrets must be kept in:

`apps/api/.env`

The `.env` file is ignored by the repository's `.gitignore`.

Only safe placeholder templates such as `.env.example` may be committed.

Never commit the actual `.env` file.

## 7. Repository structure

Current intended MercaLoop layout:

```text
merc-aloop/
│
├── apps/
│   ├── web/
│   │   ├── app/
│   │   ├── public/
│   │   ├── package.json
│   │   └── ...Next.js configuration
│   │
│   └── api/
│       ├── src/
│       │   ├── db/
│       │   ├── app.controller.ts
│       │   ├── app.module.ts
│       │   ├── app.service.ts
│       │   └── main.ts
│       │
│       ├── test/
│       ├── .env
│       ├── .env.example
│       ├── drizzle.config.ts
│       └── package.json
│
├── docs/
│   ├── PROJECT_SPEC.md
│   ├── DATABASE_SCHEMA.md
│   ├── WALLET_ACCOUNTING.md
│   ├── ARCHITECTURE.md
│   ├── DEVELOPMENT_PROGRESS.md
│   └── MODEL_CHANGELOG.md
│
├── database/
│   └── migrations/
│
├── .gitignore
└── README.md
```

The frontend and backend have their own package files.

The database migration directory is maintained at the repository root.

npm workspaces are not currently configured.

MercaLoop is an independent project located at:

`E:\MercaLoop\merc-aloop`

It must remain separate from SGECORE and all other existing projects.

## 8. Environment and local development

### 8.1 Current development environment

Operating system: Windows.

Project root:

`E:\MercaLoop\merc-aloop`

Node.js:

`v24.22.0`

npm:

`11.19.1`

PowerShell restricts execution of `npm.ps1` on the current machine.

Use:

- `npm.cmd`
- `npx.cmd`

when running package commands in the current PowerShell terminal.

### 8.2 Frontend development

```powershell
cd E:\MercaLoop\merc-aloop\apps\web
npm.cmd run dev
```

Frontend address:

`http://localhost:3000`

### 8.3 Backend development

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
$env:PORT="4000"
npm.cmd run start:dev
```

Backend address:

`http://localhost:4000`

### 8.4 Database environment

API environment file:

`apps/api/.env`

Safe example template:

`apps/api/.env.example`

Current environment variables:

```env
NODE_ENV=development
PORT=4000
DATABASE_URL=replace_with_supabase_session_pooler_connection_string
```

The actual `.env` contains the private Supabase connection string and must remain untracked.

## 9. Database architecture and migration strategy

### 9.1 Database technology

- Supabase-managed PostgreSQL.
- Drizzle ORM.
- Drizzle Kit migrations.
- Postgres.js PostgreSQL driver.
- dotenv environment configuration.

PostgreSQL remains the source of truth.

### 9.2 Drizzle configuration

File:

`apps/api/drizzle.config.ts`

Configuration:
- PostgreSQL dialect.
- Schema source at `apps/api/src/db/schema.ts`.
- Migration output at root `database/migrations`.
- Database credentials loaded from the private environment file.

The actual schema definitions are not yet implemented.

### 9.3 Migration strategy

Create reviewed, incremental migrations rather than generating the entire platform schema at once.

Recommended phases:

1. Identity, roles and shops.
2. Countries, serviceability and partner stores/locations/contacts/availability.
3. Catalogue, products and variants.
4. Wallet accounts and immutable ledger primitives.
5. Deposit configuration and manual deposit requests.
6. Reseller prepaid purchases, product entitlements, inventory lots and entitlement movements.
7. Shop listings, carts, orders and entitlement reservations.
8. Partner assignments, customer payments, payment allocations and fulfilments.
9. Order settlements, settlement payment allocations, refunds and disputes.
10. Chat, notifications, reports and audit extensions.
11. Partner compensation/payables/payments only after terms are agreed.

Each migration must:
- Be reviewed before execution.
- Be tested against a non-production database.
- Include appropriate foreign keys and constraints.
- Be reviewed for concurrency and idempotency.
- Be tested for rollback/recovery where applicable.
- Confirm the target Supabase project is MercaLoop.

No migration should be applied without reviewing its generated SQL.

## 10. Development environments and hosting

### 10.1 Local development

Local development uses:
- VS Code.
- Git.
- Node.js.
- Next.js.
- NestJS.
- Supabase PostgreSQL.

All local services and configurations must remain independent from SGECORE.

### 10.2 Development and production separation

When staging and production are introduced, use separate:
- Supabase projects.
- Database credentials.
- Environment variables.
- API deployment configurations.
- Frontend deployment configurations.

Never use production financial data for development testing.

### 10.3 Hosting

The final commercial production hosting provider has not been selected.

Do not assume that a free hosting tier permits commercial production use.

Evaluate providers based on:
- Commercial usage eligibility.
- Backend runtime support.
- PostgreSQL connectivity.
- Environment-variable security.
- Deployment and rollback support.
- Wildcard domain and SSL support.
- Expected traffic and operating costs.

Select an appropriate production plan before launch.

### 10.4 Domain and storefront subdomains

The domain will be purchased when the platform is ready for launch.

Planned storefront structure:

`shopname.mercaloop.com`

Production requires:
- Wildcard DNS configuration.
- Wildcard SSL support.
- Shop slug/hostname validation.
- Secure tenant resolution.
- Correct routing to the appropriate reseller shop.

Local development may use application routes or test host mapping before the real domain is purchased.

## 11. Development security and quality requirements

- Authentication and authorization must be tested.
- Reseller ownership must be checked server-side.
- Wallet debits must be atomic.
- Wallet credits must be idempotent.
- Inventory entitlement reservations must prevent overselling.
- Lot movements and lot counters must reconcile.
- Partner physical availability must remain separate from reseller entitlements.
- Customer payment verification must be independent of delivery verification.
- Sale settlement must require eligible delivery, customer receipt confirmation and verified/allocated payment.
- Partial fulfilment and partial settlements must be supported.
- Partner private information must never be exposed to customers or unrelated resellers.
- Posted financial records must remain immutable.
- API inputs must be validated.
- Rate limiting, monitoring and backups must be implemented before production.
- Security and financial reconciliation tests must pass before processing real money.

## 12. Technical decisions still open

The following decisions remain unresolved:

- Supabase Auth integration and server-side token validation.
- Exact NestJS database module/provider integration.
- Drizzle table definitions and first database migrations.
- Database roles and authorization policy.
- Supabase RLS configuration.
- Admin role/permission implementation.
- Product and country serviceability administration.
- Lot consumption/cost allocation policy for profit reporting.
- Partner availability verification and freshness policy.
- Partner compensation/payables model.
- Customer checkout provider and payment verification.
- Customer payment conversion to USDT.
- Refund loss-allocation policy.
- Reseller withdrawal policy.
- Production hosting provider.
- Actual domain purchase and wildcard DNS/SSL configuration.

Unresolved decisions must not be implemented through assumptions.

## 13. Current verified development status

### Completed

- Separate MercaLoop project workspace created.
- GitHub repository created and connected.
- Next.js frontend generated and initial development page tested.
- NestJS backend generated using ESM.
- NestJS initial endpoint tested successfully.
- NestJS build tested successfully after dependency security updates.
- Supabase project created independently.
- Supabase Tokyo region confirmed (`ap-northeast-1`).
- Supabase Session Pooler connection method selected.
- Private database connection string saved in `apps/api/.env`.
- PostgreSQL connection successfully verified from the API project.
- Drizzle ORM installed.
- Drizzle Kit installed.
- Postgres.js installed.
- dotenv installed.
- Root `.gitignore` checked; `.env` is ignored.
- `apps/api/.env.example` created.
- `apps/api/drizzle.config.ts` created.
- `apps/api/src/db` directory created.
- Dependency audit returned zero reported vulnerabilities after security cleanup.
- Optional NestJS Mau deployment package removed.
- Drizzle schema and accounting architecture documented and reviewed.
- Prepay / Option A wallet model confirmed.
- Local partner-store fulfilment architecture confirmed.

### Not yet completed

- Final documentation review and initial Git commit.
- Drizzle TypeScript table definitions.
- Database migration generation.
- Database migration testing and application.
- NestJS database provider/module integration.
- API health endpoint and frontend-to-backend connectivity test.
- Supabase Auth integration.
- Server-side token validation.
- User profile and MercaLoop UID implementation.
- Roles and permissions.
- Product catalogue implementation.
- Wallet ledger implementation.
- Deposit administration.
- Reseller prepaid entitlement purchasing.
- Reseller storefront implementation.
- Customer order functionality.
- Partner-store administration.
- Manual partner assignment workflow.
- Customer payment and delivery verification.
- Option A settlement implementation.
- Customer/reseller messaging.
- Security, reconciliation and production-readiness reviews.

## 14. Development working agreement

All MercaLoop development must follow these rules:

1. Keep MercaLoop completely separate from SGECORE and other existing projects.
2. Use free/open-source development tools wherever practical.
3. Do not purchase paid subscriptions unnecessarily.
4. Change one logical area at a time.
5. Review and test every important change before continuing.
6. Ask for the complete current file when a file needs modification.
7. Return a complete revised file rather than asking for manual insertion of individual lines.
8. Never request or expose private database passwords and secret keys in chat.
9. Never commit actual environment secrets to GitHub.
10. Review every database migration before execution.
11. Verify the selected Supabase project before any schema operation.
12. Do not enable production money flows until financial ledger, permissions, audit and reconciliation tests pass.
13. Do not assume unresolved partner compensation, withdrawal, conversion or refund policies.
14. Do not describe an unimplemented or untested feature as completed.

**Core development principle: Build the MercaLoop platform as a secure, modular, centrally managed marketplace with manually coordinated partner-store fulfilment and accurately controlled reseller financial accounting.**
