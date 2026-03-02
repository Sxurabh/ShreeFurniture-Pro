# 03 — Information Architecture
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Scope:** URL structure, navigation hierarchy, page inventory, routing
> **Purpose:** Define the site map, URL patterns, and navigation hierarchy for the storefront.
> **When to read:** Before adding or changing routes, navigation, or page layout structure. AI agents should read this before generating new pages or links.

---

## 1. Site Map

```
shree-furniture.in/
│
├── / (Homepage)
│
├── /collections/
│   ├── /collections/living-room
│   ├── /collections/bedroom
│   ├── /collections/dining
│   └── /collections/office
│
├── /products/
│   └── /products/[handle]          (e.g., /products/oslo-3-seater-sofa)
│
├── /search
│
├── /cart
│
├── /checkout
│   ├── /checkout                   (Step 1: Address)
│   ├── /checkout/shipping          (Step 2: Shipping)
│   └── /checkout/payment           (Step 3: Payment)
│
├── /order/
│   └── /order/confirm/[id]         (Post-payment confirmation)
│
├── /account/
│   ├── /account/login
│   ├── /account/register
│   ├── /account/forgot-password
│   ├── /account/orders             (My Orders list)
│   ├── /account/orders/[id]        (Order detail)
│   ├── /account/wishlist
│   ├── /account/addresses
│   └── /account/settings
│
└── /api/
    └── /api/webhooks/razorpay      (Internal — not user-facing)
```

---

## 2. Next.js App Router File Structure

```
apps/storefront/app/
│
├── layout.tsx                      # Root layout: fonts, global CSS, providers
├── (store)/                        # Route group — shared Header + Footer layout
│   ├── layout.tsx                  # Store layout: Header, Footer, CartDrawer
│   ├── page.tsx                    # Homepage (ISR: revalidate 60s)
│   ├── collections/
│   │   └── [handle]/
│   │       └── page.tsx            # PLP (ISR: revalidate 60s)
│   ├── products/
│   │   └── [handle]/
│   │       └── page.tsx            # PDP (ISR: revalidate 60s)
│   ├── search/
│   │   └── page.tsx                # Search results (CSR)
│   └── cart/
│       └── page.tsx                # Cart page (CSR)
│
├── (checkout)/                     # Route group — no Header/Footer, minimal layout
│   ├── layout.tsx                  # Checkout layout: logo only, progress indicator
│   ├── checkout/
│   │   ├── page.tsx                # Step 1: Address
│   │   ├── shipping/
│   │   │   └── page.tsx            # Step 2: Shipping
│   │   └── payment/
│   │       └── page.tsx            # Step 3: Payment + Razorpay
│   └── order/
│       └── confirm/
│           └── [id]/
│               └── page.tsx        # Order confirmation (SSR — fresh per request)
│
├── (account)/                      # Route group — authenticated layout
│   ├── layout.tsx                  # Account layout: sidebar nav + auth guard
│   └── account/
│       ├── login/page.tsx
│       ├── register/page.tsx
│       ├── forgot-password/page.tsx
│       ├── orders/
│       │   ├── page.tsx            # Order list
│       │   └── [id]/page.tsx       # Order detail
│       ├── wishlist/page.tsx
│       ├── addresses/page.tsx
│       └── settings/page.tsx
│
└── api/
    └── webhooks/
        └── razorpay/
            └── route.ts            # POST handler — Razorpay payment events
```

---

## 3. Navigation Structure

### 3.1 Primary Navigation (Header)

```
[Logo]   [Living Room] [Bedroom] [Dining] [Office]   [🔍 Search] [♡ Wishlist] [👤 Account] [🛒 Cart (n)]
```

- All collection links map to `/collections/{slug}`
- Search icon opens full-screen search overlay on all viewports
- Wishlist icon: links to `/account/wishlist` (or prompts login if guest)
- Account icon: links to `/account/login` (guest) or `/account/orders` (logged in)
- Cart icon: opens CartDrawer (not a page navigation); shows item count badge

### 3.2 Mobile Navigation

```
[Logo]                              [🔍] [🛒 (n)] [☰ Menu]

Hamburger menu opens full-screen overlay:
  Living Room | Bedroom | Dining | Office
  ─────────────────────────────────────
  My Account | My Wishlist | My Orders
```

### 3.3 Footer Navigation

