<!-- 🏦 Upay | Payment Gateway & Digital Banking Infrastructure -->

<div align="center">

<h1>🏦 Upay</h1>
<h3>Payment Gateway & Digital Banking Infrastructure</h3>
<p><em>Gateway de pagamentos completo com banco digital, marketplace de afiliados e painel administrativo avançado.</em></p>

<p>
  <img src="https://img.shields.io/badge/Version-2.22.0-6D28D9?style=for-the-badge" />
  <img src="https://img.shields.io/badge/TypeScript-100%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/PostgreSQL-13+-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/Prisma-ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white" />
</p>

<p>
  <a href="https://github.com/anthonymengottii" target="_blank">
    <img src="https://img.shields.io/badge/Author-Anthony_Mengotti-1C1C1C?style=for-the-badge&logo=github&logoColor=white" />
  </a>
  <a href="mailto:anthonymengottii@gmail.com">
    <img src="https://img.shields.io/badge/Contact-Email-0078D4?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
  <a href="https://www.linkedin.com/in/anthony-mengotti-50026424a/" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
</p>

</div>

---

## Overview

**Upay** is a production-grade, full-stack payment infrastructure platform built from the ground up. It handles the complete financial lifecycle — from PIX and card processing to affiliate commissions, balance management, and regulatory compliance — with a security-first, type-safe architecture.

The system was designed to rival established Brazilian fintech products (Hotmart, Iugu, Pagar.me) in depth and reliability, while remaining fully customizable as a white-label offering.

---

## Core Technical Highlights

### 🔐 Security & Compliance
- **Zero `any` TypeScript policy** — 1,289 warnings eliminated; entire codebase runs with strict types
- **KYC flow** with document upload, admin review and automated status webhooks
- **JWT authentication** with bcrypt, API Key support for external integrations
- **Role-based access control (RBAC)** with granular admin permissions (18 distinct permission flags)
- **Rate limiting** per `userId` on authenticated routes (prevents NAT/proxy penalization)
- **Idempotency keys** on transaction creation (prevents duplicate charges on retry)
- **HMAC-SHA256 webhook signatures** with timing-safe comparison and 60s skew tolerance
- **Input sanitization** against XSS in transaction metadata
- **CPF/CNPJ validation** with mod-11 check digit verification
- **Privilege escalation fix** on admin financial routes (verified at route layer)

### 💳 Payment Processing
- **Multi-method**: PIX (instant), credit/debit card (up to 12x installments), boleto
- **PSP integration**: Ameii gateway with async PIX processing, external ID tracking and status sync
- **Circuit breaker** for PSP failures — fast-fail without blocking request threads
- **Webhook delivery system** with automatic retry on failure
- **Automatic card release** — settled after 30 days (lump-sum) or per installment (split)
- **Split payments** — automatic division between gateway and merchant via Pagar.me
- **Advanced sales tracking** — native UTM capture (`utm_source`, `utm_medium`, `utm_campaign`) and click IDs (`fbclid`, `gclid`) via Utmify integration

### 🤝 Affiliate Marketing System (v2.22.0)
- **Marketplace**: merchants create affiliate programs per product; affiliates browse and request to join
- **Unique trackable links** in format `/pay/{slug}?aff={CODE}` with configurable cookie window (default 30 days)
- **Automatic commissions**: credited to affiliate wallet on payment confirmation — `PERCENTAGE` or `FIXED` types
- **Full audit trail**: per-link click and conversion counters, commission status (`PENDING → PAID → CANCELLED`)
- **Program rules engine**: description, rules, affiliate page and sales page fields per program
- Architecture mirrors Hotmart/Kirvano; commission processing uses atomic `prisma.$transaction` for consistency

### 🏗️ Architecture & Engineering
- **Monorepo** with independent `api/` (Node.js) and `src/` (React) modules
- **Service layer pattern**: domain services (`transactionService`, `affiliateService`, `referralService`) isolated from controllers
- **Prisma ORM** with PostgreSQL — all schema changes via versioned migrations
- **Redis** for caching and rate-limit counters
- **Cloudinary** for image storage (products, KYC documents, reward tiers) with automatic WebP optimization
- **gzip/brotli compression** on all responses
- **Subdomain routing**: `app.[domain].com` (dashboard) vs `checkout.[domain].com` (public checkout)
- **OpenAPI 3.0 spec** maintained in-code and published to public documentation (Mintlify)

