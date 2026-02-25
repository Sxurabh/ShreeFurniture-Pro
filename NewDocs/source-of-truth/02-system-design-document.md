# 02 — System Design Document
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Date:** Q1 2026 | **Scope:** Full System Architecture

---

## 1. Overview

This document describes the end-to-end system design for the Shree Furniture platform — covering data flow, component interactions, API contracts, caching strategy, queue architecture, and deployment topology across both Phase 1 (free tier) and Phase 2 (VPS production).

---

## 2. High-Level System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                               │
│          Next.js 15 (App Router) + TypeScript + Tailwind CSS         │
│          Vercel Edge Network (Phase 1) / PM2 on VPS (Phase 2)        │
└───────────────────────────────┬──────────────────────────────────────┘
                                │ HTTPS (REST API)
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                         COMMERCE ENGINE                              │
│                   MedusaJS v2 — Headless Commerce API                │
│                   Railway (Phase 1) / PM2 on VPS (Phase 2)           │
└─────────┬─────────────────────────────────────┬──────────────────────┘
          │                                     │
┌─────────▼──────────┐               ┌──────────▼────────────────────┐
│   PostgreSQL 16     │               │         Redis 7               │
│   (Primary DB)      │               │  Sessions / Cart / Queue      │
│   Neon.tech (P1)    │               │  Upstash (P1) / VPS (P2)     │
│   VPS (P2)          │               └───────────────────────────────┘
└────────────────────┘
          │
┌─────────▼──────────────────────────────────────────────────────────┐
│                        EXTERNAL SERVICES                            │
│  Razorpay (Payments) │ Cloudinary (Media) │ Resend (Email)         │
│  Algolia (Search)    │ PostHog (Analytics) │ Sentry (Errors)       │
└────────────────────────────────────────────────────────────────────┘
          │
┌─────────▼────────────────────────────────────────────────────────┐
│                     EDGE / CDN LAYER                              │
│              Cloudflare — DNS, CDN, WAF, DDoS Protection          │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Component Breakdown

### 3.1 Storefront (Next.js 15)

The storefront is a React Server Component-first application deployed on Vercel's Edge Network. It uses Incremental Static Regeneration (ISR) for all product and collection pages, ensuring fast page loads while keeping data fresh.

**Rendering Strategy by Page Type:**

| Page | Rendering Mode | Revalidation |
|---|---|---|
| Homepage | ISR | 60 seconds |
| Product Listing Page | ISR | 60 seconds |
| Product Detail Page | ISR | 60 seconds |
| Cart / Checkout | CSR (dynamic) | Real-time |
| My Account / Orders | SSR (authenticated) | Per request |
| Admin (Medusa Admin) | CSR | Real-time |

**State Management:**

- **Zustand** — client-side cart, wishlist, and UI state (drawer open/closed)
- **TanStack Query v5** — server state: product data, availability, search results with background refetch
- **React Hook Form + Zod** — form state for checkout steps, address, and authentication

### 3.2 Commerce Backend (MedusaJS v2)

MedusaJS v2 serves as the headless commerce engine exposing a REST API consumed by the Next.js storefront and the Medusa Admin dashboard.

**Core API Domains:**

| Domain | Endpoints | Key Operations |
|---|---|---|
| Products | `/store/products` | List, filter, retrieve, search |
| Collections | `/store/collections` | Room-based navigation |
| Cart | `/store/carts` | Create, add item, update, complete |
| Orders | `/store/orders` | Create, retrieve, fulfill |
| Customers | `/store/customers` | Register, login, profile, addresses |
| Payments | `/store/payment-sessions` | Initiate Razorpay session |
| Inventory | (admin) | Stock levels per SKU |
| Webhooks | `/store/webhooks` | Razorpay payment events |

**Custom Extensions:**

- **Subscribers** — Listen to `order.placed` event → trigger Resend transactional email
- **Workflows** — Custom fulfillment workflow for warehouse team
- **Custom API Routes** — `/store/wishlist`, `/store/reviews` (Phase 2+)

