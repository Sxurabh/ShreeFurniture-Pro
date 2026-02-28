# CLAUDE.md — Shree Furniture Platform
## Root-Level AI Agent Context File
## Auto-loaded by: Claude Code · Cursor · Antigravity · Windsurf

> **READ THIS FIRST.** This file is the single source of truth for all AI agents working on this codebase.
> Before writing any code, check: STATUS.md (what's built) → CLAUDE.md (rules) → relevant NewDocs doc.

---

## What This Project Is

Shree Furniture is an India-first, IKEA-inspired headless e-commerce platform.
The Next.js storefront is **completely decoupled** from the MedusaJS backend. They communicate **only via REST API**.

**You are in a pnpm + Turborepo monorepo.**

```
apps/storefront/     → Next.js 15 (App Router) customer-facing storefront
backend/             → MedusaJS v2 headless commerce engine
packages/types/      → Shared TypeScript interfaces (Product, Cart, Order, Address)
packages/ui/         → Shared design tokens (colours, typography, spacing)
infrastructure/      → Docker Compose (local), Nginx (VPS Phase 2), GitHub Actions
```

---

## ⚡ Before You Write Any Code — Check These First

| Step | File | Why |
|---|---|---|
| 1 | `STATUS.md` | See what's already built — never regenerate existing code |
| 2 | `PREFERENCES.md` | Owner's design taste — overrides your UI defaults for every component |
| 3 | `REJECTIONS.md` | Patterns the owner has explicitly rejected — never suggest these |
| 4 | `STACK-CHANGES.md` | Mid-project tech changes — entries here supersede the stack table below |
| 5 | `NewDocs/09-engineering-scope-definition.md` | Confirm the feature is in scope before starting |
| 6 | `NewDocs/10-development-phases.md` | Confirm the feature is in the **current phase** |
| 7 | `NewDocs/MEDUSA-V2-PATTERNS.md` | If touching backend — use v2 patterns, not v1 |

---

## Technology Stack (Non-Negotiable)

| Layer | Technology | Version |
|---|---|---|
| Frontend | Next.js App Router | 15 |
| Language | TypeScript strict mode | 5.x |
| Styling | Tailwind CSS | v4 |
| Components | shadcn/ui | latest |
| Client State | Zustand | 4.x |
| Server State | TanStack Query | v5 |
| Forms | React Hook Form + Zod | latest |
| Backend | MedusaJS | **v2** (NOT v1) |
| Runtime | Node.js LTS | 20 |
| Database | PostgreSQL | 16 |
| Cache/Queue | Redis | 7 |
| Payments | Razorpay | — |
| Images | Cloudinary CDN | — |
| Email | Resend + React Email | — |
| Search | Algolia (Phase 1) | — |
| Analytics | PostHog | — |
| Errors | Sentry | — |
| Package Manager | pnpm | — |
| Build System | Turborepo | — |

---

## Hard Rules — Non-Negotiable

### Money
- **All monetary values are integers in paise.** ₹1 = 100 paise. `4999900` = ₹49,999.
- Never use `float` or `number` with decimal math for prices.
- Convert paise → display string **only** at the render layer using `formatPrice()` from `packages/types/src/utils/price.ts`.

### Security
- **Auth tokens go in HTTP-only cookies only.** Never `localStorage`, `sessionStorage`, or Zustand for tokens.
- **Secrets are env vars only.** Never hardcode. Never `NEXT_PUBLIC_` prefix a secret.
- Razorpay Key ID (`rzp_live_...`) is public → `NEXT_PUBLIC_RAZORPAY_KEY_ID`. Razorpay Secret is not.

### TypeScript
- **Strict mode is on.** No `any` without a `// reason: ...` comment justifying the exception.
- Use `unknown` + type narrowing when type is genuinely unknown.
- Always `import type` for type-only imports.

### Data Integrity
- **Soft deletes only.** Use `deleted_at` timestamp. Never `DELETE` products, customers, or orders.
- **Line item prices are snapshots.** `unit_price` is captured at cart-add time. Never update it retroactively.
- **Webhook handlers must be idempotent.** Always check if the event was already processed before acting.

### Performance
- **Explicit image dimensions always.** All `<Image>` must have `width` and `height` to prevent CLS.
- **ISR revalidation on publish.** Call `revalidatePath()` when a product is published/updated.

### MedusaJS v2
- **Use v2 SDK patterns.** See `NewDocs/MEDUSA-V2-PATTERNS.md` before touching the backend.
- Never use v1 patterns (`medusa.products.list()`, `medusa.carts.create()` etc.) — they will break silently.

---

## What NOT to Build — Already Handled by Platforms

| Feature | Handled By | Action |
|---|---|---|
| Product/Order admin UI | Medusa Admin dashboard | Use as-is |
| Password hashing | MedusaJS Customer Module | Do not reimplement |
| JWT issuance/validation | MedusaJS auth endpoints | Do not reimplement |
| Cart CRUD logic | MedusaJS Cart Module | Call API; do not rewrite |
| Order creation | MedusaJS (`carts/:id/complete`) | Do not rewrite |
| Razorpay payment capture | `medusa-payment-razorpay` plugin | Configure plugin only |
| Image CDN delivery | Cloudinary | Do not build custom image server |
| Email delivery | Resend | Do not build SMTP server |
| Search indexing | Algolia | Do not build search engine |
| Error monitoring | Sentry SDK | Instrument only |

---

## Out of Scope — Do Not Propose for MVP

These are Phase 2/3 features. If you find yourself about to implement any of these, **stop and re-read this file.**

- Reviews & Ratings
- Loyalty / Referral program
- AR / 3D product viewer
- GST-compliant invoicing (CGST/SGST breakdown)
- SMS notifications
- Multi-currency support
- Bulk CSV product import
- Discount codes / promotions engine
- Inventory reservation on cart add
- MeiliSearch (using Algolia for MVP)
- VPS migration (using Vercel + Railway free tier for MVP)

---

## Key File Locations

| File | Purpose |
|---|---|
| `apps/storefront/lib/medusa/client.ts` | Typed Medusa v2 API client |
| `apps/storefront/store/cart-store.ts` | Zustand cart state + optimistic updates |
| `apps/storefront/app/api/webhooks/razorpay/route.ts` | Razorpay webhook handler (HMAC verify + idempotent) |
| `backend/medusa-config.ts` | MedusaJS plugin registration |
| `backend/src/subscribers/` | Event-driven handlers (order, algolia, stock) |
| `backend/src/modules/wishlist/` | Custom wishlist Medusa module |
| `packages/types/src/index.ts` | Shared TypeScript interfaces |
| `packages/ui/src/tokens.ts` | Design tokens |

---

## Context Documents Index

| When You're About To... | Read This First |
|---|---|
| Build any feature | `NewDocs/01-product-requirements.md` — check feature spec + acceptance criteria |
| Make any infra or rendering decision | `NewDocs/04-system-architecture.md` |
| Write any DB query or migration | `NewDocs/05-database-schema.md` |
| Call any Medusa API endpoint | `NewDocs/06-api-contracts.md` |
| Create new files or folders | `NewDocs/07-monorepo-structure.md` |
| Touch cart, pricing, or checkout | `NewDocs/08-cart-pricing-engine-spec.md` |
| Unsure if something should be built | `NewDocs/09-engineering-scope-definition.md` |
| Check current milestone | `NewDocs/10-development-phases.md` |
| Write any test | `NewDocs/12-testing-strategy.md` |
| Write any backend code | `NewDocs/MEDUSA-V2-PATTERNS.md` |
| Hit something unexpected | `NewDocs/KNOWN-ISSUES.md` |
| Write any code at all | `NewDocs/CONVENTIONS.md` |

---

## Design Tokens Quick Reference

```
Brand Primary:  #1A1A1A  (near-black — text, CTAs)
Brand Accent:   #C8A96E  (warm gold — hover, highlights)
Brand Warm:     #F5F0E8  (off-white — backgrounds)
Brand Subtle:   #E8E0D0  (light tan — borders, dividers)
Heading Font:   Playfair Display, serif
Body Font:      Inter, sans-serif
Max Width:      1280px
```

---

*Shree Furniture | v1.1 — Updated Q1 2026*