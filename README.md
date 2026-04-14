<!-- 🏦 Upay | Payment Gateway -->

<div align="center">

<h1>🏦 Upay</h1>
<h3>Full-Stack Payment Gateway</h3>
<p><em>Gateway de pagamentos completo com marketplace de afiliados, gestão financeira e painel administrativo avançado.</em></p>

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

**Upay** is a production-grade, full-stack payment gateway built from the ground up. It handles the complete payment lifecycle — from PIX and card processing to affiliate commissions and balance management — with a security-first, type-safe architecture.

Designed to rival established Brazilian payment platforms (Iugu, Pagar.me, Hotmart) in depth and reliability, while remaining fully customizable as a white-label offering.

---

## Core Technical Highlights

### 🔐 Security & Compliance
- **TypeScript strict mode** across the entire frontend and backend codebase
- **KYC flow** with document upload, admin review and automated status webhooks
- **JWT authentication** with bcrypt, plus API Key support for external integrations
- **Role-based access control (RBAC)** with 18 granular admin permission flags
- **Rate limiting** per `userId` on authenticated routes
- **Idempotency keys** on transaction creation
- **HMAC-SHA256 webhook signatures** with timing-safe comparison
- **CPF/CNPJ validation** with mod-11 check digit verification
- **XSS sanitization** on transaction metadata

### 💳 Payment Processing
- **Multi-method**: PIX (instant), credit/debit card (up to 12x installments), boleto
- **PSP integration**: Ameii gateway with async PIX processing and status sync
- **Circuit breaker** on PSP calls for fast failure isolation
- **Webhook delivery** with automatic retry
- **Automatic card release** — 30 days lump-sum or per installment
- **Split payments** via Pagar.me
- **Advanced sales tracking** — UTM capture (`utm_source`, `utm_medium`, `utm_campaign`) and click IDs (`fbclid`, `gclid`) via Utmify

### 🤝 Affiliate Marketing System (v2.22.0)
- **Marketplace**: merchants create affiliate programs per product; affiliates browse and request to join
- **Unique trackable links** — `/pay/{slug}?aff={CODE}` with configurable cookie window
- **Automatic commissions**: credited to affiliate wallet on payment confirmation — `PERCENTAGE` or `FIXED` types
- **Per-link metrics**: click counters, conversion counters, commission status tracking
- **Program configuration**: description, rules, affiliate page and sales page per program
- Commission processing via atomic `prisma.$transaction`

### 🏗️ Architecture & Engineering
- **Monorepo** with independent `api/` (Node.js) and `src/` (React) modules
- **Service layer**: domain services (`transactionService`, `affiliateService`, `referralService`, `webhookService`) isolated from controllers
- **Prisma ORM** with versioned PostgreSQL migrations
- **Redis** for caching and rate-limit counters
- **Cloudinary** for image storage with automatic WebP optimization
- **gzip/brotli compression** on all responses
- **Subdomain routing**: `app.[domain].com` (dashboard) vs `checkout.[domain].com` (public checkout)
- **OpenAPI 3.0 spec** maintained in-code, published via Mintlify

### 🛠️ Admin Infrastructure
- **Kanban task board** — full CRUD with priority levels, assignees and status columns
- **Granular RBAC** — custom admin roles with per-permission toggles (`approve_kyc`, `view_transactions`, `approve_withdrawals`, and more)
- **Email template management** — Handlebars templates with PDF receipt attachments
- **Audit dashboard** — KYC queue, withdrawal approvals, advance requests, transaction oversight

### 🧪 Tests & CI/CD
- **Unit test suites** (Vitest) covering transactions, wallet, payment links, profile, advances, transfers and coupons
- **E2E test pipeline** with real PostgreSQL
- **GitHub Actions** with separate `unit.yml` and `e2e.yml` workflows

---

## Tech Stack

<p align="center">
  <img src="https://skillicons.dev/icons?i=ts,nodejs,express,react,vite,tailwind,prisma,postgres,redis,docker" />
</p>

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui, Zustand, React Query |
| **Backend** | Node.js 20, Express, TypeScript, Zod |
| **Database** | PostgreSQL 13+, Prisma ORM, Redis |
| **Auth** | JWT, bcrypt, API Keys |
| **Storage** | Cloudinary |
| **Payments** | Ameii (PIX), Pagar.me (split), Utmify (tracking) |
| **DevOps** | GitHub Actions, Render (API), Vercel (Frontend) |
| **Docs** | Mintlify, OpenAPI 3.0 |

---

## System Architecture

<p align="center">
  <img src="./docs/system-flow.png" width="85%" alt="Upay System Flow Diagram" />
</p>

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
| Referral program | ✅ Production |
| **Affiliate marketplace** | ✅ **v2.22.0** |
| Balance / withdrawals / advances | ✅ Production |
| Admin RBAC (18 permissions) | ✅ Production |
| Admin Kanban board | ✅ Production |
| Idempotency keys | ✅ Production |
| Split payments | ✅ Production |
| UTM / sales tracking | ✅ Production |
| Shopify integration | ✅ Production |
| Rate limiting | ✅ Production |
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
