<!-- 🏦 Upay | Sistema de Pagamentos White-Label -->

<div align="center">

<h1>🏦 Upay</h1>
<h3>Sistema de Pagamentos White-Label Full-Stack</h3>
<p><em>Plataforma de pagamentos white-label completa — marketplace de afiliados, gestão financeira e painel administrativo avançado. "Upay" é o nome de referência do projeto; toda a identidade visual (logo, cores, domínio, SEO) é configurável por instância.</em></p>

<p>
  <img src="https://img.shields.io/badge/Versão-2.24.0-6D28D9?style=for-the-badge" />
  <img src="https://img.shields.io/badge/TypeScript-100%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/PostgreSQL-16+-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/Prisma-7_ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white" />
</p>

<p>
  <a href="https://github.com/anthonymengottii" target="_blank">
    <img src="https://img.shields.io/badge/Autor-Anthony_Mengotti-1C1C1C?style=for-the-badge&logo=github&logoColor=white" />
  </a>
  <a href="mailto:anthonymengottii@gmail.com">
    <img src="https://img.shields.io/badge/Contato-Email-0078D4?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
  <a href="https://www.linkedin.com/in/anthony-mengotti-50026424a/" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
</p>

</div>

---

## Visão Geral

**Upay** é um **sistema de pagamentos white-label** full-stack construído do zero — apesar do nome próprio, a plataforma foi projetada desde o início para ser rebrandada e operada sob qualquer marca. Gerencia o ciclo completo de pagamentos — do PIX e cartão ao marketplace de afiliados, gestão de saldo e conformidade regulatória — com arquitetura orientada a segurança e TypeScript strict em todo o projeto.

**White-label na prática**: cada instância configura logo, paleta de cores, domínio próprio (roteamento por subdomínio `app.*` / `checkout.*`), metadados de SEO e templates de email — sem alteração de código. "Upay" é apenas o nome de referência da instância demonstrativa.

Desenvolvido para competir em profundidade e confiabilidade com as principais plataformas de pagamento brasileiras (Iugu, Pagar.me, Hotmart).

---

## Destaques Técnicos

### 🔐 Segurança e Conformidade
- **TypeScript strict mode** em todo o frontend e backend
- **Fluxo de KYC** com upload de documentos, revisão administrativa e webhooks de status
- **Autenticação JWT** com bcrypt, além de suporte a API Key para integrações externas
- **MFA com Google Authenticator (TOTP)** — ativação, validação e desativação com QR code
- **Login social Google (OAuth)** — autenticação via conta Google; gestão de sessões com revogação por dispositivo
- **Controle de acesso baseado em papéis (RBAC)** com 30+ flags de permissão granular
- **Rate limiting** por `userId` nas rotas autenticadas; duplo limite por email e IP no reset de senha
- **Idempotency keys** na criação de transações e cobranças de assinatura
- **Assinaturas de webhook HMAC-SHA256** com comparação timing-safe; comprimento mínimo 32 bytes enforçado
- **Validação de CPF/CNPJ** com algoritmo mod-11
- **Sanitização XSS** nos metadados de transações e URLs de branding (white-label)
- **AuditLog imutável**: registros sem UPDATE/DELETE, retenção de 1825 dias (5 anos, conforme **BACEN 4.658/2018**)
- **LGPD — direito ao erasure**: `deleteMyData` anonimiza `AuditLog`, `Session` e `Transaction` completamente
- **Sentry** integrado em frontend e backend — rastreamento de exceções, alertas de jobs e circuit breakers em produção
- **Job de reconciliação**: verifica divergência de saldo DB vs PSP com alerta automático via Sentry

### 💳 Processamento de Pagamentos
- **Multi-método**: PIX (instantâneo), cartão de crédito/débito (até 12x), boleto
- **Multi-PSP com prioridade configurável**: 7 adquirentes (Marlim, Pagar.me, Fyntra, Citrex, Ameii, PixBR.dev, Pague.dev) — failover automático e roteamento por prioridade
- **Marlim** como adquirente principal — PIX, cartão com 3DS e split de assinatura recorrente (retry e troca de cartão inclusos)
- **Celcoin** como rail dedicado de payout — processa saques e transferências separado do fluxo de cobrança
- **Circuit breaker** em cada PSP para isolamento de falhas; alerta Sentry quando nenhuma rota ativa
- **Entrega de webhooks** com retry automático e assinatura HMAC-SHA256
- **Liberação automática de cartão** — 30 dias à vista ou por parcela (job background configurável)
- **Split de pagamentos** — divisão automática entre gateway e merchant por taxa cadastrada
- **Limite de transação configurável** — teto global da plataforma com override por empresa/usuário (bloqueio explícito quando zerado)
- **OTP em saques e transferências** — código de 6 dígitos, hash SHA256, TTL 300s no Redis, 5 tentativas, comparação timing-safe
- **Rastreamento avançado de vendas** — captura de UTMs (`utm_source`, `utm_medium`, `utm_campaign`) e click IDs (`fbclid`, `gclid`) via Utmify

