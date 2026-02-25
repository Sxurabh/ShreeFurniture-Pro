# 01 — Product Requirements
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Status:** Active | **Owner:** Product Lead

---

## 1. Vision

**"Make buying furniture online feel as effortless as ordering food — beautiful, trustworthy, and designed for India."**

Shree Furniture is an India-first, IKEA-inspired online furniture destination delivering a best-in-class browsing and buying experience — optimised for mobile, built for trust, and engineered to convert.

---

## 2. Target Users

| Persona | Age | Context | Key Needs |
|---|---|---|---|
| Urban Home Maker (Primary) | 24–38, Tier 1 & 2 cities | Setting up or upgrading home | Detailed images, dimensions, EMI, delivery timeline |
| Interior Enthusiast (Secondary) | 30–50, metros | Renovating / redecorating | Room-type filters, style collections, wishlists |
| Gift Buyer (Tertiary) | Any | Housewarming / wedding | Curated collections, fast delivery confirmation |

---

## 3. Business Goals (6 Months Post-Launch)

| Metric | Target |
|---|---|
| Monthly GMV | ₹5 Lakhs/month |
| Cart-to-purchase rate | ≥ 2.5% |
| Repeat purchase rate | ≥ 20% within 90 days |
| Monthly organic sessions | ≥ 10,000 |
| Mobile Lighthouse Score | ≥ 90 |
| Payment success rate | ≥ 97% |

---

## 4. Feature Requirements

### 4.1 Homepage

- Hero section: full-width banner, headline, primary CTA ("Shop Now" / "Explore Collections")
- Featured Collections grid: Living Room, Bedroom, Dining, Office — with cover images
- Best Sellers carousel (horizontally scrollable on mobile)
- New Arrivals section
- Trust signals: "Free Delivery above ₹5,000", "Easy Returns", "EMI Available", "Genuine Warranty"
- Fully responsive: mobile (360px+), tablet, desktop

**Acceptance Criteria:**
- LCP < 2.5s on 4G mobile
- Hero image served in WebP via Cloudinary CDN
- Collection grid links correctly to `/collections/{slug}`

---

### 4.2 Product Listing Page (PLP)

- URL: `/collections/{room-slug}` (e.g., `/collections/living-room`)
- Grid: 2 columns mobile / 3 tablet / 4 desktop
- Product card: thumbnail, name, price, discount badge, "Add to Wishlist" icon
- Filter panel: room, material, colour, price range, style
- Sort: Price Low→High, High→Low, Newest, Most Popular
- "Load More" pagination (preferred over infinite scroll for SEO)
- Empty state with helpful messaging
- Breadcrumb navigation

**Acceptance Criteria:**
- Filters update grid without full page reload
- All filter state reflected in URL query params (shareable, back-button safe)
- Algolia returns results within 200ms

---

### 4.3 Product Detail Page (PDP)

- Image gallery: primary large image + thumbnail strip; pinch-to-zoom on mobile
- Product name, SKU, price with GST note, original price + discount %
- Variant selectors: colour swatches, size/dimensions — dynamic price update
- Stock status: "In Stock" / "Only 3 left" / "Out of Stock"
- Quantity selector (capped to inventory)
- Primary CTA: "Add to Cart" (prominent, sticky on mobile)
- Secondary CTA: "Add to Wishlist"
- Rich text description: materials, dimensions, care instructions
- Delivery estimate via pin code lookup
- Specifications table: L×W×H, weight, material, finish
- Related Products / "Customers also bought" carousel
- SEO: unique `<title>`, `<meta description>`, JSON-LD Product schema

**Acceptance Criteria:**
- Variant switch updates price, images, stock status without page reload
- "Add to Cart" triggers cart drawer with item confirmation
- Out-of-stock variants are visually disabled (not hidden)
- LCP < 2.5s; above-fold product image uses `priority` loading

---

### 4.4 Cart

- Persistent header icon with item count badge
- Opens as side drawer (desktop) / bottom sheet (mobile)
- Cart item: thumbnail, name, variant, quantity stepper, unit price, line total
- Remove item
- Order subtotal, GST breakdown, estimated shipping (free above ₹5,000)
- "Proceed to Checkout" CTA; "Continue Shopping" secondary
- Empty cart state with browse CTA
- Persisted across sessions via Redis (7-day TTL)