### 3.3 Database — PostgreSQL 16

PostgreSQL is the single source of truth for all commerce data.

**Key Schema Domains:**

```
Products & Catalogue
├── product              (id, title, description, thumbnail, status)
├── product_variant      (id, product_id, sku, title, inventory_quantity)
├── product_option       (colour, size, material)
├── product_collection   (Living Room, Bedroom, Dining, Office)
└── product_image        (Cloudinary URLs)

Orders & Fulfilment
├── cart                 (id, customer_id, region_id, line_items)
├── order                (id, cart_id, status, payment_status, total)
├── line_item            (order_id, variant_id, quantity, unit_price)
├── fulfillment          (order_id, tracking_number, status)
└── payment              (order_id, razorpay_order_id, status)

Customers
├── customer             (id, email, first_name, last_name)
├── address              (customer_id, line_1, city, state, postal_code)
└── wishlist_item        (customer_id, product_id)
```

### 3.4 Cache & Queue — Redis 7

Redis serves two roles: a cache layer and a background job queue.

**Cache Use Cases:**

| Key Pattern | TTL | Purpose |
|---|---|---|
| `cart:{cart_id}` | 7 days | Cart persistence across sessions |
| `session:{token}` | 24 hours | Auth session tokens |
| `rate_limit:{ip}:{route}` | 1 minute | Rate limiting on auth and order endpoints |
| `search:{query_hash}` | 5 minutes | Search result caching |

**Queue Use Cases (via MedusaJS event bus):**

| Event | Handler | Action |
|---|---|---|
| `order.placed` | `OrderConfirmationSubscriber` | Send transactional email via Resend |
| `order.shipment_created` | `ShipmentNotificationSubscriber` | Send shipping notification email |
| `customer.password_reset` | `PasswordResetSubscriber` | Send OTP/reset link email |
| `inventory.low_stock` | `LowStockSubscriber` | Alert admin via email |

---

## 4. Data Flow — Critical User Journeys

### 4.1 Product Discovery Flow

```
User visits PLP
  → Next.js serves ISR-cached page (< 100ms TTFB)
  → Client-side: TanStack Query fetches filter options from Medusa REST API
  → User applies filters → debounced API call → Medusa queries PostgreSQL
  → Algolia powers instant search via searchbox component
  → Images served from Cloudinary CDN (WebP/AVIF auto-converted)
```

### 4.2 Checkout & Payment Flow

```
User clicks "Add to Cart"
  → POST /store/carts/:id/line-items (Medusa REST)
  → Cart persisted to Redis (7-day TTL)
  → Zustand cart state updated (UI reflects instantly)

User proceeds to Checkout
  → POST /store/carts/:id/payment-sessions (initiates Razorpay)
  → Razorpay Order ID returned to client
  → Client loads Razorpay checkout modal

User completes payment
  → Razorpay sends webhook to /api/webhooks/razorpay
  → HMAC-SHA256 signature verified server-side
  → POST /store/carts/:id/complete (Medusa completes cart → creates Order)
  → order.placed event fired → Redis queue → Resend email triggered
  → User redirected to /order/confirm/:id
```

### 4.3 Authentication Flow

```
User registers → POST /store/customers
  → Medusa creates customer record in PostgreSQL
  → JWT token issued → stored as HTTP-only cookie

User logs in → POST /store/customers/auth
  → Credentials validated → session token cached in Redis
  → Subsequent requests send token via cookie
  → Medusa middleware validates and attaches customer context
```

---

## 5. Search Architecture

Algolia is used in MVP (free tier). MeiliSearch (self-hosted on VPS) replaces it in Phase 2 for cost efficiency.

**Indexing Strategy:**

```
Product fields indexed:
  - title, description, handle
  - collection.title (room type)
  - variants: price_min, price_max
  - tags: material, colour, style
  - thumbnail URL (for result display)

Facets configured:
  - collection (Living Room, Bedroom, Dining, Office)
  - price range (₹0-5K, ₹5K-15K, ₹15K-50K, ₹50K+)
  - material (Wood, Metal, Fabric, Glass)
  - colour
```

