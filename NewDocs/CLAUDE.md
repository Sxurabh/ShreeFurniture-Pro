# CLAUDE.md — Shree Furniture Platform
## AI Agent Context File

> Read this file before writing any code or making any architectural decisions.

---

## What This Project Is

Shree Furniture is an India-first, IKEA-inspired e-commerce furniture platform. It is a **headless commerce** system: the Next.js storefront is completely decoupled from the MedusaJS backend. They communicate only via REST API.

**Monorepo structure:**
- `apps/storefront/` — Next.js 15 (App Router) customer storefront
- `backend/` — MedusaJS v2 headless commerce engine
- `packages/types/` — shared TypeScript interfaces
- `packages/ui/` — shared design tokens

---

## Technology Stack (Non-Negotiable)

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, App Router, TypeScript strict, Tailwind CSS v4, shadcn/ui |
| State | Zustand (client) + TanStack Query v5 (server) |
| Forms | React Hook Form + Zod |
| Backend | MedusaJS v2, Node.js 20 LTS |
| Database | PostgreSQL 16 (Neon.tech → Hetzner VPS) |
| Cache/Queue | Redis 7 (Upstash → VPS) |
| Payments | Razorpay (UPI, Cards, EMI, BNPL) |
| Images | Cloudinary CDN |
| Email | Resend + React Email |
| Search | Algolia (Phase 1) → MeiliSearch (Phase 2) |
| Analytics | PostHog |
| Errors | Sentry |
| Package Manager | pnpm |
| Build System | Turborepo |

---

## Hard Rules — Always Follow

1. **Money is always integers (paise).** ₹1 = 100 paise. Never use floats for prices. `4999900` = ₹49,999.
2. **Auth tokens go in HTTP-only cookies only.** Never `localStorage`, never `sessionStorage`, never JavaScript-accessible storage.
3. **Secrets are environment variables only.** Never hardcode API keys. Never `NEXT_PUBLIC_` prefix for secrets.
4. **TypeScript strict mode is on.** No `any` types without a comment justifying the exception.
5. **Soft deletes only.** Use `deleted_at` timestamp. Never `DELETE` products, customers, or orders from the database.
6. **All webhook handlers must be idempotent.** Check if event was already processed before acting on it.
7. **Explicit image dimensions.** All `<Image>` components need `width` and `height` props to prevent CLS (Core Web Vitals).
8. **Line item prices are snapshots.** `unit_price` is captured at cart-add time. Changing a product price does NOT update existing carts or orders.
9. **ISR revalidation on publish.** When a product is published/updated, call `revalidatePath()` to invalidate the ISR cache.
10. **`NEXT_PUBLIC_` only for truly public values.** Razorpay key ID (`rzp_live_...`) is public. Razorpay secret is not.

---

## What NOT to Build (Already Handled)

Do not reinvent these — they are provided by platform integrations:

| Feature | Already Handled By |
|---|---|
| Product CRUD admin UI | Medusa Admin dashboard |
| Order management admin UI | Medusa Admin dashboard |
| Password hashing | MedusaJS Customer Module (bcrypt) |
| JWT issuance | MedusaJS auth endpoints |
| Cart database logic | MedusaJS Cart Module |
| Order creation logic | MedusaJS (`carts.complete()`) |
| Razorpay payment capture | `medusa-payment-razorpay` plugin |
| Image CDN delivery | Cloudinary |
| Email delivery | Resend |
| Search indexing | Algolia |
| Error tracking | Sentry SDK |
| Analytics | PostHog SDK |

---

## Out of Scope for Current Phase (MVP)

Do not propose or build these — they are Phase 2/3:
- Reviews & Ratings
- Loyalty / Referral program
- AR / 3D product viewer
- GST-compliant invoicing (CGST/SGST breakdown)
- SMS notifications
- Multi-currency
- Bulk CSV product import
- Discount codes / promotions
- Inventory reservation on cart add
- MeiliSearch (using Algolia for now)
- VPS migration (using Vercel + Railway free tier)

---

## Key File Locations

| File | Purpose |
|---|---|
| `apps/storefront/lib/medusa/client.ts` | Typed Medusa API client |
| `apps/storefront/store/cart-store.ts` | Zustand cart state |
| `apps/storefront/app/api/webhooks/razorpay/route.ts` | Razorpay webhook handler |
| `backend/medusa-config.ts` | MedusaJS plugin configuration |
| `backend/src/subscribers/` | Event-driven handlers |
| `packages/types/src/index.ts` | Shared TypeScript types |
| `packages/ui/src/tokens.ts` | Design tokens |

---

## Context Documents (Read When Relevant)

| Doc | When to Read |
|---|---|
| `docs/01-product-requirements.md` | Before building any feature — check scope |
| `docs/04-system-architecture.md` | Before making infra or rendering decisions |
| `docs/05-database-schema.md` | Before writing any DB query or migration |
| `docs/06-api-contracts.md` | Before calling any Medusa API endpoint |
| `docs/07-monorepo-structure.md` | Before creating new files or folders |
| `docs/08-cart-pricing-engine-spec.md` | Before touching cart, pricing, or checkout logic |
| `docs/09-engineering-scope-definition.md` | When unsure if something should be built |
| `docs/10-development-phases.md` | To check what milestone we're in |

---

*Shree Furniture | v1.0 — Q1 2026*
