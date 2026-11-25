
# MercaLoop — Development Progress

**Last updated:** 26 November 2026

**Project location:** `E:\MercaLoop\merc-aloop`

**GitHub repository:** `https://github.com/aan95work-svg/merc-aloop.git`

**Current development stage:** Foundation, architecture and database preparation.

---

## 1. Completed and verified locally

### Project workspace and source control

- [x] Created separate MercaLoop workspace at `E:\MercaLoop\merc-aloop`.
- [x] Confirmed MercaLoop is independent from the existing SGECORE project.
- [x] Created dedicated GitHub repository `aan95work-svg/merc-aloop`.
- [x] Cloned the repository locally.
- [x] Verified Git remote points to the MercaLoop repository.
- [x] Verified current Git branch is `main`.
- [x] Created root folders `apps`, `docs` and `database/migrations`.
- [x] Confirmed PowerShell is operating in the correct MercaLoop directory.
- [x] Removed the temporary `ARCHIVE-README.txt` file.

### Development environment

- [x] Verified Node.js version `v24.22.0`.
- [x] Verified npm version `11.19.1`.
- [x] Identified Windows PowerShell restriction on `npm.ps1`.
- [x] Confirmed `npm.cmd` and `npx.cmd` work.
- [x] Established the rule to use `npm.cmd` and `npx.cmd` in the current PowerShell environment.

### Frontend

- [x] Generated Next.js frontend in `apps/web` using create-next-app `16.4.6`.
- [x] Installed React, TypeScript, Tailwind CSS, ESLint and App Router.
- [x] Started the frontend development server successfully.
- [x] Verified the default Next.js page loads at `http://localhost:3000`.

### Backend

- [x] Generated NestJS backend in `apps/api`.
- [x] Selected ESM (ES Modules).
- [x] Configured Vitest during NestJS generation.
- [x] Confirmed the initial NestJS endpoint returns `Hello World!` at `http://localhost:4000`.
- [x] Successfully built the NestJS backend using `npm.cmd run build`.
- [x] Successfully rebuilt the backend after dependency security updates.

### Dependency installation and security

- [x] Installed Drizzle ORM.
- [x] Installed Drizzle Kit.
- [x] Installed Postgres.js.
- [x] Installed dotenv.
- [x] Inspected the initial npm dependency vulnerabilities.
- [x] Identified deprecated nested esbuild dependency as one source of vulnerabilities.
- [x] Added a targeted esbuild dependency override instead of using `npm audit fix --force`.
- [x] Removed the optional `@nestjs/mau` deployment package because it was not needed for the planned hosting architecture.
- [x] Removed its unused NestJS deployment script.
- [x] Approved only the required esbuild installation scripts for the installed versions.
- [x] Ran the final `npm.cmd audit` successfully with zero reported vulnerabilities.
- [x] Confirmed production dependencies reported zero vulnerabilities.
- [x] Avoided unnecessary breaking dependency downgrades.

### Supabase and PostgreSQL

- [x] Created a separate MercaLoop Supabase project.
- [x] Confirmed Supabase project region as `ap-northeast-1` (Tokyo, Japan).
- [x] Selected Supabase Session Pooler as the initial PostgreSQL connection method.
- [x] Obtained the PostgreSQL URI connection string.
- [x] Created the local API environment file `apps/api/.env`.
- [x] Stored the private database connection string locally.
- [x] Successfully tested the PostgreSQL connection using Postgres.js.
- [x] Verified the test query returned database `postgres` and role `postgres`.
- [x] Confirmed database connectivity without creating or modifying business tables.

### Environment security

- [x] Reviewed the root `.gitignore`.
- [x] Confirmed `.env` is excluded from Git.
- [x] Confirmed `.env.example` is allowed by the ignore rules.
- [x] Created `apps/api/.env.example` with safe placeholder values.
- [x] Created the local `apps/api/.env` file.
- [x] Verified Git ignores `apps/api/.env`.
- [x] Kept actual database credentials out of shared documentation.

