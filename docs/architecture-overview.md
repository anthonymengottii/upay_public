# 🏗️ Upay – Visão Geral da Arquitetura

## Visão Geral

A plataforma **Upay** segue uma **arquitetura de serviços em camadas** que separa responsabilidades entre transporte (controllers/routes), lógica de negócio (services) e acesso a dados (Prisma ORM). Cada domínio possui seu próprio módulo de serviço, garantindo manutenibilidade em escala.

---

## Componentes Principais

### 1. Frontend (React 19 + Vite)
- Construído com **React 19 + Vite** e **TypeScript** strict mode
- Gerenciamento de estado via **Zustand** (estado global de autenticação) e **React Query** (estado do servidor, cache e invalidação)
- Camada de UI: **Tailwind CSS** + componentes **shadcn/ui** com suporte completo a dark mode
- Roteamento multi-página: dashboard, painel admin, catálogo de produtos, marketplace de afiliados, checkout
- **Roteamento por subdomínio**: `app.[dominio].com` para a aplicação principal, `checkout.[dominio].com` para checkout público
- Fluxo de KYC com formulários multi-etapa e upload de documentos

### 2. Mobile (React Native + Expo) — MVP em desenvolvimento
- **Expo SDK 54** + **React Native 0.81**, TypeScript strict, mesmo backend/API do web
- **NativeWind (Tailwind)** para UI + **React Navigation** (Stack + Bottom Tabs)
- **Zustand** (auth/branding) com persistência via **expo-secure-store**; **TanStack Query** para dados de servidor
- Telas: dashboard (KPIs + gráficos SVG nativos), transações, saldo/saques com OTP, perfil + upload de KYC
- Login social Google via `expo-auth-session`
- Sem impersonação admin (app é exclusivo para o usuário final)
- Build de produção (EAS) e submissão à Play Store ainda pendentes

### 3. Backend (Node.js 20 + Express)
- **Express.js** com **TypeScript** — todas as rotas, middlewares e controllers totalmente tipados
- **Zod** para validação de requisições em tempo de execução (body, params, query) nos controllers
- **Prisma ORM** com **PostgreSQL** — migrações versionadas
- **Redis** para contadores de rate limit e cache
- **JWT** para autenticação de usuários + **API Keys** para integrações externas
- **Idempotency keys** em `POST /transactions`

### 4. Camada de Serviços
Cada domínio possui um módulo de serviço isolado:

| Serviço | Responsabilidade |
|---------|------------------|
| `transactionService` | Ciclo de vida dos pagamentos, transições de status, liquidação |
| `affiliateService` | Geração de códigos, cálculo de comissão, pagamento atômico |
| `referralService` | Rastreamento de indicações e crédito de recompensas |
| `balanceService` | Créditos, débitos e consultas de saldo da carteira |
| `webhookService` | Entrega de webhooks de saída com lógica de retry |
| `notificationService` | Envio de emails via templates Handlebars + comprovantes PDF |
| `subscriptionService` | Cobrança recorrente, gestão de planos, retry automático, métricas MRR/churn |
| `auditService` | Registro imutável de ações — conformidade BACEN 4.658/2018 e LGPD |
| `transactionLimitService` | Resolve limite máximo de transação: override por empresa ou fallback pro teto global da plataforma |
| `marlimService` / `pagarmeService` / etc. | Adaptadores por PSP com circuit breaker integrado — Marlim como adquirente principal (PIX, cartão, split de assinatura) |

### 5. Sistema de Marketing de Afiliados (v2.22.0)
- Merchants criam registros **AffiliateProgram** vinculados 1:1 a um produto
- Afiliados ingressam em programas e recebem um **AffiliateLink** único com código legível (`NOME-XXXXXX`)
- Compras via `?aff=CODIGO` são vinculadas ao afiliado no checkout
- `affiliateService.processAffiliateCommission()` executa dentro de um bloco `prisma.$transaction`:
  1. Calcula a comissão (PERCENTAGE ou FIXED)
  2. Cria o registro `AffiliateCommission`
  3. Cria `BalanceEntry` de crédito na carteira do afiliado
  4. Incrementa contadores de conversão atomicamente

### 6. Painel Administrativo
- **Controle de Acesso Baseado em Papéis (RBAC)**: model `AdminRole` com 18 flags booleanas de permissão granular
- Middleware `requirePermission(flag)` injetado por rota
- **Quadro Kanban**: model `KanbanTask` com enum de prioridade, responsáveis e status por coluna
- **Gestão de templates de email**: templates editáveis pelo admin com comprovantes PDF

### 7. Segurança e Conformidade
- Senhas com hash via **bcrypt**; **MFA TOTP** (Google Authenticator) com QR code
- Assinaturas de webhook: **HMAC-SHA256** com comparação timing-safe; mínimo 32 bytes no secret
- Rate limiting por `userId` nas rotas autenticadas; duplo limite por email + IP no reset de senha — contadores atômicos via Redis INCR (sem race condition sob concorrência)
- **OTP em saques e transferências**: código de 6 dígitos hash SHA256, TTL 300s no Redis, máximo 5 tentativas, comparação timing-safe
- **Circuit breaker** por PSP; alerta Sentry quando nenhuma rota ativa
- Sanitização XSS nos metadados de transações e URLs de branding (white-label)
- Validação de CPF/CNPJ com algoritmo **mod-11**
- CORS com lista explícita de origens permitidas; roteamento por subdomínio (`app.*` / `checkout.*`)
- `expectedCode`/`receivedCode` removidos dos logs TOTP — sem PII em logs de autenticação
- **AuditLog imutável**: sem UPDATE/DELETE, retenção 1825 dias (**BACEN 4.658/2018**)
- **LGPD — art. 18**: `deleteMyData` anonimiza `AuditLog`, `Session` e `Transaction` completamente

---

## Modelo de Dados

```
User ──────────────── Transaction ──── AffiliateCommission
  │                       │
  ├── AffiliateProgram     └── CartAbandonment
  │       │
  │   AffiliateLink ──── AffiliateCommission
  │
  ├── BalanceEntry
  ├── Withdrawal
  ├── SubscriptionPlan ──── Subscription
  ├── PaymentLink ──── PaymentLinkProduct ──── Product
  ├── AuditLog (imutável)
  └── KYC
```

- `AffiliateProgram` → único por `productId`
- `AffiliateLink` → único por `(programId, affiliateId)`
- `AffiliateCommission` → único por `transactionId`
- `Transaction.affiliateLinkId` → FK nullable
- `Subscription` → vinculada a `SubscriptionPlan` e ao `User` (merchant)
- `CartAbandonment` → vinculada a `PaymentLink`, marcada como recuperada na conclusão do pagamento
- `AuditLog` → sem FK de update/delete; retenção mínima 1825 dias

---

## Deploy

| Camada | Plataforma |
|--------|------------|
| Frontend | Vercel (build Vite, deploys de preview) |
| Mobile | EAS Build (Android/.aab) → Play Store — pendente publicação |
| Backend API | Render (Node.js 20, auto-deploy no push) |
| Banco de Dados | PostgreSQL 16+ gerenciado (migrações Prisma 7) |
| Cache | Redis 7 gerenciado |
| Imagens | Cloudinary (WebP, CDN) |
| Observabilidade | Sentry (frontend + backend) |
| CI/CD | GitHub Actions (`unit.yml` + `e2e.yml`) |
| Self-hosted | Docker Compose (postgres:16 + redis:7 + api + nginx frontend) |

---

> _Desenvolvido por Anthony Mengotti._