### 📋 Assinaturas Recorrentes e Cart Abandonment
- **Planos configuráveis**: nome, preço, intervalo (mensal, anual, etc.), período de trial e limite de assinantes
- **Checkout público sem autenticação**: clientes se inscrevem via `POST /api/subscriptions/checkout`
- **Métricas MRR/churn**: dashboard com MRR, churn rate e assinantes ativos por merchant
- **Retry automático**: cobranças falhas reprocessadas automaticamente
- **Cart abandonment**: captura de clientes que iniciaram mas não finalizaram o checkout — marcação de recuperação quando retornam
- **Painel admin**: listagem filtrada por link e período com análise de conversão

### 🤝 Sistema de Marketing de Afiliados
- **Marketplace**: merchants criam programas de afiliados por produto; afiliados navegam e solicitam afiliação
- **Links rastreáveis únicos** — `/pay/{slug}?aff={CODIGO}` com janela de cookie configurável
- **Comissões automáticas**: creditadas na carteira do afiliado após confirmação do pagamento — tipos `PERCENTAGE` ou `FIXED`
- **Métricas por link**: contadores de cliques, conversões e status de comissão
- **Configuração de programa**: descrição, regras, página de afiliados e página de vendas
- Processamento de comissão via `prisma.$transaction` atômico

### 🏗️ Arquitetura e Engenharia
- **Monorepo** com módulos independentes `api/` (Node.js) e `src/` (React)
- **Camada de serviços**: serviços de domínio (`transactionService`, `affiliateService`, `referralService`, `webhookService`) isolados dos controllers
- **Prisma ORM** com migrações versionadas no PostgreSQL
- **Redis** para cache e contadores de rate limit
- **Cloudinary** para armazenamento de imagens com otimização automática para WebP
- **Compressão gzip/brotli** em todas as respostas
- **Roteamento por subdomínio**: `app.[dominio].com` (dashboard) vs `checkout.[dominio].com` (checkout público)
- **Especificação OpenAPI 3.0** mantida no código e publicada via Mintlify

### 🛠️ Painel Administrativo
- **Quadro Kanban** — CRUD completo com níveis de prioridade, responsáveis e colunas de status
- **RBAC granular** — papéis personalizados com toggles de permissão (`approve_kyc`, `view_transactions`, `approve_withdrawals`, entre outros)
- **Gestão de templates de email** — templates Handlebars com comprovantes PDF anexados
- **Dashboard de auditoria** — fila de KYC, aprovações de saque, solicitações de antecipação, gestão de transações

### 📱 App Mobile (Android)
- **React Native + Expo SDK 54** — companion app do dashboard web, mesmo backend/API
- **NativeWind (Tailwind)** + React Navigation (Stack + Bottom Tabs) + Zustand + TanStack Query
- Telas: dashboard (KPIs + gráficos SVG), transações, saldo/saques com **OTP de confirmação**, perfil com upload de KYC
- **expo-secure-store** para token de auth; login social Google
- **Status: MVP em desenvolvimento** — build de produção (EAS) e publicação na Play Store ainda pendentes

### 🧪 Testes e CI/CD
- **340+ suítes de testes** — Jest (backend) + Vitest (frontend) — cobrindo controllers, serviços, middlewares, hooks, screens e stores
- **~7.000+ test cases** unitários passando; 19 fluxos E2E com **PostgreSQL real** (sem mocks de banco)
- **GitHub Actions** com workflows separados `unit.yml` (Jest + Vitest + ESLint) e `e2e.yml`
- **ESLint 0 warnings** — backend e frontend com `--max-warnings 0`; step de lint no CI

---

## Stack Tecnológico

<p align="center">
  <img src="https://skillicons.dev/icons?i=ts,nodejs,express,react,vite,tailwind,prisma,postgres,redis,docker" />
</p>

