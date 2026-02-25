# 03 — Product Requirements Document (PRD)
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Date:** Q1 2026 | **Product Owner:** Shree Furniture Business Lead

---

## 1. Product Vision

Shree Furniture is an India-first, IKEA-inspired online furniture destination that makes high-quality, design-forward furniture accessible to every Indian home. The platform will deliver a best-in-class browsing and buying experience — optimised for mobile, built for trust, and engineered to convert.

**Vision Statement:** *"Make buying furniture online feel as effortless as ordering food — beautiful, trustworthy, and designed for India."*

---

## 2. Target Audience

### Primary Persona — "The Urban Home Maker"
- Age 24–38, Tier 1 & Tier 2 cities
- Setting up a new home or upgrading existing furniture
- Mobile-first browsing habit (Instagram, Pinterest for inspiration)
- Price-sensitive but quality-conscious; willing to pay EMI for premium pieces
- Needs: detailed product images, dimensions, material info, delivery timeline, EMI option

### Secondary Persona — "The Interior Enthusiast"
- Age 30–50, metro cities
- Renovating or redecorating; shops for aesthetics, not just need
- Researches thoroughly before buying; reads reviews, compares materials
- Needs: room-type filtering, style collections, wishlists, room visualisation (future)

### Tertiary Persona — "The Gift Buyer"
- Buying for a new home, wedding, or housewarming
- Needs: curated gift-worthy collections, gift messaging (future), fast delivery confirmation

---

## 3. Business Goals

| Goal | Metric | Target (6 months post-launch) |
|---|---|---|
| Revenue | Monthly GMV | ₹5 Lakhs / month |
| Conversion | Cart-to-purchase rate | ≥ 2.5% |
| Retention | Repeat purchase rate | ≥ 20% within 90 days |
| Acquisition | Monthly organic sessions | ≥ 10,000 |
| Performance | Mobile Lighthouse Score | ≥ 90 |
| Trust | Payment success rate | ≥ 97% |

---

## 4. Product Requirements

### 4.1 Homepage

**Purpose:** Create a strong first impression, communicate brand identity, and funnel users toward product discovery.

**Requirements:**
- Hero section with full-width banner, headline, and primary CTA ("Shop Now" / "Explore Collections")
- Featured Collections grid (Living Room, Bedroom, Dining, Office) with cover images
- Best Sellers product carousel (horizontally scrollable on mobile)
- New Arrivals section
- Trust signals section: "Free Delivery above ₹5,000", "Easy Returns", "EMI Available", "Genuine Warranty"
- Newsletter signup strip (optional for MVP)
- Fully responsive: mobile, tablet, desktop breakpoints

**Acceptance Criteria:**
- Page loads with LCP < 2.5s on 4G mobile
- Hero image served in WebP format via Cloudinary CDN
- Collection grid links correctly to respective PLP

---

### 4.2 Product Listing Page (PLP)

**Purpose:** Enable product discovery through browsing and filtering.

**Requirements:**
- URL structure: `/collections/{room-slug}` (e.g., `/collections/living-room`)
- Product grid: 2 columns on mobile, 3 on tablet, 4 on desktop
- Each product card shows: thumbnail, name, price, discount badge (if applicable), "Add to Wishlist" icon
- Filter sidebar / bottom sheet (mobile): room, material, colour, price range, style
- Sort options: Price Low→High, High→Low, Newest, Most Popular
- Pagination or infinite scroll (preference: Load More button for SEO)
- Empty state with helpful message when no products match filters
- Breadcrumb navigation

**Acceptance Criteria:**
- Filters update product grid without full page reload (client-side filtering via TanStack Query)
- Faceted filters powered by Algolia return results within 200ms
- All filter states reflected in URL query params (shareable, back-button friendly)

---

### 4.3 Product Detail Page (PDP)

**Purpose:** Provide complete product information to drive purchase confidence and conversion.

**Requirements:**
- Image gallery: primary large image + thumbnail strip; pinch-to-zoom on mobile
- Product name, SKU, price (with GST note), original price + discount %
- Variant selectors: colour swatches, size/dimensions options — dynamic price update
- Stock status indicator: "In Stock", "Only 3 left", "Out of Stock"
- Quantity selector (capped to available inventory)
- Primary CTA: "Add to Cart" button (prominent, sticky on mobile)
- Secondary CTA: "Add to Wishlist"
- Product description with rich text: materials, dimensions, care instructions
- Delivery info: estimated delivery date based on pin code lookup
- Product specifications table: dimensions (L×W×H), weight, material, finish
- Related Products / "Customers also bought" carousel
- Breadcrumb navigation
- SEO: unique `<title>`, `<meta description>`, structured data (JSON-LD Product schema)

**Acceptance Criteria:**
- Variant switching updates price, images, and stock status without page reload
- "Add to Cart" triggers cart drawer open with item confirmation
- Out-of-stock variants are visually disabled, not hidden
- Page LCP < 2.5s; product images use `priority` loading for above-fold image

---

### 4.4 Cart

**Purpose:** Provide a clear, low-friction path from product selection to checkout initiation.

**Requirements:**
- Cart accessible via persistent header icon with item count badge
- Cart opens as a side drawer (desktop) / bottom sheet (mobile)
- Cart items show: thumbnail, name, variant, quantity stepper, unit price, line total
- Remove item functionality
- Order subtotal, GST breakdown, estimated shipping (free above ₹5,000)
- "Proceed to Checkout" primary CTA
- "Continue Shopping" secondary action
- Empty cart state with CTA to browse products
- Cart persisted across sessions (Redis, 7-day TTL)

