# STATUS.md — Shree Furniture Platform
## Current Build State — Updated Before Every AI Session

> **AI Agent Instruction:** Read this file BEFORE writing any code.
> If a component is marked ✅ DONE — do not regenerate it. Extend or edit the existing file instead.
> If marked 🚧 IN PROGRESS — ask the user for the current file before modifying.
> If marked ⬜ NOT STARTED — you can scaffold from scratch.

---

## 📍 Current Phase

**Phase:** 1 — Foundation
**Current Week:** 1
**Active Milestone:** Monorepo setup + MedusaJS backend live

> Update this section at the start of each work session.

---

## 🗂 Infrastructure & Config

| Item | Status | Notes |
|---|---|---|
| pnpm + Turborepo monorepo init | ⬜ NOT STARTED | |
| `apps/storefront/` scaffold (Next.js 15) | ⬜ NOT STARTED | |
| `backend/` scaffold (MedusaJS v2) | ⬜ NOT STARTED | |
| `packages/types/` initial interfaces | ⬜ NOT STARTED | |
| `packages/ui/` design tokens | ⬜ NOT STARTED | |
| Docker Compose (PostgreSQL + Redis) | ⬜ NOT STARTED | |
| `.env.example` | ⬜ NOT STARTED | |
| GitHub repo + branch protection | ⬜ NOT STARTED | |
| CI pipeline (`ci.yml`) | ⬜ NOT STARTED | |
| Deploy pipeline (`deploy.yml`) | ⬜ NOT STARTED | |

---

## 🗄 Backend (MedusaJS v2)

| Item | Status | Notes |
|---|---|---|
| MedusaJS deployed to Railway | ⬜ NOT STARTED | |
| PostgreSQL (Neon.tech) connected | ⬜ NOT STARTED | |
| Redis (Upstash) connected | ⬜ NOT STARTED | |
| India/INR region seeded | ⬜ NOT STARTED | |
| Sample products + collections seeded | ⬜ NOT STARTED | |
| Cloudinary plugin configured | ⬜ NOT STARTED | |
| Razorpay plugin configured (sandbox) | ⬜ NOT STARTED | |
| Resend plugin configured | ⬜ NOT STARTED | |
| Algolia index + sync subscriber | ⬜ NOT STARTED | |
| `medusa-config.ts` | ⬜ NOT STARTED | |
| Wishlist module | ⬜ NOT STARTED | |
| `subscribers/order-placed.ts` | ⬜ NOT STARTED | |
| `subscribers/order-shipped.ts` | ⬜ NOT STARTED | |
| `subscribers/low-stock.ts` | ⬜ NOT STARTED | |
| `subscribers/algolia-sync.ts` | ⬜ NOT STARTED | |
| `workflows/fulfillment-workflow.ts` | ⬜ NOT STARTED | |

---

## 🖥 Storefront — Layout & Navigation

| Item | Status | Notes |
|---|---|---|
| Root layout (`app/layout.tsx`) | ⬜ NOT STARTED | |
| `Header.tsx` | ⬜ NOT STARTED | |
| `Footer.tsx` | ⬜ NOT STARTED | |
| `Navigation.tsx` | ⬜ NOT STARTED | |
| `MobileMenu.tsx` | ⬜ NOT STARTED | |
| `Breadcrumb.tsx` | ⬜ NOT STARTED | |
| Store route group layout | ⬜ NOT STARTED | |
| Checkout route group layout | ⬜ NOT STARTED | |
| Account route group layout + auth guard | ⬜ NOT STARTED | |

---

## 🖥 Storefront — Pages

| Page | Status | Notes |
|---|---|---|
| Homepage (`/`) | ⬜ NOT STARTED | ISR 60s |
| Product Listing (`/collections/[handle]`) | ⬜ NOT STARTED | ISR 60s |
| Product Detail (`/products/[handle]`) | ⬜ NOT STARTED | ISR 60s |
| Search (`/search`) | ⬜ NOT STARTED | CSR |
| Cart (`/cart`) | ⬜ NOT STARTED | CSR |
| Checkout — Address | ⬜ NOT STARTED | |
| Checkout — Shipping | ⬜ NOT STARTED | |
| Checkout — Payment | ⬜ NOT STARTED | |
| Order Confirmation (`/order/confirm/[id]`) | ⬜ NOT STARTED | SSR |
| Account — Login | ⬜ NOT STARTED | |
| Account — Register | ⬜ NOT STARTED | |
| Account — Orders | ⬜ NOT STARTED | |
| Account — Wishlist | ⬜ NOT STARTED | |
| Account — Addresses | ⬜ NOT STARTED | |

