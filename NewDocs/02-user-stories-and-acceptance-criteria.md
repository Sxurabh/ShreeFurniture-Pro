# 02 — User Stories & Acceptance Criteria
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Format:** As a [persona], I want [action], so that [value]

---

## Epic 1 — Product Discovery

### US-001 | Browse Products by Collection
**As a** visitor, **I want to** browse furniture by room type (Living Room, Bedroom, Dining, Office), **so that** I can find products relevant to my specific decorating need.

**Acceptance Criteria:**
- [ ] `/collections/living-room`, `/collections/bedroom`, etc. routes exist and are indexable by search engines
- [ ] Each PLP renders in < 2.5s LCP on 4G mobile simulation
- [ ] Page is served from ISR cache (revalidate 60s); stale-while-revalidate behaviour confirmed
- [ ] Collection grid on homepage links correctly to each PLP

---

### US-002 | Filter and Sort Products
**As a** visitor, **I want to** filter products by material, colour, and price range and sort results, **so that** I can narrow down choices quickly without scrolling through irrelevant items.

**Acceptance Criteria:**
- [ ] Filter state is reflected in URL query params (e.g., `?material=wood&price=5000-15000`)
- [ ] Applying or removing a filter does not trigger a full page reload
- [ ] Back button restores previous filter state
- [ ] Sort options available: Price Low→High, High→Low, Newest, Most Popular
- [ ] Empty state message shown when no products match filters
- [ ] Filter panel renders as sidebar on desktop, bottom sheet on mobile

---

### US-003 | Search for Products
**As a** visitor, **I want to** search by product name or keyword, **so that** I can find specific items without navigating categories.

**Acceptance Criteria:**
- [ ] Search bar accessible from all pages via header
- [ ] First result appears within 200ms of typing (debounced at 300ms)
- [ ] Typo-tolerance: "sopha" returns sofa results
- [ ] "No results" state suggests related categories
- [ ] On mobile, search expands to full-screen overlay
- [ ] Search query is logged to PostHog for analytics

---

### US-004 | View Product Details
**As a** visitor, **I want to** see complete product information including images, dimensions, materials, and price, **so that** I can make a confident purchase decision.

**Acceptance Criteria:**
- [ ] Primary product image loads with `priority` flag (LCP target met)
- [ ] Image gallery supports thumbnail navigation; pinch-to-zoom works on mobile
- [ ] Switching colour or size variant updates: price, images, stock status — without page reload
- [ ] Out-of-stock variants are visually disabled (greyed out) but not removed from UI
- [ ] Delivery estimate shows when user enters PIN code
- [ ] Specifications table shows: L×W×H, weight, material, finish
- [ ] JSON-LD Product schema is present in page `<head>`

---

## Epic 2 — Cart & Wishlist

### US-005 | Add Item to Cart
**As a** visitor, **I want to** add a product to my cart with a selected variant, **so that** I can collect items before purchasing.

**Acceptance Criteria:**
- [ ] "Add to Cart" button is prominent and sticky on mobile PDP
- [ ] Cart drawer opens automatically after successful add
- [ ] UI updates optimistically in < 500ms (before server confirmation)
- [ ] Cart badge in header increments immediately
- [ ] Quantity cannot exceed available stock (UI enforced)
- [ ] If variant not selected, user is prompted to select before adding

---

### US-006 | Manage Cart Items
**As a** visitor, **I want to** update quantities or remove items from my cart, **so that** I can adjust my order before checkout.

**Acceptance Criteria:**
- [ ] Quantity stepper increments/decrements correctly; cannot go below 1 or above stock
- [ ] Remove button deletes line item; cart re-totals immediately
- [ ] Cart persists across browser refresh and new tab (Redis, 7-day TTL)
- [ ] Empty cart state shows a CTA to continue browsing
- [ ] Free shipping threshold (₹5,000) is clearly communicated

---

### US-007 | Save Items to Wishlist
**As a** logged-in user, **I want to** save products to a wishlist, **so that** I can track items I'm interested in without committing to purchase.

**Acceptance Criteria:**
- [ ] Wishlist icon on product card and PDP toggles saved state
- [ ] Wishlist is persisted to user account (not lost on logout)
- [ ] Wishlist page shows all saved products with Add to Cart action
- [ ] Adding a wishlisted item to cart does not remove it from wishlist

---

## Epic 3 — Checkout & Payment

### US-008 | Complete Checkout as Guest
**As a** visitor (not logged in), **I want to** complete a purchase without creating an account, **so that** I can buy quickly without friction.

**Acceptance Criteria:**
- [ ] Checkout accessible without login; no forced registration step
- [ ] Email field required for order confirmation delivery
- [ ] Address form validates: all required fields, PIN code format, phone number format
- [ ] Guest order is retrievable post-purchase via order confirmation email link

