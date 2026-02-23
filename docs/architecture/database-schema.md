# Database Schema

> **This is the canonical schema reference.** All other docs defer to this file.
> The SQL source of truth lives in `/supabase/migrations/`.
> TypeScript types are auto-generated to `/src/types/database.ts` — never edit that file manually.

---

## Quick Reference — All Tables

| Table | Description |
|-------|-------------|
| `profiles` | Public user profile linked to `auth.users` |
| `addresses` | Saved delivery addresses per user |
| `categories` | Product categories (supports subcategories via `parent_id`) |
| `products` | Main product records |
| `product_images` | Product images (separate table, ordered, with primary flag) |
| `product_variants` | Product variants (color, size, material, etc.) |
| `cart_items` | Persistent cart for logged-in users |
| `wishlist_items` | Saved wishlist products per user |
| `orders` | Order records with JSONB address snapshot |
| `order_items` | Line items with price/name snapshot at time of order |
| `coupons` | Discount codes |
| `banners` | Homepage promotional banners |

---

## Profiles

```sql
CREATE TABLE public.profiles (
  id            UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name     TEXT,                          -- single field (not first_name/last_name)
  phone         TEXT,
  avatar_url    TEXT,
  role          TEXT DEFAULT 'customer'
                  CHECK (role IN ('customer', 'admin')),
  created_at    TIMESTAMPTZ DEFAULT now(),
  updated_at    TIMESTAMPTZ DEFAULT now()
);
```

**Notes:**
- `full_name` is used (not `first_name`/`last_name`) — consistent with Supabase Auth pattern
- `role` column drives all admin access checks
- Created automatically on user signup via Supabase Auth trigger

---

## Addresses

```sql
CREATE TABLE public.addresses (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  label         TEXT DEFAULT 'Home',           -- 'Home', 'Work', 'Other'
  full_name     TEXT NOT NULL,
  phone         TEXT NOT NULL,
  line1         TEXT NOT NULL,                 -- house/flat number, building
  line2         TEXT,                          -- area, landmark (optional)
  city          TEXT NOT NULL,
  state         TEXT NOT NULL,
  pincode       TEXT NOT NULL,                 -- 6-digit Indian PIN code
  is_default    BOOLEAN DEFAULT false,
  created_at    TIMESTAMPTZ DEFAULT now()
);
```

**Notes:**
- Uses `line1`/`line2`, not `street` — matches checkout form fields
- Uses `pincode`, not `postal_code` — Indian market convention
- No `country` field — India only in MVP
- Address is **copied as JSONB snapshot** into `orders.shipping_address` at checkout time — changing this record later does NOT affect past orders

---

## Categories

```sql
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
```

**Notes:**
- Self-referencing `parent_id` for subcategory support
- `slug` is used in URL: `/products?category={slug}`

---

## Products

```sql
CREATE TABLE public.products (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category_id           UUID REFERENCES public.categories(id) ON DELETE SET NULL,
  name                  TEXT NOT NULL,
  slug                  TEXT NOT NULL UNIQUE,
  description           TEXT,
  short_desc            TEXT,
  base_price            NUMERIC(10,2) NOT NULL,
  sale_price            NUMERIC(10,2),            -- null means no sale
  sku                   TEXT UNIQUE,
  material              TEXT,
  brand                 TEXT DEFAULT 'Shree Furniture',
  weight_kg             NUMERIC(6,2),
  dimensions_cm         JSONB,                    -- { "length": 120, "width": 60, "height": 75 }
  is_published          BOOLEAN DEFAULT false,
  is_featured           BOOLEAN DEFAULT false,
  stock_quantity        INT DEFAULT 0,
  low_stock_threshold   INT DEFAULT 5,
  search_vector         TSVECTOR,                 -- auto-updated via trigger
  meta_title            TEXT,
  meta_description      TEXT,
  created_at            TIMESTAMPTZ DEFAULT now(),
  updated_at            TIMESTAMPTZ DEFAULT now()
);

-- Auto-update search_vector on insert/update
CREATE OR REPLACE FUNCTION update_product_search_vector()
RETURNS TRIGGER AS $$
BEGIN
  NEW.search_vector := to_tsvector('english',
    COALESCE(NEW.name, '')        || ' ' ||
    COALESCE(NEW.description, '') || ' ' ||
    COALESCE(NEW.material, '')    || ' ' ||
    COALESCE(NEW.brand, '')
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_search_vector_update
  BEFORE INSERT OR UPDATE ON public.products
  FOR EACH ROW EXECUTE FUNCTION update_product_search_vector();
```

**Notes:**
- `stock_quantity` is the total across all variants for quick availability checks
- Individual variant stock is in `product_variants.stock_quantity`
- `is_published = false` means not visible to customers (RLS enforced)
- `dimensions_cm` uses JSONB: `{ "length": 120, "width": 60, "height": 75 }`

---

## Product Images

```sql
CREATE TABLE public.product_images (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id    UUID NOT NULL REFERENCES public.products(id) ON DELETE CASCADE,
  url           TEXT NOT NULL,
  alt_text      TEXT,
  display_order INT DEFAULT 0,
  is_primary    BOOLEAN DEFAULT false,
  created_at    TIMESTAMPTZ DEFAULT now()
);
```