---

## 🧩 Storefront — Components

| Component | Status | File Path | Notes |
|---|---|---|---|
| `ProductCard.tsx` | ⬜ NOT STARTED | `components/product/` | |
| `ProductGallery.tsx` | ⬜ NOT STARTED | `components/product/` | |
| `ProductVariants.tsx` | ⬜ NOT STARTED | `components/product/` | |
| `ProductBadge.tsx` | ⬜ NOT STARTED | `components/product/` | |
| `ProductSpecsTable.tsx` | ⬜ NOT STARTED | `components/product/` | |
| `CartDrawer.tsx` | ⬜ NOT STARTED | `components/cart/` | |
| `CartItem.tsx` | ⬜ NOT STARTED | `components/cart/` | |
| `CartSummary.tsx` | ⬜ NOT STARTED | `components/cart/` | |
| `AddressForm.tsx` | ⬜ NOT STARTED | `components/checkout/` | |
| `ShippingOptions.tsx` | ⬜ NOT STARTED | `components/checkout/` | |
| `PaymentStep.tsx` | ⬜ NOT STARTED | `components/checkout/` | |
| `CheckoutProgress.tsx` | ⬜ NOT STARTED | `components/checkout/` | |
| `SearchBar.tsx` | ⬜ NOT STARTED | `components/search/` | |
| `SearchOverlay.tsx` | ⬜ NOT STARTED | `components/search/` | |
| `SearchResults.tsx` | ⬜ NOT STARTED | `components/search/` | |
| `PriceDisplay.tsx` | ⬜ NOT STARTED | `components/shared/` | |
| `SkeletonCard.tsx` | ⬜ NOT STARTED | `components/shared/` | |
| `ErrorBoundary.tsx` | ⬜ NOT STARTED | `components/shared/` | |

---

## 🪝 Storefront — Lib / Hooks / Store

| Item | Status | Notes |
|---|---|---|
| `lib/medusa/client.ts` | ⬜ NOT STARTED | v2 SDK client |
| `lib/medusa/products.ts` | ⬜ NOT STARTED | |
| `lib/medusa/cart.ts` | ⬜ NOT STARTED | |
| `lib/medusa/orders.ts` | ⬜ NOT STARTED | |
| `lib/medusa/customers.ts` | ⬜ NOT STARTED | |
| `lib/algolia/client.ts` | ⬜ NOT STARTED | |
| `lib/utils/price.ts` | ⬜ NOT STARTED | paise utils |
| `lib/utils/date.ts` | ⬜ NOT STARTED | |
| `lib/utils/validators.ts` | ⬜ NOT STARTED | PIN, phone |
| `store/cart-store.ts` | ⬜ NOT STARTED | Zustand |
| `store/ui-store.ts` | ⬜ NOT STARTED | Zustand |
| `hooks/useCart.ts` | ⬜ NOT STARTED | TanStack Query |
| `hooks/useProducts.ts` | ⬜ NOT STARTED | TanStack Query |
| `hooks/useCustomer.ts` | ⬜ NOT STARTED | TanStack Query |
| Razorpay webhook handler | ⬜ NOT STARTED | `app/api/webhooks/razorpay/route.ts` |

---

## 🧪 Tests

| Test File | Status | Notes |
|---|---|---|
| `price.test.ts` | ⬜ NOT STARTED | paise arithmetic, GST |
| `cart-store.test.ts` | ⬜ NOT STARTED | Zustand optimistic updates |
| `razorpay-webhook.test.ts` | ⬜ NOT STARTED | HMAC verify + idempotency |
| Playwright purchase flow E2E | ⬜ NOT STARTED | Critical path |

---

## 📝 How to Update This File

After each coding session, update the status of completed items:

- `⬜ NOT STARTED` → `🚧 IN PROGRESS` → `✅ DONE`
- Add a **Notes** column entry if there's anything the next AI session needs to know about the implementation (e.g., "uses custom hook pattern, see useCart.ts for reference", or "TODO: add pagination").

---

*Shree Furniture | v1.0 — Q1 2026 | Update this file every session*
