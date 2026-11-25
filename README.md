# MercaLoop

**A Global Wholesale and Reseller Marketplace**

MercaLoop is a centrally managed wholesale and reseller marketplace that allows approved resellers to purchase prepaid product-selling entitlements using an internal USDT wallet, operate their own branded storefronts and sell products to customers worldwide where local fulfilment is supported.

Actual physical products remain with independent local partner stores owned by friends, family members and other approved partners. MercaLoop administrators manually contact suitable local store owners and coordinate direct delivery to customers.

MercaLoop does not depend on automated supplier APIs or automatic courier integrations in its initial operating model.

---

## 1. Core Business Model

The MercaLoop operating model has four main participants:

1. **Platform administrator:** Manages the central catalogue, wholesale prices, reseller accounts, wallet approvals, orders, partner stores and operational processes.
2. **Reseller:** Purchases prepaid product entitlements using their USDT wallet, creates a branded online shop, sets retail prices and sells products.
3. **Customer:** Visits a reseller's shop, orders products, provides a delivery address and confirms receipt.
4. **Local partner store:** Holds the physical products and delivers them directly to the customer's address after manual coordination with MercaLoop administrators.

### Prepaid reseller inventory

Resellers purchase product quantities at wholesale prices using their internal USDT wallet.

These purchases create commercial selling entitlements, not physical inventory held by the reseller.

### Local partner fulfilment

When a customer places an order, MercaLoop administrators manually contact a suitable local partner store based on the customer's delivery destination.

The partner confirms availability and delivers directly to the customer.

### Finalized Option A settlement

The reseller pays the wholesale entitlement cost upfront.

After successful delivery, customer receipt confirmation and independent verification of customer payment, MercaLoop credits the reseller with the full eligible customer sale proceeds, less separately configured and disclosed fees or adjustments.

**The original wholesale cost must never be deducted a second time during settlement.**

See `docs/WALLET_ACCOUNTING.md` for the complete accounting rules.

---

## 2. Repository Structure

```text
merc-aloop/
│
├── apps/
│   ├── web/                  # Next.js frontend
│   │   ├── app/
│   │   ├── public/
│   │   └── package.json
│   │
│   └── api/                  # NestJS backend
│       ├── src/
│       │   └── db/           # Planned Drizzle schema location
│       ├── test/
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
│   └── migrations/           # Reviewed migration output
│
├── .gitignore
└── README.md
```

The frontend and backend currently maintain separate package configurations.

npm workspaces have not been configured.

The Drizzle TypeScript schema and actual database migrations have not yet been created.

---

## 3. Technology Stack

| Component | Technology |
|---|---|
| Frontend | Next.js 16 |
| UI framework | React |
| Programming language | TypeScript |
| Styling | Tailwind CSS |
| Backend | NestJS |
| Runtime | Node.js 24 |
| Database | Supabase PostgreSQL |
| ORM | Drizzle ORM |
| Migration tooling | Drizzle Kit |
| PostgreSQL driver | Postgres.js |
| Environment configuration | dotenv |
| Testing | Vitest |
| Version control | Git and GitHub |
| Code editor | Visual Studio Code |

The project uses free or open-source development tools wherever practical, subject to individual service and usage terms.

---

## 4. Prerequisites

For local development, install:

- Node.js 24 or compatible supported version.
- npm.
- Git.
- Visual Studio Code.

The current Windows development environment uses:

- Node.js `v24.22.0`
- npm `11.19.1`
- PowerShell.

**Windows PowerShell note:** The current machine blocks the `npm.ps1` and `npx.ps1` command shims through its execution policy.

Use:

- `npm.cmd`
- `npx.cmd`

Do not change the machine-wide PowerShell execution policy just for MercaLoop.

---

## 5. Clone the Repository

GitHub repository:

`https://github.com/aan95work-svg/merc-aloop.git`

Clone:

```powershell
git clone https://github.com/aan95work-svg/merc-aloop.git
cd merc-aloop
```

The current local development workspace is:

`E:\MercaLoop\merc-aloop`

MercaLoop must remain completely separate from SGECORE and all other existing projects.

---

## 6. Install Dependencies

The frontend and backend use separate package files.

### Frontend dependencies

```powershell
cd .\apps\web
npm.cmd install
```

### Backend dependencies