### Drizzle configuration

- [x] Confirmed the backend uses ESM through its package configuration.
- [x] Created `apps/api/drizzle.config.ts`.
- [x] Configured the PostgreSQL dialect.
- [x] Configured the Drizzle schema path as `apps/api/src/db/schema.ts`.
- [x] Configured migration output at the repository-level `database/migrations` directory.
- [x] Configured Drizzle to read `DATABASE_URL` from the private environment file.
- [x] Created `apps/api/src/db` directory.

### Product and business architecture

- [x] Confirmed the central MercaLoop product catalogue model.
- [x] Confirmed reseller prepayment using the internal USDT wallet.
- [x] Confirmed Option A settlement.
- [x] Confirmed wholesale entitlement cost is deducted at purchase.
- [x] Confirmed full eligible customer sale proceeds are credited after delivery, customer receipt confirmation and verified payment, less separately approved fees/adjustments.
- [x] Confirmed the original wholesale cost must never be deducted a second time at sale.
- [x] Confirmed reseller product entitlements and partner-store physical stock are separate concepts.
- [x] Confirmed physical products remain with independent local partner stores.
- [x] Confirmed administrators manually contact suitable local partners based on the customer's delivery destination.
- [x] Confirmed partners deliver directly to customers.
- [x] Confirmed automatic supplier APIs and automatic courier integrations are not assumed for the MVP.
- [x] Confirmed partner compensation and withdrawals remain unresolved production decisions.

### Project documentation

- [x] Updated the conceptual `PROJECT_SPEC.md` for the reseller and partner-store operating model.
- [x] Reworked `DATABASE_SCHEMA.md` to include reseller product entitlements, inventory lots, entitlement movements, partner stores, shop listings, orders, fulfilments and settlement allocations.
- [x] Added explicit relationships between reseller entitlements, inventory lots and movements.
- [x] Added rules requiring inventory lot counter updates and immutable movement records in the same database transaction.
- [x] Connected shop listings to reseller product entitlements.
- [x] Updated `WALLET_ACCOUNTING.md` with the complete Option A settlement process.
- [x] Added partial order settlement, refunds, reconciliation and financial safeguards to wallet documentation.
- [x] Updated `ARCHITECTURE.md` to reflect the actual technology choices and verified development progress.
- [x] Verified key content in the revised documentation using PowerShell.
- [x] Replaced the local `DEVELOPMENT_PROGRESS.md` with this updated progress record.

---

## 2. Current project structure