---

### US-009 | Pay via Indian Payment Methods
**As a** customer, **I want to** pay using UPI, card, EMI, or BNPL options, **so that** I can use my preferred payment method including flexible financing.

**Acceptance Criteria:**
- [ ] Razorpay modal loads within 2 seconds of "Place Order" click
- [ ] UPI, Credit/Debit Cards, Net Banking, EMI, and BNPL (Simpl, LazyPay) are available
- [ ] Payment failure shows a user-friendly error message with retry option (no page reload)
- [ ] Successful payment redirects to `/order/confirm/:id` within 3 seconds
- [ ] Duplicate webhook events do not create duplicate orders (idempotent processing)
- [ ] HMAC-SHA256 signature is verified before any webhook event is processed

---

### US-010 | Receive Order Confirmation
**As a** customer, **I want to** receive an order confirmation after paying, **so that** I have a record of my purchase and expected delivery.

**Acceptance Criteria:**
- [ ] Order confirmation page shows: order ID, items summary, total, estimated delivery
- [ ] Transactional confirmation email sent within 2 minutes via Resend
- [ ] Email contains: order ID, itemised list, total, delivery address, estimated date
- [ ] Order confirmation page is accessible without login for guest users (via unique URL)

---

## Epic 4 — User Account

### US-011 | Register and Log In
**As a** visitor, **I want to** create an account and log in, **so that** I can access order history, saved addresses, and wishlist.

**Acceptance Criteria:**
- [ ] Registration requires: email, password (min 8 chars), first and last name
- [ ] Login with email + password; JWT stored in HTTP-only cookie
- [ ] Session expires after 24 hours of inactivity
- [ ] Forgot password flow: email OTP received within 60 seconds; link valid for 15 minutes
- [ ] Auth token is never exposed in localStorage or JavaScript-accessible storage

---

### US-012 | View Order History
**As a** logged-in user, **I want to** view all my past orders with their status, **so that** I can track deliveries and reference previous purchases.

**Acceptance Criteria:**
- [ ] My Orders page lists all orders: order ID, date, total, payment status, fulfilment status
- [ ] Clicking an order shows full order detail: items, shipping address, payment method, tracking link
- [ ] Tracking link is shown once admin marks order as fulfilled with tracking number

---

## Epic 5 — Admin Operations

### US-013 | Publish a New Product
**As an** admin, **I want to** create a product with images, variants, and pricing in under 5 minutes, **so that** the product is live for customers to purchase.

**Acceptance Criteria:**
- [ ] Product can be created via Medusa Admin with: title, description, images, collection, tags
- [ ] Image upload accepts JPG, PNG, WebP up to 10MB; auto-converted by Cloudinary
- [ ] Variants can be added: colour, size, SKU, price, stock count per variant
- [ ] Publishing a product triggers ISR revalidation; storefront reflects change within 60 seconds
- [ ] Products in "Draft" status are not visible on the storefront

---

### US-014 | Process and Fulfil an Order
**As an** admin, **I want to** view incoming orders, mark them as fulfilled with a tracking number, and issue refunds, **so that** I can manage post-purchase operations efficiently.

**Acceptance Criteria:**
- [ ] Orders list is filterable by status, date, and searchable by order ID / customer email
- [ ] Marking an order as fulfilled requires a tracking number input
- [ ] Fulfilment status change automatically sends shipping notification email to customer
- [ ] Partial and full refunds can be initiated; reflected in Razorpay within 30 seconds
- [ ] Internal notes can be added to any order

---

## Epic 6 — Performance & SEO

### US-015 | Fast Page Loads on Mobile
**As a** visitor on a 4G mobile connection, **I want to** pages to load quickly, **so that** I can browse and purchase without frustration.

**Acceptance Criteria:**
- [ ] LCP < 2.5s across Homepage, PLP, and PDP on 4G mobile simulation (Lighthouse)
- [ ] INP < 200ms on all interactive elements
- [ ] CLS < 0.1 — no layout shifts caused by image loading or font swap
- [ ] Initial JS bundle < 200KB (measured via `next build` output)

---

### US-016 | Search Engine Indexability
**As the** business, **I want** all product and collection pages to be indexable by Google, **so that** organic traffic can be acquired without paid ads.

**Acceptance Criteria:**
- [ ] All PLP and PDP pages are server-rendered (ISR) with `<title>` and `<meta description>`
- [ ] JSON-LD Product schema present on all PDP pages
- [ ] Canonical URLs set correctly; no duplicate content issues
- [ ] `robots.txt` and `sitemap.xml` are present and correctly configured
- [ ] Checkout, account, and cart pages are excluded from indexing via `noindex`

---

*Owner: Product Lead | Shree Furniture | v1.0 — Q1 2026*
