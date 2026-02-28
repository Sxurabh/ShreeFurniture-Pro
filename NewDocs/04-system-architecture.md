# 04 — System Architecture
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Status:** Active
> This document was converted from diagram format to text + Mermaid for AI readability.

---

## 1. Architecture Philosophy

**Headless Commerce.** The storefront has zero direct database access. It knows nothing about PostgreSQL, Redis, or internal Medusa data models. All data flows through the MedusaJS REST API only. This means:
- The frontend and backend can be deployed, scaled, and failed independently.
- The storefront can be replaced entirely without touching business logic.
- API contracts (doc 06) are the binding interface between the two.

---

## 2. High-Level System Diagram

```mermaid
graph TB
  subgraph Client["Browser / Mobile"]
    Browser["Customer Browser"]
  end

  subgraph Vercel["Vercel (Edge + Serverless)"]
    NextJS["Next.js 15 App Router\n(apps/storefront)"]
    ISR["ISR Cache\n(CDN Edge)"]
    Webhook["Razorpay Webhook\n/api/webhooks/razorpay"]
  end

  subgraph Railway["Railway.app"]
    Medusa["MedusaJS v2\n(backend/)"]
    MedusaAdmin["Medusa Admin\n/admin"]
  end

  subgraph Neon["Neon.tech"]
    Postgres["PostgreSQL 16\n(primary data store)"]
  end

  subgraph Upstash["Upstash"]
    Redis["Redis 7\n(sessions + cart TTL + queue)"]
  end

  subgraph ThirdParty["Third-Party Services"]
    Razorpay["Razorpay\n(payments)"]
    Cloudinary["Cloudinary CDN\n(images)"]
    Algolia["Algolia\n(search)"]
    Resend["Resend\n(email)"]
    PostHog["PostHog\n(analytics)"]
    Sentry["Sentry\n(errors)"]
  end

  Browser --> Vercel
  Vercel --> ISR
  NextJS -- "REST API (store)" --> Medusa
  Webhook -- "HMAC verified POST" --> Medusa
  Medusa --> Postgres
  Medusa --> Redis
  Medusa --> Razorpay
  Medusa --> Cloudinary
  Medusa --> Algolia
  Medusa --> Resend
  MedusaAdmin --> Medusa
  NextJS --> PostHog
  NextJS --> Sentry
  NextJS --> Algolia
```

---

## 3. Data Flow: Product Page Request

This is the most common request type and illustrates ISR caching:

```mermaid
sequenceDiagram
  participant Browser
  participant Vercel_Edge as Vercel Edge (CDN)
  participant NextJS as Next.js Server
  participant Medusa as MedusaJS v2
  participant Postgres as PostgreSQL

  Browser->>Vercel_Edge: GET /products/oslo-3-seater-sofa
  Vercel_Edge-->>Browser: Serve ISR cached HTML (if < 60s old)

  Note over Vercel_Edge: If cache is stale (> 60s):
  Vercel_Edge->>NextJS: Trigger background revalidation
  NextJS->>Medusa: GET /store/products?handle=oslo-3-seater-sofa
  Medusa->>Postgres: SELECT product + variants + images
  Postgres-->>Medusa: Product data
  Medusa-->>NextJS: Product JSON
  NextJS-->>Vercel_Edge: New rendered HTML
  Vercel_Edge-->>Browser: New HTML (next request)
```

**Key rules:**
- PDP, PLP, and Homepage all use `revalidate = 60` (60-second ISR).
- On product publish/update via Medusa Admin, an Algolia sync subscriber calls the storefront's `/api/revalidate` endpoint to force **immediate** cache invalidation.
- Search results (`/search`) are CSR — no ISR. Algolia handles freshness.
- Cart and checkout pages are CSR — no ISR.

---

## 4. Data Flow: Checkout & Payment

```mermaid
sequenceDiagram
  participant Browser
  participant NextJS as Next.js
  participant Medusa as MedusaJS v2
  participant Razorpay as Razorpay
  participant Postgres as PostgreSQL

  Browser->>Medusa: POST /store/carts (create cart)
  Browser->>Medusa: POST /store/carts/:id/line-items (add items)
  Browser->>Medusa: POST /store/carts/:id (set email + address)
  Browser->>Medusa: POST /store/carts/:id/shipping-methods
  Browser->>Medusa: POST /store/carts/:id/payment-sessions (init Razorpay)
  Medusa-->>Browser: payment_session with Razorpay order_id

  Browser->>Razorpay: Open Razorpay JS modal (order_id)
  Razorpay-->>Browser: Payment captured → razorpay_payment_id

  Browser->>Medusa: POST /store/carts/:id/complete
  Medusa->>Postgres: Create order record
  Medusa-->>Browser: { type: "order", data: { id, display_id } }

  Note over Razorpay,NextJS: Async: Razorpay sends webhook
  Razorpay->>NextJS: POST /api/webhooks/razorpay (HMAC signed)
  NextJS->>NextJS: Verify HMAC signature
  NextJS->>Postgres: Check idempotency (event_id already processed?)
  NextJS->>Medusa: Capture payment if not yet processed
  Medusa->>Medusa: Trigger order.placed subscriber → send email
```

