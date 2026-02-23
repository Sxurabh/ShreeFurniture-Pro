# Database Schema Overview

## Core Entities

### Users (Supabase Auth)
- Managed by Supabase internally (`auth.users`).

### Profiles
- `id`: uuid (PK, references auth.users)
- `first_name`: text
- `last_name`: text
- `phone`: text (unique)
- `role`: text (default 'customer')
- `created_at`: timestamptz
- `updated_at`: timestamptz

### Addresses
- `id`: uuid (PK)
- `user_id`: uuid (references profiles)
- `street`: text
- `city`: text
- `state`: text
- `postal_code`: text
- `country`: text
- `is_default`: boolean

### Categories
- `id`: uuid (PK)
- `name`: text
- `slug`: text (unique)
- `description`: text
- `image_url`: text
- `created_at`: timestamptz

### Products
- `id`: uuid (PK)
- `category_id`: uuid (references categories)
- `title`: text
- `slug`: text (unique)
- `description`: text
- `base_price`: numeric
- `is_published`: boolean
- `created_at`: timestamptz
- `updated_at`: timestamptz

### Product Variants
- `id`: uuid (PK)
- `product_id`: uuid (references products)
- `sku`: text (unique)
- `name`: text (e.g., 'Oak', 'Mahogany')
- `price_adjustment`: numeric
- `stock_quantity`: integer
- `images`: text[] (array of URLs)

### Orders
- `id`: uuid (PK)
- `user_id`: uuid (references profiles)
- `total_amount`: numeric
- `status`: text (e.g., 'pending', 'paid', 'shipped', 'delivered', 'cancelled')
- `payment_method`: text
- `razorpay_order_id`: text
- `razorpay_payment_id`: text
- `shipping_address_id`: uuid (references addresses)
- `created_at`: timestamptz
- `updated_at`: timestamptz

### Order Items
- `id`: uuid (PK)
- `order_id`: uuid (references orders)
- `product_variant_id`: uuid (references product_variants)
- `quantity`: integer
- `unit_price`: numeric
- `subtotal`: numeric

### Cart Items
- `id`: uuid (PK)
- `user_id`: uuid (references profiles)
- `product_variant_id`: uuid (references product_variants)
- `quantity`: integer
- `created_at`: timestamptz

### Wishlist
- `id`: uuid (PK)
- `user_id`: uuid (references profiles)
- `product_id`: uuid (references products)
- `created_at`: timestamptz

### Coupons
- `id`: uuid (PK)
- `code`: text (unique)
- `discount_type`: text ('percentage', 'fixed')
- `discount_value`: numeric
- `min_order_amount`: numeric
- `expires_at`: timestamptz
- `is_active`: boolean

### Banners
- `id`: uuid (PK)
- `title`: text
- `image_url`: text
- `link_url`: text
- `is_active`: boolean
- `sort_order`: integer

---

## Relationships

User → Orders  
Order → Order Items  
Product → Variants  
Category → Products  

---

## Performance

Indexes:
- product search vector
- slug unique index
- order date index

---

## Security

RLS:
- users access own data
- public read published products
- admin full access

---

## Multi-tenant Readiness

- role column
- future vendor_id column