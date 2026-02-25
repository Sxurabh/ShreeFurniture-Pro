# 07 — Monorepo Structure
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Tooling:** pnpm workspaces + Turborepo

---

## 1. Overview

The project uses a **pnpm monorepo** with Turborepo for task orchestration. All deployable units (storefront, backend, admin) live in the same repository but are independently deployable. Shared code (TypeScript types, design tokens) lives in `packages/`.

```
shree-furniture/                     # Monorepo root
├── apps/
│   ├── storefront/                  # Next.js 15 — customer-facing storefront
│   └── admin/                       # Medusa Admin UI (or future custom admin)
├── backend/                         # MedusaJS v2 commerce engine
├── packages/
│   ├── types/                       # Shared TypeScript interfaces
│   └── ui/                          # Shared design tokens (colours, spacing, typography)
├── infrastructure/                  # Nginx config, Docker, CI/CD workflows
├── package.json                     # Root workspace config
├── pnpm-workspace.yaml
├── turbo.json                       # Turborepo pipeline config
├── .env.example                     # Root env variable reference (no secrets)
├── CLAUDE.md                        # AI agent context file
├── CONVENTIONS.md                   # Naming, coding, and folder conventions
└── DECISIONS.md                     # Architecture Decision Records (ADR log)
```

---

## 2. `apps/storefront/` — Next.js 15 Storefront