```text
merc-aloop/
│
├── apps/
│   ├── web/
│   │   ├── app/
│   │   ├── public/
│   │   └── package.json
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

This is the intended project organization. The actual folders and package files should be checked locally before the first Git commit.

The frontend and backend have separate package files.

npm workspaces have not been configured.

---

## 3. Confirmed technical configuration

| Component | Technology | Current status |
|---|---|---|
| Frontend | Next.js 16 | Installed and tested |
| UI | React | Installed |
| Language | TypeScript | Installed |
| Styling | Tailwind CSS | Installed |
| Backend | NestJS 12 | Installed and tested |
| Runtime | Node.js 24 | Verified |
| Package manager | npm 11 | Verified |
| Database | Supabase PostgreSQL | Created and connected |
| Database region | Tokyo `ap-northeast-1` | Confirmed |
| Connection method | Supabase Session Pooler | Connected |
| ORM | Drizzle ORM | Installed |
| Migrations | Drizzle Kit | Installed |
| PostgreSQL driver | Postgres.js | Installed |
| Environment loader | dotenv | Installed |
| Testing | Vitest | Installed |
| Version control | GitHub | Connected |
| Editor | Visual Studio Code | Installed |

---

## 4. Not yet completed

### Source control and documentation

- [ ] Review root `README.md` and confirm its setup instructions.
- [ ] Review `MODEL_CHANGELOG.md` against the final business decisions.
- [ ] Perform final documentation consistency review.
- [ ] Create the initial Git commit.
- [ ] Push the initial MercaLoop foundation to GitHub.
- [ ] Confirm no private credentials or generated build files are included in the commit.

### Database implementation

- [ ] Create actual Drizzle TypeScript table definitions in `apps/api/src/db/schema.ts`.
- [ ] Define shared database column conventions and reusable types.
- [ ] Decide the exact database role and authorization structure.
- [ ] Define foreign-key actions and deletion restrictions.
- [ ] Define unique constraints, indexes and validation constraints.
- [ ] Configure the NestJS database provider/module.
- [ ] Integrate Postgres.js and Drizzle into the NestJS application.
- [ ] Implement a safe database health check.
- [ ] Generate the first reviewed Drizzle migration.
- [ ] Inspect generated SQL before execution.
- [ ] Establish a non-production migration testing strategy.
- [ ] Test migrations before applying them to the MercaLoop Supabase project.
- [ ] Confirm database backup and recovery arrangements before production.

### Authentication and authorization

- [ ] Configure Supabase Auth.
- [ ] Implement server-side access-token validation in NestJS.
- [ ] Implement user profiles.
- [ ] Generate unique immutable MercaLoop UIDs.
- [ ] Implement roles and permissions.
- [ ] Implement backend authorization and ownership checks.
- [ ] Review Supabase Row Level Security policies.
- [ ] Test access isolation between customers, resellers and admins.

### Backend API foundation

- [ ] Configure API modules and service boundaries.
- [ ] Add a proper health endpoint.
- [ ] Test frontend-to-backend communication.
- [ ] Configure request validation and error handling.
- [ ] Configure structured logging and request correlation IDs.
- [ ] Add API security, rate limiting and permission guards.
- [ ] Configure API integration testing.

### Business modules

- [ ] Identity, registration and account management.
- [ ] Admin roles and permission management.
- [ ] Countries and serviceability.
- [ ] Categories, products, variants and images.
- [ ] Catalogue wholesale pricing.
- [ ] Partner stores, locations and private contacts.
- [ ] Partner product availability and manual verification.
- [ ] Wallet accounts and immutable double-entry ledger.
- [ ] Crypto deposit configuration and manual verification.
- [ ] Support-assisted bank deposits.
- [ ] Reseller prepaid inventory purchases.
- [ ] Reseller product entitlements and historical inventory lots.
- [ ] Entitlement movements and quantity reconciliation.
- [ ] Reseller shop listings and retail pricing.
- [ ] Customer storefronts and checkout.
- [ ] Orders and entitlement reservations.
- [ ] Manual partner order assignments.
- [ ] Customer payment verification and payment allocations.
- [ ] Partial fulfilment and customer receipt confirmation.
- [ ] Option A reseller sale settlement.
- [ ] Refunds, disputes and audit history.
- [ ] Customer/reseller messaging.
- [ ] Notifications and reporting.

### Remaining business decisions

- [ ] Choose the entitlement inventory lot consumption/cost-allocation policy, such as FIFO.
- [ ] Define partner compensation terms.
- [ ] Define how partners confirm availability and how stale availability is handled.
- [ ] Define customer checkout payment provider and verification process.
- [ ] Define customer payment currency conversion to USDT.
- [ ] Define reseller refund-loss allocation.
- [ ] Define whether reseller withdrawals will be supported.
- [ ] Define applicable supported countries and operational delivery coverage.
- [ ] Complete legal, licensing, KYC/AML, sanctions, consumer protection, privacy and tax reviews.

### Production readiness

- [ ] Select commercial-eligible production hosting.
- [ ] Establish separate development, staging and production configurations.
- [ ] Configure production environment secrets securely.
- [ ] Purchase the MercaLoop domain when ready.
- [ ] Configure wildcard DNS and SSL for reseller subdomains.
- [ ] Configure backups, monitoring and incident procedures.
- [ ] Complete security testing.
- [ ] Complete wallet and inventory reconciliation testing.
- [ ] Complete order, payment and settlement testing.
- [ ] Complete operational and compliance reviews before enabling live-money workflows.

---

## 5. Current local development commands

### Frontend terminal

```powershell
cd E:\MercaLoop\merc-aloop\apps\web
npm.cmd run dev
```

Frontend address:

`http://localhost:3000`

