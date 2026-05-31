# 🔗 Integrações

## Processadores de Pagamento (7 PSPs)

O Upay opera com **7 adquirentes** via sistema de roteamento com prioridade configurável. Se o PSP de maior prioridade falhar (circuit breaker aberto), o sistema faz failover automático para o próximo ativo. Cada PSP possui seu próprio adaptador com circuit breaker independente.

### Fyntra (PIX)
- PIX instantâneo com geração de QR code
- `FYNTRA_ACTIVE` + prioridade configurável pelo painel admin

### Pagar.me (PIX, Cartão, Boleto, Split)
- PIX instantâneo, cartão de crédito/débito (parcelamento até 12x), boleto bancário
- **Split de pagamentos**: divisão automática entre gateway e merchant usando recipients — cálculo baseado nas taxas cadastradas na plataforma
- Captura manual, cancelamento e estorno de transações
- Sistema de recebedores (recipients) para cada merchant

### Citrex (PIX)
- PIX com certificado público para validação de webhooks (TLS mútuo)

### Ameii (PIX)
- **Processamento assíncrono**: QR codes gerados de forma assíncrona; confirmações via webhook
- **Rastreamento de status**: endpoint `/sync-status` mantém o estado sincronizado
- `externalId` e `ameiiTxId` persistidos mesmo quando o PSP retorna erro — sem perda de rastreabilidade

### Ezzy Banking (PIX)
- PIX com webhook de confirmação; `EZZY_ACTIVE` configurável

### Pagflex (PIX)
- PIX com webhook de confirmação; `PAGFLEX_ACTIVE` configurável

### Axis (PIX)
- PIX com webhook de confirmação; `AXIS_ACTIVE` configurável

---

## Observabilidade

### Sentry
Integrado em **frontend e backend** para rastreamento proativo de exceções em produção:

- Captura automática de erros não tratados e rejeições de Promise
- **Alertas automáticos** para eventos críticos de infraestrutura:
  - `autoReleaseCard` com >50% de falhas por execução
  - Nenhuma rota de adquirente ativa disponível
  - Circuit breaker não encontrado para um PSP
  - Divergência detectada entre saldo no banco e saldo no PSP (job de reconciliação)
- Configurável via `SENTRY_DSN` (backend) e `VITE_SENTRY_DSN` (frontend)
- Zero impacto quando DSN não configurado (falha silenciosa)

---

## Autenticação Social

### Google OAuth
- Login via conta Google sem necessidade de senha Upay
- Fluxo OAuth 2.0 padrão com callback para `/api/auth/google`
- **Gestão de sessões**: visualização e revogação de sessões ativas por dispositivo
  - `DELETE /api/auth/sessions/:id` — revogar sessão específica
  - `DELETE /api/auth/sessions/others` — revogar todas exceto a atual

---

## Cloud e Armazenamento

### Cloudinary
Todas as imagens geradas pelos usuários (fotos de produtos, documentos KYC, imagens de recompensas) são armazenadas via Cloudinary:

- **Otimização automática**: imagens convertidas para WebP no upload
- **Entrega via CDN**: todas as URLs de imagem servidas pela CDN global do Cloudinary
- **Upload server-side**: processado via SDK do Cloudinary no backend

---

## Vendas e Marketing

### Utmify (Rastreamento Avançado de Vendas)
Integração nativa para atribuição de campanhas:

- Captura `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` nas URLs do checkout
- Suporte a click IDs de anúncios: `fbclid` (Meta), `gclid` (Google)
- Parâmetro `src` customizado para atribuição direta
- Endereço IP capturado por transação
- Dados encaminhados à API do Utmify na confirmação do pagamento

### Shopify
Integração para merchants que vendem via lojas Shopify:

- Configurável pela página de Parcerias no dashboard
- Sincronização de produtos e reconciliação de pedidos entre Shopify e Upay

---

## Comunicação

### Email (SMTP + Handlebars)
- Emails transacionais construídos com templates **Handlebars**
- Comprovantes de pagamento em PDF gerados no servidor e anexados automaticamente
- Templates editáveis pelo admin direto no dashboard (sem necessidade de redeploy)

### Webhooks (Saída)
O Upay entrega eventos em tempo real para os endpoints dos merchants:

- Merchant assina tipos de eventos específicos via `POST /api/v1/webhooks/subscriptions`
- Eventos: `transaction.created`, `transaction.completed`, `transaction.failed`, `transaction.updated`, `payment_link.created`, `payment_link.updated`, `balance.updated`
- **Assinatura**: cada entrega inclui `X-Webhook-Signature` (HMAC-SHA256 do payload + secret)
- **Retry automático**: entregas com falha são reprocessadas automaticamente

---

## SDKs para Desenvolvedores

SDKs oficiais disponíveis para as principais linguagens backend:

| SDK | Pacote | Linguagem |
|-----|--------|-----------|
| Node.js / TypeScript | `@upay/upay-js` | JavaScript / TypeScript |
| Python | `upay-python` | Python 3.8+ |
| PHP | `upay/upay-php` | PHP 7.4+ |
| Java | `com.upay:upay-java` | Java 11+ |

Todos os SDKs incluem utilitários de validação de assinatura HMAC para webhooks e tratamento de erros tipado.
