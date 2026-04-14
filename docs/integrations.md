# 🔗 Integrações

## Processadores de Pagamento

### Ameii (Gateway PIX)
Integração principal para pagamentos via PIX:

- **Processamento assíncrono**: QR codes PIX gerados de forma assíncrona; o sistema recebe confirmações via webhook
- **Rastreamento de status**: endpoint `/sync-status` mantém o estado da transação sincronizado com o gateway
- **Salvamento de referências externas**: `externalId` e `ameiiTxId` persistidos na transação para rastreabilidade completa

### Pagar.me (Split de Pagamentos)
Utilizado para divisão automática de receita entre gateway e merchants:

- Padrão de adapter modular — Pagar.me opera em seu próprio módulo de serviço
- Orquestração de transações independente do fluxo de pagamento principal
- Dados de liquidação sincronizados de volta ao sistema de saldo do Upay

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
