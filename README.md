<!-- 🏦 Upay | Gateway de Pagamentos -->

<div align="center">

<h1>🏦 Upay</h1>
<h3>Gateway de Pagamentos Full-Stack</h3>
<p><em>Gateway de pagamentos completo com marketplace de afiliados, gestão financeira e painel administrativo avançado.</em></p>

<p>
  <img src="https://img.shields.io/badge/Versão-2.22.0-6D28D9?style=for-the-badge" />
  <img src="https://img.shields.io/badge/TypeScript-100%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/PostgreSQL-13+-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/Prisma-ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white" />
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

**Upay** é um gateway de pagamentos full-stack construído do zero. Gerencia o ciclo completo de pagamentos — do PIX e cartão ao marketplace de afiliados, gestão de saldo e conformidade regulatória — com arquitetura orientada a segurança e TypeScript strict em todo o projeto.

Desenvolvido para competir em profundidade e confiabilidade com as principais plataformas de pagamento brasileiras (Iugu, Pagar.me, Hotmart), mantendo flexibilidade para uso como solução white-label.

---

## Destaques Técnicos

### 🔐 Segurança e Conformidade
- **TypeScript strict mode** em todo o frontend e backend
- **Fluxo de KYC** com upload de documentos, revisão administrativa e webhooks de status
- **Autenticação JWT** com bcrypt, além de suporte a API Key para integrações externas
- **Controle de acesso baseado em papéis (RBAC)** com 18 flags de permissão granular
- **Rate limiting** por `userId` nas rotas autenticadas
- **Idempotency keys** na criação de transações
- **Assinaturas de webhook HMAC-SHA256** com comparação timing-safe
- **Validação de CPF/CNPJ** com algoritmo mod-11
- **Sanitização XSS** nos metadados de transações

### 💳 Processamento de Pagamentos
- **Multi-método**: PIX (instantâneo), cartão de crédito/débito (até 12x), boleto
- **Integração PSP**: gateway Ameii com processamento PIX assíncrono e sincronização de status
- **Circuit breaker** nas chamadas ao PSP para isolamento de falhas
- **Entrega de webhooks** com retry automático
- **Liberação automática de cartão** — 30 dias à vista ou por parcela
- **Split de pagamentos** via Pagar.me
- **Rastreamento avançado de vendas** — captura de UTMs (`utm_source`, `utm_medium`, `utm_campaign`) e click IDs (`fbclid`, `gclid`) via Utmify

### 🤝 Sistema de Marketing de Afiliados (v2.22.0)
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

### 🧪 Testes e CI/CD
- **Suítes de testes unitários** (Vitest) cobrindo transações, carteira, links de pagamento, perfil, antecipações, transferências e cupons
- **Pipeline E2E** com PostgreSQL real
- **GitHub Actions** com workflows separados `unit.yml` e `e2e.yml`

---

## Stack Tecnológico

<p align="center">
  <img src="https://skillicons.dev/icons?i=ts,nodejs,express,react,vite,tailwind,prisma,postgres,redis,docker" />
</p>

| Camada | Tecnologias |
|--------|-------------|
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui, Zustand, React Query |
| **Backend** | Node.js 20, Express, TypeScript, Zod |
| **Banco de Dados** | PostgreSQL 13+, Prisma ORM, Redis |
| **Autenticação** | JWT, bcrypt, API Keys |
| **Armazenamento** | Cloudinary |
| **Pagamentos** | Ameii (PIX), Pagar.me (split), Utmify (rastreamento) |
| **DevOps** | GitHub Actions, Render (API), Vercel (Frontend) |
| **Docs** | Mintlify, OpenAPI 3.0 |

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
│   PostgreSQL     │       Redis           │   Cloudinary / PSPs   │
│   (Prisma ORM)   │  (cache · rate limit) │   (Ameii · Pagar.me)  │
└──────────────────┴──────────────────────┴───────────────────────┘
```

---

## Funcionalidades

| Funcionalidade | Status |
|----------------|--------|
| Pagamento via PIX instantâneo | ✅ Produção |
| Cartão de crédito/débito (12x) | ✅ Produção |
| Boleto bancário | ✅ Produção |
| KYC com revisão administrativa | ✅ Produção |
| Assinaturas de webhook | ✅ Produção |
| Sistema de cupons (% / fixo) | ✅ Produção |
| Catálogo de produtos + estoque | ✅ Produção |
| Programa de indicações | ✅ Produção |
| **Marketplace de afiliados** | ✅ **v2.22.0** |
| Saldo / saques / antecipações | ✅ Produção |
| RBAC administrativo (18 permissões) | ✅ Produção |
| Quadro Kanban administrativo | ✅ Produção |
| Idempotency keys | ✅ Produção |
| Split de pagamentos | ✅ Produção |
| Rastreamento UTM / vendas | ✅ Produção |
| Integração com Shopify | ✅ Produção |
| Rate limiting | ✅ Produção |
| Assinaturas HMAC nos webhooks | ✅ Produção |
| Circuit breaker (PSP) | ✅ Produção |
| Suítes de testes unitários e E2E | ✅ Produção |
| OpenAPI 3.0 + documentação pública | ✅ Produção |

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