**Key rules:**
- `unit_price` on each line item is a **snapshot** taken at the moment the item is added.
- Webhook handler at `/api/webhooks/razorpay/route.ts` must be idempotent.
- Cart `complete()` is the single trigger for order creation — never create orders directly.

---

## 5. Rendering Strategy by Page

| Page | Strategy | Rationale |
|---|---|---|
| Homepage (`/`) | ISR — `revalidate: 60` | SEO-critical; content changes infrequently |
| PLP (`/collections/[handle]`) | ISR — `revalidate: 60` | SEO-critical; catalogue updates via admin, not real-time |
| PDP (`/products/[handle]`) | ISR — `revalidate: 60` | SEO most important here; stock changes tolerated at 60s lag |
| Search (`/search`) | CSR (Algolia InstantSearch) | Needs real-time results; not crawled by SEO |
| Cart (`/cart`) | CSR | User-specific, not cacheable |
| Checkout (`/checkout/*`) | CSR | Authenticated + user-specific |
| Order Confirm (`/order/confirm/[id]`) | SSR | Fresh per-request; shows live order status |
| Account pages (`/account/*`) | CSR | Authenticated; all user-specific data |

---

## 6. Monorepo Build Dependency Graph

```mermaid
graph LR
  types["packages/types"]
  ui["packages/ui"]
  storefront["apps/storefront"]
  backend["backend/"]

  types --> storefront
  types --> backend
  ui --> storefront
```

**Build order enforced by Turborepo:** `packages/types` and `packages/ui` must build before `apps/storefront` and `backend/`.

---

## 7. Infrastructure: Phase 1 (MVP) vs Phase 2

| Layer | Phase 1 (MVP) | Phase 2 |
|---|---|---|
| Storefront hosting | Vercel (free/pro) | Vercel (stays) |
| Backend hosting | Railway.app | Hetzner VPS (Docker) |
| Database | Neon.tech (serverless PG) | Hetzner VPS (PostgreSQL container) |
| Redis | Upstash (serverless) | Hetzner VPS (Redis container) |
| Search | Algolia (free tier) | MeiliSearch (self-hosted on VPS) |
| Nginx | N/A | Reverse proxy on VPS |

---

## 8. Security Boundaries

```
PUBLIC (no auth required):
  GET  /store/products
  GET  /store/products/:handle
  GET  /store/collections
  POST /store/carts
  POST /store/carts/:id/line-items
  GET  /store/regions
  GET  /store/shipping-options

AUTHENTICATED (customer session cookie required):
  GET  /store/customers/me
  POST /store/customers/me/addresses
  GET  /store/orders

ADMIN (Medusa Admin JWT required — different auth):
  All  /admin/*

WEBHOOK (HMAC signature required — no user auth):
  POST /api/webhooks/razorpay
```

---

## 9. Environment Variable Matrix

| Variable | Used By | Public? | Notes |
|---|---|---|---|
| `DATABASE_URL` | Backend only | ❌ No | PostgreSQL connection string |
| `REDIS_URL` | Backend only | ❌ No | Redis connection string |
| `JWT_SECRET` | Backend only | ❌ No | MedusaJS JWT signing |
| `COOKIE_SECRET` | Backend only | ❌ No | MedusaJS cookie signing |
| `RAZORPAY_SECRET` | Backend only | ❌ No | Razorpay API secret |
| `RAZORPAY_WEBHOOK_SECRET` | Storefront webhook handler | ❌ No | HMAC verification |
| `CLOUDINARY_API_SECRET` | Backend only | ❌ No | |
| `ALGOLIA_WRITE_API_KEY` | Backend (sync) | ❌ No | Write-access key |
| `RESEND_API_KEY` | Backend only | ❌ No | |
| `REVALIDATION_SECRET` | Storefront server | ❌ No | ISR on-demand revalidation |
| `NEXT_PUBLIC_MEDUSA_BACKEND_URL` | Storefront | ✅ Yes | Backend API URL |
| `NEXT_PUBLIC_RAZORPAY_KEY_ID` | Storefront (Razorpay modal) | ✅ Yes | Confirmed public by Razorpay docs |
| `NEXT_PUBLIC_ALGOLIA_APP_ID` | Storefront (search) | ✅ Yes | |
| `NEXT_PUBLIC_ALGOLIA_SEARCH_KEY` | Storefront (search) | ✅ Yes | Read-only search key only |
| `NEXT_PUBLIC_POSTHOG_KEY` | Storefront (analytics) | ✅ Yes | |
| `NEXT_PUBLIC_SENTRY_DSN` | Storefront (errors) | ✅ Yes | |

---

*Shree Furniture | v1.1 — Q1 2026 | Converted from binary diagram to Mermaid*
