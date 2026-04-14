# 🔗 Integrations Overview

## Payment Processors

### Ameii (PIX Gateway)
The primary PIX integration. Key design decisions made during implementation:

- **Async processing**: PIX QR codes are generated asynchronously; the system polls or receives webhooks for confirmation
- **Fault tolerance**: `processPixPaymentAsync` saves `externalId` and `ameiiTxId` even when the gateway returns an error — preventing lost transaction references
- **Status sync**: `/sync-status` endpoint falls back to `paymentDetails.ameiiTxId` when `displayId` is null (handles edge cases in Ameii's response format)
- **Graceful degradation**: returns `pix_copia_cola: null` instead of throwing when QR code is unavailable, allowing the UI to handle the partial state

### Pagar.me (Split Payments)
Used for automatic revenue split between gateway and merchants:

- Modular adapter pattern — Pagar.me runs under its own service module
- Handles transaction orchestration independently from the core payment flow
- Settlement data synced back to the Upay balance system

---

## Cloud & Storage

### Cloudinary
All user-generated images (product photos, KYC documents, reward tier images) are stored via Cloudinary:

- **Automatic optimization**: images converted to WebP with quality tuning on upload
- **CDN delivery**: all image URLs served from Cloudinary's global CDN
- **Migration scripts**: idempotent scripts to move existing local images to cloud storage
- Upload handled server-side via Cloudinary SDK — client never receives direct upload credentials

---

## Sales & Marketing

### Utmify (Advanced Sales Tracking)
Native integration for campaign attribution:

- Captures `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` from checkout URLs
- Supports ad click IDs: `fbclid` (Meta), `gclid` (Google)
- Custom `src` parameter for direct attribution
- IP address captured per transaction for geolocation reporting
- Data forwarded to Utmify's API on payment confirmation

### Shopify
Integration for merchants who sell via Shopify storefronts:

- Configurable from the Partnerships page in the dashboard
- Enables product sync and order reconciliation between Shopify and Upay

---

## Communication

### Email (SMTP + Handlebars)
- Transactional emails built with **Handlebars** templates
- PDF payment receipts generated server-side and attached automatically
- Templates are admin-editable from the dashboard (no redeploy required)
- Dark mode prevention applied to all email HTML (consistent rendering across clients)

### Webhooks (Outbound)
Upay delivers real-time events to merchant endpoints:

- Merchant subscribes to specific event types via `POST /api/v1/webhooks/subscriptions`
- Events: `transaction.created`, `transaction.completed`, `transaction.failed`, `transaction.updated`, `payment_link.created`, `payment_link.updated`, `balance.updated`
- **Signature**: each delivery includes `X-Webhook-Signature` (HMAC-SHA256 of payload + secret)
- **Retry logic**: failed deliveries are retried with exponential backoff
- **Clock skew tolerance**: ±60s between server and receiver clocks

---

## Developer SDKs

Official SDKs available for all major backend languages:

| SDK | Package | Language |
|-----|---------|----------|
| Node.js / TypeScript | `@upay/upay-js` | JavaScript / TypeScript |
| Python | `upay-python` | Python 3.8+ |
| PHP | `upay/upay-php` | PHP 7.4+ |
| Java | `com.upay:upay-java` | Java 11+ |

All SDKs include HMAC webhook signature validation utilities and typed error handling.
