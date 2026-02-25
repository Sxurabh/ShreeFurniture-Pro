# 04 — System Architecture
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Scope:** Full system design, layers, data flows, deployment topology

---

## 1. Architecture Principles

**Headless Commerce** — The Next.js storefront is fully decoupled from MedusaJS. They communicate only via REST API over HTTPS. This enables independent deployment and future mobile app support from the same API layer.

**Performance-First** — Every architectural decision is evaluated against Core Web Vitals. ISR, CDN delivery, Redis caching, and React Server Components are core constraints, not optional optimisations.

**Owned Infrastructure** — MedusaJS is self-hosted (not SaaS). PostgreSQL is directly managed. No per-transaction fees; the business owns all commerce data and logic.

---

## 2. Architecture Layers

```
┌────────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — PRESENTATION                                                 │
│  Next.js 15 (App Router) · TypeScript · Tailwind CSS · shadcn/ui       │
│  Framer Motion · Zustand · TanStack Query v5 · React Hook Form + Zod   │
│  Vercel Edge Network (Phase 1) / PM2 on Hetzner VPS (Phase 2)          │
└───────────────────────────────┬────────────────────────────────────────┘
                                │ REST API over HTTPS
┌───────────────────────────────▼────────────────────────────────────────┐
│  LAYER 2 — COMMERCE ENGINE                                               │
│  MedusaJS v2 · Node.js 20 LTS                                           │
│  Products · Cart · Orders · Customers · Payments · Inventory            │
│  Railway.app (Phase 1) / PM2 on VPS (Phase 2)                          │
└──────────┬──────────────────────────────────────┬──────────────────────┘
           │                                      │
┌──────────▼─────────────┐            ┌───────────▼────────────────────┐
│  LAYER 3A — PRIMARY DB  │            │  LAYER 3B — CACHE & QUEUE      │
│  PostgreSQL 16          │            │  Redis 7                        │
│  Neon.tech (Phase 1)    │            │  Sessions, cart, rate limits,  │
│  VPS internal (Phase 2) │            │  background job queue           │
└────────────────────────┘            │  Upstash (Phase 1) / VPS (P2)  │
                                       └────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 4 — EXTERNAL INTEGRATIONS                                       │
│  Razorpay (Payments) · Cloudinary (Media CDN) · Resend (Email)        │
│  Algolia (Search, Phase 1) · MeiliSearch (Search, Phase 2)            │
│  PostHog (Analytics) · Sentry (Error tracking)                        │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 5 — EDGE & INFRASTRUCTURE                                       │
│  Cloudflare CDN + WAF + DDoS · Nginx reverse proxy (Phase 2)          │
│  GitHub Actions CI/CD · PM2 process manager · Certbot SSL             │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. Rendering Strategy

| Route | Mode | Revalidation | Reason |
|---|---|---|---|
| `/` | ISR | 60 seconds | High traffic; content infrequently changes |
| `/collections/[handle]` | ISR | 60 seconds | SEO-critical; filters are client-side |
| `/products/[handle]` | ISR | 60 seconds | SEO-critical; stale price/stock is acceptable |
| `/checkout` | CSR (dynamic) | Real-time | Payment state cannot be cached |
| `/account/*` | SSR (per request) | Per request | Personalised; authentication required |
| `/order/confirm/[id]` | SSR (per request) | Per request | Order data must be fresh |
| `/api/webhooks/*` | Edge API Route | Stateless | Low-latency, stateless processing |

---

## 4. State Management Architecture

```
Application State Split:

Server State (TanStack Query v5)         Client State (Zustand)
──────────────────────────────           ──────────────────────
• Products list + filters                • Cart items (optimistic updates)
• Product detail + variants              • Cart drawer open/closed
• Search results                         • Wishlist items (UI cache)
• Order history                          • Active filter selections
• Customer profile + addresses           • Modal / overlay state
• Shipping options
```

---

## 5. Critical Data Flows

### 5.1 Product Discovery

```
User visits PLP
  → Next.js serves ISR-cached page (TTFB < 600ms)
  → Client: TanStack Query hydrates filter options from Medusa REST API
  → User applies filter → debounced API call → Medusa queries PostgreSQL
  → Algolia powers instant search results via InstantSearch hooks
  → Images served from Cloudinary CDN (WebP/AVIF, responsive widths)
  → Filter state written to URL query params (shareable)
```

### 5.2 Add to Cart

```
User clicks "Add to Cart"
  → Zustand cart state updated immediately (optimistic)
  → POST /store/carts/:id/line-items (Medusa REST)
  → Cart persisted to Redis (key: cart:{cart_id}, TTL: 7 days)
  → On error: optimistic update rolled back, error toast shown
  → CartDrawer opens automatically
```

### 5.3 Checkout & Payment

```
User proceeds to Checkout
  → POST /store/carts/:id/payment-sessions
  → MedusaJS Razorpay plugin → Razorpay Orders API
  → razorpay_order_id, amount, currency, key_id returned to client

User interacts with Razorpay checkout modal
  → Razorpay JS SDK handles payment UI

Payment completed by Razorpay
  → POST /api/webhooks/razorpay (Razorpay fires webhook)
  → HMAC-SHA256 signature verified (reject 400 if invalid)
  → Idempotency check: order already processed? Skip.
  → POST /store/carts/:id/complete (Medusa → creates Order)
  → order.placed event → Redis queue → Resend email subscriber fires
  → Client polls /store/orders/:id for confirmation
  → Redirect to /order/confirm/:id
```

### 5.4 Authentication

```
Register: POST /store/customers
  → Medusa creates customer in PostgreSQL
  → JWT issued → HTTP-only, Secure, SameSite=Strict cookie

Login: POST /store/customers/auth
  → Credentials validated → JWT issued
  → Session cached in Redis (key: session:{customer_id}, TTL: 24h)
  → All subsequent requests authenticate via cookie
  → Medusa middleware validates JWT and attaches customer context
```

### 5.5 Media Upload & Delivery

```
Admin uploads image via Medusa Admin
  → Cloudinary Upload API → auto-converts to WebP + AVIF
  → Responsive variants generated: 400w, 800w, 1200w
  → CDN URL stored in product_image table

Storefront Next/Image request
  → Cloudinary URL with transform params (f_auto, q_auto, w_{width})
  → Cloudflare CDN cache check → hit: edge-served
  → Cache miss → Cloudinary generates + returns → Cloudflare caches
```

---

## 6. Redis Cache Schema

| Key Pattern | TTL | Purpose |
|---|---|---|
| `cart:{cart_id}` | 7 days | Cart persistence across sessions |
| `session:{customer_id}` | 24 hours | Auth session token validation |
| `rate_limit:{ip}:{route}` | 1 minute | Per-IP rate limiting |
| `search:{query_hash}` | 5 minutes | Search result caching |

---

## 7. Event-Driven Queue (Redis via MedusaJS Event Bus)

| Event | Subscriber | Action |
|---|---|---|
| `order.placed` | `OrderConfirmationSubscriber` | Send confirmation email via Resend |
| `order.shipment_created` | `ShipmentNotificationSubscriber` | Send shipping notification email |
| `customer.password_reset` | `PasswordResetSubscriber` | Send OTP/reset link email |
| `customer.created` | `WelcomeEmailSubscriber` | Send welcome email |
| `inventory.low_stock` | `LowStockSubscriber` | Alert admin via email |
| `product.created` / `product.updated` | `AlgoliaIndexSubscriber` | Sync product to Algolia index |

---

## 8. Security Architecture

| Layer | Control | Implementation |
|---|---|---|
| Transport | HTTPS everywhere | Cloudflare SSL + Certbot (VPS) |
| Authentication | JWT in HTTP-only cookies | Medusa auth middleware |
| Payment webhooks | HMAC-SHA256 verification | Razorpay plugin + custom handler |
| API rate limiting | Redis per-IP limits | `/store/auth` (5/15min), `/store/carts/complete` (3/min) |
| Database | Not publicly exposed | Bound to `127.0.0.1` (VPS); Neon private pool (Phase 1) |
| WAF | Cloudflare WAF rules | OWASP ruleset; bot challenge |
| Secrets | Environment variables only | `.env.local`; never committed to Git |
| Dependencies | Automated audit | `npm audit` runs in every CI pipeline |
| Backups | Daily `pg_dump` | Stored on Backblaze B2 |
| Admin routes | IP allowlist (Phase 2) | Nginx `allow` directive for `/admin/*` |

---

## 9. Deployment Topology

### Phase 1 — Managed Free Tier

```
Cloudflare DNS + CDN + WAF
  ├── Vercel Edge Network
  │     └── Next.js 15 Storefront
  └── Railway.app
        └── MedusaJS v2 (Node.js 20)
              ├── Neon.tech (PostgreSQL 16, serverless)
              └── Upstash (Redis 7, serverless)

External: Cloudinary · Algolia · Resend · PostHog · Sentry
```

### Phase 2 — Hetzner CX31 VPS (Ubuntu 22.04)

```
Cloudflare CDN + WAF + DDoS
  └── Nginx (reverse proxy + Certbot SSL)
        ├── /*         → Next.js (PM2, :3000)
        ├── /api/*     → MedusaJS (PM2, :9000)
        └── /admin/*   → Medusa Admin (PM2, :7001)

VPS internal (127.0.0.1 only):
  ├── PostgreSQL 16 (:5432)
  ├── Redis 7       (:6379, requirepass set)
  └── MeiliSearch   (:7700)  [replaces Algolia]

Monitoring: Netdata (:19999) · Uptime Kuma (HTTP checks every 60s)
```

### CI/CD Pipeline

```
PR opened
  → GitHub Actions CI:
      tsc --noEmit → eslint + prettier → vitest run → next build

Merge to main
  → GitHub Actions Deploy:
      Build Next.js → Deploy to Vercel (storefront)
      SSH to VPS → git pull → npm ci → pm2 reload medusa
      npx medusa db:migrate → pm2 reload storefront
      Smoke tests: /api/health check + checkout flow ping
```

---

## 10. Core Web Vitals Architecture Map

| Metric | Target | How It's Achieved |
|---|---|---|
| LCP | < 2.5s | ISR caching, `priority` image loading, Cloudflare CDN |
| INP | < 200ms | RSC reduces client JS; Zustand minimises re-renders |
| CLS | < 0.1 | Explicit image dimensions; skeleton loaders match layout |
| TTFB | < 600ms | Edge middleware, Redis-cached responses, Vercel edge |
| Bundle | < 200KB | Dynamic imports for below-fold; Tailwind tree-shaking |

---

## 11. Disaster Recovery

| Scenario | Detection | Recovery | RTO |
|---|---|---|---|
| DB corruption | Netdata + app errors | Restore from latest pg_dump (Backblaze B2) | < 2 hours |
| VPS failure | Uptime Kuma alert | Redeploy to new Hetzner instance via scripts | < 4 hours |
| Webhook loss | Order stuck "pending" | Manual replay via Razorpay dashboard | < 1 hour |
| Vercel deploy failure | GitHub Actions alert | 1-click rollback in Vercel dashboard | < 15 minutes |
| Redis data loss | Cart/session errors | Carts rebuilt from PostgreSQL; sessions require re-login | < 30 minutes |

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