Open a separate terminal:

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
npm.cmd install
```

Do not introduce npm workspaces or migrate package files without a deliberate architecture decision.

---

## 7. Configure Environment Variables

The backend uses a local environment file:

`apps/api/.env`

A safe example template is maintained at:

`apps/api/.env.example`

For a fresh local setup, copy the example:

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
Copy-Item .env.example .env
```

Configure the private environment variables in `apps/api/.env`:

```env
NODE_ENV=development
PORT=4000
DATABASE_URL=your_private_supabase_postgresql_connection_string
```

Replace the database URL placeholder with the actual connection string from the MercaLoop Supabase project.

### Environment security rules

- Never commit `.env` files.
- Never share database passwords or private connection strings.
- Never expose Supabase service-role keys in the frontend.
- Commit only safe `.env.example` templates.
- Keep all production credentials separate from local development credentials.

The root `.gitignore` is configured to ignore `.env` files while allowing safe example templates.

---

## 8. Supabase Database Configuration

MercaLoop uses its own independent Supabase project.

Confirmed configuration:

| Setting | Value |
|---|---|
| Database provider | Supabase PostgreSQL |
| Region | Tokyo |
| AWS region | `ap-northeast-1` |
| Connection method | Session Pooler |
| Database | `postgres` |
| PostgreSQL driver | Postgres.js |
| ORM | Drizzle ORM |

The local PostgreSQL connection has been successfully tested using a read-only database query.

However, the following remain outstanding:

- Drizzle schema definitions.
- NestJS database module/provider integration.
- Actual PostgreSQL business tables.
- Database migrations.
- Supabase Auth integration.
- Database authorization and RLS policies.

A successful connection test does not mean that the MercaLoop business schema has already been created.

---

## 9. Run the Application Locally

Open two separate VS Code terminals.

### Terminal 1 — Frontend

```powershell
cd E:\MercaLoop\merc-aloop\apps\web
npm.cmd run dev
```

Frontend address:

`http://localhost:3000`

### Terminal 2 — Backend

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
$env:PORT="4000"
npm.cmd run start:dev
```

Backend address:

`http://localhost:4000`

The initial NestJS endpoint was successfully tested and returned:

`Hello World!`

The frontend successfully displayed the default Next.js welcome page.

These are development starter pages, not the completed MercaLoop application.

---

## 10. Database Development

The selected database tooling is:

- Drizzle ORM.
- Drizzle Kit.
- Postgres.js.
- Supabase PostgreSQL.

Drizzle configuration is maintained in:

`apps/api/drizzle.config.ts`

The intended TypeScript schema location is:

`apps/api/src/db/schema.ts`

Migration output location:

`database/migrations/`

Database implementation will proceed in reviewed phases:

1. Identity, roles and shops.
2. Countries, serviceability and partner stores.
3. Product catalogue and variants.
4. Wallet accounts and immutable ledger.
5. Deposit configuration and manual deposit requests.
6. Reseller prepaid purchases and product entitlements.
7. Inventory lots and entitlement movements.
8. Shop listings, carts, orders and reservations.
9. Partner assignments and fulfilment records.
10. Customer payments and allocations.
11. Reseller sale settlements.
12. Chat, notifications and audit extensions.

Every generated migration must be inspected before execution.

Do not run unreviewed migrations against the Supabase project.

---

## 11. Important Financial Rules

MercaLoop's initial internal reseller wallet is denominated in USDT.

### Example: Reseller purchases 10 mobile phones

| Description | Amount |
|---|---:|
| Opening wallet | 1,000 USDT |
| Wholesale price per unit | 80 USDT |
| Quantity purchased | 10 |
| Wholesale purchase debit | -800 USDT |
| Remaining wallet | 200 USDT |

The reseller lists the mobile phones at 100 USDT each.

A customer orders all 10 units.

After successful delivery, customer receipt confirmation and verified payment:

| Description | Amount |
|---|---:|
| Customer sale proceeds | 1,000 USDT |
| Reseller wallet credit | +1,000 USDT |
| Final wallet balance | 1,200 USDT |
| Gross reseller spread | 200 USDT |

The wholesale purchase cost has already been deducted.

**Never deduct the original 800 USDT again.**

Additional rules:

