# 🏗️ Upay – Architecture Overview

## Overview

The **Upay** platform follows a **layered service architecture** that separates concerns between transport (controllers/routes), business logic (services), and data access (Prisma ORM). Every domain has its own service module, ensuring the codebase remains maintainable at scale.

---

## Core Components

### 1. Frontend (React 18 + Vite)
- Built with **React 18 + Vite** and **TypeScript** strict mode
- State management via **Zustand** (global auth/user state) and **React Query** (server state, caching, invalidation)
- UI layer: **Tailwind CSS** + **shadcn/ui** components with full dark mode support
- Multi-page routing: dashboard, admin panel, product catalog, affiliate marketplace, checkout
- **Subdomain routing**: `app.[domain].com` for the main application, `checkout.[domain].com` for public checkout pages
- KYC onboarding flows with multi-step forms and document upload

### 2. Backend (Node.js 20 + Express)
- **Express.js** with **TypeScript** — all routes, middleware and controllers fully typed
- **Zod** for runtime request validation (body, params, query) at controller entry points
- **Prisma ORM** with **PostgreSQL** — versioned migrations
- **Redis** for rate-limit counters and caching
- **JWT** for user authentication + **API Keys** for external integrations
- **Idempotency keys** on `POST /transactions`

### 3. Service Layer
Each domain has an isolated service module:

| Service | Responsibility |
|---------|----------------|
| `transactionService` | Payment lifecycle, status transitions, settlement |
| `affiliateService` | Code generation, commission calculation, atomic payout |
| `referralService` | User referral tracking and reward credits |
| `balanceService` | Wallet credits, debits and balance queries |
| `webhookService` | Outbound webhook delivery with retry logic |
| `notificationService` | Email dispatch via Handlebars templates + PDF receipts |
| `ameiiService` | PIX gateway integration with async status sync |

### 4. Affiliate Marketing System (v2.22.0)
- Merchants create **AffiliateProgram** records linked 1:1 to a product
- Affiliates join programs and receive a unique **AffiliateLink** with a human-readable code (`NAME-XXXXXX`)
- Purchases via `?aff=CODE` are linked to the affiliate at checkout
- `affiliateService.processAffiliateCommission()` runs inside a `prisma.$transaction` block:
  1. Calculates commission (PERCENTAGE or FIXED)
  2. Creates `AffiliateCommission` record
  3. Creates `BalanceEntry` credit for the affiliate wallet
  4. Increments conversion counters atomically

### 5. Admin Infrastructure
- **Role-Based Access Control (RBAC)**: `AdminRole` model with 18 granular boolean permission flags
- `requirePermission(flag)` middleware injected per route
- **Kanban board**: `KanbanTask` model with priority enum, assignees and column-based status
- **Email template management**: admin-editable Handlebars templates with PDF receipts

### 6. Security
- Passwords hashed with **bcrypt**
- Webhook signatures: **HMAC-SHA256** with timing-safe comparison
- Rate limiting per `userId` on authenticated routes
- **Circuit breaker** on PSP calls
- XSS sanitization on transaction metadata
- CPF/CNPJ validated with **mod-11 check digit** algorithm
- CORS with explicit origin allowlist

---

## Data Model

```
User ──────────────── Transaction
  │                       │
  ├── AffiliateProgram     └── AffiliateCommission
  │       │
  │   AffiliateLink ──── AffiliateCommission
  │
  ├── BalanceEntry
  ├── Withdrawal
  ├── PaymentLink ──── PaymentLinkProduct ──── Product
  └── KYC
```

- `AffiliateProgram` → unique per `productId`
- `AffiliateLink` → unique per `(programId, affiliateId)`
- `AffiliateCommission` → unique per `transactionId`
- `Transaction.affiliateLinkId` → nullable FK

---

## Deployment

| Layer | Platform |
|-------|----------|
| Frontend | Vercel (Vite build, preview deployments) |
| Backend API | Render (Node.js 20, auto-deploy on push) |
| Database | PostgreSQL managed (Prisma migrations) |
| Cache | Redis managed |
| Images | Cloudinary (WebP, CDN) |
| CI/CD | GitHub Actions (`unit.yml` + `e2e.yml`) |

---

> _Designed and built by Anthony Mengotti._
