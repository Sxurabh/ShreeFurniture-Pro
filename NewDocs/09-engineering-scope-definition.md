# 09 — Engineering Scope Definition
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Purpose:** Define exactly what engineers build, extend, configure vs. leave to platform defaults

---

## 1. Purpose

This document prevents scope creep and ensures the AI agent and engineers build *only* what is required. Everything in "Do Not Build" is either handled by a third-party platform, deferred to a later phase, or intentionally out of scope.

---

## 2. What We BUILD (Custom Code)

### 2.1 Storefront (`apps/storefront/`)

| Component | What We Build | Notes |
|---|---|---|
| All page layouts | App Router pages, route groups, layouts | All custom React/Next.js code |
| Product Card | `ProductCard.tsx` with wishlist toggle, discount badge | Custom UI |
| Product Gallery | `ProductGallery.tsx` with thumbnail strip, pinch-zoom | Custom UI |
| Variant Selector | `ProductVariants.tsx` — colour swatches, size buttons | Custom UI |
| Cart Drawer | `CartDrawer.tsx` with quantity steppers, remove | Custom UI |
| Checkout flow | `AddressForm`, `ShippingOptions`, `PaymentStep` | Multi-step custom flow |
| Razorpay modal | Integration in `PaymentStep.tsx` | Thin wrapper over Razorpay JS SDK |
| Search overlay | `SearchOverlay.tsx`, `SearchResults.tsx` | Algolia InstantSearch components |
| Zustand stores | `cart-store.ts`, `ui-store.ts` | Client state management |
| TanStack Query hooks | `useCart.ts`, `useProducts.ts`, `useCustomer.ts` | Server state wrappers |
| Medusa client | `lib/medusa/` — typed fetch wrappers | Typed API layer |
| Price utilities | `lib/utils/price.ts` | paise → INR formatting |
| Webhook handler | `/api/webhooks/razorpay/route.ts` | HMAC verify + idempotent processing |
| SEO metadata | `generateMetadata()` on PLP and PDP pages | Next.js Metadata API |
| JSON-LD schemas | Product, BreadcrumbList schemas in PDP | Structured data |
| ISR revalidation | On-demand via `revalidatePath()` triggered by Medusa subscriber | Cache invalidation |

---

### 2.2 Backend (`backend/`)

| Component | What We Build | Notes |
|---|---|---|
| Wishlist module | `modules/wishlist/` — model, service, API routes | Full custom Medusa module |
| Order email subscriber | `subscribers/order-placed.ts` | Hooks into `order.placed` event |
| Shipping email subscriber | `subscribers/order-shipped.ts` | Hooks into `order.shipment_created` |
| Low stock subscriber | `subscribers/low-stock.ts` | Hooks into `inventory.low_stock` |
| Algolia sync subscriber | `subscribers/algolia-sync.ts` | Syncs product changes to Algolia |
| Fulfilment workflow | `workflows/fulfillment-workflow.ts` | Custom Medusa workflow |
| medusa-config.ts | Plugin registration, module config | Configuration file |

---

### 2.3 Shared Packages

| Package | What We Build |
|---|---|
| `packages/types/` | TypeScript interfaces for Product, Cart, Order, Customer, Address |
| `packages/ui/` | Design token constants: colours, typography, spacing |

---

### 2.4 Infrastructure

| Item | What We Build |
|---|---|
| `docker-compose.yml` | Local dev: PostgreSQL + Redis containers |
| `.github/workflows/ci.yml` | PR pipeline: tsc, eslint, vitest, build check |
| `.github/workflows/deploy.yml` | Deploy pipeline: Vercel + VPS SSH deploy |
| `infrastructure/nginx/shree-furniture.conf` | Phase 2 reverse proxy config |

---

## 3. What We CONFIGURE (Not Build)

These are third-party platforms or tools we set up and configure, not code from scratch.