**Acceptance Criteria:**
- Add to cart completes in < 500ms (optimistic UI update)
- Cart survives browser refresh and new tab
- Quantity cannot exceed available stock

---

### 4.5 Checkout (Multi-Step)

**Step 1 — Address:**
- Email (supports guest checkout)
- Full Name, Phone, Address Line 1 & 2, City, State, PIN Code
- Saved addresses selectable for logged-in users
- Real-time PIN code serviceability validation

**Step 2 — Shipping:**
- Available methods with estimated delivery dates
- Free shipping threshold shown prominently

**Step 3 — Payment:**
- Razorpay: UPI, Credit/Debit Cards, Net Banking, EMI, BNPL (Simpl, LazyPay)
- Order summary sidebar (desktop) / collapsed section (mobile)
- Place Order CTA with total confirmation

**Post-Payment:**
- Order confirmation page: order ID, summary, estimated delivery, email note
- Automatic order confirmation email via Resend

**Acceptance Criteria:**
- Guest checkout — no forced account creation
- Razorpay modal loads within 2 seconds
- Webhook processing is idempotent — no duplicate orders
- Payment failure shows friendly error with retry option

---

### 4.6 User Account

- Register / Login (email + password); forgot password with OTP reset
- My Orders: list with status, order ID, total, date; clickable for detail
- Order Detail: items, address, payment method, fulfilment status, tracking link
- My Wishlist: saved products with Add to Cart action
- My Addresses: add, edit, delete
- Account Settings: update name, email, password

**Acceptance Criteria:**
- Auth tokens in HTTP-only cookies (not localStorage)
- Session expires after 24 hours of inactivity
- Password reset email within 60 seconds

---

### 4.7 Search

- Search bar accessible from all pages via header
- Instant results as user types (debounced 300ms)
- Result shows: name, thumbnail, price, collection badge
- Typo-tolerance enabled
- Faceted filtering within search results
- "No results" state with suggested categories
- Search queries logged to PostHog

**Acceptance Criteria:**
- First result appears within 200ms
- Mobile: expands to full-screen search overlay

---

### 4.8 Admin — Product Management

- Create / edit / delete products: title, description, images (Cloudinary upload), collection, tags
- Manage variants: colour, size, price, SKU, inventory count per variant
- Product statuses: publish / draft / archive
- Collection management: create, reorder, assign products

**Acceptance Criteria:**
- New product live on storefront within 5 minutes of creation
- Image upload: JPG, PNG, WebP up to 10MB; auto-converted by Cloudinary
- ISR revalidation triggered on publish/update; storefront reflects change within 60 seconds

---

### 4.9 Admin — Order Management

- Order list: filterable/searchable by order ID, customer, date, total, status
- Order detail: items, address, payment info, timeline
- Mark as fulfilled + add tracking number
- Initiate partial or full refund via Razorpay
- Internal notes on orders

**Acceptance Criteria:**
- Fulfilment status change triggers shipping notification email to customer
- Refund initiates in Razorpay within 30 seconds

---

## 5. Non-Functional Requirements

| Requirement | Target |
|---|---|
| LCP | < 2.5s on 4G mobile |
| INP | < 200ms |
| CLS | < 0.1 |
| TTFB | < 600ms |
| Initial JS Bundle | < 200KB |
| Availability | 99.5% (Phase 1); 99.9% (Phase 2) |
| Mobile viewport | Fully usable at 360px |
| SEO | All PLP/PDP indexable with structured data |
| Accessibility | WCAG 2.1 AA — core purchase flow |
| Security | OWASP Top 10 compliance |
| Browser Support | Chrome, Safari, Firefox — latest 2 versions |

---

## 6. Out of Scope (MVP)

The following are explicitly deferred to Phase 2–3:

- AR / 3D Product Viewer (React Three Fiber)
- Loyalty & Referral Program
- Multi-currency / GST-compliant invoicing
- SMS & Push Notifications
- Reviews & Ratings system
- Marketing email automation
- Bulk CSV inventory import
- Gift messaging

---

*Owner: Product Lead | Shree Furniture | v1.0 — Q1 2026*