**Acceptance Criteria:**
- Adding an item to cart takes < 500ms (optimistic UI update via TanStack Query)
- Cart state survives browser refresh and new tab open
- Quantity cannot exceed available stock

---

### 4.5 Checkout

**Purpose:** Complete the purchase with maximum trust and minimum friction.

**Requirements:**

**Step 1 — Contact & Address:**
- Email address (required for guest checkout)
- Delivery address: Full Name, Phone, Address Line 1 & 2, City, State, PIN Code
- Saved addresses for logged-in customers (selectable)
- Real-time PIN code validation (delivery serviceability check)

**Step 2 — Shipping:**
- Display available shipping methods and estimated delivery dates
- Free shipping threshold prominently shown

**Step 3 — Payment:**
- Razorpay integration: UPI, Credit/Debit Cards, Net Banking, EMI, BNPL (Simpl, LazyPay)
- Order summary in sidebar/below on mobile
- GST invoice generation note
- Place Order CTA with order total confirmation

**Post-Payment:**
- Order confirmation page: order ID, summary, estimated delivery, email confirmation note
- Automatic order confirmation email via Resend

**Acceptance Criteria:**
- Guest checkout supported without forced account creation
- Razorpay modal loads within 2 seconds
- Webhook processing is idempotent — duplicate webhook events do not create duplicate orders
- Payment failure shows user-friendly error with retry option

---

### 4.6 User Account

**Purpose:** Improve retention through personalised experience and order tracking.

**Requirements:**
- Register / Login (email + password); forgot password with email OTP reset
- My Orders: list of past orders with status, order ID, total, date; click to view order detail
- Order Detail: items, shipping address, payment method, fulfilment status, tracking link
- My Wishlist: saved products with Add to Cart action
- My Addresses: add, edit, delete saved delivery addresses
- Account Settings: update name, email, password

**Acceptance Criteria:**
- Auth tokens stored in HTTP-only cookies (not localStorage)
- Session expires after 24 hours of inactivity
- Password reset email arrives within 60 seconds

---

### 4.7 Search

**Purpose:** Provide instant, intelligent product discovery for users who know what they want.

**Requirements:**
- Search bar accessible from all pages via header
- Instant search results as user types (debounced at 300ms)
- Results show: product name, thumbnail, price, collection badge
- Typo-tolerance (Algolia / MeiliSearch feature)
- Faceted filtering within search results
- "No results" state with suggested categories
- Search query logged to PostHog for analytics

**Acceptance Criteria:**
- First search result appears within 200ms of typing
- Search is accessible on mobile (expands to full-screen search overlay)

---

### 4.8 Admin — Product Management

**Purpose:** Allow the business team to manage the catalogue efficiently.

**Requirements:**
- Create / edit / delete products with: title, description, images (Cloudinary upload), collection assignment, tags
- Manage product variants: colour, size, price, SKU, inventory count per variant
- Publish / draft / archive product status
- Bulk inventory update (CSV import — Phase 2)
- Collection (room type) management: create, reorder, assign products

**Acceptance Criteria:**
- A new product can be published live on the storefront within 5 minutes of creation
- Image upload accepts JPG, PNG, WebP up to 10MB; auto-converted on Cloudinary
- ISR revalidation triggered on product publish/update so storefront reflects change within 60 seconds

---

### 4.9 Admin — Order Management

**Purpose:** Allow fulfilment and customer service teams to process orders efficiently.

**Requirements:**
- Order list with: order ID, customer, date, total, payment status, fulfilment status; filterable and searchable
- Order detail: items, shipping address, payment info, timeline
- Mark order as fulfilled + add tracking number
- Initiate refund (partial or full) via Razorpay
- Add internal notes to orders

**Acceptance Criteria:**
- Fulfilment status change triggers automatic shipping notification email to customer
- Refund initiation reflects in Razorpay within 30 seconds

---

## 5. Non-Functional Requirements

| Requirement | Target |
|---|---|
| **Performance — LCP** | < 2.5s on 4G mobile |
| **Performance — INP** | < 200ms |
| **Performance — CLS** | < 0.1 |
| **Performance — TTFB** | < 600ms |
| **Availability** | 99.5% uptime (Phase 1); 99.9% (Phase 2 VPS) |
| **Mobile Responsiveness** | All pages fully usable on 360px viewport |
| **SEO** | All PLP and PDP pages indexable with structured data |
| **Accessibility** | WCAG 2.1 AA for core purchase flow |
| **Security** | OWASP Top 10 compliance |
| **Browser Support** | Chrome, Safari, Firefox — latest 2 versions |

---

## 6. Feature Roadmap

### Phase 1 — Foundation (Month 1–2)
Home, PLP, PDP, Cart, Checkout, Razorpay, Order Confirmation, Admin Product & Order Management. Live on Vercel + Railway free tier.

### Phase 2 — Commerce Complete (Month 3–4)
User authentication, Wishlist, Order history, Algolia/MeiliSearch, Transactional emails, Admin customisation, Migration to Hetzner VPS.

### Phase 3 — Growth (Month 5–6)
Reviews & Ratings, Loyalty/Referral program, AR/3D product viewer (React Three Fiber), Multi-currency, GST-compliant invoicing, SEO structured data, SMS notifications.

---

## 7. Success Metrics

| Phase | Metric | Target |
|---|---|---|
| Phase 1 Launch | First successful live transaction | Day 1 post-launch |
| Month 1 | Checkout completion rate | ≥ 60% of initiated checkouts |
| Month 3 | Monthly returning customer % | ≥ 20% |
| Month 6 | Monthly GMV | ₹5 Lakhs |
| Month 6 | Organic search sessions | ≥ 10,000/month |

---

*Document Owner: Product Lead | Shree Furniture | v1.0 — Q1 2026*