**Notes:**
- Images are a **separate table**, NOT stored as `text[]` in products or variants
- `display_order` allows drag-and-drop reordering in admin
- `is_primary` marks the main image shown on product cards
- URLs point to Supabase Storage CDN

---

## Product Variants

```sql
CREATE TABLE public.product_variants (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id      UUID NOT NULL REFERENCES public.products(id) ON DELETE CASCADE,
  name            TEXT NOT NULL,    -- e.g., "Oak / Large", "Walnut"
  sku             TEXT UNIQUE,
  price_delta     NUMERIC(10,2) DEFAULT 0, -- added to products.base_price
  stock_quantity  INT DEFAULT 0,
  attributes      JSONB,            -- { "color": "Oak", "size": "Large" }
  is_active       BOOLEAN DEFAULT true,
  created_at      TIMESTAMPTZ DEFAULT now()
);
```

**Notes:**
- Final price = `products.base_price + product_variants.price_delta`
- `attributes` is flexible JSONB — no fixed color/size columns
- Variants do **not** have their own image arrays — images are on the product level

---

## Cart Items

```sql
CREATE TABLE public.cart_items (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  product_id  UUID NOT NULL REFERENCES public.products(id) ON DELETE CASCADE,
  variant_id  UUID REFERENCES public.product_variants(id) ON DELETE SET NULL,
  quantity    INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
  created_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, product_id, variant_id)   -- prevents duplicate rows
);
```

**Notes:**
- Only exists for **logged-in users** — guest cart lives in localStorage (Zustand)
- On login, localStorage cart is merged into this table (quantities combined)
- Max quantity per item is enforced in application logic (default: 10)
- `variant_id` is nullable — some products have no variants

---

## Wishlist Items

```sql
CREATE TABLE public.wishlist_items (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  product_id  UUID NOT NULL REFERENCES public.products(id) ON DELETE CASCADE,
  created_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, product_id)
);
```

---

## Orders

```sql
CREATE TYPE order_status AS ENUM (
  'pending_payment',
  'payment_failed',
  'confirmed',
  'processing',
  'shipped',
  'out_for_delivery',
  'delivered',
  'cancelled',
  'refunded'
);

CREATE TABLE public.orders (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number         TEXT UNIQUE NOT NULL,  -- e.g., SF-2026-00001
  user_id              UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  guest_email          TEXT,                  -- non-null for guest orders
  status               order_status DEFAULT 'pending_payment',
  subtotal             NUMERIC(10,2) NOT NULL,
  discount_amount      NUMERIC(10,2) DEFAULT 0,
  shipping_amount      NUMERIC(10,2) DEFAULT 0,
  tax_amount           NUMERIC(10,2) DEFAULT 0,
  total_amount         NUMERIC(10,2) NOT NULL,
  shipping_address     JSONB NOT NULL,        -- SNAPSHOT — see note below
  coupon_id            UUID REFERENCES public.coupons(id) ON DELETE SET NULL,
  razorpay_order_id    TEXT,
  razorpay_payment_id  TEXT,
  payment_method       TEXT,                  -- e.g., 'upi', 'card', 'netbanking'
  tracking_number      TEXT,                  -- entered manually by admin
  notes                TEXT,
  shipped_at           TIMESTAMPTZ,
  delivered_at         TIMESTAMPTZ,
  created_at           TIMESTAMPTZ DEFAULT now(),
  updated_at           TIMESTAMPTZ DEFAULT now()
);
```

**Critical: shipping_address is a JSONB snapshot**

At checkout, copy the address into the order as JSONB — do NOT use a foreign key reference. This ensures the order record is never affected by future changes to the user's address book.

```typescript
// Example JSONB shape for shipping_address
{
  "full_name": "Priya Sharma",
  "phone": "9876543210",
  "line1": "Flat 4B, Sunrise Apartments",
  "line2": "Near Inorbit Mall",
  "city": "Mumbai",
  "state": "Maharashtra",
  "pincode": "400063"
}
```

---

## Order Items

```sql
CREATE TABLE public.order_items (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id      UUID NOT NULL REFERENCES public.orders(id) ON DELETE CASCADE,
  product_id    UUID REFERENCES public.products(id) ON DELETE SET NULL,
  variant_id    UUID REFERENCES public.product_variants(id) ON DELETE SET NULL,
  product_name  TEXT NOT NULL,    -- SNAPSHOT of name at time of order
  variant_name  TEXT,             -- SNAPSHOT of variant name at time of order
  unit_price    NUMERIC(10,2) NOT NULL,  -- SNAPSHOT of price at time of order
  quantity      INT NOT NULL CHECK (quantity > 0),
  total_price   NUMERIC(10,2) NOT NULL  -- unit_price * quantity
);
```

**Critical:** `product_name` and `unit_price` are **snapshots** taken at checkout time. If a product is later deleted or its price changes, the order history remains accurate.

---

## Coupons

