# 04 — Architecture Document
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Date:** Q1 2026 | **Scope:** Technical Architecture Reference

---

## 1. Architecture Philosophy

The Shree Furniture platform is built on three architectural principles:

**Headless Commerce** — The frontend (storefront) is completely decoupled from the commerce backend (MedusaJS). This enables independent deployment, technology upgrades, and the ability to support multiple frontends (web, mobile app) from a single API layer.

**Performance-First** — Every architectural decision is evaluated through the lens of Core Web Vitals. ISR, CDN-first delivery, Redis caching, and React Server Components are not optional optimisations — they are core to the architecture.

**Owned Infrastructure** — MedusaJS is self-hosted, not SaaS. PostgreSQL is directly managed. No per-transaction fees, no vendor lock-in on commerce data. The business owns all its data and logic.

---

## 2. Architecture Layers

```
┌────────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — PRESENTATION                                                 │
│  Next.js 15 (App Router) · TypeScript · Tailwind CSS · shadcn/ui       │
│  Framer Motion · Zustand · TanStack Query · React Hook Form + Zod      │
└───────────────────────────────┬────────────────────────────────────────┘
                                │ REST API over HTTPS
┌───────────────────────────────▼────────────────────────────────────────┐
│  LAYER 2 — COMMERCE API                                                  │
│  MedusaJS v2 · Node.js 20 LTS                                           │
│  Products · Cart · Orders · Customers · Payments · Inventory            │
└──────────┬──────────────────────────────────────┬──────────────────────┘
           │                                      │
┌──────────▼─────────────┐            ┌───────────▼────────────────────┐
│  LAYER 3 — DATA         │            │  LAYER 3 — CACHE & QUEUE       │
│  PostgreSQL 16          │            │  Redis 7                        │
│  (Primary data store)   │            │  (Sessions, cart, rate limit,  │
└────────────────────────┘            │   background jobs)              │
                                       └────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 4 — EXTERNAL INTEGRATIONS                                       │
│  Razorpay · Cloudinary · Resend · Algolia · PostHog · Sentry          │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 5 — INFRASTRUCTURE & EDGE                                       │
│  Cloudflare CDN + WAF · Vercel Edge Network · Nginx (VPS Phase 2)     │
│  GitHub Actions CI/CD · PM2 · Certbot · Netdata                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. Monorepo Structure

The project uses a monorepo architecture to share types and design tokens between the storefront, admin, and backend while keeping each deployable unit independent.

```
shree-furniture/
├── apps/
│   ├── storefront/              # Next.js 15 (App Router) — customer-facing
│   │   ├── app/
│   │   │   ├── (store)/
│   │   │   │   ├── page.tsx              # Homepage
│   │   │   │   ├── products/[handle]/    # Product Detail Page (ISR)
│   │   │   │   ├── collections/[handle]/ # Product Listing Page (ISR)
│   │   │   │   ├── cart/                 # Cart page
│   │   │   │   └── checkout/             # Multi-step checkout (CSR)
│   │   │   └── api/
│   │   │       └── webhooks/razorpay/    # Razorpay payment webhook handler
│   │   ├── components/
│   │   │   ├── ui/                       # shadcn/ui base components
│   │   │   ├── product/                  # ProductCard, Gallery, Variants, Badges
│   │   │   ├── cart/                     # CartDrawer, CartItem, CartSummary
│   │   │   ├── checkout/                 # AddressForm, ShippingStep, PaymentStep
│   │   │   ├── layout/                   # Header, Footer, Navigation, Breadcrumb
│   │   │   └── shared/                   # Skeleton loaders, Error boundaries
│   │   ├── lib/
│   │   │   ├── medusa/                   # Typed Medusa client and query hooks
│   │   │   ├── algolia/                  # Search client initialisation
│   │   │   └── utils/                    # Price formatters, date utils, validators
│   │   └── store/
│   │       ├── cart-store.ts             # Zustand cart state
│   │       └── ui-store.ts               # Zustand UI state (drawer, modal)
│   │
│   └── admin/                   # Medusa Admin (or custom admin — future)
│
├── backend/                     # MedusaJS v2
│   ├── src/
│   │   ├── modules/             # Custom Medusa modules (wishlist, reviews)
│   │   ├── subscribers/         # Event-driven handlers
│   │   │   ├── order-placed.ts  # → Resend email on order.placed
│   │   │   └── low-stock.ts     # → Admin alert on low inventory
│   │   ├── workflows/           # MedusaJS custom workflows
│   │   └── api/
│   │       └── store/           # Custom storefront API routes
│   └── medusa-config.ts         # MedusaJS configuration (plugins, modules)
│
├── packages/
│   ├── ui/                      # Shared design tokens (colours, spacing, typography)
│   └── types/                   # Shared TypeScript types (Product, Order, Cart, etc.)
│
└── infrastructure/
    ├── nginx/
    │   └── shree-furniture.conf # Nginx reverse proxy configuration
    ├── docker-compose.yml       # Local development: PostgreSQL + Redis
    └── .github/
        └── workflows/
            ├── ci.yml           # Type check + lint + test on every PR
            └── deploy.yml       # Build + deploy on merge to main