- Pending deposits cannot be spent.
- Deposit screenshots alone cannot authorize wallet credits.
- Orders must reserve reseller entitlements transactionally.
- Partner physical stock and reseller entitlements are separate records.
- Delivery and payment verification are independent conditions.
- Partial settlements are supported.
- Duplicate wallet credits are prevented with idempotency controls.
- Posted ledger records are immutable.
- Partner compensation is separate from reseller settlement and remains a business decision to be finalized.

See `docs/WALLET_ACCOUNTING.md` for the complete rules.

---

## 12. Project Documentation

The `docs/` directory contains the main product and technical specifications.

| Document | Purpose |
|---|---|
| `PROJECT_SPEC.md` | Product requirements and business workflow |
| `DATABASE_SCHEMA.md` | Proposed database structure and relationships |
| `WALLET_ACCOUNTING.md` | USDT wallet and Option A accounting rules |
| `ARCHITECTURE.md` | Frontend, backend, database and security architecture |
| `DEVELOPMENT_PROGRESS.md` | Completed work and outstanding development tasks |
| `MODEL_CHANGELOG.md` | Business and architecture decision history |

These documents are the working project specifications.

Their presence does not mean every documented feature has already been implemented.

---

## 13. Development and Security Rules

- Keep MercaLoop separate from SGECORE.
- Use free/open-source tools wherever practical.
- Change one logical area at a time and test before continuing.
- Keep the frontend and backend properly separated.
- Perform privileged business operations through NestJS.
- Validate wallet balances, prices and quantities on the backend.
- Use PostgreSQL transactions for financial and inventory mutations.
- Use idempotency keys for financial commands and settlements.
- Keep historical wholesale purchase costs immutable.
- Maintain reseller inventory entitlement quantities separately from local partner-store physical availability.
- Manually coordinate partner-store fulfilment in the MVP.
- Restrict access to customer information and private partner contacts.
- Record important administrative actions in audit logs.
- Inspect every database migration before execution.
- Confirm the selected Supabase project is MercaLoop before applying migrations.
- Never expose real credentials or private keys.
- Do not enable production financial transactions before security and reconciliation tests pass.

---

## 14. Features Still Under Development

The following functionality has not yet been completed:

- MercaLoop registration and authentication.
- Unique user UID generation.
- Roles and permission management.
- Public marketplace interface.
- Product catalogue administration.
- Reseller dashboards.
- Customer storefronts and shopping cart.
- USDT wallet ledger.
- Crypto and bank deposit workflows.
- Prepaid reseller entitlement purchases.
- Inventory lots and movement tracking.
- Reseller shop listings.
- Customer order management.
- Partner-store administration.
- Manual partner order assignments.
- Customer payment verification.
- Delivery confirmation.
- Option A wallet settlement.
- Customer/reseller chat.
- Notifications.
- Admin reporting and audit management.

These will be developed and tested in separate phases.

---

## 15. Production Deployment

Production hosting has not yet been selected.

A commercial-eligible hosting plan must be selected before launch.

The MercaLoop domain will be purchased when the application is ready.

Planned storefront subdomain structure:

`shopname.mercaloop.com`

Production deployment will require:

- Wildcard DNS.
- Wildcard SSL.
- Secure subdomain-to-shop resolution.
- Tenant and ownership isolation.
- Production environment secrets.
- Separate development, staging and production configurations.
- Database backups and recovery procedures.
- Monitoring and operational support.

Do not assume a free hosting plan permits commercial production use.

No real-money production flows should be enabled before security, legal, compliance, financial and operational reviews are complete.

---

## 16. Development Working Agreement

1. Keep MercaLoop completely independent from SGECORE.
2. Use free tools wherever practical.
3. Work one step at a time.
4. When an existing file needs modification, use its complete current contents as the starting point.
5. Return complete revised files rather than requiring manual insertion of individual lines.
6. Verify each important change before continuing.
7. Never ask for private database credentials in chat.
8. Do not commit real environment secrets.
9. Review all migrations before application.
10. Never claim that an unimplemented feature is complete.
11. Do not automate partner communication, supplier orders or courier integrations in the MVP.
12. Preserve the confirmed reseller prepayment and Option A accounting model.
13. Do not enable real-money operations before financial integrity, audit, authorization and reconciliation tests pass.

**MercaLoop development principle: Build a secure, centrally managed global reseller marketplace with prepaid selling entitlements, manually coordinated local partner-store fulfilment and reliable financial accounting.**