| Platform | Configuration Required |
|---|---|
| **MedusaJS v2** | Products, collections, regions (India/INR), shipping options, tax rates, admin users |
| **Razorpay** | Account setup, webhook URL, HMAC secret, payment methods enabled (UPI, EMI, BNPL) |
| **Cloudinary** | Upload presets, auto-format (WebP/AVIF), responsive breakpoints, folder structure |
| **Algolia** | Index name `products`, searchable attributes, facets, ranking rules |
| **Resend** | Domain verification (DKIM/SPF), React Email templates, sender address |
| **PostHog** | Project setup, custom event names, funnels (browse → cart → checkout → purchase) |
| **Sentry** | DSN, source maps upload, alert rules |
| **Vercel** | Project linked to monorepo, env vars, domain, build settings (`apps/storefront`) |
| **Railway.app** | MedusaJS service, env vars, health check path `/health` |
| **Neon.tech** | Database branch setup, connection string |
| **Upstash** | Redis database, REST API token |
| **Cloudflare** | DNS records, SSL mode (Full Strict), WAF rules, caching rules |

---

## 4. What We DO NOT BUILD (Platform Defaults)

These features are provided out-of-the-box by MedusaJS or other platforms and **must not be rebuilt from scratch**.

| Feature | Provided By | Action |
|---|---|---|
| Admin product CRUD UI | Medusa Admin dashboard | Use as-is |
| Admin order management UI | Medusa Admin dashboard | Use as-is |
| Admin customer management | Medusa Admin dashboard | Use as-is |
| JWT auth (customers) | MedusaJS Customer Module | Configure, do not reimplement |
| Password hashing (bcrypt) | MedusaJS | Do not reimplement |
| Cart CRUD logic | MedusaJS Cart Module | Call API endpoints; do not rewrite |
| Order creation logic | MedusaJS Order Module | Trigger via `carts.complete()`; do not rewrite |
| Razorpay payment capture | Razorpay + `medusa-payment-razorpay` plugin | Configure plugin; do not rewrite payment flow |
| Image CDN delivery | Cloudinary | Do not build custom image server |
| Email delivery | Resend | Do not build SMTP server |
| Full-text search indexing | Algolia | Do not build custom search engine |
| Error monitoring | Sentry SDK | Instrument app; do not build custom error logger |
| Analytics event collection | PostHog SDK | Instrument; do not build custom analytics |

---

## 5. Phase Boundaries — What NOT to Build in MVP

The following are explicitly **deferred**. If an AI agent proposes implementing these during MVP, it is out of scope.

| Feature | Phase | Reason Deferred |
|---|---|---|
| Reviews & Ratings | Phase 3 | Requires moderation; low MVP priority |
| Loyalty / Referral system | Phase 3 | Complexity outweighs MVP value |
| AR / 3D product viewer | Phase 3 | React Three Fiber; high effort |
| GST-compliant invoicing | Phase 3 | Regulatory complexity; legal requirement |
| SMS notifications | Phase 3 | Requires SMS gateway setup |
| Multi-currency support | Phase 3 | India-only for MVP |
| Bulk CSV product import | Phase 2 | Not needed until catalogue grows |
| MeiliSearch (self-hosted) | Phase 2 | Algolia free tier sufficient for MVP |
| VPS migration (Hetzner) | Phase 2 | Free tier infrastructure is sufficient for MVP |
| Inventory reservation | Phase 2 | Accepted race condition for MVP |
| Advanced discount engine | Phase 2 | Launch first, promotions come later |

---

## 6. Technical Constraints (Non-Negotiable)

| Constraint | Rule |
|---|---|
| Monetary values | Always integers (paise). Never floats. |
| Auth storage | JWT in HTTP-only cookies only. Never localStorage. |
| Secrets | Environment variables only. Never hardcoded. Never committed to Git. |
| TypeScript | Strict mode (`"strict": true`). No `any` types without explicit comment justification. |
| Soft deletes | Use `deleted_at` timestamp. Never hard-delete products, customers, or orders. |
| Idempotency | All webhook handlers must check if event was already processed before acting. |
| ISR invalidation | On product publish/update, `revalidatePath()` must be called to invalidate ISR cache. |
| Image dimensions | All `<Image>` components must have explicit `width` and `height` to prevent CLS. |
| Price snapshots | Line item `unit_price` is a snapshot at add time; price changes do not retroactively update carts. |

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
