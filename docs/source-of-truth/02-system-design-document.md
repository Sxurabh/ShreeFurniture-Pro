# Shree Furniture — System Design Document

> **Project:** Shree Furniture E-Commerce Platform
> **Document Type:** System Design Document
> **Version:** 1.0
> **Date:** February 2026
> **Stack:** Next.js · TypeScript · Supabase · ShadCN UI · Tailwind CSS

---

## Table of Contents

1. [System Components Overview](#1-system-components-overview)
2. [Data Flow Diagrams](#2-data-flow-diagrams)
3. [Database Design Strategy](#3-database-design-strategy)
4. [API Structure & Conventions](#4-api-structure--conventions)
5. [Scalability Considerations](#5-scalability-considerations)
6. [Performance Optimization Strategy](#6-performance-optimization-strategy)
7. [Recommended Additional Tools](#7-recommended-additional-tools)

---

## 1. System Components Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                   │
│                                                                         │
│   ┌──────────────────┐         ┌──────────────────┐                    │
│   │  Customer Web    │         │   Admin Web App  │                    │
│   │  App (Next.js)   │         │   (Next.js)      │                    │
│   │  /app/(store)    │         │   /app/admin     │                    │
│   └────────┬─────────┘         └────────┬─────────┘                    │
│            │                            │                               │
└────────────┼────────────────────────────┼───────────────────────────────┘
             │                            │
┌────────────▼────────────────────────────▼───────────────────────────────┐
│                        NEXT.JS APPLICATION LAYER                        │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │  App Router │  │  Server     │  │  Route      │  │  Middleware   │  │
│  │  (RSC +     │  │  Actions    │  │  Handlers   │  │  (Auth Guard) │  │
│  │   Client)   │  │             │  │  /api/*     │  │               │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └──────────────┘  │
│                                                                         │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
┌────────────────────────────────────▼────────────────────────────────────┐
│                         SUPABASE PLATFORM                               │
│                                                                         │
│  ┌──────────────┐  ┌────────────────┐  ┌──────────┐  ┌─────────────┐  │
│  │  PostgreSQL  │  │  Auth          │  │  Storage │  │  Edge       │  │
│  │  (RLS)       │  │  (JWT + OAuth) │  │  (CDN)   │  │  Functions  │  │
│  └──────────────┘  └────────────────┘  └──────────┘  └─────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                     │
┌────────────────────────────────────▼────────────────────────────────────┐
│                       EXTERNAL SERVICES                                 │
│                                                                         │
│     ┌──────────────┐         ┌──────────────┐    ┌──────────────┐      │
│     │  Razorpay    │         │  Resend      │    │  Vercel      │      │
│     │  (Payments)  │         │  (Email)     │    │  (CDN/Edge)  │      │
│     └──────────────┘         └──────────────┘    └──────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

**Next.js Application Layer**

- **App Router (RSC):** Server Components handle data fetching, SEO, and initial HTML rendering. Client Components handle interactivity (cart, filters, forms).
- **Server Actions:** Secure server-side mutations (cart sync, order creation, admin operations) without exposing API endpoints.
- **Route Handlers (`/api/*`):** Used exclusively for Razorpay webhook reception and any third-party callback endpoints.
- **Middleware:** JWT-based session validation for protected routes (`/admin/*`, `/account/*`, `/checkout`).

**Supabase Platform**

- **PostgreSQL:** Primary data store for all business entities (products, orders, users, categories).
- **Auth:** JWT-based authentication with email/password and Google OAuth. Session tokens stored in cookies for SSR compatibility.
- **Storage:** Product image hosting with built-in CDN for fast global delivery.
- **Edge Functions (Deno):** Business-critical server-side logic — Razorpay order creation, payment verification, webhook processing. Isolated from Next.js for security.

**External Services**

- **Razorpay:** Payment processing. Orders created via Edge Function (server-side). Webhooks handled via Next.js Route Handler.
- **Resend:** Transactional emails (order confirmation, shipping update, password reset).
- **Vercel Edge Network:** CDN for static assets, serverless function hosting, and Next.js ISR/SSR execution.

---

## 2. Data Flow Diagrams

### 2.1 Customer Browse & Search Flow

```
User types search query
        │
        ▼
Next.js Client Component (debounce 300ms)
        │
        ▼
Supabase JS Client (anon key)
  └── products table query
       ├── Full-text search: to_tsvector('english', name || ' ' || description)
       ├── Filter: category_id, price_range, material, in_stock
       ├── Sort: price ASC/DESC, created_at DESC
       └── Pagination: LIMIT 20 OFFSET n
        │
        ▼
PostgreSQL (RLS: all can read published products)
        │
        ▼
Results streamed to client via React Suspense
        │
        ▼
URL state updated with search params (shareable, crawlable)
```

### 2.2 Add to Cart Flow

```
Customer clicks "Add to Cart"
        │
        ▼
Zustand cartStore.addItem(product, quantity, variant)
        │
  ┌─────┴──────┐
  │            │
  ▼            ▼
Guest?      Logged In?
  │            │
  ▼            ▼
localStorage  Server Action: syncCartToSupabase()
persist         └── Upsert into cart_items table
                    (user_id, product_id, variant_id, quantity)
        │
        ▼
Cart drawer updates (optimistic UI)
Cart count badge in header updates
```

### 2.3 Checkout & Payment Flow

```
Customer fills checkout form
        │
        ▼
Client-side form validation (React Hook Form + Zod)
        │
        ▼
Server Action: initiateCheckout()
  ├── Validate cart items (prices, stock availability)
  ├── Apply discount code if present
  ├── Calculate totals (subtotal, discount, shipping, tax, grand total)
  └── Call Supabase Edge Function: create-razorpay-order
           │
           ▼
      Razorpay API: POST /v1/orders
           │
           ▼
      Returns: { order_id, amount, currency }
           │
           ▼
Client receives Razorpay order_id
        │
        ▼
Razorpay JS SDK opens payment modal
        │
     ┌──┴──┐
     │     │
     ▼     ▼
 Success  Failure
     │     │
     ▼     ▼
Client receives  Show error toast
razorpay_payment_id  Allow retry
razorpay_order_id
razorpay_signature
     │
     ▼
Server Action: verifyPayment()
  └── Call Supabase Edge Function: verify-razorpay-payment
           │
           ▼
      HMAC signature verification (server-side)
           │
        ┌──┴──┐
        │     │
        ▼     ▼
    Valid   Invalid
        │     │
        ▼     ▼
  Update order status to confirmed  Return error
  (order already pending_payment)   (do not confirm order)
  Clear cart
  Send confirmation email (Resend)
  Redirect to /checkout/confirmation/[orderId]
```

### 2.4 Razorpay Webhook Flow (Backup Mechanism)

```
Razorpay Server → POST /api/webhooks/razorpay
        │
        ▼
Route Handler validates webhook signature
  └── crypto.createHmac('sha256', RAZORPAY_WEBHOOK_SECRET)
        │
        ▼
Event type check:
  ├── payment.captured → ensure order exists, mark as paid
  ├── payment.failed   → update order status to failed
  └── refund.created   → update order status to refunded
        │
        ▼
Idempotency check: has this payment_id been processed?
  └── If yes → return 200 OK (no-op)
  └── If no  → process + mark as processed
        │
        ▼
Return 200 OK (Razorpay retries on non-200)
```

### 2.5 Admin Product Creation Flow

```
Admin fills product form
        │
        ▼
Client: Image files selected
        │
        ▼
Upload to Supabase Storage
  ├── Bucket: product-images (private bucket, public read policy)
  ├── Path: products/{product_id}/{uuid}.webp
  └── Returns public CDN URL
        │
        ▼
Server Action: createProduct()
  ├── Auth check: req user has role='admin'
  ├── Validate product data (Zod schema)
  ├── Insert into products table
  ├── Insert into product_images table (multiple images)
  ├── Insert into product_variants table
  └── Insert into product_attributes table
        │
        ▼
Supabase Realtime triggers → invalidate product cache
        │
        ▼
Redirect to /admin/products/[id]
```

---

## 3. Database Design Strategy

### 3.1 Schema Overview

```sql
-- USERS & AUTH (Managed by Supabase Auth)
-- auth.users (Supabase managed, do not modify directly)

-- Public user profile linked to auth.users
CREATE TABLE public.profiles (
  id            UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name     TEXT,
  phone         TEXT,
  role          TEXT DEFAULT 'customer' CHECK (role IN ('customer', 'admin')),
  avatar_url    TEXT,
  created_at    TIMESTAMPTZ DEFAULT now(),
  updated_at    TIMESTAMPTZ DEFAULT now()
);

-- Customer delivery addresses
CREATE TABLE public.addresses (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  label         TEXT DEFAULT 'Home', -- Home, Work, Other
  full_name     TEXT NOT NULL,
  phone         TEXT NOT NULL,
  line1         TEXT NOT NULL,
  line2         TEXT,
  city          TEXT NOT NULL,
  state         TEXT NOT NULL,
  pincode       TEXT NOT NULL,
  is_default    BOOLEAN DEFAULT false,
  created_at    TIMESTAMPTZ DEFAULT now()
);

-- CATALOG

CREATE TABLE public.categories (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id     UUID REFERENCES public.categories(id) ON DELETE SET NULL,
  name          TEXT NOT NULL,
  slug          TEXT NOT NULL UNIQUE,
  description   TEXT,
  image_url     TEXT,
  display_order INT DEFAULT 0,
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE public.products (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category_id     UUID REFERENCES public.categories(id) ON DELETE SET NULL,
  name            TEXT NOT NULL,
  slug            TEXT NOT NULL UNIQUE,
  description     TEXT,
  short_desc      TEXT,
  base_price      NUMERIC(10,2) NOT NULL,
  sale_price      NUMERIC(10,2),
  sku             TEXT UNIQUE,
  material        TEXT,
  brand           TEXT DEFAULT 'Shree Furniture',
  weight_kg       NUMERIC(6,2),
  dimensions_cm   JSONB,  -- { "length": 120, "width": 60, "height": 75 }
  is_published    BOOLEAN DEFAULT false,
  is_featured     BOOLEAN DEFAULT false,
  stock_quantity  INT DEFAULT 0,
  low_stock_threshold INT DEFAULT 5,
  search_vector   TSVECTOR,  -- auto-updated by trigger
  meta_title      TEXT,
  meta_description TEXT,
  created_at      TIMESTAMPTZ DEFAULT now(),
  updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Auto-update search_vector
CREATE FUNCTION update_product_search_vector() RETURNS TRIGGER AS $$
BEGIN
  NEW.search_vector := to_tsvector('english',
    COALESCE(NEW.name, '') || ' ' ||
    COALESCE(NEW.description, '') || ' ' ||
    COALESCE(NEW.material, '') || ' ' ||
    COALESCE(NEW.brand, '')
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_search_vector_update
  BEFORE INSERT OR UPDATE ON public.products
  FOR EACH ROW EXECUTE FUNCTION update_product_search_vector();

CREATE INDEX idx_products_search ON public.products USING GIN (search_vector);

CREATE TABLE public.product_images (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id  UUID REFERENCES public.products(id) ON DELETE CASCADE,
  url         TEXT NOT NULL,
  alt_text    TEXT,
  display_order INT DEFAULT 0,
  is_primary  BOOLEAN DEFAULT false
);

CREATE TABLE public.product_variants (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id    UUID REFERENCES public.products(id) ON DELETE CASCADE,
  name          TEXT NOT NULL,  -- e.g., "Oak / Large"
  sku           TEXT UNIQUE,
  price_delta   NUMERIC(10,2) DEFAULT 0,  -- added to base_price
  stock_quantity INT DEFAULT 0,
  attributes    JSONB,  -- { "color": "Oak", "size": "Large" }
  is_active     BOOLEAN DEFAULT true
);

-- CART

CREATE TABLE public.cart_items (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  product_id  UUID REFERENCES public.products(id) ON DELETE CASCADE,
  variant_id  UUID REFERENCES public.product_variants(id) ON DELETE SET NULL,
  quantity    INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
  created_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, product_id, variant_id)
);

-- WISHLIST

CREATE TABLE public.wishlist_items (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  product_id  UUID REFERENCES public.products(id) ON DELETE CASCADE,
  created_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, product_id)
);

-- ORDERS

CREATE TYPE order_status AS ENUM (
  'pending_payment', 'payment_failed', 'confirmed',
  'processing', 'shipped', 'out_for_delivery', 'delivered', 'cancelled', 'refunded'
);

CREATE TABLE public.orders (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number         TEXT UNIQUE NOT NULL,  -- e.g., SF-2026-00001
  user_id              UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  guest_email          TEXT,  -- for guest orders
  status               order_status DEFAULT 'pending_payment',
  subtotal             NUMERIC(10,2) NOT NULL,
  discount_amount      NUMERIC(10,2) DEFAULT 0,
  shipping_amount      NUMERIC(10,2) DEFAULT 0,
  tax_amount           NUMERIC(10,2) DEFAULT 0,
  total_amount         NUMERIC(10,2) NOT NULL,
  shipping_address     JSONB NOT NULL,
  coupon_id            UUID REFERENCES public.coupons(id) ON DELETE SET NULL,
  razorpay_order_id    TEXT,
  razorpay_payment_id  TEXT,
  notes                TEXT,
  shipped_at           TIMESTAMPTZ,
  delivered_at         TIMESTAMPTZ,
  created_at           TIMESTAMPTZ DEFAULT now(),
  updated_at           TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE public.order_items (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id      UUID REFERENCES public.orders(id) ON DELETE CASCADE,
  product_id    UUID REFERENCES public.products(id) ON DELETE SET NULL,
  variant_id    UUID REFERENCES public.product_variants(id) ON DELETE SET NULL,
  product_name  TEXT NOT NULL,  -- snapshot at time of order
  variant_name  TEXT,
  unit_price    NUMERIC(10,2) NOT NULL,  -- snapshot at time of order
  quantity      INT NOT NULL,
  total_price   NUMERIC(10,2) NOT NULL
);

-- PROMOTIONS

CREATE TABLE public.banners (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title         TEXT NOT NULL,
  subtitle      TEXT,
  image_url     TEXT NOT NULL,
  link_url      TEXT,
  display_order INT DEFAULT 0,
  is_active     BOOLEAN DEFAULT true,
  starts_at     TIMESTAMPTZ,
  ends_at       TIMESTAMPTZ,
  created_at    TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE public.coupons (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code              TEXT NOT NULL UNIQUE,
  description       TEXT,
  discount_type     TEXT CHECK (discount_type IN ('percentage', 'fixed')),
  discount_value    NUMERIC(10,2) NOT NULL,
  min_order_amount  NUMERIC(10,2) DEFAULT 0,
  max_uses          INT,
  used_count        INT DEFAULT 0,
  is_active         BOOLEAN DEFAULT true,
  starts_at         TIMESTAMPTZ,
  expires_at        TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT now()
);
```

### 3.2 Row Level Security (RLS) Policies

```sql
-- PROFILES
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

-- Users can read and update their own profile
CREATE POLICY "profiles_own_access" ON public.profiles
  FOR ALL USING (auth.uid() = id);

-- Admins can read all profiles
CREATE POLICY "profiles_admin_read" ON public.profiles
  FOR SELECT USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );

-- PRODUCTS (Public Read)
ALTER TABLE public.products ENABLE ROW LEVEL SECURITY;

CREATE POLICY "products_public_read" ON public.products
  FOR SELECT USING (is_published = true);

CREATE POLICY "products_admin_all" ON public.products
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );

-- CART ITEMS
ALTER TABLE public.cart_items ENABLE ROW LEVEL SECURITY;

CREATE POLICY "cart_own_access" ON public.cart_items
  FOR ALL USING (auth.uid() = user_id);

-- ORDERS
ALTER TABLE public.orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "orders_own_read" ON public.orders
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "orders_admin_all" ON public.orders
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );
```

### 3.3 Key Indexes

```sql
-- Product performance indexes
CREATE INDEX idx_products_category     ON public.products(category_id);
CREATE INDEX idx_products_price        ON public.products(base_price);
CREATE INDEX idx_products_published    ON public.products(is_published);
CREATE INDEX idx_products_featured     ON public.products(is_featured);
CREATE INDEX idx_products_slug         ON public.products(slug);
CREATE INDEX idx_products_search       ON public.products USING GIN(search_vector);

-- Order performance indexes
CREATE INDEX idx_orders_user           ON public.orders(user_id);
CREATE INDEX idx_orders_status         ON public.orders(status);
CREATE INDEX idx_orders_created_at     ON public.orders(created_at DESC);
CREATE INDEX idx_order_items_order     ON public.order_items(order_id);

-- Category
CREATE INDEX idx_categories_parent     ON public.categories(parent_id);
CREATE INDEX idx_categories_slug       ON public.categories(slug);
```

### 3.4 Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Tables | `snake_case`, plural | `order_items`, `product_images` |
| Columns | `snake_case` | `created_at`, `user_id` |
| Primary Keys | `id` (UUID) | `id UUID DEFAULT gen_random_uuid()` |
| Foreign Keys | `{table_singular}_id` | `product_id`, `order_id` |
| Indexes | `idx_{table}_{column}` | `idx_products_category` |
| Enums | `snake_case` | `order_status` |
| Policies | Descriptive string | `"products_public_read"` |

---

## 4. API Structure & Conventions

### 4.1 Architecture Decision: Server Actions vs Route Handlers

| Mechanism | Use Case |
|-----------|---------|
| **Server Actions** | All mutations triggered by UI (cart, checkout, profile, admin CRUD) |
| **Route Handlers** (`/api/*`) | External webhooks (Razorpay), third-party callbacks |
| **Supabase Edge Functions** | Sensitive operations requiring service role key (payment creation/verification) |
| **Supabase JS Client** | Direct data fetching in Server Components (RSC) |

### 4.2 Server Action Structure

```typescript
// /src/actions/products.ts

'use server'

import { createServerClient } from '@/lib/supabase/server'
import { productSchema } from '@/lib/validations/product'
import { revalidatePath } from 'next/cache'
import type { ActionResult } from '@/types/actions'

export async function createProduct(
  formData: FormData
): Promise<ActionResult<{ id: string }>> {
  const supabase = createServerClient()

  // 1. Auth check
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return { success: false, error: 'Unauthorized' }

  const { data: profile } = await supabase
    .from('profiles')
    .select('role')
    .eq('id', user.id)
    .single()

  if (profile?.role !== 'admin') return { success: false, error: 'Forbidden' }

  // 2. Validate
  const raw = Object.fromEntries(formData)
  const parsed = productSchema.safeParse(raw)
  if (!parsed.success) {
    return { success: false, error: parsed.error.flatten().fieldErrors }
  }

  // 3. Mutate
  const { data, error } = await supabase
    .from('products')
    .insert(parsed.data)
    .select('id')
    .single()

  if (error) return { success: false, error: error.message }

  // 4. Revalidate
  revalidatePath('/admin/products')
  revalidatePath('/products')

  return { success: true, data }
}
```

### 4.3 Route Handler Convention (Webhooks Only)

```
POST /api/webhooks/razorpay    → Razorpay payment events
POST /api/webhooks/resend      → Resend email delivery events (optional)
```

```typescript
// /src/app/api/webhooks/razorpay/route.ts

import { headers } from 'next/headers'
import crypto from 'crypto'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const body = await request.text()
  const signature = headers().get('x-razorpay-signature') ?? ''

  const expectedSignature = crypto
    .createHmac('sha256', process.env.RAZORPAY_WEBHOOK_SECRET!)
    .update(body)
    .digest('hex')

  if (signature !== expectedSignature) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 401 })
  }

  const event = JSON.parse(body)
  // Handle event types...
  return NextResponse.json({ received: true })
}
```

### 4.4 Supabase Edge Function Convention

```
/supabase/functions/
  ├── create-razorpay-order/    → Creates Razorpay order (uses service role)
  ├── verify-razorpay-payment/  → Verifies payment signature
  └── send-order-email/         → Sends order confirmation via Resend
```

```typescript
// /supabase/functions/create-razorpay-order/index.ts

import { serve } from 'https://deno.land/std/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const { orderId, amount, currency = 'INR' } = await req.json()

  // Validate caller is authenticated
  const authHeader = req.headers.get('Authorization')!
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_ANON_KEY')!,
    { global: { headers: { Authorization: authHeader } } }
  )

  const { data: { user } } = await supabase.auth.getUser()
  // Validate user...

  // Create Razorpay order
  const response = await fetch('https://api.razorpay.com/v1/orders', {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${btoa(`${Deno.env.get('RAZORPAY_KEY_ID')}:${Deno.env.get('RAZORPAY_KEY_SECRET')}`)}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      amount: amount * 100,  // paise
      currency,
      receipt: orderId,
      payment_capture: 1,
    }),
  })

  const razorpayOrder = await response.json()
  return new Response(JSON.stringify(razorpayOrder), {
    headers: { 'Content-Type': 'application/json' },
  })
})
```

### 4.5 Data Fetching Conventions (Server Components)

```typescript
// Pattern: Direct Supabase client in RSC

// /src/app/(store)/products/page.tsx
import { createServerClient } from '@/lib/supabase/server'

export default async function ProductsPage({
  searchParams
}: {
  searchParams: { category?: string; min_price?: string; q?: string; page?: string }
}) {
  const supabase = createServerClient()
  const page = Number(searchParams.page) || 1
  const pageSize = 20

  let query = supabase
    .from('products')
    .select(`
      id, name, slug, base_price, sale_price, stock_quantity,
      product_images(url, is_primary),
      categories(name, slug)
    `, { count: 'exact' })
    .eq('is_published', true)
    .range((page - 1) * pageSize, page * pageSize - 1)

  if (searchParams.category) {
    query = query.eq('categories.slug', searchParams.category)
  }
  if (searchParams.q) {
    query = query.textSearch('search_vector', searchParams.q)
  }

  const { data: products, count } = await query

  return <ProductGrid products={products} total={count} />
}
```

### 4.6 Error Handling Convention

```typescript
// Standard action result type
type ActionResult<T = void> =
  | { success: true; data: T }
  | { success: false; error: string | Record<string, string[]> }

// Client-side handling
const result = await createProduct(formData)
if (!result.success) {
  toast.error(typeof result.error === 'string' ? result.error : 'Validation failed')
  return
}
toast.success('Product created successfully')
router.push(`/admin/products/${result.data.id}`)
```

---

## 5. Scalability Considerations

### 5.1 Database Scalability

**Current (Free Tier):** Single Supabase project, shared compute, 500MB storage.

**Scale Path:**
- **~1,000 products, ~10,000 orders:** Free tier comfortably handles this. No changes needed.
- **~10,000 products, ~100,000 orders:** Upgrade to Supabase Pro. Enable connection pooling via PgBouncer (already included in Pro). Add read replicas for analytics queries.
- **~100,000+ products:** Consider partitioning `orders` table by `created_at` (monthly partitions). Move analytics to separate read replica or pg_analytics extension.

**Query Optimization Strategy:**
- All list queries use cursor-based pagination (via `id > last_id`) for large datasets, replacing OFFSET pagination
- Expensive admin analytics run as materialized views, refreshed every 15 minutes

```sql
-- Materialized view for admin dashboard
CREATE MATERIALIZED VIEW daily_sales_summary AS
SELECT
  DATE(created_at) as sale_date,
  COUNT(*) as order_count,
  SUM(total_amount) as revenue,
  AVG(total_amount) as avg_order_value
FROM public.orders
WHERE status NOT IN ('pending_payment', 'payment_failed', 'cancelled')
GROUP BY DATE(created_at);

CREATE UNIQUE INDEX ON daily_sales_summary(sale_date);

-- Refresh every 15 minutes via pg_cron (Supabase Pro)
SELECT cron.schedule('refresh-sales-summary', '*/15 * * * *',
  'REFRESH MATERIALIZED VIEW CONCURRENTLY daily_sales_summary');
```

### 5.2 Application Scalability

**Next.js Serverless:** Vercel auto-scales Next.js functions horizontally — no configuration needed. Each request is independent.

**State Management:** Zustand stores are client-side per-session. No server-side state to scale.

**Image Delivery:** Supabase Storage CDN handles image delivery at scale. For very high traffic (>1M requests/month), consider migrating image delivery to Cloudflare R2 + Cloudflare CDN (still free up to 10GB egress).

### 5.3 Traffic Scalability

```
Current Setup → Handles: ~50,000 page views/month comfortably

At ~500,000 PV/month:
  ├── Upgrade Vercel to Pro
  ├── Upgrade Supabase to Pro
  └── Enable Next.js full-page caching (ISR) for product/category pages

At ~5,000,000 PV/month:
  ├── Add Redis (Upstash — serverless, pay-per-use) for session/cart caching
  ├── Add Supabase read replica
  └── Enable aggressive CDN caching for catalog pages
```

---

## 6. Performance Optimization Strategy

### 6.1 Rendering Strategy by Page Type

| Page | Strategy | Rationale |
|------|---------|-----------|
| Homepage | ISR (revalidate: 300s) | Changes infrequently; full SSR is wasteful |
| Category listing | ISR (revalidate: 60s) | Product availability changes; filters are URL-param based |
| Product Detail Page | ISR (revalidate: 60s) + on-demand revalidation | Stock/price can change; admin triggers revalidation on update |
| Search results | SSR | Unique per query; cannot be cached generically |
| Cart / Checkout | CSR (Client Component) | User-specific; should not be cached |
| Admin pages | SSR (no-cache) | Always fresh data required |

```typescript
// ISR example for product page
export const revalidate = 60

// On-demand revalidation when admin updates a product
import { revalidatePath } from 'next/cache'
revalidatePath(`/products/${product.slug}`)
revalidatePath('/products')  // Invalidate listing too
```

### 6.2 Image Optimization

```typescript
// next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '*.supabase.co',
        pathname: '/storage/v1/object/public/**',
      },
    ],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
}
```

**Upload-time optimization:**
- Use `browser-image-compression` on client before upload
- Target: < 200KB per image at 1200px wide
- Always upload as WebP or JPEG (not PNG for photos)

### 6.3 Caching Strategy

```
Browser Cache (static assets)
  └── Vercel CDN caches JS/CSS/fonts with immutable headers
  └── Cache-Control: public, max-age=31536000, immutable

Next.js Data Cache (fetch deduplication)
  └── Supabase queries in RSC are automatically deduplicated per request

CDN Cache (pages)
  └── ISR pages cached at Vercel edge
  └── s-maxage=60, stale-while-revalidate=300

Client Cache (TanStack Query)
  └── Cart and wishlist synced with staleTime: 30000
  └── Product listings: staleTime: 60000
```

### 6.4 Bundle Optimization

```typescript
// Dynamic imports for heavy components
const ProductImageGallery = dynamic(
  () => import('@/components/products/ProductImageGallery'),
  { loading: () => <GallerySkeleton />, ssr: false }
)

// Lazy load admin chart library
const RevenueChart = dynamic(
  () => import('@/components/admin/RevenueChart'),
  { ssr: false }
)
```

### 6.5 Core Web Vitals Targets

| Metric | Target | Strategy |
|--------|--------|---------|
| LCP (Largest Contentful Paint) | < 2.5s | Priority image on hero, ISR, CDN |
| FID / INP | < 100ms | Minimize JS on initial load, Server Components |
| CLS (Cumulative Layout Shift) | < 0.1 | Reserve image dimensions, skeleton screens |
| TTFB | < 800ms | ISR edge caching, Supabase regional hosting |

---

## 7. Recommended Additional Tools

These tools are justified additions beyond the core stack, selected for low cost and high value:

### 7.1 State Management — Zustand + TanStack Query

**Zustand** (cart, wishlist, UI state): Zero dependencies, tiny bundle (~1KB). Perfect for client-side cart state that needs to persist across page navigations without prop drilling.

**TanStack Query** (server state synchronization): Handles caching, background refetching, optimistic updates, and loading states for cart sync and wishlist. Eliminates custom `useState + useEffect` fetch patterns.

```bash
npm install zustand @tanstack/react-query
```

### 7.2 Email — Resend

**Why Resend over alternatives:**
- Free tier: 3,000 emails/month (sufficient for MVP)
- React Email for type-safe email templates
- Excellent deliverability
- Simple API, no SMTP complexity

```bash
npm install resend react-email @react-email/components
```

### 7.3 Form Validation — React Hook Form + Zod

**Why both are necessary:**
- `react-hook-form`: Performant form state without re-renders
- `zod`: Type-safe validation shared between client and server (Server Actions)

```bash
npm install react-hook-form zod @hookform/resolvers
```

### 7.4 Analytics — Vercel Analytics + Vercel Speed Insights

**Why:** Zero cost on Hobby plan, zero config, no cookie banner needed (privacy-first), directly integrated with Vercel deployment.

```bash
npm install @vercel/analytics @vercel/speed-insights
```

### 7.5 SEO — Next SEO (or native Next.js Metadata API)

Use **Next.js built-in Metadata API** (App Router) — no extra package needed. Add `next-sitemap` for XML sitemap generation.

```bash
npm install next-sitemap
```

### 7.6 Logging & Monitoring

**Vercel Dashboard** covers basic function logs and error tracking at no cost. For production error monitoring, add **Sentry** free tier:

```bash
npm install @sentry/nextjs
```

Sentry free tier: 5,000 errors/month — sufficient for early-stage production.

### Tool Summary Table

| Tool | Purpose | Monthly Cost |
|------|---------|-------------|
| Zustand | Client state (cart, UI) | Free |
| TanStack Query | Server state sync, caching | Free |
| Resend | Transactional email | Free (3K/month) |
| React Hook Form + Zod | Form handling + validation | Free |
| Vercel Analytics | Traffic analytics | Free |
| next-sitemap | XML sitemap generation | Free |
| Sentry (free) | Error monitoring | Free (5K errors/mo) |

---

*Document Version 1.0 — Shree Furniture System Design Document*
