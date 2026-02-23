# Shree Furniture — Product Requirements Document (PRD)

> **Project:** Shree Furniture E-Commerce Platform
> **Document Type:** Product Requirements Document
> **Version:** 1.0
> **Date:** February 2026
> **Stack:** Next.js · TypeScript · Supabase · ShadCN UI · Tailwind CSS

---

## Table of Contents

1. [Product Vision & Goals](#1-product-vision--goals)
2. [User Personas](#2-user-personas)
3. [User Journeys](#3-user-journeys)
4. [Feature Specifications](#4-feature-specifications)
5. [Acceptance Criteria](#5-acceptance-criteria)
6. [Success Metrics](#6-success-metrics)
7. [Constraints & Assumptions](#7-constraints--assumptions)

---

## 1. Product Vision & Goals

### 1.1 Vision Statement

> Shree Furniture aims to be the **go-to online destination for quality furniture in India**, delivering a premium, trust-inspiring shopping experience where customers can confidently discover, evaluate, and purchase furniture that fits their home and budget — from the comfort of any device.

### 1.2 Product Goals

| Goal | Description | Timeframe |
|------|-------------|-----------|
| **G1: Launch** | Ship a fully functional e-commerce platform with end-to-end purchase capability | Month 3 |
| **G2: Revenue** | Achieve ₹5,00,000 in monthly GMV (Gross Merchandise Value) | Month 6 |
| **G3: Experience** | Deliver a sub-3s page load time and ≥ 4.0 average customer rating | Month 4 |
| **G4: Catalog** | List 200+ products across 8+ categories with complete specifications | Month 3 |
| **G5: Conversion** | Achieve ≥ 2.5% cart-to-purchase conversion rate | Month 5 |

### 1.3 Business Objectives

- **Expand market reach** beyond the physical store's local geography
- **Reduce manual order processing** overhead through automated order management
- **Increase average order value** through smart product recommendations and bundle presentation
- **Build a loyal customer base** with accounts, order history, and personalized wishlist
- **Enable data-driven decisions** through analytics on top products, traffic, and funnel performance

### 1.4 Design Principles

Inspired by IKEA's digital experience, Shree Furniture adheres to the following principles:

1. **Clarity over cleverness** — Every UI element serves a purpose. No decorative complexity.
2. **Mobile-first always** — Design for the smallest screen first; enhance for larger screens.
3. **Trust through transparency** — Clear pricing, stock availability, delivery timelines, and return policies everywhere.
4. **Speed as a feature** — Every millisecond of delay costs conversions. Performance is non-negotiable.
5. **Accessibility is default** — The platform is usable by everyone, regardless of ability.

---

## 2. User Personas

### Persona 1 — Priya, The Thoughtful Home Buyer

| Attribute | Detail |
|-----------|--------|
| **Age** | 28–35 |
| **Location** | Metro city (Mumbai, Bangalore, Delhi) |
| **Device** | 70% mobile, 30% desktop |
| **Occupation** | Working professional, first home owner |
| **Income** | ₹8L–₹20L per annum |
| **Tech Comfort** | High — daily app user, comfortable with UPI |

**Scenario:** Priya just moved into her first apartment. She's searching for a dining table and 4 chairs within ₹25,000. She browses on her phone during lunch breaks, saves options to a wishlist, and makes the final purchase on her laptop on weekends.

**Goals:**
- Compare multiple products side by side based on dimensions, materials, and price
- Understand exactly what she's buying (detailed images from multiple angles, exact dimensions)
- Trust the platform before sharing payment details
- Track her order after purchase without calling customer support

**Pain Points:**
- Product pages that hide dimensions or material quality
- Inability to filter by specific price brackets
- No order tracking after payment — leaves her anxious
- Checkout forms that ask for too much information

---

### Persona 2 — Rajesh, The Budget-Conscious Parent

| Attribute | Detail |
|-----------|--------|
| **Age** | 40–50 |
| **Location** | Tier 2 city (Pune, Jaipur, Surat) |
| **Device** | 90% mobile (Android) |
| **Occupation** | Small business owner |
| **Income** | ₹5L–₹12L per annum |
| **Tech Comfort** | Medium — uses WhatsApp, Google Pay |

**Scenario:** Rajesh needs a study table and chair set for his son starting college. Budget is under ₹8,000. He's skeptical of online shopping for furniture and needs reassurance at every step.

**Goals:**
- Find affordable, functional furniture quickly
- See clear "In Stock" and delivery timeline information
- Pay via UPI (preferred over cards)
- Have a simple checkout — not too many steps

**Pain Points:**
- Complex multi-page checkouts abandon him
- Products that look different in real life than in photos
- Unclear whether the price includes delivery

---

### Persona 3 — Sunita, The Interior Design Enthusiast

| Attribute | Detail |
|-----------|--------|
| **Age** | 30–42 |
| **Location** | Metro city |
| **Device** | 50% tablet, 50% desktop |
| **Occupation** | Interior designer or design-conscious home maker |
| **Income** | ₹15L–₹40L per annum |
| **Tech Comfort** | High — heavy Pinterest, Instagram user |

**Scenario:** Sunita is furnishing her entire living room. She needs to buy multiple pieces that coordinate aesthetically. She'll browse extensively, create a wishlist, share it with her partner for approval, and then place a consolidated order.

**Goals:**
- Browse by design style / aesthetic category
- See high-quality lifestyle images of furniture in room settings
- Save multiple wishlists for different rooms
- View product in context (color variants, material variants)

**Pain Points:**
- Poor image quality or only one photo per product
- No variant options (e.g., can't choose color)
- Products listed without material or assembly information

---

### Persona 4 — Admin Arjun, The Store Manager

| Attribute | Detail |
|-----------|--------|
| **Age** | 25–40 |
| **Location** | In-store or remote |
| **Device** | Desktop (laptop) |
| **Role** | Store manager / e-commerce operations |
| **Tech Comfort** | Medium — comfortable with spreadsheets, basic software |

**Scenario:** Arjun manages the Shree Furniture online catalog and fulfillment. Every morning he checks new orders, updates inventory after stock arrivals, and uploads new product photos to the catalog. He runs promotions during festival seasons.

**Goals:**
- Quickly see all new orders and update their status
- Upload product photos and descriptions without technical help
- Set discount codes for Diwali/New Year promotions
- See at a glance which products are low on stock

**Pain Points:**
- Complex admin tools that require developer assistance
- No low-stock alerts until items are completely out of stock
- Difficulty managing multiple product images and reordering them
- No consolidated view of daily/weekly revenue

---

## 3. User Journeys

### Journey 1 — First-Time Visitor to Purchase (Priya)

```
STEP 1: Discovery
  Channel: Google Search "solid wood dining table online India"
  Lands on: Product Detail Page (SEO organic result)
  Experience: Fast-loading page (< 2.5s), professional product images,
              clear price, specifications, and "Add to Cart" CTA visible above fold

STEP 2: Exploration
  Action: Clicks breadcrumb → Dining Furniture category
  Views: 24 products in grid, filter sidebar on desktop / filter drawer on mobile
  Filters by: Price ₹15,000–₹30,000 + In Stock only
  Sorts by: "Price: Low to High"
  Time on page: ~5 minutes

STEP 3: Product Evaluation
  Clicks: 3 different products, compares
  On PDP: Views image gallery (5 images), reads dimensions, checks material
  Action: Adds 2 products to Wishlist (prompted to create account)

STEP 4: Account Creation
  Trigger: "Save to Wishlist" CTA
  Flow: Email signup → Email verification → Returns to same product
  Time: ~2 minutes

STEP 5: Decision & Cart
  Returns: Same evening on desktop
  Views: Wishlist page with saved items
  Decides: Adds dining table to cart
  Cart view: Reviews item, quantity = 1, sees order subtotal

STEP 6: Checkout
  Fills: Shipping address (saved to profile for next time)
  Selects: Standard Delivery (5-7 days) — Free
  Reviews: Order summary with product image, price, delivery date estimate
  Applies: Discount code from Instagram post (WELCOME10 → 10% off)
  Pays: Razorpay payment modal (selects UPI)
  
STEP 7: Confirmation
  Sees: Order confirmation page with order number SF-2026-00142
  Receives: Email confirmation with order details and expected delivery date

STEP 8: Post-Purchase
  Day 3: Visits /orders → sees status: "Shipped" with tracking info
  Day 7: Receives furniture, status updated to "Delivered"
  Follow-up: Review request email sent (post-MVP)
```

---

### Journey 2 — Quick Mobile Purchase (Rajesh)

```
STEP 1: Search
  Channel: Google "study table buy online cheap"
  Lands on: Products listing page → filters already applied by URL params

STEP 2: Browse
  Views product grid on mobile, scrolls through 8–10 items
  Clicks item with clear price tag ₹4,500

STEP 3: PDP on Mobile
  Taps through image gallery (swipe gestures)
  Taps "Add to Cart"
  Cart badge updates: 1

STEP 4: Checkout (Guest)
  Taps Cart → Taps "Proceed to Checkout"
  Selects: "Continue as Guest" → enters email
  Fills: Name, phone, address (auto-complete from browser)
  Selects: UPI payment

STEP 5: Payment
  Razorpay opens UPI app selection
  Pays via Google Pay
  Returns to confirmation page

Total time from landing to purchase: ~8 minutes
```

---

### Journey 3 — Admin Order Management (Arjun)

```
STEP 1: Morning Review
  Logs into /admin → Dashboard shows:
  - 12 new orders since yesterday
  - Revenue: ₹1,24,500 today
  - 3 products below low-stock threshold (highlighted in red)

STEP 2: Process Orders
  Navigates to Orders → Filters: Status = "Confirmed"
  Sees 5 confirmed orders ready to process
  Updates status: "Confirmed" → "Processing" for all 5 (bulk action)

STEP 3: Update Inventory
  Navigates to Products → Inventory view
  Updates stock for 2 newly arrived items
  Sets low-stock threshold to 3 for a high-demand item

STEP 4: Upload New Product
  Navigates to Products → New Product
  Fills: Name, category, description, price, dimensions, materials
  Uploads: 5 images (drag & drop), reorders them, sets primary
  Adds: 2 variants (color: Natural Oak, Walnut)
  Sets: Published = true
  Saves → Product live on storefront immediately

STEP 5: Create Promotion
  Navigates to Promotions → New Coupon
  Creates: DIWALI25 → 25% off → expires Oct 31
  Activates homepage banner with festival imagery
```

---

## 4. Feature Specifications

### 4.1 Homepage

**Description:** The primary entry point that communicates brand identity, showcases offers, and drives navigation to catalog.

**Components:**

| Component | Specification |
|-----------|--------------|
| Hero Banner Carousel | Max 3 slides, auto-advance every 5s, pause on hover/touch, manual dots navigation. Admin-managed images (1440×600px recommended). Each slide has title, subtitle, CTA button with link. |
| Featured Products Section | 8 products grid/carousel. Admin marks products as `is_featured`. Shows product image, name, price (with strikethrough if on sale), "Add to Cart" button. |
| Category Grid | 6 top-level categories displayed as image cards with category name overlay. Clicking navigates to `/products?category={slug}`. |
| Promotional Banner | Static single banner between sections. Admin-managed. Optional. |
| Newsletter Signup | Email capture form (post-MVP). |

**SEO Requirements:**
- Page title: "Shree Furniture — Quality Furniture for Every Home"
- Meta description: Unique, ≤ 160 characters
- Hero image must have descriptive `alt` text
- First featured product image is `priority` loaded (LCP optimization)

---

### 4.2 Product Listing Page (PLP)

**URL Pattern:** `/products` or `/products?category={slug}&min_price=5000&max_price=20000&in_stock=true&q=sofa`

**Layout:**
- Desktop: Left filter sidebar (280px) + right product grid (4 columns)
- Tablet: Collapsible filter sidebar + 3-column grid
- Mobile: Filter drawer (bottom sheet) + 2-column grid

**Filter Specifications:**

| Filter | Type | Options |
|--------|------|---------|
| Category | Checkbox (multi-select) | All top-level + subcategory options |
| Price Range | Dual-handle slider + text inputs | Min: ₹0, Max: ₹1,00,000 |
| Material | Checkbox (multi-select) | Solid Wood, MDF, Metal, Fabric, Leather, Rattan |
| In Stock Only | Toggle | Excludes items with `stock_quantity = 0` |
| Rating | Star selector (min rating) | 3★+, 4★+ (post-MVP when reviews added) |

**Sort Options:**
- Relevance (default for search)
- Price: Low to High
- Price: High to Low
- Newest First
- Best Seller (post-MVP)

**Product Card (Grid Item):**
- Primary image (hover shows second image on desktop)
- Product name (2 lines max, truncated)
- Category label
- Price (base price; sale price with strikethrough if discounted)
- "In Stock" / "Out of Stock" badge
- Wishlist heart icon (top-right corner)
- "Add to Cart" button (appears on hover desktop / always visible mobile)

**Pagination:**
- 20 items per page
- "Load More" button pattern (appends to existing results without page reload)
- URL updates with page number for shareability

---

### 4.3 Product Detail Page (PDP)

**URL Pattern:** `/products/{slug}`

**Layout Structure:**
```
[Breadcrumb: Home > Dining Furniture > Solid Wood Dining Table]

[Image Gallery (left 60%)]     [Product Info (right 40%)]
  - Primary large image          - Name (H1)
  - Thumbnail strip              - Rating & Review count
  - Zoom on hover (desktop)      - Price (+ sale price if applicable)
  - Swipe gallery (mobile)       - Material / Color / Size variants selector
                                 - Stock status ("In Stock – 8 remaining")
                                 - Quantity selector
                                 - [Add to Cart] CTA button (primary)
                                 - [Add to Wishlist] button (secondary)
                                 - Delivery estimate
                                 - Pincode-based delivery check (future)

[Tabs: Description | Specifications | Dimensions | Delivery & Returns]
[Related Products Section]
```

**Image Gallery Specifications:**
- Minimum 3 images, recommended 5 (front, back, side, detail, lifestyle)
- Thumbnail strip: horizontal scroll on mobile, vertical on desktop
- Primary image: 800×800px minimum display
- Lightbox/zoom on click (desktop)
- Swipe gesture support on mobile

**Specifications Tab:**
```
Material: Solid Sheesham Wood
Finish: Walnut Polish
Weight: 45 kg
Seating Capacity: 6 Persons
Assembly Required: Yes (instructions included)
Warranty: 1 Year
Country of Origin: India
```

**Dimensions Tab:**
- Visual diagram (SVG or image showing length × width × height)
- Numerical values: L × W × H in cm
- Seat height (for seating products)

**Structured Data (JSON-LD):**
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Solid Wood Dining Table",
  "image": ["url1", "url2"],
  "description": "...",
  "sku": "SF-DT-001",
  "brand": { "@type": "Brand", "name": "Shree Furniture" },
  "offers": {
    "@type": "Offer",
    "price": "24999",
    "priceCurrency": "INR",
    "availability": "https://schema.org/InStock"
  }
}
```

---

### 4.4 Search

**Implementation:** PostgreSQL full-text search using `tsvector` + `GIN` index (already defined in schema).

**UX Behavior:**
- Search bar always visible in header
- Debounced input: 300ms delay before triggering query
- Results update in real-time as user types (React state)
- Empty state: "No products found for '{query}'" with suggested categories
- Recent searches stored in localStorage (max 5)

**Search Result Page (`/search?q={query}`):**
- Same layout as PLP with filter sidebar
- Shows: "{N} results for '{query}'"
- Results ranked by full-text search relevance + sales volume weight

**What is searched:**
- Product name (weight: 3)
- Product description (weight: 1)
- Material (weight: 2)
- Category name (weight: 2)

---

### 4.5 Cart

**Persistence Strategy:**
- **Guest:** localStorage only (survives browser refresh, not cross-device)
- **Logged In:** Supabase `cart_items` table (synced on login, persists cross-device)
- **Login Merge:** On login, merge localStorage cart with Supabase cart (combine quantities, resolve duplicates)

**Cart UI — Slide-Out Drawer:**
- Opens from right side on all devices
- Shows: Product image thumbnail, name, variant, unit price, quantity stepper, line total, remove button
- Footer: Subtotal + "Proceed to Checkout" CTA
- Coupon field visible in cart (or at checkout)

**Cart Rules:**
- Max quantity per line item: 10 (configurable in admin)
- "Out of Stock" items in cart: Show warning, disable checkout until removed
- Price shown in cart always reflects current product price (not price at time of addition)

---

### 4.6 Checkout

**Flow (3 Steps):**

```
Step 1: Information
  ├── Contact: Email (pre-filled if logged in)
  ├── Guest option: "Continue as guest" or "Sign in"
  └── Shipping address form (with saved addresses for logged-in users)

Step 2: Shipping
  ├── Free standard delivery (5–7 business days) — default
  └── [Future: Express delivery option]

Step 3: Payment
  ├── Order summary (collapsed, expandable)
  ├── Coupon code field
  ├── Price breakdown (subtotal, discount, shipping, tax, total)
  └── "Pay Now" button → opens Razorpay modal
```

**Address Form Fields:**
- Full Name *
- Phone Number * (10-digit Indian mobile)
- Address Line 1 * (house/flat number, building)
- Address Line 2 (area, landmark)
- City *
- State * (dropdown with all Indian states)
- PIN Code * (6-digit, validated)

**Logged-In Users:**
- Pre-fill address from default address in profile
- Option to select from saved addresses
- Option to add new address and optionally save to profile

**Checkout Security:**
- HTTPS only (Vercel enforces)
- No credit card data handled by our server (Razorpay's PCI-DSS compliant iframe handles everything)
- CSRF protection via Next.js Server Actions
- Rate limiting on checkout attempts (5 per minute per IP)

---

### 4.7 Order Tracking

**Order Status States:**

```
pending_payment → payment_failed
               ↓
           confirmed → processing → shipped → out_for_delivery → delivered
               ↓
           cancelled
               ↓
           refunded
```

**Order History Page (`/account/orders`):**
- List of all orders sorted by date (newest first)
- Each order card: Order number, date, status badge, items summary, total, "View Details" link

**Order Detail Page (`/account/orders/{id}`):**
- Full order breakdown: items, prices, address, payment method
- Status timeline (step indicator showing progression)
- Tracking number (when available — manually entered by admin)
- Estimated delivery date

---

### 4.8 Admin Dashboard

**Dashboard KPIs (Home Screen):**

| Metric | Display | Period Options |
|--------|---------|---------------|
| Total Revenue | ₹ amount, trend % vs previous period | Today, 7d, 30d, 90d |
| Order Count | Number + trend | Today, 7d, 30d |
| Average Order Value | ₹ amount | 7d, 30d |
| New Customers | Count + trend | 7d, 30d |
| Top 5 Products | Table with product name, units sold, revenue | 30d |
| Low Stock Alerts | Products below threshold | Always |

**Charts:**
- Revenue trend line chart (daily, last 30 days)
- Orders by status (pie chart)

---

### 4.9 Admin Product Management

**Product List View:**
- Table with columns: Image thumbnail, Name, Category, Price, Stock, Status (Published/Draft), Actions
- Filters: Category, Status, Stock status
- Search by name or SKU
- Bulk actions: Publish, Unpublish, Delete

**Product Form (Create/Edit):**

| Section | Fields |
|---------|-------|
| Basic Info | Name*, Slug* (auto-generated, editable), Short Description, Full Description (rich text) |
| Pricing | Base Price*, Sale Price, Cost Price (for margin tracking) |
| Categorization | Category* (dropdown), Tags |
| Media | Image upload (drag & drop, reorder, set primary, delete) |
| Physical | Weight (kg), Dimensions (L×W×H in cm), Material, SKU |
| Inventory | Stock Quantity*, Low Stock Threshold |
| SEO | Meta Title, Meta Description (with character count) |
| Status | Published toggle, Featured toggle |

---

## 5. Acceptance Criteria

### AC-01: Homepage

- [ ] Hero banner cycles through admin-configured slides automatically every 5 seconds
- [ ] Manual navigation via dots pauses auto-cycle
- [ ] Featured products section shows exactly 8 products marked as featured in admin
- [ ] Category grid shows all active top-level categories
- [ ] Page loads in under 3 seconds on a simulated 3G mobile connection (Lighthouse)
- [ ] All images have descriptive alt text

### AC-02: Product Listing & Filtering

- [ ] Selecting multiple filters combines them with AND logic (e.g., Dining + ₹10K–₹30K shows only dining products in that price range)
- [ ] Filter state is reflected in URL query params
- [ ] Sharing filtered URL shows the same results on opening
- [ ] "Load More" appends 20 products without scrolling to top
- [ ] Products with `stock_quantity = 0` show "Out of Stock" and are excluded when "In Stock Only" filter is active
- [ ] Filter panel is accessible via keyboard navigation

### AC-03: Product Detail Page

- [ ] All product images are displayed in gallery; clicking a thumbnail updates the main image
- [ ] Variant selection (color/size) updates price if variant has price delta
- [ ] Stock status reflects real-time database value
- [ ] "Add to Cart" increments cart count in header immediately (optimistic update)
- [ ] Structured JSON-LD data is present in page `<head>` (verified via Google Rich Results Test)
- [ ] Product page is indexable (no `noindex` directive)

### AC-04: Cart

- [ ] Cart persists across browser refresh for guest users (localStorage)
- [ ] Cart persists across devices for logged-in users (Supabase)
- [ ] Merging guest cart with logged-in cart on login combines quantities correctly
- [ ] Removing an item updates cart total immediately
- [ ] Out-of-stock items show a warning and block checkout

### AC-05: Checkout & Payment

- [ ] Checkout requires valid email (for guest and logged-in)
- [ ] Address form validates all required fields before proceeding
- [ ] Invalid PIN code shows field error
- [ ] Razorpay payment modal opens successfully
- [ ] On payment success: order is created in DB, cart is cleared, user is redirected to confirmation page
- [ ] On payment failure: user sees error message and can retry without re-entering address
- [ ] Order confirmation email is received within 5 minutes of successful payment
- [ ] Applying a valid coupon reduces the total correctly
- [ ] Applying an expired or invalid coupon shows a clear error message
- [ ] Razorpay signature is verified server-side (via Edge Function) before order creation

### AC-06: Authentication

- [ ] Email/password signup creates a profile record in `public.profiles`
- [ ] Google OAuth login creates profile if it doesn't exist
- [ ] Protected routes (`/account/*`, `/checkout`) redirect unauthenticated users to `/login?redirect={originalPath}`
- [ ] After login, user is redirected back to the original intended URL
- [ ] Admin routes (`/admin/*`) are inaccessible to non-admin users (403 page displayed)

### AC-07: Admin — Product Management

- [ ] Admin can create a product with all required fields and it immediately appears in the storefront when Published
- [ ] Admin can upload up to 10 images per product; reordering is drag-and-drop
- [ ] Deleting a product removes it from storefront but preserves it in order history records
- [ ] Low-stock products (below threshold) are highlighted in the admin products table

### AC-08: Admin — Order Management

- [ ] All orders are visible in admin orders list
- [ ] Admin can update order status; change is reflected immediately on customer's order detail page
- [ ] Admin can filter orders by status, date range, and customer name/email

### AC-09: Security

- [ ] Supabase RLS prevents any customer from reading another customer's orders or profile
- [ ] Admin API actions verify `role = 'admin'` before executing (defense in depth beyond RLS)
- [ ] No secret keys (Razorpay secret, Supabase service role) are exposed in client-side JavaScript bundle
- [ ] Content Security Policy headers are set on all responses

### AC-10: SEO & Performance

- [ ] Lighthouse mobile score ≥ 85 (Performance, Accessibility, Best Practices, SEO) on Homepage, PLP, and PDP
- [ ] `sitemap.xml` is accessible at `/sitemap.xml` and includes all published product and category URLs
- [ ] `robots.txt` is accessible and does not block product/category pages
- [ ] All pages have unique `<title>` and `<meta description>` tags

---

## 6. Success Metrics

### 6.1 Business Metrics

| Metric | MVP Target (Month 3) | Growth Target (Month 6) |
|--------|---------------------|------------------------|
| Monthly GMV | ₹1,00,000 | ₹5,00,000 |
| Monthly Orders | 25+ orders | 100+ orders |
| Average Order Value | ₹4,000 | ₹5,000 |
| Repeat Purchase Rate | — | ≥ 15% |
| Discount Code Redemption Rate | — | ≥ 8% |

### 6.2 User Experience Metrics

| Metric | Target |
|--------|--------|
| Cart Abandonment Rate | ≤ 65% |
| Cart-to-Purchase Conversion | ≥ 2.5% |
| Average Session Duration | ≥ 3 minutes |
| Bounce Rate (Homepage) | ≤ 55% |
| Customer Satisfaction (post-purchase survey) | ≥ 4.0/5.0 |
| Mobile Traffic Conversion | ≥ 1.5% |

### 6.3 Technical Performance Metrics

| Metric | Target |
|--------|--------|
| Lighthouse Performance (mobile) | ≥ 85 |
| LCP (Largest Contentful Paint) | < 2.5s |
| CLS (Cumulative Layout Shift) | < 0.1 |
| TTFB (Time to First Byte) | < 800ms |
| Error Rate (5xx) | < 0.1% of requests |
| Uptime | ≥ 99.5% |
| Page Load Time (P95) | < 4s on 3G |

### 6.4 SEO Metrics

| Metric | Target (Month 6) |
|--------|-----------------|
| Google Indexed Pages | ≥ 80% of published products |
| Organic Traffic | ≥ 500 sessions/month |
| Core Web Vitals Pass Rate | ≥ 90% of pages |
| Rich Results (Product Schema) | Implemented and verified |

### 6.5 Admin Efficiency Metrics

| Metric | Target |
|--------|--------|
| Time to publish a new product | < 10 minutes (for admin with photos ready) |
| Time to process and update 10 orders | < 15 minutes |
| Order accuracy (correct status at time of customer inquiry) | ≥ 99% |

---

## 7. Constraints & Assumptions

### Constraints

- **Payment:** Only Razorpay is supported. No PayPal, Stripe, or bank transfer in MVP.
- **Currency:** INR only. No multi-currency.
- **Language:** English only. No Hindi or regional language support in MVP.
- **Shipping:** No real-time shipping rate calculation. Standard free delivery is the only option in MVP. Actual fulfillment is handled offline by the store team.
- **Returns/Refunds:** No automated returns flow. Customer contacts via phone/email. Admin manually updates status.
- **Tax:** GST calculation is simplified (single rate). Not GST-invoice-compliant in MVP (post-MVP scope).
- **Inventory:** Single warehouse / single location inventory. No multi-location stock.

### Assumptions

- The client (Shree Furniture) will provide all product photography and content.
- The client has or will create a Razorpay business account before Phase 2 begins.
- Product catalog will be populated by the client/admin team, not migrated from another system.
- Actual delivery and fulfillment operations remain manual (admin enters tracking number; no carrier API integration).
- The client will handle customer support via phone/WhatsApp. No chat widget in scope.
- All products share a flat shipping policy (free shipping). No weight-based or zone-based shipping in MVP.
- The domain name is already purchased or will be purchased before Phase 5.

---

*Document Version 1.0 — Shree Furniture Product Requirements Document*