```
apps/storefront/
├── app/
│   ├── layout.tsx                   # Root layout: fonts, providers, global CSS
│   ├── (store)/                     # Route group: Header + Footer + CartDrawer
│   │   ├── layout.tsx
│   │   ├── page.tsx                 # Homepage (ISR: revalidate 60s)
│   │   ├── collections/
│   │   │   └── [handle]/
│   │   │       ├── page.tsx         # PLP (ISR: revalidate 60s)
│   │   │       └── loading.tsx      # Skeleton loader
│   │   ├── products/
│   │   │   └── [handle]/
│   │   │       ├── page.tsx         # PDP (ISR: revalidate 60s)
│   │   │       ├── loading.tsx
│   │   │       └── not-found.tsx
│   │   ├── search/
│   │   │   └── page.tsx             # Search results (CSR)
│   │   └── cart/
│   │       └── page.tsx             # Cart page (CSR)
│   ├── (checkout)/                  # Route group: minimal checkout layout
│   │   ├── layout.tsx
│   │   ├── checkout/
│   │   │   ├── page.tsx             # Step 1: Address
│   │   │   ├── shipping/page.tsx    # Step 2: Shipping method
│   │   │   └── payment/page.tsx     # Step 3: Payment + Razorpay
│   │   └── order/
│   │       └── confirm/
│   │           └── [id]/page.tsx    # Post-payment confirmation (SSR)
│   ├── (account)/                   # Route group: auth-guarded layout
│   │   ├── layout.tsx               # Auth guard middleware + account sidebar
│   │   └── account/
│   │       ├── login/page.tsx
│   │       ├── register/page.tsx
│   │       ├── forgot-password/page.tsx
│   │       ├── orders/
│   │       │   ├── page.tsx         # My Orders list
│   │       │   └── [id]/page.tsx    # Order detail
│   │       ├── wishlist/page.tsx
│   │       ├── addresses/page.tsx
│   │       └── settings/page.tsx
│   └── api/
│       └── webhooks/
│           └── razorpay/
│               └── route.ts         # POST: Razorpay payment webhook handler
│
├── components/
│   ├── ui/                          # shadcn/ui base: Button, Input, Dialog, Badge, etc.
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── Navigation.tsx
│   │   ├── MobileMenu.tsx
│   │   └── Breadcrumb.tsx
│   ├── product/
│   │   ├── ProductCard.tsx          # Grid card: thumbnail, name, price, wishlist icon
│   │   ├── ProductGallery.tsx       # Image gallery with thumbnail strip
│   │   ├── ProductVariants.tsx      # Colour swatches + size selectors
│   │   ├── ProductBadge.tsx         # "New", "Sale", "Only 3 left" badges
│   │   └── ProductSpecsTable.tsx    # L×W×H, weight, material table
│   ├── cart/
│   │   ├── CartDrawer.tsx           # Slide-in cart panel
│   │   ├── CartItem.tsx             # Line item with quantity stepper + remove
│   │   └── CartSummary.tsx          # Subtotal, shipping, GST, total
│   ├── checkout/
│   │   ├── AddressForm.tsx          # React Hook Form + Zod validated address
│   │   ├── ShippingOptions.tsx      # Available shipping method selector
│   │   ├── PaymentStep.tsx          # Razorpay modal trigger
│   │   └── CheckoutProgress.tsx     # Step indicator (Address → Shipping → Payment)
│   ├── search/
│   │   ├── SearchBar.tsx            # Header search input + overlay trigger
│   │   ├── SearchOverlay.tsx        # Full-screen search UI
│   │   └── SearchResults.tsx        # Algolia InstantSearch result list
│   └── shared/
│       ├── SkeletonCard.tsx         # Product card skeleton loader
│       ├── ErrorBoundary.tsx        # React error boundary wrapper
│       ├── EmptyState.tsx           # Empty cart / no results UI
│       └── PriceDisplay.tsx         # Formats paise → ₹ with GST note
│
├── lib/
│   ├── medusa/
│   │   ├── client.ts                # Typed MedusaJS storefront client instance
│   │   ├── products.ts              # getProduct(), getProducts(), getCollections()
│   │   ├── cart.ts                  # createCart(), addItem(), removeItem(), etc.
│   │   ├── orders.ts                # getOrder(), getOrders()
│   │   └── customers.ts             # login(), register(), getMe(), etc.
│   ├── algolia/
│   │   └── client.ts                # Algolia InstantSearch client initialisation
│   └── utils/
│       ├── price.ts                 # paiseToInr(), formatPrice(), calculateGST()
│       ├── date.ts                  # formatDate(), getDeliveryEstimate()
│       └── validators.ts            # PIN code format, phone number validation
│
├── store/
│   ├── cart-store.ts                # Zustand: cart items, optimistic updates
│   └── ui-store.ts                  # Zustand: cartDrawerOpen, searchOverlayOpen, etc.
│
├── hooks/
│   ├── useCart.ts                   # TanStack Query wrapper for cart operations
│   ├── useProducts.ts               # TanStack Query wrapper for product fetching
│   └── useCustomer.ts               # TanStack Query wrapper for auth state
│
├── public/
│   ├── fonts/                       # Self-hosted font files
│   └── images/                      # Static images (logo, placeholder, OG image)
│
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## 3. `backend/` — MedusaJS v2

```
backend/
├── src/
│   ├── modules/
│   │   └── wishlist/                # Custom wishlist module
│   │       ├── index.ts
│   │       ├── models/
│   │       │   └── wishlist-item.ts
│   │       └── service.ts
│   ├── subscribers/
│   │   ├── order-placed.ts          # order.placed → Resend confirmation email
│   │   ├── order-shipped.ts         # order.shipment_created → shipping email
│   │   ├── low-stock.ts             # inventory.low_stock → admin alert email
│   │   └── algolia-sync.ts          # product.created/updated → Algolia index
│   ├── workflows/
│   │   └── fulfillment-workflow.ts  # Custom order fulfilment logic
│   └── api/
│       └── store/
│           ├── wishlist/
│           │   └── route.ts         # GET, POST /store/wishlist
│           └── wishlist/
│               └── [id]/
│                   └── route.ts     # DELETE /store/wishlist/:id
├── medusa-config.ts                 # Plugins, modules, database, redis config
├── tsconfig.json
└── package.json
```

---

## 4. `packages/types/` — Shared TypeScript Types

```typescript
// packages/types/src/index.ts

export interface Product {
  id: string
  title: string
  handle: string
  thumbnail: string | null
  description: string | null
  status: 'draft' | 'published' | 'proposed' | 'rejected'
  collection: ProductCollection | null
  variants: ProductVariant[]
  images: ProductImage[]
  tags: ProductTag[]
  room_type: string | null
  assembly_required: boolean
}