### 🛠️ Admin Infrastructure
- **Kanban task board** — full CRUD with priority levels, assignees and drag-and-drop columns (Idea → In Progress → Done)
- **Granular RBAC** — custom admin roles with per-permission toggles (e.g., `approve_kyc`, `view_transactions`, `approve_withdrawals`)
- **Email template management** — Handlebars templates with PDF receipt attachments, dark-mode prevention
- **Audit dashboard** — transaction oversight, KYC queue, withdrawal approvals, advance requests

### 🧪 Test Coverage
- **Unit test suites** (Vitest) for all major controllers: transactions, wallet, payment links, profile, advances, transfers, coupons
- **E2E test pipeline** configured with real PostgreSQL (no mocks — after a production incident where mock/prod divergence masked a broken migration)
- **CI/CD**: GitHub Actions with separate `unit.yml` and `e2e.yml` workflows

---

## Tech Stack

<p align="center">
  <img src="https://skillicons.dev/icons?i=ts,nodejs,express,react,vite,tailwind,prisma,postgres,redis,docker" />
</p>

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui, Zustand, React Query |
| **Backend** | Node.js 20, Express, TypeScript, Zod (validation) |
| **Database** | PostgreSQL 13+, Prisma ORM, Redis |
| **Auth** | JWT, bcrypt, API Keys |
| **Storage** | Cloudinary (images), local (documents) |
| **Payments** | Ameii (PIX), Pagar.me (split), Utmify (tracking) |
| **DevOps** | GitHub Actions, Render (API), Vercel (Frontend) |
| **Docs** | Mintlify, OpenAPI 3.0 |

---

## System Architecture

<p align="center">
  <img src="./docs/system-flow.png" width="85%" alt="Upay System Flow Diagram" />
</p>

The platform follows a **layered service architecture**:

```
┌─────────────────────────────────────────────────────────────────┐
│                        React Frontend                           │
│  Dashboard · Admin Panel · Checkout · Affiliate Marketplace     │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST + Webhooks
┌────────────────────────────▼────────────────────────────────────┐
│                       Express API                               │
│  Auth · Payments · Products · Affiliates · KYC · Admin          │
├─────────────────────────────────────────────────────────────────┤
│  Service Layer                                                  │
│  transactionService · affiliateService · referralService        │
│  balanceService · notificationService · webhookService          │
├──────────────────┬──────────────────────┬───────────────────────┤
│   PostgreSQL     │       Redis           │   Cloudinary / PSPs   │
│   (Prisma ORM)   │   (cache · rate limit)│   (Ameii · Pagar.me)  │
└──────────────────┴──────────────────────┴───────────────────────┘
```

---

## Feature Matrix

| Feature | Status |
|---------|--------|
| PIX instant payments | ✅ Production |
| Credit/debit card (12x) | ✅ Production |
| Boleto bancário | ✅ Production |
| KYC with admin review | ✅ Production |
| Webhook subscriptions | ✅ Production |
| Coupon system (% / fixed) | ✅ Production |
| Product catalog + stock | ✅ Production |
| Referral program (Indique e Ganhe) | ✅ Production |
| **Affiliate marketplace** | ✅ **v2.22.0** |
| Balance / withdrawals / advances | ✅ Production |
| Admin RBAC (18 permissions) | ✅ Production |
| Admin Kanban board | ✅ Production |
| Idempotency keys | ✅ Production |
| Split payments | ✅ Production |
| UTM / sales tracking | ✅ Production |
| Shopify integration | ✅ Production |
| Rate limiting (per userId) | ✅ Production |
| HMAC webhook signatures | ✅ Production |
| Circuit breaker (PSP) | ✅ Production |
| Unit + E2E test suites | ✅ Production |
| OpenAPI 3.0 + public docs | ✅ Production |

---

## Public Documentation

Full API documentation available including:
- Getting started guide and authentication
- REST API reference (OpenAPI 3.0 spec)
- Affiliate system guide
- Webhook events reference
- SDKs for Node.js, Python, PHP and Java

---

## Repository

🔒 **Private repository** — full codebase available to partners and technical reviewers upon request.

📩 [anthonymengottii@gmail.com](mailto:anthonymengottii@gmail.com)  
💼 [LinkedIn — Anthony Mengotti](https://www.linkedin.com/in/anthony-mengotti-50026424a/)