**Sync Strategy:** MedusaJS `product.created` and `product.updated` events trigger Algolia index sync via subscriber.

---

## 6. Media & Image Strategy

All product images are stored and served through Cloudinary.

**Upload Pipeline:**

```
Admin uploads image via Medusa Admin
  → Cloudinary upload API receives file
  → Auto-converts to WebP + AVIF
  → Generates responsive variants (400w, 800w, 1200w)
  → Returns CDN URL stored in product_image table
```

**Delivery Pipeline:**

```
Next/Image component requests image
  → Cloudinary URL with transform params (f_auto, q_auto, w_{width})
  → Served from Cloudflare CDN (edge cache)
  → Browser receives optimal format + size
```

---

## 7. Security Architecture

| Layer | Control | Implementation |
|---|---|---|
| Transport | HTTPS everywhere | Cloudflare SSL + Let's Encrypt |
| Authentication | JWT in HTTP-only cookies | Medusa auth middleware |
| Payment Webhooks | HMAC-SHA256 signature verification | Razorpay plugin |
| API Rate Limiting | Redis-backed per-IP limits | `/store/auth`, `/store/orders` |
| Network | PostgreSQL not publicly exposed | Internal VPC/network only |
| WAF | Cloudflare WAF rules | Block common attack patterns |
| Secrets | Environment variables | `.env.local`, never in Git |
| Dependencies | Automated audit | `npm audit` in GitHub Actions CI |
| Backups | Automated daily `pg_dump` | Stored on Backblaze B2 / S3 |

---

## 8. Deployment Topology

### Phase 1 — Managed Services

```
Cloudflare DNS
  → Vercel Edge Network → Next.js Storefront
  → Railway.app → MedusaJS Backend (Node.js)
                → Neon.tech (PostgreSQL)
                → Upstash (Redis)
  → Cloudinary (media)
  → Algolia (search)
  → Resend (email)
```

### Phase 2 — VPS (Hetzner CX31)

```
Internet
  → Cloudflare CDN + WAF + DDoS protection
  → Nginx (reverse proxy + SSL termination)
      ├── /* → Next.js (PM2, :3000)
      ├── /api/* → MedusaJS (PM2, :9000)
      └── /admin/* → Medusa Admin (PM2, :7001)

VPS Internal Services:
  ├── PostgreSQL 16 (internal only, :5432)
  ├── Redis 7 (internal only, :6379)
  ├── MeiliSearch (internal, :7700)
  ├── Netdata (monitoring, :19999)
  └── Certbot (auto-renew SSL)

CI/CD:
  GitHub Actions → SSH deploy → VPS
  → npm ci → build → PM2 reload → db:migrate → smoke test
```

---

## 9. Monitoring & Observability

| Layer | Tool | What is Monitored |
|---|---|---|
| Frontend Performance | Vercel Analytics | Core Web Vitals, TTFB per route |
| Product Analytics | PostHog | Funnels, session replay, conversion |
| Application Errors | Sentry | JS errors, API errors, traces |
| Server Health | Netdata | CPU, RAM, disk I/O, network |
| SEO | Google Search Console | Index coverage, CTR, rankings |
| Uptime | Uptime Kuma (self-hosted) | HTTP health checks every 60s |

---

## 10. Scalability Considerations

The system is designed to scale horizontally at the frontend layer and vertically at the backend during the growth phase. Key decisions that enable this:

- **ISR** on Next.js means product pages serve millions of requests without hitting the database for every view
- **Cloudflare CDN** absorbs static asset traffic before it reaches origin servers
- **Redis** decouples session and queue load from PostgreSQL
- **MedusaJS modularity** allows swapping individual services (search, email, payment) without rewriting business logic
- **Stateless MedusaJS API** allows horizontal scaling behind a load balancer when required

---

*Document Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