| Camada | Tecnologias |
|--------|-------------|
| **Frontend** | React 19, TypeScript, Vite 7, Tailwind CSS, shadcn/ui, Zustand, React Query |
| **Mobile** | React Native, Expo SDK 54, NativeWind, React Navigation, Zustand, TanStack Query |
| **Backend** | Node.js 20, Express, TypeScript, Zod |
| **Banco de Dados** | PostgreSQL 16+, Prisma 7 ORM, Redis 7 |
| **Autenticação** | JWT, bcrypt, API Keys, Google OAuth, TOTP (MFA) |
| **Armazenamento** | Cloudinary (WebP, CDN) |
| **Pagamentos** | Marlim, Pagar.me, Fyntra, Citrex, Ameii, PixBR.dev, Pague.dev + Celcoin (payout) + Utmify (rastreamento) |
| **Observabilidade** | Sentry (frontend + backend), circuit breaker, job de reconciliação |
| **DevOps** | Docker, GitHub Actions, Render (API), Vercel (Frontend) |
| **Docs** | Mintlify, OpenAPI 3.0, Swagger UI |

---

## Arquitetura do Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                       Frontend React                            │
│  Dashboard · Painel Admin · Checkout · Marketplace de Afiliados │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST + Webhooks
┌────────────────────────────▼────────────────────────────────────┐
│                       Express API                               │
│  Auth · Pagamentos · Produtos · Afiliados · KYC · Admin         │
├─────────────────────────────────────────────────────────────────┤
│  Camada de Serviços                                             │
│  transactionService · affiliateService · referralService        │
│  balanceService · notificationService · webhookService          │
├──────────────────┬──────────────────────┬───────────────────────┤
│   PostgreSQL     │       Redis           │   Cloudinary / PSPs            │
│   (Prisma ORM)   │  (cache · rate limit) │   Marlim · Pagar.me · +5 PSPs  │
└──────────────────┴──────────────────────┴───────────────────────┘
```

---

## Funcionalidades

| Funcionalidade | Status |
|----------------|--------|
| Pagamento via PIX instantâneo | ✅ Produção |
| Cartão de crédito/débito (12x) | ✅ Produção |
| Boleto bancário | ✅ Produção |
| Multi-PSP com prioridade configurável (7 adquirentes: Marlim, Pagar.me, Fyntra, Citrex, Ameii, PixBR.dev, Pague.dev) | ✅ Produção |
| Limite de transação configurável (global + override por empresa) | ✅ Produção |
| OTP em saques e transferências (SHA256, Redis TTL, rate-limited) | ✅ Produção |
| KYC com revisão administrativa | ✅ Produção |
| MFA (TOTP Google Authenticator) | ✅ Produção |
| Login social Google (OAuth) + gestão de sessões | ✅ Produção |
| Assinaturas de webhook HMAC-SHA256 | ✅ Produção |
| Sistema de cupons (% / fixo) | ✅ Produção |
| Catálogo de produtos + controle de estoque | ✅ Produção |
| Programa de indicações (recompensa fixa + comissão) | ✅ Produção |
| Marketplace de afiliados | ✅ Produção |
| **Assinaturas recorrentes** (MRR/churn, retry, trial) | ✅ Produção |
| **Cart abandonment** (captura + recuperação) | ✅ Produção |
| Saldo / saques / antecipações | ✅ Produção |
| RBAC administrativo (30+ permissões granulares) | ✅ Produção |
| Quadro Kanban administrativo | ✅ Produção |
| Idempotency keys | ✅ Produção |
| Split de pagamentos | ✅ Produção |
| Rastreamento UTM / vendas (Utmify) | ✅ Produção |
| Integração com Shopify | ✅ Produção |
| Rate limiting distribuído (Redis) | ✅ Produção |
| Circuit breaker por PSP | ✅ Produção |
| **AuditLog imutável** (BACEN 4.658/2018 — 5 anos) | ✅ Produção |
| **LGPD — direito ao erasure** (`deleteMyData`) | ✅ Produção |
| Sentry (observabilidade + alertas automáticos) | ✅ Produção |
| White-label branding completo (logo, cor, SEO) | ✅ Produção |
| 340+ suítes / ~7.000+ TCs (unitários + E2E PostgreSQL) | ✅ Produção |
| OpenAPI 3.0 + Swagger UI | ✅ Produção |
| App mobile Android (React Native + Expo) | 🚧 MVP em desenvolvimento |

---

## Documentação Pública

Documentação completa da API disponível em **[docs.upaybr.com](https://docs.upaybr.com)**, incluindo:
- Guia de início rápido e autenticação
- Referência da API REST (especificação OpenAPI 3.0)
- Guia do sistema de afiliados
- Referência de eventos de webhook
- SDKs para Node.js, Python, PHP e Java

---

## Repositório

🔒 **Repositório privado** — código completo disponível para parceiros e revisores técnicos mediante solicitação.

📩 [anthonymengottii@gmail.com](mailto:anthonymengottii@gmail.com)  
💼 [LinkedIn — Anthony Mengotti](https://www.linkedin.com/in/anthony-mengotti-50026424a/)