export interface ProductVariant {
  id: string
  title: string
  sku: string | null
  inventory_quantity: number
  prices: MoneyAmount[]
  options: { value: string }[]
}

export interface MoneyAmount {
  currency_code: string
  amount: number  // paise
}

export interface Cart {
  id: string
  email: string | null
  items: LineItem[]
  subtotal: number
  shipping_total: number
  tax_total: number
  total: number
  region: { currency_code: string }
}

export interface LineItem {
  id: string
  title: string
  thumbnail: string | null
  quantity: number
  unit_price: number
  total: number
  variant: ProductVariant
}

export interface Order {
  id: string
  display_id: number
  status: OrderStatus
  fulfillment_status: FulfillmentStatus
  payment_status: PaymentStatus
  total: number
  currency_code: string
  created_at: string
  items: LineItem[]
  shipping_address: Address
}

export type OrderStatus = 'pending' | 'completed' | 'archived' | 'canceled'
export type FulfillmentStatus = 'not_fulfilled' | 'fulfilled' | 'shipped' | 'canceled'
export type PaymentStatus = 'not_paid' | 'awaiting' | 'captured' | 'refunded'
```

---

## 5. `packages/ui/` — Shared Design Tokens

```typescript
// packages/ui/src/tokens.ts

export const colors = {
  brand: {
    primary: '#1A1A1A',    // near-black — primary text, CTAs
    accent: '#C8A96E',     // warm gold — hover states, highlights
    warm: '#F5F0E8',       // off-white — backgrounds
    subtle: '#E8E0D0',     // light tan — card borders, dividers
  },
  semantic: {
    success: '#2D7A4F',
    error: '#C0392B',
    warning: '#E67E22',
    info: '#2980B9',
  },
  text: {
    primary: '#1A1A1A',
    secondary: '#5A5A5A',
    muted: '#9A9A9A',
    inverse: '#FFFFFF',
  }
}

export const typography = {
  fontFamily: {
    heading: 'Playfair Display, serif',
    body: 'Inter, sans-serif',
  },
  fontSize: { xs: '12px', sm: '14px', base: '16px', lg: '18px', xl: '20px', '2xl': '24px', '3xl': '30px', '4xl': '36px' }
}

export const spacing = {
  page: { x: '24px', maxWidth: '1280px' }
}
```

---

## 6. `infrastructure/` — DevOps & Config

```
infrastructure/
├── nginx/
│   └── shree-furniture.conf         # Nginx reverse proxy for VPS Phase 2
├── docker-compose.yml               # Local dev: PostgreSQL 16 + Redis 7
└── .github/
    └── workflows/
        ├── ci.yml                   # PR: tsc + eslint + vitest + build check
        └── deploy.yml               # main merge: deploy to Vercel + VPS
```

---

## 7. Root Config Files

| File | Purpose |
|---|---|
| `package.json` | Workspace root: `turbo`, `pnpm` scripts |
| `pnpm-workspace.yaml` | Declares `apps/*`, `backend`, `packages/*` as workspaces |
| `turbo.json` | Pipeline: `build`, `dev`, `lint`, `test` tasks with caching |
| `.env.example` | Documents all required env vars (no values) |
| `.gitignore` | Excludes: `node_modules`, `.env*`, `.next`, `dist` |
| `CLAUDE.md` | AI agent context: stack summary, conventions, what NOT to do |
| `CONVENTIONS.md` | Naming, import, component, and file conventions |
| `DECISIONS.md` | ADR log: why each major tech choice was made |

---

## 8. Turbo Pipeline (turbo.json)

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "outputs": ["coverage/**"]
    }
  }
}
```

---

## 9. Key Scripts (Root package.json)

```json
{
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "db:migrate": "cd backend && npx medusa db:migrate",
    "db:seed": "cd backend && npx medusa exec ./src/seed.ts",
    "type-check": "turbo run type-check"
  }
}
```

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