### Backend terminal

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
$env:PORT="4000"
npm.cmd run start:dev
```

Backend address:

`http://localhost:4000`

### Backend build test

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
npm.cmd run build
```

### Dependency security test

```powershell
cd E:\MercaLoop\merc-aloop\apps\api
npm.cmd audit
```

Last verified result:

`found 0 vulnerabilities`

### Supabase connection configuration

API environment file:

`apps/api/.env`

Safe example:

`apps/api/.env.example`

Database configuration:

`apps/api/drizzle.config.ts`

Real database credentials must remain local and untracked.

Never paste connection strings, database passwords or service-role keys into public chats, documentation or GitHub.

---

## 6. Git and repository status

GitHub repository:

`https://github.com/aan95work-svg/merc-aloop.git`

Local project:

`E:\MercaLoop\merc-aloop`

Current branch:

`main`

Last reported Git status before the latest documentation updates showed untracked project files and no initial commit.

**Initial commit and GitHub push remain pending.**

Before committing:

- Review `git status`.
- Confirm `.env` is ignored.
- Confirm `.env.example` contains placeholders only.
- Review root `.gitignore`.
- Inspect all changed and untracked files.
- Confirm `node_modules`, `.next`, `dist` and other generated folders are excluded.
- Do not commit database credentials or private keys.

---

## 7. Current Supabase connection details

Project:

MercaLoop's independent Supabase project.

Region:

`ap-northeast-1` — Tokyo, Japan.

Connection method:

Session Pooler.

Database:

`postgres`.

Successfully verified database role:

`postgres`.

Connection driver:

Postgres.js.

The connection was tested with a read-only SQL query.

The following have not yet been implemented:

- Drizzle database provider inside NestJS.
- TypeScript table definitions.
- SQL migrations.
- Business tables.
- Database triggers or financial functions.
- Authentication integration.
- RLS policies.

**Important:** Successful connectivity does not mean that the platform schema has been created.

No production wallet, order or customer financial records exist yet.

---

## 8. Working agreement

- Keep MercaLoop completely separate from SGECORE and all other projects.
- Use free/open-source development tools wherever practical.
- Do not purchase paid development subscriptions unnecessarily.
- Work one step at a time.
- Provide one command or one logical task at a time when guiding local development.
- Wait for the user's output before proceeding.
- When modifying a file, ask for its complete current contents.
- Return the complete revised file so the user can replace it in one operation.
- Do not ask the user to manually insert individual lines when a complete-file replacement is practical.
- Verify every important change before moving forward.
- Never request or expose private database credentials in chat.
- Keep real secrets in local untracked environment files.
- Review every database migration before execution.
- Confirm the selected Supabase project is MercaLoop before applying migrations.
- Do not configure real-money flows during local development.
- Do not represent reseller prepaid entitlements as physical inventory at a reseller or confirmed partner-store stock.
- Do not automate partner contact, supplier orders, partner stock updates or courier integrations in the MVP.
- Never deduct the original reseller wholesale entitlement cost twice.
- Do not enable production money flows until ledger integrity, permissions, audit and reconciliation tests pass.
- Do not mark features complete until they are implemented and tested.

**Core development principle:** Build MercaLoop as a secure, centrally managed global reseller marketplace with prepaid selling entitlements, independent local partner-store fulfilment, manually managed operational processes and accurately controlled reseller financial accounting.
