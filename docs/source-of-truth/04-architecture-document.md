# Shree Furniture — Architecture Document

> **Project:** Shree Furniture E-Commerce Platform
> **Document Type:** Architecture Document
> **Version:** 1.0
> **Date:** February 2026
> **Stack:** Next.js · TypeScript · Supabase · ShadCN UI · Tailwind CSS

---

## Table of Contents

1. [High-Level Architecture](#1-high-level-architecture)
2. [Frontend Architecture](#2-frontend-architecture)
3. [Backend Architecture](#3-backend-architecture)
4. [Database Architecture](#4-database-architecture)
5. [Security Architecture](#5-security-architecture)
6. [Deployment Architecture](#6-deployment-architecture)
7. [Future Scalability Roadmap](#7-future-scalability-roadmap)

---

## 1. High-Level Architecture

### 1.1 Architecture Overview

Shree Furniture uses a **Jamstack-inspired, serverless architecture** that combines Next.js App Router with Supabase's Backend-as-a-Service (BaaS) platform. This approach delivers the performance and SEO characteristics of static sites with the dynamic capabilities needed for a full e-commerce platform — all without managing any servers.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET                                        │
└─────────────────────────────────┬────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         VERCEL EDGE NETWORK (CDN)                            │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                    Next.js 14 Application                           │   │
│   │                                                                     │   │
│   │  ┌──────────────────────┐     ┌──────────────────────────────────┐  │   │
│   │  │  STOREFRONT          │     │  ADMIN PANEL                     │  │   │
│   │  │  /app/(store)/       │     │  /app/admin/                     │  │   │
│   │  │                      │     │                                  │  │   │
│   │  │  • Homepage          │     │  • Dashboard                     │  │   │
│   │  │  • Categories/PLP    │     │  • Products CRUD                 │  │   │
│   │  │  • Product PDP       │     │  • Orders Management             │  │   │
│   │  │  • Search            │     │  • Inventory                     │  │   │
│   │  │  • Cart/Checkout     │     │  • Customers                     │  │   │
│   │  │  • Account           │     │  • Analytics                     │  │   │
│   │  │  • Order Tracking    │     │  • Promotions                    │  │   │
│   │  └──────────────────────┘     └──────────────────────────────────┘  │   │
│   │                                                                     │   │
│   │  ┌──────────────────┐  ┌────────────────┐  ┌──────────────────┐    │   │
│   │  │  Middleware       │  │  Server        │  │  Route Handlers  │    │   │
│   │  │  (Auth Guard)     │  │  Actions       │  │  /api/webhooks/  │    │   │
│   │  └──────────────────┘  └────────────────┘  └──────────────────┘    │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└──────────────────────────────────────┬───────────────────────────────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
              ▼                        ▼                        ▼
┌─────────────────────┐ ┌──────────────────────┐ ┌─────────────────────────┐
│   SUPABASE PLATFORM │ │   RAZORPAY           │ │   RESEND (Email)        │
│                     │ │                      │ │                         │
│  ┌───────────────┐  │ │  • Payment Modal     │ │  • Order Confirmation   │
│  │  PostgreSQL   │  │ │  • Webhook Events    │ │  • Shipping Updates     │
│  │  (+ RLS)      │  │ │  • Refunds API       │ │  • Password Reset       │
│  └───────────────┘  │ └──────────────────────┘ └─────────────────────────┘
│  ┌───────────────┐  │
│  │  Auth         │  │
│  │  (JWT/OAuth)  │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │  Storage      │  │
│  │  (Image CDN)  │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │  Edge Fns     │  │
│  │  (Deno)       │  │
│  └───────────────┘  │
└─────────────────────┘
```

### 1.2 Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Rendering Strategy** | RSC + Selective CSR | SEO for public pages; interactivity where needed |
| **Auth Pattern** | Supabase Auth via cookie-based sessions | SSR-compatible; no JWT in localStorage |
| **Data Access** | Direct Supabase client in RSC + Server Actions for mutations | Eliminates traditional REST API layer; reduces latency |
| **Payment Security** | Edge Function + signature verification | Keeps Razorpay secret server-side; verifies payment integrity |
| **Image Storage** | Supabase Storage + Next.js Image optimization | Zero-cost CDN; automatic WebP conversion |
| **Admin Access** | Same Next.js app, `/admin` route group | Single codebase; role-based access via middleware + RLS |
| **State Management** | Zustand (client) + TanStack Query (server state) | Minimal boilerplate; zero additional infrastructure |

---

## 2. Frontend Architecture

### 2.1 Project Structure

```
shree-furniture/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (store)/                  # Route group: Storefront
│   │   │   ├── page.tsx              # Homepage
│   │   │   ├── products/
│   │   │   │   ├── page.tsx          # PLP (Product Listing)
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx      # PDP (Product Detail)
│   │   │   ├── search/
│   │   │   │   └── page.tsx
│   │   │   ├── cart/
│   │   │   │   └── page.tsx
│   │   │   ├── checkout/
│   │   │   │   ├── page.tsx
│   │   │   │   └── confirmation/[orderId]/page.tsx
│   │   │   └── account/
│   │   │       ├── layout.tsx        # Auth-protected layout
│   │   │       ├── page.tsx          # Profile
│   │   │       ├── orders/
│   │   │       │   ├── page.tsx
│   │   │       │   └── [id]/page.tsx
│   │   │       └── wishlist/page.tsx
│   │   ├── admin/                    # Route group: Admin Panel
│   │   │   ├── layout.tsx            # Admin auth check layout
│   │   │   ├── page.tsx              # Dashboard
│   │   │   ├── products/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── new/page.tsx
│   │   │   │   └── [id]/edit/page.tsx
│   │   │   ├── orders/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [id]/page.tsx
│   │   │   ├── categories/page.tsx
│   │   │   ├── customers/page.tsx
│   │   │   ├── promotions/page.tsx
│   │   │   └── inventory/page.tsx
│   │   ├── auth/
│   │   │   ├── login/page.tsx
│   │   │   ├── signup/page.tsx
│   │   │   └── callback/route.ts     # OAuth callback
│   │   ├── api/
│   │   │   └── webhooks/
│   │   │       └── razorpay/route.ts
│   │   ├── layout.tsx                # Root layout (Providers)
│   │   ├── not-found.tsx
│   │   └── error.tsx
│   │
│   ├── components/
│   │   ├── ui/                       # ShadCN UI base components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   └── ...
│   │   ├── layout/                   # Layout components
│   │   │   ├── Header/
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── NavMenu.tsx
│   │   │   │   └── CartIcon.tsx
│   │   │   ├── Footer/
│   │   │   └── AdminSidebar/
│   │   ├── products/                 # Product-specific components
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductGrid.tsx
│   │   │   ├── ProductImageGallery.tsx
│   │   │   ├── ProductFilters.tsx
│   │   │   ├── ProductVariantSelector.tsx
│   │   │   └── AddToCartButton.tsx
│   │   ├── cart/
│   │   │   ├── CartDrawer.tsx
│   │   │   ├── CartItem.tsx
│   │   │   └── CartSummary.tsx
│   │   ├── checkout/
│   │   │   ├── AddressForm.tsx
│   │   │   ├── OrderSummary.tsx
│   │   │   ├── CouponInput.tsx
│   │   │   └── RazorpayButton.tsx
│   │   ├── home/
│   │   │   ├── HeroBanner.tsx
│   │   │   ├── FeaturedProducts.tsx
│   │   │   └── CategoryGrid.tsx
│   │   └── admin/
│   │       ├── ProductForm.tsx
│   │       ├── ImageUploader.tsx
│   │       ├── OrdersTable.tsx
│   │       └── RevenueChart.tsx
│   │
│   ├── actions/                      # Next.js Server Actions
│   │   ├── auth.ts
│   │   ├── cart.ts
│   │   ├── checkout.ts
│   │   ├── products.ts
│   │   ├── orders.ts
│   │   └── admin/
│   │       ├── products.ts
│   │       ├── orders.ts
│   │       └── promotions.ts
│   │
│   ├── stores/                       # Zustand stores
│   │   ├── cartStore.ts
│   │   ├── wishlistStore.ts
│   │   └── uiStore.ts
│   │
│   ├── hooks/                        # Custom React hooks
│   │   ├── useCart.ts
│   │   ├── useWishlist.ts
│   │   ├── useProducts.ts
│   │   └── useDebounce.ts
│   │
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts             # Browser client
│   │   │   ├── server.ts             # Server component client
│   │   │   └── middleware.ts         # Middleware client
│   │   ├── validations/
│   │   │   ├── product.ts            # Zod schemas
│   │   │   ├── checkout.ts
│   │   │   └── auth.ts
│   │   ├── razorpay.ts
│   │   ├── resend.ts
│   │   └── utils.ts
│   │
│   ├── types/
│   │   ├── database.ts               # Generated from Supabase (supabase gen types)
│   │   ├── actions.ts
│   │   └── index.ts
│   │
│   └── middleware.ts                 # Next.js Edge Middleware (auth guard)
│
├── supabase/
│   ├── functions/
│   │   ├── create-razorpay-order/
│   │   ├── verify-razorpay-payment/
│   │   └── send-order-email/
│   ├── migrations/                   # Versioned SQL migration files
│   └── seed.sql                      # Development seed data
│
├── emails/                           # React Email templates
│   ├── OrderConfirmation.tsx
│   └── ShippingUpdate.tsx
│
├── public/
│   ├── icons/
│   └── og-default.png
│
├── next.config.ts
├── tailwind.config.ts
├── components.json                   # ShadCN config
└── package.json
```

### 2.2 Component Architecture

**Component Hierarchy Principles:**

```
Server Components (RSC)         Client Components
─────────────────────           ─────────────────
Page layouts                    Cart drawer
Product data fetching           Filters (interactive)
SEO metadata                    Search input
Static content                  Forms
Admin data tables               Image galleries
                                Payment button
                                Variant selectors
```

**Component Composition Pattern:**
```typescript
// Server Component (data fetching)
// app/(store)/products/[slug]/page.tsx
export default async function ProductPage({ params }) {
  const product = await fetchProduct(params.slug)  // RSC direct DB call

  return (
    <div className="container grid grid-cols-1 md:grid-cols-2 gap-8">
      {/* Client Component for interactivity */}
      <ProductImageGallery images={product.images} />

      <div>
        <ProductInfo product={product} />
        {/* Client Component — needs cart store */}
        <AddToCartSection product={product} />
      </div>
    </div>
  )
}

// Client Component (interactivity)
// components/products/AddToCartSection.tsx
'use client'

export function AddToCartSection({ product }) {
  const { addItem } = useCartStore()
  const [selectedVariant, setSelectedVariant] = useState(null)
  const [quantity, setQuantity] = useState(1)

  return (
    <>
      <ProductVariantSelector
        variants={product.variants}
        onChange={setSelectedVariant}
      />
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <Button onClick={() => addItem(product, selectedVariant, quantity)}>
        Add to Cart
      </Button>
    </>
  )
}
```

### 2.3 State Management Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    STATE MANAGEMENT                         │
│                                                             │
│  Zustand (Client State)         TanStack Query (Server State)│
│  ─────────────────────         ──────────────────────────── │
│  • cartItems[]                 • useProducts(filters)        │
│  • wishlistIds[]               • useProduct(slug)            │
│  • isCartOpen                  • useOrders()                 │
│  • isMobileMenuOpen            • useWishlistItems()          │
│                                • useAdminProducts()          │
│  Persists to:                                               │
│  • localStorage (guest)        Caches in:                   │
│  • Supabase (logged in)        • TanStack Query cache        │
│                                • Configurable stale time     │
└─────────────────────────────────────────────────────────────┘
```

**Zustand Cart Store:**
```typescript
// stores/cartStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface CartItem {
  productId: string
  variantId: string | null
  name: string
  price: number
  quantity: number
  imageUrl: string
}

interface CartStore {
  items: CartItem[]
  addItem: (item: CartItem) => void
  removeItem: (productId: string, variantId?: string) => void
  updateQuantity: (productId: string, variantId: string | null, quantity: number) => void
  clearCart: () => void
  totalItems: number
  totalPrice: number
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      addItem: (item) => set((state) => {
        const existing = state.items.find(
          i => i.productId === item.productId && i.variantId === item.variantId
        )
        if (existing) {
          return {
            items: state.items.map(i =>
              i.productId === item.productId && i.variantId === item.variantId
                ? { ...i, quantity: Math.min(i.quantity + item.quantity, 10) }
                : i
            )
          }
        }
        return { items: [...state.items, item] }
      }),
      // ... other methods
      get totalItems() { return get().items.reduce((sum, i) => sum + i.quantity, 0) },
      get totalPrice() { return get().items.reduce((sum, i) => sum + i.price * i.quantity, 0) },
    }),
    { name: 'shree-cart' }
  )
)
```

### 2.4 Routing & Navigation Architecture

```
/ (root layout — Providers, Header, Footer)
├── (store)/ (storefront layout group)
│   ├── /                    Homepage
│   ├── /products            PLP
│   ├── /products/[slug]     PDP
│   ├── /search              Search Results
│   ├── /cart                Cart Page
│   ├── /checkout            Checkout (auth optional)
│   │   └── /confirmation/[orderId]
│   └── /account             (requires auth — middleware protected)
│       ├── /                Profile
│       ├── /orders
│       ├── /orders/[id]
│       └── /wishlist
│
├── /admin/ (admin layout — requires admin role)
│   ├── /                    Dashboard
│   ├── /products
│   ├── /products/new
│   ├── /products/[id]/edit
│   ├── /orders
│   ├── /orders/[id]
│   ├── /categories
│   ├── /customers
│   ├── /promotions
│   └── /inventory
│
├── /auth/
│   ├── /login
│   ├── /signup
│   └── /callback            OAuth redirect handler
│
└── /api/
    └── /webhooks/razorpay
```

**Middleware (Auth Guard):**
```typescript
// middleware.ts
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(req: NextRequest) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })

  const { data: { session } } = await supabase.auth.getSession()

  // Protect /account/* routes
  if (req.nextUrl.pathname.startsWith('/account') && !session) {
    return NextResponse.redirect(
      new URL(`/auth/login?redirect=${req.nextUrl.pathname}`, req.url)
    )
  }

  // Protect /admin/* routes — check role
  if (req.nextUrl.pathname.startsWith('/admin')) {
    if (!session) {
      return NextResponse.redirect(new URL('/auth/login', req.url))
    }

    const { data: profile } = await supabase
      .from('profiles')
      .select('role')
      .eq('id', session.user.id)
      .single()

    if (profile?.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', req.url))
    }
  }

  return res
}

export const config = {
  matcher: ['/account/:path*', '/admin/:path*']
}
```

---

## 3. Backend Architecture

### 3.1 Server Actions Architecture

Server Actions are the primary mechanism for all data mutations. They provide type-safe, server-side execution without needing separate API endpoints.

```
Client Component
      │
      │ calls
      ▼
Server Action (runs on server)
      ├── Authenticate user (Supabase)
      ├── Authorize (role check)
      ├── Validate input (Zod)
      ├── Execute business logic
      ├── Mutate database (Supabase client with service role or user session)
      ├── Revalidate Next.js cache (revalidatePath / revalidateTag)
      └── Return typed result to client
```

**Server Action Template:**
```typescript
'use server'

import { createServerClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const schema = z.object({ /* ... */ })

export async function myAction(input: unknown) {
  // 1. Create Supabase server client (uses cookies for auth)
  const supabase = createServerClient()

  // 2. Authenticate
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  if (!user || authError) return { success: false, error: 'Unauthorized' }

  // 3. Validate input
  const parsed = schema.safeParse(input)
  if (!parsed.success) return { success: false, error: parsed.error.flatten() }

  // 4. Execute
  const { data, error } = await supabase.from('table').insert(parsed.data).select().single()
  if (error) return { success: false, error: error.message }

  // 5. Invalidate cache
  revalidatePath('/some-path')

  return { success: true, data }
}
```

### 3.2 Supabase Edge Functions Architecture

Edge Functions handle operations requiring the Supabase service role key or Razorpay secret — credentials that must never reach the client or Next.js server bundle.

```
supabase/functions/
├── create-razorpay-order/      Creates Razorpay order server-side
│   └── index.ts
├── verify-razorpay-payment/    Verifies payment signature (HMAC)
│   └── index.ts
└── send-order-email/           Sends transactional email via Resend
    └── index.ts
```

**Calling Edge Functions from Server Actions:**
```typescript
// actions/checkout.ts
export async function verifyAndCompleteOrder(paymentData: {
  razorpay_order_id: string
  razorpay_payment_id: string
  razorpay_signature: string
  orderId: string
}) {
  const supabase = createServerClient()

  // Call Edge Function (Edge Function handles the HMAC verification)
  const { data, error } = await supabase.functions.invoke('verify-razorpay-payment', {
    body: paymentData,
  })

  if (error || !data.verified) {
    return { success: false, error: 'Payment verification failed' }
  }

  // Create order in database
  const { data: order } = await supabase
    .from('orders')
    .update({ status: 'confirmed', razorpay_payment_id: paymentData.razorpay_payment_id })
    .eq('id', paymentData.orderId)
    .select()
    .single()

  // Trigger email
  await supabase.functions.invoke('send-order-email', {
    body: { orderId: order.id }
  })

  return { success: true, data: { orderId: order.id } }
}
```

### 3.3 API Routes Architecture (Minimal)

Only two Route Handlers exist in the application, both for receiving external webhooks:

```typescript
// app/api/webhooks/razorpay/route.ts
// Purpose: Backup mechanism for payment events missed by the main flow
// Security: HMAC signature verification on every request

export async function POST(request: Request) {
  // Verify Razorpay signature
  // Handle event (payment.captured, payment.failed)
  // Idempotency: check if already processed
  // Update order status if needed
  return NextResponse.json({ received: true })
}
```

### 3.4 Email Architecture

```
Order Placed
     │
     ▼
Server Action: verifyAndCompleteOrder()
     │
     ▼
Supabase Edge Function: send-order-email
     │
     ▼
React Email: renders OrderConfirmation template
     │
     ▼
Resend API: sends to customer email
     │
     ▼
Customer receives branded order confirmation email
```

**Email Templates (React Email):**
```typescript
// emails/OrderConfirmation.tsx
import { Html, Head, Body, Container, Section, Text, Img, Button } from '@react-email/components'

export function OrderConfirmationEmail({ order }: { order: Order }) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'Arial, sans-serif' }}>
        <Container>
          <Img src="https://shreefurniture.in/logo.png" alt="Shree Furniture" width={150} />
          <Text>Thank you for your order, {order.customerName}!</Text>
          <Text>Order #{order.orderNumber} has been confirmed.</Text>
          {/* Order items table */}
          {/* Shipping address */}
          <Button href={`https://shreefurniture.in/account/orders/${order.id}`}>
            Track Your Order
          </Button>
        </Container>
      </Body>
    </Html>
  )
}
```

---

## 4. Database Architecture

### 4.1 Entity Relationship Diagram (Text)

```
auth.users (Supabase managed)
    │ 1:1
    ▼
profiles ────────────────────────────────┐
    │ 1:N                                │ 1:N
    ▼                                    ▼
addresses                           cart_items ──────► products
                                         │
orders ◄─────────────────────────────────┘ (on checkout)
    │
    ├── order_items ──► products
    │                └► product_variants
    └── coupons


products
    ├── product_images (1:N)
    ├── product_variants (1:N)
    └── categories (N:1)

categories (self-referencing for subcategories)
    └── parent_id → categories.id

banners (standalone, no foreign keys)
coupons (standalone, referenced by orders)
wishlist_items → profiles + products
```

### 4.2 Data Partitioning Strategy

**Current (MVP):** Single table for all orders and products. Sufficient for up to ~500,000 orders.

**Future (Scale):** When orders exceed 1M rows:
```sql
-- Partition orders by year
CREATE TABLE orders_2026 PARTITION OF orders
  FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

CREATE TABLE orders_2027 PARTITION OF orders
  FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
```

### 4.3 Type Generation

Supabase CLI generates TypeScript types from the database schema, ensuring end-to-end type safety:

```bash
# Generate types (run after schema changes)
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > src/types/database.ts
```

```typescript
// Auto-generated — never edit manually
export interface Database {
  public: {
    Tables: {
      products: {
        Row: {
          id: string
          name: string
          base_price: number
          // ...
        }
        Insert: { /* ... */ }
        Update: { /* ... */ }
      }
    }
  }
}
```

### 4.4 Migration Strategy

```bash
# All schema changes go through versioned migration files
supabase/migrations/
├── 20260201000001_create_profiles.sql
├── 20260201000002_create_categories.sql
├── 20260201000003_create_products.sql
├── 20260201000004_create_cart_orders.sql
├── 20260201000005_create_promotions.sql
└── 20260201000006_create_indexes_rls.sql

# Apply to local
npx supabase db reset

# Apply to production
npx supabase db push
```

**Migration Rules:**
- Never modify existing migration files
- Always use `IF NOT EXISTS` guards
- Destructive operations (DROP, ALTER) require a new migration with a data backup step

---

## 5. Security Architecture

### 5.1 Authentication & Authorization Layers

```
Request arrives at Next.js
         │
         ▼
Layer 1: Middleware (Edge)
  └── Validates Supabase session cookie
  └── Redirects unauthenticated users from protected routes
  └── Checks admin role for /admin/* routes
         │
         ▼
Layer 2: Server Action / Route Handler
  └── Re-validates session (auth.getUser())
  └── Never trusts middleware alone (defense in depth)
  └── Checks role from profiles table for admin actions
         │
         ▼
Layer 3: Supabase RLS (Database level)
  └── Row-level policies enforced at PostgreSQL level
  └── Even if application code has a bug, data is protected
  └── Every query runs as authenticated user (not service role)
         │
         ▼
Layer 4: Supabase Edge Functions
  └── Service role key locked in Supabase secrets vault
  └── Callable only from authenticated Next.js context
  └── Payment verification cannot be bypassed
```

### 5.2 Secret Management

```
Secret Type              Storage Location           Accessible By
─────────────            ────────────────           ─────────────
SUPABASE_ANON_KEY        Vercel env (public)        Client + Server
SUPABASE_SERVICE_ROLE    Vercel env (private)       Server only ❌ never client
RAZORPAY_KEY_ID          Vercel env (public prefix) Client (for SDK init)
RAZORPAY_KEY_SECRET      Vercel env (private)       Server only ❌ never client
RAZORPAY_WEBHOOK_SECRET  Vercel env (private)       Webhook handler only
RESEND_API_KEY           Vercel env (private)       Server only
RAZORPAY secrets         Supabase Secrets Vault     Edge Functions only
```

**Client Bundle Audit (enforced in CI):**
```bash
# Check that no private keys are in client bundle
grep -r "RAZORPAY_KEY_SECRET\|SUPABASE_SERVICE_ROLE\|RESEND_API_KEY" .next/
# Must return zero results
```

### 5.3 Content Security Policy

```typescript
// next.config.ts
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://checkout.razorpay.com",
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https://*.supabase.co https://lh3.googleusercontent.com",
      "connect-src 'self' https://*.supabase.co https://api.razorpay.com",
      "frame-src https://api.razorpay.com",
      "font-src 'self'",
    ].join('; ')
  },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
]
```

### 5.4 Input Validation Strategy

**Every input is validated at two layers:**

```
Client Side (UX feedback)          Server Side (security)
─────────────────────────          ──────────────────────
React Hook Form validation         Zod schema validation
Shows field errors instantly       Runs in Server Action
                                   Rejects malformed input
                                   before DB query
```

**Zod schemas are shared between client and server:**
```typescript
// lib/validations/checkout.ts — used by both form and server action
export const checkoutSchema = z.object({
  email: z.string().email('Invalid email address'),
  fullName: z.string().min(2, 'Name must be at least 2 characters'),
  phone: z.string().regex(/^[6-9]\d{9}$/, 'Invalid Indian mobile number'),
  line1: z.string().min(5),
  city: z.string().min(2),
  state: z.string().min(2),
  pincode: z.string().regex(/^\d{6}$/, 'PIN code must be 6 digits'),
})
```

### 5.5 Rate Limiting

**Checkout flow:** Vercel's built-in edge rate limiting (configurable via `vercel.json`):
```json
{
  "functions": {
    "app/api/webhooks/razorpay/route.ts": {
      "maxDuration": 30
    }
  }
}
```

For Supabase, enable **Auth rate limiting** via Supabase dashboard:
- Sign up: 3 per hour per IP
- Login: 10 per 5 minutes per IP
- Password reset: 3 per hour

### 5.6 Razorpay Payment Security

```
Payment Security Checklist
─────────────────────────
✅ Order created server-side (Edge Function) — amount cannot be tampered client-side
✅ HMAC-SHA256 signature verified server-side before order creation
✅ Idempotency: duplicate payment_id rejected gracefully
✅ Webhook signature verified independently
✅ Order amount in DB compared to Razorpay amount on webhook receipt
✅ HTTPS enforced (Vercel automatic)
✅ Razorpay secret never in client bundle
```

---

## 6. Deployment Architecture

### 6.1 Infrastructure Topology

```
┌───────────────────────────────────────────────────────────────────┐
│                          DNS / Domain                              │
│                      shreefurniture.in                            │
│                      (Namecheap / GoDaddy)                        │
└─────────────────────────────┬─────────────────────────────────────┘
                              │ DNS CNAME → Vercel
                              ▼
┌───────────────────────────────────────────────────────────────────┐
│                     VERCEL EDGE NETWORK                           │
│                                                                   │
│  shreefurniture.in          → Production (main branch)            │
│  staging.shreefurniture.in  → Staging (develop branch)            │
│  *.vercel.app               → Preview (PR branches)               │
│                                                                   │
│  Edge Locations: Global CDN — Mumbai, Singapore, Frankfurt, etc.  │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
                    ┌─────────┴──────────┐
                    │                    │
                    ▼                    ▼
         ┌────────────────┐   ┌────────────────────┐
         │ Supabase (SG)  │   │ External Services  │
         │                │   │                    │
         │ • PostgreSQL   │   │ • Razorpay API     │
         │ • Auth         │   │ • Resend API       │
         │ • Storage CDN  │   │ • Google OAuth     │
         │ • Edge Fns     │   │                    │
         └────────────────┘   └────────────────────┘
```

### 6.2 CI/CD Pipeline

```yaml
# .github/workflows/ci.yml

name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run type-check      # tsc --noEmit
      - run: npm run lint            # ESLint
      - run: npm run test            # Vitest
      - run: npm run build           # Next.js build
      - name: Check no secrets in bundle
        run: |
          grep -r "KEY_SECRET\|SERVICE_ROLE\|RESEND_API" .next/ && exit 1 || exit 0

  # Vercel auto-deploys on push (no manual step needed)
  # Preview: on every PR
  # Production: on push to main
```

### 6.3 Environment Configuration

```bash
# Production (Vercel Dashboard → Settings → Environment Variables)
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://shreefurniture.in
NEXT_PUBLIC_SUPABASE_URL=https://[project].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon-key]
SUPABASE_SERVICE_ROLE_KEY=[service-role-key]
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_live_[key]
RAZORPAY_KEY_SECRET=[secret]
RAZORPAY_WEBHOOK_SECRET=[webhook-secret]
RESEND_API_KEY=re_[key]

# Staging (separate Supabase project for isolation)
NEXT_PUBLIC_SUPABASE_URL=https://[staging-project].supabase.co
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_test_[key]   # Test mode for staging
```

### 6.4 Supabase Project Structure

```
Supabase Projects
├── shree-furniture-prod         # Production database
│   ├── Database: prod data
│   ├── Auth: real users
│   ├── Storage: real product images
│   └── Edge Functions: deployed
│
└── shree-furniture-staging      # Staging/dev database
    ├── Database: test data
    ├── Auth: test accounts
    ├── Storage: test images
    └── Edge Functions: dev versions
```

### 6.5 Backup & Recovery

**Supabase Free Tier:** Daily automated backups (7-day retention) — enabled by default.

**Supabase Pro:** PITR (Point-in-Time Recovery) — recommended for production after launch.

**Pre-deployment backup procedure:**
```bash
# Before every production deployment with DB migration
npx supabase db dump --file backup_$(date +%Y%m%d_%H%M%S).sql

# Rollback procedure
npx supabase db execute --file backup_YYYYMMDD_HHMMSS.sql
```

### 6.6 Monitoring & Observability

| Tool | What It Monitors | Cost |
|------|-----------------|------|
| Vercel Analytics | Page views, traffic, geographic distribution | Free |
| Vercel Speed Insights | Real-user Core Web Vitals | Free |
| Vercel Function Logs | Serverless function errors and execution times | Free |
| Supabase Dashboard | DB query performance, Auth events, Storage usage | Free |
| Sentry (free tier) | JavaScript errors, exception tracking | Free |

**Error Tracking Setup:**
```typescript
// next.config.ts (Sentry integration)
import { withSentryConfig } from '@sentry/nextjs'

export default withSentryConfig(nextConfig, {
  org: 'shree-furniture',
  project: 'shree-furniture-web',
  silent: true,
  widenClientFileUpload: true,
  hideSourceMaps: true,
})
```

---

## 7. Future Scalability Roadmap

### Phase 1 → Phase 2 (Month 6–12)

**Product Reviews & Ratings**
- Add `reviews` table with `user_id, product_id, rating, comment, is_approved`
- RLS: users can read approved reviews; can only write their own; admin can approve
- Display star rating on PDP and PLP cards

**Abandoned Cart Recovery**
- Supabase Edge Function triggered by pg_cron: scan carts not checked out in 24h
- Send recovery email via Resend with cart contents and link
- Track recovery conversion in `cart_recovery_events` table

**Advanced Search (Algolia or Typesense)**
- When product catalog exceeds 5,000 items, PostgreSQL full-text search may feel slow
- Migrate to Typesense (self-hostable, open-source) or Algolia (SaaS)
- Sync Supabase products → search index via Edge Function trigger on product insert/update

### Phase 2 → Phase 3 (Month 12–18)

**Shipping Integration**
- Integrate Shiprocket or Delhivery API for automated label generation
- Real-time tracking updates via webhook → stored in `shipments` table
- Customer receives automated tracking updates via email

**GST-Compliant Invoicing**
- Generate PDF invoices with HSN codes, GSTIN, tax breakdown
- Use React PDF or Puppeteer in Edge Function for PDF generation
- Store in Supabase Storage, attach to order confirmation email

**Redis Caching (Upstash)**
- Add when database connection pooling becomes a bottleneck (>100 concurrent users)
- Cache: product catalog (TTL: 5min), category tree (TTL: 30min), homepage banners (TTL: 10min)
- Upstash Redis: serverless, pay-per-request, no idle cost

```typescript
import { Redis } from '@upstash/redis'
const redis = new Redis({ url: process.env.UPSTASH_URL, token: process.env.UPSTASH_TOKEN })

// Cache product listing
const cacheKey = `products:${JSON.stringify(filters)}`
const cached = await redis.get(cacheKey)
if (cached) return cached

const products = await supabase.from('products').select(...)
await redis.setex(cacheKey, 300, products)  // Cache 5 minutes
```

**Multi-warehouse Inventory**
- Add `warehouses` and `inventory_locations` tables
- Show delivery time based on nearest warehouse to customer PIN code

### Phase 3 → Phase 4 (Month 18+)

**Mobile App (React Native / Expo)**
- Reuse Supabase backend directly (same Auth, same DB, same Storage)
- Share TypeScript types and validation schemas between web and mobile
- Push notifications via Expo Notifications for order updates

**AI-Powered Recommendations**
- Track `product_views` events in Supabase
- Build collaborative filtering model (pgvector extension in Supabase)
- "Customers who viewed this also viewed" section on PDP

**Multi-City / Multi-Branch**
- Add `branches` table with city, contact, operating hours
- Customer selects preferred branch at checkout
- Inventory managed per branch

**B2B / Bulk Order Portal**
- Separate `b2b_customers` with custom pricing tiers
- RLS policy: B2B customer sees wholesale prices
- Bulk order form with quotation workflow

### Infrastructure Upgrade Triggers

| Trigger | Action |
|---------|--------|
| DB > 450MB | Upgrade Supabase to Pro (₹2,100/month) |
| Monthly page views > 100,000 | Upgrade Vercel to Pro (₹1,700/month) |
| Email volume > 3,000/month | Upgrade Resend to Pro ($20/month) |
| Error rate > 0.5% | Investigate + Upgrade Sentry plan |
| Search latency > 500ms | Add Typesense or Algolia |
| DB connections > 80% pool | Add PgBouncer pooler (included in Supabase Pro) |

### Technology Evolution Path

```
Current (MVP)                Future (Scale)
─────────────────────────    ──────────────────────────────
Supabase Free               → Supabase Pro + Read Replicas
Vercel Hobby                → Vercel Pro
In-memory cart (Zustand)    → Redis (Upstash) session store
PG full-text search         → Typesense / Algolia
No cache layer              → Upstash Redis
Manual shipping             → Shiprocket / Delhivery API
Basic analytics             → Mixpanel / PostHog
Email via Resend            → Resend + drip campaign tooling
```

---

*Document Version 1.0 — Shree Furniture Architecture Document*