```
Collections          Customer Service      Company
Living Room          Track Your Order      About Us
Bedroom              Returns & Refunds     Contact
Dining               FAQs                 Privacy Policy
Office               Contact Us            Terms of Service
                                           
                     Payment: [UPI] [Visa] [Mastercard] [EMI]
                     © 2026 Shree Furniture
```

---

## 4. Breadcrumb Structure

Breadcrumbs are shown on PLP and PDP pages for navigation and SEO (BreadcrumbList JSON-LD).

```
Homepage > Living Room > Oslo 3-Seater Sofa
Home     > [Collection Name] > [Product Name]
  /      > /collections/living-room > /products/oslo-3-seater-sofa
```

---

## 5. URL Conventions

| Pattern | Example | Notes |
|---|---|---|
| Collections | `/collections/living-room` | Hyphenated, lowercase slug |
| Products | `/products/oslo-3-seater-sofa-grey` | `{name}-{variant}` slug from Medusa |
| Orders | `/order/confirm/ord_01ABCDEF` | Medusa order ID |
| Account sub-pages | `/account/orders/ord_01ABCDEF` | Authenticated only |
| Search | `/search?q=sofa&collection=living-room` | Query params preserved in URL |
| Filters | `/collections/living-room?material=wood&price=5000-15000&sort=price_asc` | All filter state in URL |

---

## 6. Page Inventory & Rendering Modes

| Page | Route | Rendering | Auth Required | Indexed by Google |
|---|---|---|---|---|
| Homepage | `/` | ISR (60s) | No | ✅ Yes |
| Product Listing | `/collections/[handle]` | ISR (60s) | No | ✅ Yes |
| Product Detail | `/products/[handle]` | ISR (60s) | No | ✅ Yes |
| Search | `/search` | CSR | No | ❌ No |
| Cart | `/cart` | CSR | No | ❌ No |
| Checkout — Address | `/checkout` | CSR | No | ❌ No |
| Checkout — Shipping | `/checkout/shipping` | CSR | No | ❌ No |
| Checkout — Payment | `/checkout/payment` | CSR | No | ❌ No |
| Order Confirmation | `/order/confirm/[id]` | SSR | No (guest link) | ❌ No |
| Login | `/account/login` | CSR | No | ❌ No |
| Register | `/account/register` | CSR | No | ❌ No |
| My Orders | `/account/orders` | SSR | ✅ Yes | ❌ No |
| Order Detail | `/account/orders/[id]` | SSR | ✅ Yes | ❌ No |
| Wishlist | `/account/wishlist` | SSR | ✅ Yes | ❌ No |
| My Addresses | `/account/addresses` | SSR | ✅ Yes | ❌ No |
| Account Settings | `/account/settings` | SSR | ✅ Yes | ❌ No |

---

## 7. Checkout Flow State Machine

```
                      ┌─────────────────────────────────────┐
                      │           Cart (non-empty)           │
                      └──────────────────┬──────────────────┘
                                         │ "Proceed to Checkout"
                      ┌──────────────────▼──────────────────┐
                      │   Step 1: Contact & Address          │
                      │   /checkout                          │
                      └──────────────────┬──────────────────┘
                                         │ "Continue to Shipping"
                      ┌──────────────────▼──────────────────┐
                      │   Step 2: Shipping Method            │
                      │   /checkout/shipping                 │
                      └──────────────────┬──────────────────┘
                                         │ "Continue to Payment"
                      ┌──────────────────▼──────────────────┐
                      │   Step 3: Payment (Razorpay modal)   │
                      │   /checkout/payment                  │
                      └────────┬──────────────────┬─────────┘
                               │                  │
                    Payment    │                  │  Payment
                    Success    │                  │  Failure
                               │                  │
              ┌────────────────▼──┐    ┌──────────▼──────────────┐
              │  Order Confirm    │    │  Error state — retry    │
              │  /order/confirm/  │    │  shown inline           │
              │  [id]             │    │  (no page redirect)     │
              └───────────────────┘    └─────────────────────────┘
```

---

## 8. Search Architecture — User Flow

```
User focuses search input (header)
  → Search overlay opens (full-screen on mobile)
  
User types ≥ 2 characters (debounced 300ms)
  → Algolia InstantSearch fires query
  → Results rendered: product name, thumbnail, price, collection badge
  
User presses Enter or clicks "See all results"
  → Navigates to /search?q={query}
  → Full results page with faceted filters
  
No results state
  → "No results for '{query}'" message
  → Suggested collections shown as fallback CTAs
```

---

*Owner: Product Lead | Shree Furniture | v1.0 — Q1 2026*