```sql
CREATE TABLE public.coupons (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code              TEXT NOT NULL UNIQUE,
  description       TEXT,
  discount_type     TEXT NOT NULL CHECK (discount_type IN ('percentage', 'fixed')),
  discount_value    NUMERIC(10,2) NOT NULL,
  min_order_amount  NUMERIC(10,2) DEFAULT 0,
  max_uses          INT,                   -- null = unlimited
  used_count        INT DEFAULT 0,
  is_active         BOOLEAN DEFAULT true,
  starts_at         TIMESTAMPTZ,
  expires_at        TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT now()
);
```

---

## Banners

```sql
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
```

---

## Relationships

```
auth.users ──── profiles
                  │
                  ├── addresses (1:N)
                  ├── cart_items (1:N)
                  ├── wishlist_items (1:N)
                  └── orders (1:N)
                              │
                              └── order_items (1:N)
                                      │
                                      ├── products (ref, nullable)
                                      └── product_variants (ref, nullable)

categories (self-ref parent_id)
    └── products (1:N)
            ├── product_images (1:N)
            └── product_variants (1:N)

coupons ← referenced by orders.coupon_id
banners (standalone)
```

---

## Indexes

```sql
-- Product performance
CREATE INDEX idx_products_category    ON public.products(category_id);
CREATE INDEX idx_products_slug        ON public.products(slug);
CREATE INDEX idx_products_published   ON public.products(is_published);
CREATE INDEX idx_products_featured    ON public.products(is_featured);
CREATE INDEX idx_products_price       ON public.products(base_price);
CREATE INDEX idx_products_search      ON public.products USING GIN(search_vector);

-- Order performance
CREATE INDEX idx_orders_user          ON public.orders(user_id);
CREATE INDEX idx_orders_status        ON public.orders(status);
CREATE INDEX idx_orders_created_at    ON public.orders(created_at DESC);
CREATE INDEX idx_order_items_order    ON public.order_items(order_id);

-- Category
CREATE INDEX idx_categories_parent    ON public.categories(parent_id);
CREATE INDEX idx_categories_slug      ON public.categories(slug);

-- Cart
CREATE INDEX idx_cart_user            ON public.cart_items(user_id);

-- Product images
CREATE INDEX idx_product_images_product ON public.product_images(product_id);
```

---

## Row Level Security (RLS)

### Profiles
```sql
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

-- Users access their own profile only
CREATE POLICY "profiles_own_access" ON public.profiles
  FOR ALL USING (auth.uid() = id);

-- Admins can read all profiles
CREATE POLICY "profiles_admin_read" ON public.profiles
  FOR SELECT USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );
```

### Products
```sql
ALTER TABLE public.products ENABLE ROW LEVEL SECURITY;

-- Anyone can read published products
CREATE POLICY "products_public_read" ON public.products
  FOR SELECT USING (is_published = true);

-- Admins have full access
CREATE POLICY "products_admin_all" ON public.products
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );
```

### Cart Items
```sql
ALTER TABLE public.cart_items ENABLE ROW LEVEL SECURITY;

CREATE POLICY "cart_own_access" ON public.cart_items
  FOR ALL USING (auth.uid() = user_id);
```

### Orders
```sql
ALTER TABLE public.orders ENABLE ROW LEVEL SECURITY;

-- Customers read their own orders
CREATE POLICY "orders_own_read" ON public.orders
  FOR SELECT USING (auth.uid() = user_id);

-- Admins have full access
CREATE POLICY "orders_admin_all" ON public.orders
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );
```

### Addresses, Wishlist Items
```sql
ALTER TABLE public.addresses ENABLE ROW LEVEL SECURITY;
CREATE POLICY "addresses_own_access" ON public.addresses
  FOR ALL USING (auth.uid() = user_id);

ALTER TABLE public.wishlist_items ENABLE ROW LEVEL SECURITY;
CREATE POLICY "wishlist_own_access" ON public.wishlist_items
  FOR ALL USING (auth.uid() = user_id);
```

### Banners & Coupons (public read)
```sql
ALTER TABLE public.banners ENABLE ROW LEVEL SECURITY;
CREATE POLICY "banners_public_read" ON public.banners
  FOR SELECT USING (is_active = true);
CREATE POLICY "banners_admin_all" ON public.banners
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );

ALTER TABLE public.coupons ENABLE ROW LEVEL SECURITY;
-- Coupons: only admins can read the full list; customers interact via server actions only
CREATE POLICY "coupons_admin_all" ON public.coupons
  FOR ALL USING (
    EXISTS (SELECT 1 FROM public.profiles WHERE id = auth.uid() AND role = 'admin')
  );
```

---

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Tables | `snake_case`, plural | `order_items`, `product_images` |
| Columns | `snake_case` | `created_at`, `user_id` |
| Primary keys | `id` UUID | `id UUID DEFAULT gen_random_uuid()` |
| Foreign keys | `{singular_table}_id` | `product_id`, `order_id` |
| Indexes | `idx_{table}_{column}` | `idx_products_category` |
| Enum types | `snake_case` | `order_status` |
| RLS policies | Descriptive string | `"products_public_read"` |