```

---

## 4. Frontend Architecture

### 4.1 Rendering Strategy

Next.js 15 App Router provides four rendering modes. Each route is assigned the most appropriate mode:

| Route | Mode | Reason |
|---|---|---|
| `/` (Homepage) | ISR (revalidate: 60s) | High traffic; content changes infrequently |
| `/collections/[handle]` | ISR (revalidate: 60s) | SEO-critical; filter state is client-side |
| `/products/[handle]` | ISR (revalidate: 60s) | SEO-critical; price/stock updates are tolerably stale |
| `/checkout` | CSR (dynamic) | Real-time cart and payment state |
| `/account/*` | SSR (per request, authenticated) | Personalised, cannot be cached |
| `/api/webhooks/*` | Edge API Route | Low latency, stateless webhook processing |

### 4.2 State Architecture

```
Application State Split:

Server State (TanStack Query)          Client State (Zustand)
─────────────────────────────          ──────────────────────
• Products list                        • Cart items (optimistic)
• Product detail                       • Cart drawer open/closed
• Search results                       • Wishlist items (cached)
• Order history                        • Active filters (UI)
• Customer profile                     • Modal / overlay state
```

### 4.3 Component Architecture

Components are organised in a hierarchy from generic to domain-specific:

```
packages/ui/          ← Design tokens (colours, fonts, spacing)
  ↓
apps/storefront/components/ui/     ← shadcn/ui base components (Button, Input, Dialog)
  ↓
apps/storefront/components/shared/ ← Generic app components (Skeleton, ErrorBoundary)
  ↓
apps/storefront/components/{domain}/ ← Domain components (ProductCard, CartItem, AddressForm)
  ↓
apps/storefront/app/(store)/       ← Page-level Server Components (data fetching)
```

### 4.4 Performance Implementation

```typescript
// ISR for product pages
export const revalidate = 60;

// Priority loading for above-fold images
<Image src={product.thumbnail} width={800} height={800}
       priority={true} placeholder="blur" blurDataURL={blurUrl} />

// Dynamic import for below-fold / heavy components
const ProductGallery = dynamic(() => import('@/components/product/Gallery'), {
  loading: () => <GallerySkeleton />
});

// Parallel data fetching in Server Components
const [product, recommendations, relatedProducts] = await Promise.all([
  getProduct(handle),
  getRecommendations(handle),
  getCollectionProducts(collectionId)
]);
```

---

## 5. Backend Architecture (MedusaJS v2)

### 5.1 MedusaJS Module Architecture

MedusaJS v2 is built around independent, composable modules. Each module owns its own data layer and can be extended or swapped:

| Module | Responsibility | Custom Extension |
|---|---|---|
| Product Module | Products, variants, categories, collections | Add custom fields (room_type, assembly_required) |
| Cart Module | Cart lifecycle, line items, promotions | — |
| Order Module | Order creation, fulfilment, returns | Custom fulfilment workflow |
| Customer Module | Customer accounts, addresses, auth | — |
| Payment Module | Payment sessions via provider plugins | Razorpay plugin |
| Inventory Module | Stock levels per location | — |
| Notification Module | Email/SMS via provider plugins | Resend plugin |
| Search Module | Product indexing via provider plugins | Algolia / MeiliSearch plugin |

### 5.2 Event-Driven Architecture

MedusaJS uses an internal event bus (Redis-backed) for loose coupling between modules:

```
Order Placed Flow:
order.placed event
  → OrderConfirmationSubscriber
      → Resend: send "Your order is confirmed" email to customer
      → Resend: send "New order received" alert to admin

Shipment Created Flow:
order.shipment_created event
  → ShipmentNotificationSubscriber
      → Resend: send "Your order has shipped" email with tracking link

Low Stock Alert:
inventory.quantity_updated (below threshold) event
  → LowStockSubscriber
      → Resend: alert admin with SKU and current quantity
```

### 5.3 API Design

All Medusa REST APIs follow a consistent pattern:

```
Store APIs (public, customer-facing):
  GET    /store/products              # List products with filters
  GET    /store/products/:handle      # Single product
  GET    /store/collections           # List collections
  POST   /store/carts                 # Create cart
  POST   /store/carts/:id/line-items  # Add item to cart
  DELETE /store/carts/:id/line-items/:item_id
  POST   /store/carts/:id/complete    # Complete cart → create order
  POST   /store/customers             # Register
  POST   /store/customers/auth        # Login

Admin APIs (authenticated, internal):
  GET/POST/PUT/DELETE /admin/products
  GET/POST/PUT/DELETE /admin/orders
  GET/POST/PUT/DELETE /admin/customers
  POST /admin/orders/:id/fulfillments
  POST /admin/orders/:id/refunds

Custom Routes:
  POST /store/wishlist/items          # Add to wishlist
  DELETE /store/wishlist/items/:id    # Remove from wishlist
  GET  /store/wishlist                # Get customer wishlist
```

---

## 6. Database Architecture

### 6.1 Schema Overview

```sql
-- Core Commerce Tables (managed by MedusaJS)
product              → product_variant → product_option_value
product_collection   → product (many-to-many via product_collection_product)
cart                 → line_item → product_variant
order                → line_item, payment, fulfillment, return
customer             → address, order
payment_session      → payment_provider (razorpay)
inventory_item       → inventory_level (per location)

-- Custom Tables (added via MedusaJS module extensions)
wishlist_item        (customer_id, product_variant_id, created_at)
product_review       (product_id, customer_id, rating, body, created_at) -- Phase 2
```

### 6.2 Indexing Strategy

Critical indexes for query performance:

```sql
-- Product discovery queries
CREATE INDEX idx_product_collection ON product_collection_product(collection_id);
CREATE INDEX idx_product_status ON product(status);
CREATE INDEX idx_product_handle ON product(handle);  -- ISR lookup

-- Order management
CREATE INDEX idx_order_customer ON order(customer_id);
CREATE INDEX idx_order_status ON order(status, created_at);
CREATE INDEX idx_payment_razorpay ON payment(provider_id, data->>'razorpay_order_id');

-- Cart queries
CREATE INDEX idx_cart_customer ON cart(customer_id);
CREATE INDEX idx_line_item_cart ON line_item(cart_id);
```

### 6.3 Backup Strategy

```
Daily automated backup:
  pg_dump shree_furniture | gzip → Backblaze B2 (retained 30 days)
  
Cron schedule: 2:00 AM IST daily
Retention: 30 daily snapshots
Restore test: Manual monthly restore drill
Alert: Netdata alert if backup job fails
```

---

## 7. Infrastructure Architecture

### 7.1 Phase 1 — Managed Services

```
┌─────────────────────────────────────────────────────────┐
│                   Cloudflare (Edge)                     │
│          DNS Resolution + CDN + WAF + DDoS              │
└───────────┬─────────────────────────┬───────────────────┘
            │                         │
┌───────────▼──────────┐   ┌──────────▼──────────────────┐
│  Vercel              │   │  Railway.app                 │
│  Next.js Storefront  │   │  MedusaJS API (:9000)        │
│  Edge Network        │   │                              │
└──────────────────────┘   └──────┬───────────────────────┘
                                  │
                    ┌─────────────┼────────────────┐
                    │             │                │
             ┌──────▼─────┐ ┌────▼────┐    ┌──────▼──────┐
             │ Neon.tech   │ │Upstash  │    │ Cloudinary  │
             │ PostgreSQL  │ │ Redis   │    │ Image CDN   │
             └────────────┘ └─────────┘    └─────────────┘
```

### 7.2 Phase 2 — Hetzner VPS

```
┌─────────────────────────────────────────────────────────┐
│               Cloudflare (retained for CDN + WAF)        │
└───────────────────────────┬─────────────────────────────┘
                            │ HTTPS :443
┌───────────────────────────▼─────────────────────────────┐
│              Hetzner CX31 VPS (Ubuntu 22.04)             │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Nginx (Reverse Proxy + SSL via Certbot)        │    │
│  │   /          → Next.js (PM2, :3000)             │    │
│  │   /api/*     → MedusaJS (PM2, :9000)            │    │
│  │   /admin/*   → Medusa Admin (PM2, :7001)        │    │
│  └────────────────────┬────────────────────────────┘    │
│                        │ Internal network                │
│  ┌─────────────────────┴────────────────────────────┐   │
│  │  Internal Services                               │   │
│  │   PostgreSQL 16       (:5432, internal only)     │   │
│  │   Redis 7             (:6379, internal only)     │   │
│  │   MeiliSearch         (:7700, internal only)     │   │
│  │   Netdata             (:19999, monitoring)       │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### 7.3 CI/CD Pipeline

```
Developer pushes code
  → GitHub PR opened
    → GitHub Actions: CI Pipeline
        ├── tsc --noEmit                    (TypeScript check)
        ├── eslint + prettier --check       (Code quality)
        ├── vitest run                      (Unit tests)
        └── next build (dry run)            (Build validation)

PR approved + merged to main
  → GitHub Actions: Deploy Pipeline
        ├── Build Next.js
        ├── Deploy to Vercel (staging) via Vercel CLI
        ├── SSH into VPS → git pull → npm ci → pm2 reload medusa
        ├── npx medusa db:migrate           (DB migrations)
        ├── pm2 reload storefront
        └── Smoke tests: health check /api/health + checkout flow ping
```

---

## 8. Security Architecture

### 8.1 Authentication & Authorisation

```
Customer Auth:
  POST /store/customers/auth → JWT issued by MedusaJS
  JWT stored in HTTP-only, Secure, SameSite=Strict cookie
  Session cached in Redis (key: session:{customer_id}, TTL: 24h)
  Medusa auth middleware validates JWT on protected routes

Admin Auth:
  Separate JWT for admin routes (/admin/*)
  Admin users managed in Medusa Admin panel
  IP allowlist recommended for /admin/* in Nginx (Phase 2)
```

### 8.2 Payment Security

```
Razorpay Webhook Verification:
  1. Receive POST /api/webhooks/razorpay
  2. Extract X-Razorpay-Signature header
  3. Compute HMAC-SHA256(webhook_secret, raw_body)
  4. Compare computed signature to header value
  5. Reject with 400 if mismatch
  6. Process event (idempotent — check if order already processed)
```

### 8.3 Rate Limiting

```
Redis-backed rate limiting (via MedusaJS middleware):
  /store/customers/auth    → 5 attempts per 15 minutes per IP
  /store/customers         → 10 registrations per hour per IP
  /store/carts/:id/complete → 3 attempts per minute per customer
  /api/webhooks/*          → Whitelisted Razorpay IPs only
```

### 8.4 Infrastructure Security

```
VPS Hardening:
  ├── UFW Firewall: allow only :80, :443, :22 (SSH from known IPs)
  ├── PostgreSQL: bind to 127.0.0.1 only (no external access)
  ├── Redis: bind to 127.0.0.1 only + requirepass
  ├── SSH: key-based auth only, disable password auth
  └── Automatic security updates: unattended-upgrades enabled

Cloudflare Protection:
  ├── WAF: OWASP ruleset enabled
  ├── DDoS: automatic mitigation
  ├── Bot management: challenge suspicious traffic
  └── Always-on HTTPS: HTTP requests force-redirected to HTTPS
```

---

## 9. Integration Architecture

### 9.1 Razorpay Integration

```
Storefront → POST /store/carts/:id/payment-sessions
  → MedusaJS Razorpay plugin → Razorpay Orders API
  → Returns: razorpay_order_id, amount, currency, key_id

Storefront → Razorpay JS SDK (checkout modal)
  → User completes payment
  → Razorpay fires webhook → /api/webhooks/razorpay
  → Signature verified → Medusa completes cart → Order created
  → Storefront polls /store/orders/:id for confirmation
```

### 9.2 Cloudinary Integration

```
Admin uploads image → Medusa Admin
  → MedusaJS Cloudinary plugin → Cloudinary Upload API
  → Returns CDN URL (f_auto,q_auto applied at delivery)
  → URL stored in product_image table

Storefront Next/Image → Cloudinary URL
  → Cloudflare CDN checks cache
  → Cache miss → Cloudinary generates WebP/AVIF at requested size
  → Cached at Cloudflare edge for subsequent requests
```

### 9.3 Resend Email Integration

```
Event: order.placed
  → MedusaJS Notification Module → Resend API
  → React Email template rendered server-side
  → HTML email sent to customer

Templates:
  order-confirmation.tsx   (Order ID, items, total, delivery estimate)
  order-shipped.tsx        (Tracking number, carrier, expected delivery)
  password-reset.tsx       (OTP / reset link, expiry time)
  welcome.tsx              (Account created confirmation)
```

### 9.4 Algolia / MeiliSearch Integration

```
Product sync (via MedusaJS subscriber):
  product.created / product.updated / product.deleted events
    → AlgoliaIndexSubscriber
        → algolia.index('products').saveObject(productRecord)

Storefront search:
  User types in search box (debounced 300ms)
    → Algolia InstantSearch hook → Algolia API
    → Results rendered client-side with facets
    → Query event logged to PostHog
```

---

## 10. Core Web Vitals Architecture

Every architectural decision ties back to measurable performance:

| Metric | Target | Architectural Solution |
|---|---|---|
| **LCP** | < 2.5s | ISR-cached pages, priority image loading, Cloudflare CDN |
| **INP** | < 200ms | React Server Components reduce client JS bundle; Zustand minimal re-renders |
| **CLS** | < 0.1 | Explicit image dimensions everywhere; skeleton loaders match final layout |
| **TTFB** | < 600ms | Edge middleware, Redis-cached API responses, Vercel Edge Network |
| **Bundle Size** | < 200KB JS (initial) | Dynamic imports for below-fold components; Tree shaking via Tailwind |

---

## 11. Disaster Recovery

| Scenario | Detection | Recovery Procedure | RTO |
|---|---|---|---|
| Database corruption | Netdata alert, application errors | Restore from latest pg_dump (Backblaze B2) | < 2 hours |
| VPS hardware failure | Uptime Kuma alert | Redeploy to new Hetzner instance via infrastructure scripts | < 4 hours |
| Razorpay webhook loss | Order stuck in "pending" | Manual webhook replay via Razorpay dashboard | < 1 hour |
| Vercel deployment failure | GitHub Actions failure alert | Rollback to previous Vercel deployment (1-click) | < 15 minutes |
| Redis data loss | Application errors on cart/session | Carts rebuilt from PostgreSQL; sessions require re-login | < 30 minutes |

---

*Document Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
