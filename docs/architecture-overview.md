# 🏗️ Upay – Architecture Overview

## Overview

The **Upay** platform is built as a **production-grade, layered service architecture** that separates concerns cleanly between transport (controllers/routes), business logic (services), and data access (Prisma ORM). Every domain has its own service module, ensuring the codebase remains maintainable at scale.

---

## Core Components

### 1. Frontend (React 18 + Vite)
- Built with **React 18 + Vite** and **TypeScript** with strict mode — zero `any` warnings across the entire codebase
- State management via **Zustand** (global auth/user state) and **React Query** (server state, caching, invalidation)
- UI layer: **Tailwind CSS** + **shadcn/ui** components with full dark mode support
- Multi-page routing: dashboard, admin panel, product catalog, affiliate marketplace, checkout
- **Subdomain routing**: `app.[domain].com` for the main application, `checkout.[domain].com` for public checkout pages
- KYC onboarding flows with multi-step forms and document upload

### 2. Backend (Node.js 20 + Express)
- **Express.js** with **TypeScript** — all routes, middleware and controllers fully typed
- **Zod** for runtime request validation (body, params, query) at controller entry points
- **Prisma ORM** with **PostgreSQL** in production — versioned migrations, no raw SQL for schema changes
- **Redis** for rate-limit counters and caching layers
- **JWT** for user authentication + **API Keys** for external integrations
- **Idempotency keys** on `POST /transactions` — prevents duplicate charges on client retries

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
- Each purchase via `?aff=CODE` links the transaction to the affiliate link
- On payment confirmation, `affiliateService.processAffiliateCommission()` runs inside a `prisma.$transaction` block:
  1. Reads the link + program
  2. Calculates commission (PERCENTAGE or FIXED)
  3. Creates `AffiliateCommission` record (PENDING)
  4. Creates `BalanceEntry` credit for the affiliate wallet
  5. Marks commission as PAID, increments conversion counters
- Designed to be idempotent — commission is tied to `transactionId` with a unique constraint

### 5. Admin Infrastructure
- **Role-Based Access Control (RBAC)**: `AdminRole` model with 18 granular boolean permission flags
- Super admins (`adminRoleId = null`) bypass all permission checks
- `requirePermission(flag)` middleware injected per route
- **Kanban board**: `KanbanTask` model with priority enum, assignees and column-based status
- **Email template management**: admin-editable Handlebars templates, PDF comprovantes auto-attached

### 6. Security Architecture
- Passwords hashed with **bcrypt** (12 rounds)
- Webhook signatures: **HMAC-SHA256** with timing-safe comparison and ±60s clock skew tolerance
- Rate limiting: per `userId` on authenticated routes (not per IP — avoids penalizing users behind NAT)
- **Circuit breaker** on PSP calls — fast-fail after consecutive errors without blocking thread pool
- XSS sanitization on `metadata` field in transaction payloads
- CPF/CNPJ validated with **mod-11 check digit** algorithm
- CORS with explicit origin allowlist (no wildcard subdomain derivation)

---

## Data Model Highlights

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

Key design decisions:
- `AffiliateProgram` has a unique constraint on `productId` (one program per product)
- `AffiliateLink` has a unique constraint on `(programId, affiliateId)` (one link per affiliate per program)
- `AffiliateCommission.transactionId` is unique (one commission per transaction, no double-paying)
- `Transaction.affiliateLinkId` is nullable (most transactions are not affiliate-referred)

---

## Deployment

| Layer | Platform | Notes |
|-------|----------|-------|
| Frontend | Vercel | Vite build, automatic preview deployments |
| Backend API | Render | Node.js 20, environment secrets, auto-deploy on push |
| Database | PostgreSQL (managed) | Prisma migrations on deploy |
| Cache | Redis (managed) | Rate limiting and session data |
| Images | Cloudinary | Auto WebP conversion, CDN delivery |
| CI/CD | GitHub Actions | `unit.yml` (Vitest) + `e2e.yml` (real PostgreSQL) |

---

## Engineering Standards

- **TypeScript strict mode** — zero `any` across frontend and backend (1,289 warnings eliminated)
- **No mock databases in E2E tests** — real PostgreSQL after a production incident where mock divergence masked a broken migration
- **Prisma migrations only** — no ad-hoc SQL for schema changes (drift corrected and locked)
- **Service isolation** — controllers never access Prisma directly; all DB calls go through service layer
- **Atomic operations** — financial state changes use `prisma.$transaction` for consistency

---

> _Designed and built by Anthony Mengotti._
