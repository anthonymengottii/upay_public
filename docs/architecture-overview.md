# 🏗️ Upay – Visão Geral da Arquitetura

## Visão Geral

A plataforma **Upay** segue uma **arquitetura de serviços em camadas** que separa responsabilidades entre transporte (controllers/routes), lógica de negócio (services) e acesso a dados (Prisma ORM). Cada domínio possui seu próprio módulo de serviço, garantindo manutenibilidade em escala.

---

## Componentes Principais

### 1. Frontend (React 18 + Vite)
- Construído com **React 18 + Vite** e **TypeScript** strict mode
- Gerenciamento de estado via **Zustand** (estado global de autenticação) e **React Query** (estado do servidor, cache e invalidação)
- Camada de UI: **Tailwind CSS** + componentes **shadcn/ui** com suporte completo a dark mode
- Roteamento multi-página: dashboard, painel admin, catálogo de produtos, marketplace de afiliados, checkout
- **Roteamento por subdomínio**: `app.[dominio].com` para a aplicação principal, `checkout.[dominio].com` para checkout público
- Fluxo de KYC com formulários multi-etapa e upload de documentos

### 2. Backend (Node.js 20 + Express)
- **Express.js** com **TypeScript** — todas as rotas, middlewares e controllers totalmente tipados
- **Zod** para validação de requisições em tempo de execução (body, params, query) nos controllers
- **Prisma ORM** com **PostgreSQL** — migrações versionadas
- **Redis** para contadores de rate limit e cache
- **JWT** para autenticação de usuários + **API Keys** para integrações externas
- **Idempotency keys** em `POST /transactions`

### 3. Camada de Serviços
Cada domínio possui um módulo de serviço isolado:

| Serviço | Responsabilidade |
|---------|------------------|
| `transactionService` | Ciclo de vida dos pagamentos, transições de status, liquidação |
| `affiliateService` | Geração de códigos, cálculo de comissão, pagamento atômico |
| `referralService` | Rastreamento de indicações e crédito de recompensas |
| `balanceService` | Créditos, débitos e consultas de saldo da carteira |
| `webhookService` | Entrega de webhooks de saída com lógica de retry |
| `notificationService` | Envio de emails via templates Handlebars + comprovantes PDF |
| `ameiiService` | Integração com gateway PIX com sincronização assíncrona de status |

### 4. Sistema de Marketing de Afiliados (v2.22.0)
- Merchants criam registros **AffiliateProgram** vinculados 1:1 a um produto
- Afiliados ingressam em programas e recebem um **AffiliateLink** único com código legível (`NOME-XXXXXX`)
- Compras via `?aff=CODIGO` são vinculadas ao afiliado no checkout
- `affiliateService.processAffiliateCommission()` executa dentro de um bloco `prisma.$transaction`:
  1. Calcula a comissão (PERCENTAGE ou FIXED)
  2. Cria o registro `AffiliateCommission`
  3. Cria `BalanceEntry` de crédito na carteira do afiliado
  4. Incrementa contadores de conversão atomicamente

### 5. Painel Administrativo
- **Controle de Acesso Baseado em Papéis (RBAC)**: model `AdminRole` com 18 flags booleanas de permissão granular
- Middleware `requirePermission(flag)` injetado por rota
- **Quadro Kanban**: model `KanbanTask` com enum de prioridade, responsáveis e status por coluna
- **Gestão de templates de email**: templates editáveis pelo admin com comprovantes PDF

### 6. Segurança
- Senhas com hash via **bcrypt**
- Assinaturas de webhook: **HMAC-SHA256** com comparação timing-safe
- Rate limiting por `userId` nas rotas autenticadas
- **Circuit breaker** nas chamadas ao PSP
- Sanitização XSS nos metadados de transações
- Validação de CPF/CNPJ com algoritmo **mod-11**
- CORS com lista explícita de origens permitidas

---

## Modelo de Dados

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

- `AffiliateProgram` → único por `productId`
- `AffiliateLink` → único por `(programId, affiliateId)`
- `AffiliateCommission` → único por `transactionId`
- `Transaction.affiliateLinkId` → FK nullable

---

## Deploy

| Camada | Plataforma |
|--------|------------|
| Frontend | Vercel (build Vite, deploys de preview) |
| Backend API | Render (Node.js 20, auto-deploy no push) |
| Banco de Dados | PostgreSQL gerenciado (migrações Prisma) |
| Cache | Redis gerenciado |
| Imagens | Cloudinary (WebP, CDN) |
| CI/CD | GitHub Actions (`unit.yml` + `e2e.yml`) |

---

> _Desenvolvido por Anthony Mengotti._
