# Shree Furniture — Complete AI Prompt Guide
## Antigravity / Claude Code | Start-to-Deployment | v2.0

> **How to use:** Find your situation, copy the prompt, fill `[brackets]`, paste into Antigravity.
> Your stack is already in CLAUDE.md — never explain it in a prompt again.
> Every prompt that builds something ends with: **"Update STATUS.md when done."**

---

## Table of Contents

1. [Session Start — Always First](#1-session-start--always-first)
2. [Phase 1 — Foundation](#2-phase-1--foundation)
   - 2a. Monorepo & Dev Environment (Week 1)
   - 2b. Backend Live (Week 2)
   - 2c. Third-Party Services Setup
3. [Phase 2 — Storefront Core](#3-phase-2--storefront-core)
   - 3a. Lib, Hooks & API Client
   - 3b. Shared Components
   - 3c. Homepage
   - 3d. Product Pages (PLP + PDP)
   - 3e. Search
4. [Phase 3 — Transactions](#4-phase-3--transactions)
   - 4a. Cart
   - 4b. Checkout
   - 4c. Auth & Account
5. [Phase 3 — Backend Subscribers & Workflows](#5-phase-3--backend-subscribers--workflows)
6. [Phase 4 — Launch Prep](#6-phase-4--launch-prep)
   - 6a. SEO & Indexing
   - 6b. Analytics & Monitoring
   - 6c. Security Hardening
   - 6d. Performance Audit
   - 6e. Go-Live
7. [Testing — Full Suite](#7-testing--full-suite)
8. [Debugging](#8-debugging)
9. [DevOps & CI/CD](#9-devops--cicd)
10. [Session End — Always Last](#10-session-end--always-last)
11. [Anti-Patterns & Quick Reference](#11-anti-patterns--quick-reference)

---

## 1. Session Start — Always First

### Standard Session Start
```
Read these files before doing anything:

1. STATUS.md        ← What's already built
2. PREFERENCES.md   ← Design taste overrides
3. REJECTIONS.md    ← Never-suggest-again log
4. STACK-CHANGES.md ← Tech overrides (check BEFORE reading CLAUDE's tech stack)
5. CLAUDE.md        ← Hard rules (now informed by any stack changes)

Tell me:
- Current phase and what was last worked on
- Any active rejections or preference overrides I should know about
- The next 2–3 tasks from NewDocs/10-development-phases.md
Do not write any code yet.
```

### Session Start With a Specific Goal
```
Read STATUS.md, CLAUDE.md, PREFERENCES.md, REJECTIONS.md, and STACK-CHANGES.md first.
Today's goal: [what you want to accomplish]
Check REJECTIONS.md — confirm this feature/approach hasn't already been rejected.
Confirm this goal is in scope for the current phase before we start.
```

### Resuming After a Gap (Days/Weeks)
```
Read these files before doing anything:
STATUS.md, CLAUDE.md, PREFERENCES.md, REJECTIONS.md, STACK-CHANGES.md, NewDocs/KNOWN-ISSUES.md

Summarise:
- Current phase and last completed item
- Any preferences I've added since the project started
- Any rejections that are active
- Any stack changes that have been made
- Any known issues to be aware of
Do not write code yet.
```

---

## 2. Phase 1 — Foundation

### 2a. Monorepo & Dev Environment (Week 1)

#### Scaffold the Full Monorepo
```
@orchestrator Scaffold the full monorepo for Shree Furniture.
Ref: NewDocs/07-monorepo-structure.md for exact folder structure and file contents.

Create:
- pnpm-workspace.yaml (declares apps/*, backend, packages/*)
- turbo.json (pipeline from doc 07 section 8)
- Root package.json (scripts from doc 07 section 9)
- apps/storefront/ (Next.js 15 App Router, TypeScript strict, Tailwind v4)
- backend/ (MedusaJS v2 scaffold, Node.js 20)
- packages/types/src/index.ts (all interfaces from doc 07 section 4)
- packages/ui/src/tokens.ts (design tokens from doc 07 section 5)
- .env.example (document all variables — no actual values)
- .gitignore (node_modules, .env*, .next, dist, coverage)

Acceptance: pnpm dev starts all apps, pnpm build completes without errors.
Update STATUS.md when done.
```

#### Set Up Root Layout (Fonts, Providers, Global CSS)
```
@frontend-specialist Set up apps/storefront/app/layout.tsx — the root layout.
Configure:
- Self-hosted fonts: Playfair Display (headings) and Inter (body) via next/font/local
  Place font files in public/fonts/
- Global CSS: apps/storefront/app/globals.css — Tailwind base + custom CSS variables
  Map design tokens from packages/ui/src/tokens.ts to CSS custom properties
- Providers wrapper (create apps/storefront/components/providers.tsx):
  - TanStack Query QueryClientProvider
  - PostHog provider (PostHogProvider from posthog-js/react)
- Sentry instrumentation: create apps/storefront/sentry.client.config.ts
- Root metadata: title template "%s | Shree Furniture", default OG image
- Viewport: width=device-width, initial-scale=1
Update STATUS.md when done.
```

#### Set Up Route Group Layouts
```
@frontend-specialist Create the three Next.js App Router route group layouts.

1. apps/storefront/app/(store)/layout.tsx
   - Wraps: Header, Footer, CartDrawer (CartDrawer is always mounted here)
   - This layout applies to: homepage, PLP, PDP, search, cart

2. apps/storefront/app/(checkout)/layout.tsx
   - Minimal layout: logo only, no Header/Footer/CartDrawer
   - Shows CheckoutProgress component at top
   - This layout applies to: all /checkout/* and /order/confirm/* pages

3. apps/storefront/app/(account)/layout.tsx
   - Auth guard: redirect to /account/login if no valid session (check via getMe())
   - Account sidebar nav: My Orders, Wishlist, Addresses, Settings, Logout
   - This layout applies to: all /account/* pages EXCEPT login/register/forgot-password

Ref: NewDocs/03-information-architecture.md section 2 for the full route group structure.
Update STATUS.md when done.
```

#### Set Up Docker Compose for Local Dev
```
@devops-engineer Create infrastructure/docker-compose.yml for local development.
Services:
- PostgreSQL 16: port 5432, named volume for data persistence, health check
- Redis 7: port 6379, health check
Create .env.docker with local connection strings for both services.
Update STATUS.md when done.
```

#### Set Up GitHub Repository + Branch Protection
```
@devops-engineer Document the GitHub repository setup required.
Create a setup checklist in infrastructure/GITHUB-SETUP.md covering:
- Branch protection rules for main: require PR, require CI to pass, no force push
- Required status checks: tsc, eslint, vitest, build
- Auto-delete head branches after merge
- Dependabot config (.github/dependabot.yml): weekly pnpm updates
Also create .github/pull_request_template.md with checklist:
- [ ] STATUS.md updated
- [ ] TypeScript strict passes
- [ ] Tests pass
- [ ] No hardcoded secrets
```

#### Configure next.config.ts
```
@frontend-specialist Create apps/storefront/next.config.ts — the Next.js configuration file.

Configure:
1. Cloudinary image domains:
   images: {
     remotePatterns: [{ protocol: 'https', hostname: 'res.cloudinary.com' }]
   }

2. Sentry wrapper (wraps the entire config):
   import { withSentryConfig } from "@sentry/nextjs"
   — widenClientFileUpload: true, hideSourceMaps: true (source maps uploaded but not shipped to browser)

3. Bundle analyzer (enabled only when ANALYZE=true):
   import withBundleAnalyzer from "@next/bundle-analyzer"
   const withBundleAnalyzer = require('@next/bundle-analyzer')({ enabled: process.env.ANALYZE === 'true' })

4. Turbopack for dev (Next.js 15 default):
   experimental: { turbo: {} }

5. TypeScript and ESLint: ignoreBuildErrors: false (strict — must pass in CI)

6. Trailing slash: false (consistent URLs)

7. PoweredByHeader: false (don't expose Next.js version)

Export order: withBundleAnalyzer(withSentryConfig(nextConfig, sentryOptions))

Also create apps/storefront/.eslintrc.json:
  Extends: next/core-web-vitals, plugin:@typescript-eslint/recommended
  Rules: no-any as error (except with inline comment), no-console as warn

Also create apps/storefront/tailwind.config.ts:
  Content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"]
  Theme: extend with design tokens from packages/ui/src/tokens.ts as CSS vars
  Tailwind v4: use @import "tailwindcss" in globals.css, minimal config file needed

Update STATUS.md when done.
```

#### Set Up CI Pipeline
```
@devops-engineer Create .github/workflows/ci.yml.
Runs on every PR to main:
1. actions/checkout + pnpm setup
2. pnpm install --frozen-lockfile
3. pnpm type-check (turbo run type-check)
4. pnpm lint (turbo run lint)
5. pnpm test (turbo run test -- --reporter=verbose)
6. pnpm build (turbo run build)
Cache: Turborepo remote cache via TURBO_TOKEN secret.
Node version: 20 LTS.
Update STATUS.md when done.
```

---

### 2b. Backend Live (Week 2)

#### Configure MedusaJS v2 Backend
```
@backend-specialist Configure the MedusaJS v2 backend in backend/medusa-config.ts.
Ref: NewDocs/MEDUSA-V2-PATTERNS.md for correct v2 config syntax.

Configure plugins:
- medusa-payment-razorpay (razorpay_key_id, razorpay_secret from env)
- medusa-file-cloudinary (cloud_name, api_key, api_secret from env)
- @medusajs/medusa-plugin-resend (api_key, from from env)
- @medusajs/medusa-plugin-algolia (applicationId, adminApiKey, settings from env)

Configure modules:
- database: DATABASE_URL env var
- redis: REDIS_URL env var
- Custom wishlist module (./src/modules/wishlist)

Do NOT configure MeiliSearch — using Algolia for MVP.
Update STATUS.md when done.
```

#### Create Seed Script
```
@backend-specialist Create backend/src/seed.ts to seed MedusaJS with initial data.
Seed in this order:
1. India region: currency INR, countries: ["in"]
2. Tax rates: 12% on furniture collections (living-room, bedroom, dining)
          18% on storage/shelving collection
3. Collections: living-room, bedroom, dining, office (with handle + title)
4. Shipping options:
   - "Free Shipping" — ₹0 — requires order >= ₹500000 (₹5,000 in paise)
   - "Standard Shipping" — ₹49900 (₹499 in paise) — always available
5. 3 sample products per collection with:
   - 2 variants each (colour options)
   - Prices in paise (e.g., ₹49,999 = 4999900)
   - Cloudinary placeholder thumbnail URL
   - room_type metadata field

All prices as integers (paise). Ref: NewDocs/05-database-schema.md for field names.
Update STATUS.md when done.
```

#### Run Migrations and Verify Backend
```
@backend-specialist Run the MedusaJS database setup and verify the backend works.
Execute in order:
1. npx medusa db:create (create database if not exists)
2. npx medusa db:migrate (run all migrations)
3. npx medusa exec ./src/seed.ts (run seed)

Then verify with these smoke tests:
- GET /store/products → should return seeded products
- GET /store/collections → should return 4 collections
- GET /health → should return 200
- Medusa Admin at /admin → should load and product CRUD should work

Document any migration errors in NewDocs/KNOWN-ISSUES.md.
Update STATUS.md when done.
```

---

### 2c. Third-Party Services Setup

#### Configure Algolia Index
```
@backend-specialist Set up the Algolia search index for products.
Index name: "products" (from ALGOLIA_INDEX_NAME env var)

Configure these settings via Algolia dashboard or init script:
Searchable attributes (in priority order):
  1. title
  2. description
  3. collection.title
  4. tags (unordered)

Facets (for filtering):
  - collection.handle
  - room_type
  - status

Ranking: by Most Popular (use a numeric custom ranking field)

Create backend/src/scripts/algolia-init.ts to configure index settings programmatically.
Run: npx ts-node src/scripts/algolia-init.ts

Then manually sync existing products:
npx medusa exec ./src/scripts/algolia-sync-all.ts

Verify: search for "sofa" in Algolia dashboard returns results.
Update STATUS.md when done.
```

#### Configure Cloudinary
```
@devops-engineer Document the Cloudinary setup required for this project.
Create infrastructure/CLOUDINARY-SETUP.md covering:
- Upload preset name: "shree_furniture_products"
  - Mode: UNSIGNED (required for Medusa Admin uploads)
  - Auto-format: f_auto (WebP/AVIF auto-conversion)
  - Auto-quality: q_auto
  - Folder: shree-furniture/products
- Responsive breakpoints: 400w, 800w, 1200w, 1600w
- Transformation: c_fill, g_auto (smart cropping)

Verify: Upload a test image via Medusa Admin → confirm WebP is served from Cloudinary CDN.
Update STATUS.md when done.
```

#### Set Up Resend Email
```
@devops-engineer Document the Resend email setup.
Create infrastructure/RESEND-SETUP.md covering:
- Add domain: shree-furniture.in to Resend
- Configure DNS: add DKIM TXT records and SPF record to Cloudflare
- Sender address: orders@shree-furniture.in
- Verify: send a test email to confirm delivery

After setup, run a backend smoke test:
- Trigger a test order.placed event via Medusa Admin
- Confirm email received within 2 minutes
Update STATUS.md when done.
```

#### Instrument PostHog
```
@frontend-specialist Add PostHog analytics instrumentation to the storefront.
Install: posthog-js

1. PostHog provider is already in app/layout.tsx (from root layout setup).
   Verify NEXT_PUBLIC_POSTHOG_KEY and NEXT_PUBLIC_POSTHOG_HOST are set.

2. Add these tracking calls in the correct components:
   - Product view: posthog.capture('product_viewed', { productId, productTitle, collection })
     → in PDP page.tsx (server component — use PostHog server-side capture)
   - Add to cart: posthog.capture('add_to_cart', { variantId, quantity, price })
     → in cart-store.ts addItemOptimistic
   - Search query: posthog.capture('search_query', { query })
     → in SearchBar.tsx
   - Checkout started: posthog.capture('checkout_started', { cartTotal })
     → in CartSummary proceed-to-checkout click
   - Order completed: posthog.capture('order_complete', { orderId, total })
     → in order confirmation page

3. Create apps/storefront/lib/analytics.ts with typed wrappers for all events.
Verify: events appear in PostHog dashboard after testing each flow.
Update STATUS.md when done.
```

#### Instrument Sentry
```
@frontend-specialist Add Sentry error monitoring to the storefront.
Sentry config files are already created in root layout setup.

1. Verify apps/storefront/sentry.client.config.ts is correct:
   - DSN from NEXT_PUBLIC_SENTRY_DSN
   - tracesSampleRate: 0.1 (10% in production)
   - replaysOnErrorSampleRate: 1.0

2. Create apps/storefront/sentry.server.config.ts (same DSN, server-side)

3. Create apps/storefront/sentry.edge.config.ts (for Vercel edge runtime)

4. Add to next.config.ts: withSentryConfig wrapper

5. Wrap the global error boundary with Sentry.ErrorBoundary

6. Add custom Sentry context on cart errors and payment failures:
   Sentry.setTag('cart_id', cartId) when a payment error occurs

Verify: trigger a test error → confirm it appears in Sentry dashboard.
Update STATUS.md when done.
```

---

## 3. Phase 2 — Storefront Core

### 3a. Lib, Hooks & API Client

#### Build the Medusa v2 API Client
```
@frontend-specialist Create apps/storefront/lib/medusa/client.ts.
Use @medusajs/js-sdk (MedusaJS v2 — NOT medusa-js v1).
Config: baseUrl: NEXT_PUBLIC_MEDUSA_BACKEND_URL, auth: { type: "session" }

Then create typed wrapper files:
- lib/medusa/products.ts: getProduct(handle), getProducts(params), getCollections(), getBestSellers()
- lib/medusa/cart.ts: createCart(), getCart(id), addItem(), updateItem(), removeItem(), setAddress(), setShippingMethod(), initPaymentSession(), completeCart()
- lib/medusa/customers.ts: login(), register(), getMe(), logout(), requestPasswordReset(), resetPassword()
- lib/medusa/orders.ts: getOrder(id), getOrders(), getOrderByDisplayId(displayId)
- lib/medusa/wishlist.ts: getWishlist(), addToWishlist(productId), removeFromWishlist(itemId)

Ref: NewDocs/MEDUSA-V2-PATTERNS.md for v2 method names.
Ref: NewDocs/06-api-contracts.md for all endpoint signatures.
Update STATUS.md when done.
```

#### Build Algolia Client
```
@frontend-specialist Create apps/storefront/lib/algolia/client.ts.
Set up Algolia InstantSearch client:
- App ID: NEXT_PUBLIC_ALGOLIA_APP_ID
- Search-only API key: NEXT_PUBLIC_ALGOLIA_SEARCH_KEY (read-only — safe for browser)
- Index name: NEXT_PUBLIC_ALGOLIA_INDEX_NAME (default: "products")

Export: algoliasearch client instance + index name constant.
This client is used by SearchOverlay and SearchResults components.
Update STATUS.md when done.
```

#### Build Price and Date Utilities
```
@frontend-specialist Create the utility functions.

1. apps/storefront/lib/utils/price.ts:
   - formatPrice(paise: number): string → 4999900 → "₹49,999"
   - getDiscountPercent(original: number, sale: number): number
   - calculateGST(priceInclGST: number, rate: number): { gstAmount: number, baseAmount: number }
   - formatShipping(paise: number): string → 0 → "Free", 49900 → "₹499"
   
2. apps/storefront/lib/utils/date.ts:
   - formatDate(isoString: string): string → "15 Jan 2026"
   - getDeliveryEstimate(businessDays: number): string → "22–25 Jan 2026" (skips weekends)
   - isWeekend(date: Date): boolean

3. apps/storefront/lib/utils/validators.ts:
   - isPinCode(pin: string): boolean → 6 digits
   - isIndianPhone(phone: string): boolean → 10 digits, starts with 6-9
   - isServiceablePinCode(pin: string): Promise<boolean> → stub returning true for MVP

All monetary inputs/outputs in paise (integers).
Update STATUS.md when done.
```

#### Build Zustand Stores
```
@frontend-specialist Create the two Zustand stores for client-side state.
Ref: NewDocs/08-cart-pricing-engine-spec.md for cart state rules.

1. apps/storefront/store/cart-store.ts
   State:
   - items: LineItem[] (from @shree/types)
   - cartId: string | null
   - subtotal: number (paise)
   - total: number (paise)
   - shippingTotal: number (paise)
   - _snapshot: LineItem[] | null (for rollback)

   Actions:
   - addItemOptimistic(item: LineItem): void
     → add to items or increment quantity if variant already in cart
     → save _snapshot before mutating (for rollback)
   - removeItemOptimistic(lineItemId: string): void
   - updateQuantityOptimistic(lineItemId: string, quantity: number): void
   - rollbackOptimisticAdd(variantId: string): void
     → restore _snapshot if API call failed
   - setCart(cart: Cart): void — sync full cart from server response
   - clearCart(): void — on logout or cart completion

2. apps/storefront/store/ui-store.ts
   State:
   - cartDrawerOpen: boolean
   - searchOverlayOpen: boolean
   - mobileMenuOpen: boolean

   Actions:
   - openCartDrawer() / closeCartDrawer() / toggleCartDrawer()
   - openSearchOverlay() / closeSearchOverlay()
   - openMobileMenu() / closeMobileMenu()

Cart ID persistence: store cartId in a cookie (name: "cart_id", 7-day max-age).
Use zustand/middleware persist for cartId only (not items — always sync from server).

Update STATUS.md when done.
```

#### Build Next.js Middleware for Cart Persistence
```
@frontend-specialist Create apps/storefront/middleware.ts.
Purpose: Read cart_id cookie on every request so the cart is available SSR.

Logic:
- Read cart_id from request cookies
- If no cart_id → do not create one (creation happens on first "Add to Cart")
- Pass cart_id to response headers or request headers for server components to read
- Do NOT run middleware on: /api/*, /_next/*, /favicon.ico, /robots.txt, /sitemap.xml

Matcher config:
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml).*)']

Also create apps/storefront/lib/cart-id.ts:
  getCartId(): string | null — reads from cookies() in server components
  setCartId(id: string): void — sets cookie via cookies().set()
  clearCartId(): void — deletes cookie

Update STATUS.md when done.
```

#### Build TanStack Query Hooks
```
@frontend-specialist Build the TanStack Query hooks for server state.
These are used only in Client Components (with 'use client').

1. apps/storefront/hooks/useCart.ts
   - useCart(): query for current cart (from cart ID in cookie)
   - useAddToCart(): mutation with optimistic update + rollback
   - useRemoveFromCart(): mutation with optimistic update
   - useUpdateCartItem(): mutation for quantity changes

2. apps/storefront/hooks/useProducts.ts
   - useProducts(params): paginated product list query
   - useProduct(handle): single product query

3. apps/storefront/hooks/useCustomer.ts
   - useCustomer(): query for authenticated customer (null if guest)
   - useLogin(): mutation
   - useLogout(): mutation
   - useRegister(): mutation

Ref: NewDocs/MEDUSA-V2-PATTERNS.md for v2 client method names.
Query keys should be stable and typed.
Update STATUS.md when done.
```

---

### 3b. Shared Components

#### Build Header
```
@frontend-specialist Build apps/storefront/components/layout/Header.tsx.
Ref: NewDocs/03-information-architecture.md section 3.1 for exact navigation structure.
Ref: NewDocs/01-product-requirements.md section 4.1.

Desktop layout (left → right):
  [Logo → /]   [Living Room] [Bedroom] [Dining] [Office]   [🔍] [♡] [👤] [🛒 (n)]

Mobile layout:
  [Logo]   [🔍] [🛒 (n)] [☰]

Navigation links: Living Room, Bedroom, Dining, Office → /collections/[slug]

Icons and behaviour:
- Search (🔍): calls openSearchOverlay() from ui-store on click
- Wishlist (♡): links to /account/wishlist
- Account (👤):
    Guest → links to /account/login
    Logged in → links to /account/orders
    Detect auth state via useCustomer() hook (no flash — render guest state server-side)
- Cart (🛒): calls openCartDrawer() from ui-store on click
    Badge: shows item count from cart-store items.length
    Badge hidden when count is 0
    data-testid="cart-icon"
- Hamburger (☰): calls openMobileMenu() from ui-store (mobile only, <md breakpoint)

Sticky: header is sticky (position: sticky, top: 0) with subtle shadow on scroll
Background: white (#FFFFFF) with bottom border (#E8E0D0)
Height: 64px desktop, 56px mobile

Named export only.
Update STATUS.md when done.
```

#### Build ProductCard Component
```
@frontend-specialist Build apps/storefront/components/product/ProductCard.tsx.
This component is used in: Homepage BestSellers, PLP grid, Wishlist page, Search results.
Ref: NewDocs/01-product-requirements.md section 4.2.

Props interface:
  interface ProductCardProps {
    product: Product           // from @shree/types
    showWishlist?: boolean     // default: true
    priority?: boolean         // true for above-fold cards (LCP)
  }

Layout (mobile-first):
- Thumbnail: next/image, width=400 height=400 (explicit — prevents CLS), object-cover
  Use priority prop for above-fold cards
- Discount badge (top-left): if sale price < original — shows "−17%" in Brand Accent colour
- "New" badge (top-right): if created within 30 days
- Product name: Playfair Display, line-clamp-2
- PriceDisplay component (sale price + struck-through original)
- "Only [n] left" if inventory_quantity <= 3 and > 0
- Wishlist toggle (♡ icon, top-right): calls addToWishlist/removeFromWishlist
  If guest: redirects to /account/login?redirect=[current path]
- Entire card is a link to /products/[handle]
- Hover: subtle lift (translateY(-2px)) + shadow transition

data-testid="product-card" on root element.
Named export only. No default export.
Update STATUS.md when done.
```

#### Build PriceDisplay Component
```
@frontend-specialist Build apps/storefront/components/shared/PriceDisplay.tsx.
Props: { amount: number, originalAmount?: number, size?: 'sm' | 'md' | 'lg' }
- Renders formatted INR price using formatPrice() from lib/utils/price.ts
- If originalAmount provided and > amount: show strikethrough original + discount badge
- Discount badge uses Brand Accent colour (#C8A96E)
- Size variants affect font size only
- Small GST note below price: "Incl. 12% GST"
Named export. data-testid="price-display".
Update STATUS.md when done.
```

#### Build Skeleton, ErrorBoundary, EmptyState
```
@frontend-specialist Build the three shared utility components.

1. apps/storefront/components/shared/SkeletonCard.tsx
   - Animated skeleton matching ProductCard dimensions (thumbnail + 2 text lines)
   - Uses Tailwind animate-pulse
   - Used in PLP loading.tsx

2. apps/storefront/components/shared/ErrorBoundary.tsx
   - React class component (ErrorBoundary must be a class component)
   - Fallback UI: "Something went wrong" with a retry button
   - Reports error to Sentry on componentDidCatch

3. apps/storefront/components/shared/EmptyState.tsx
   - Props: { title: string, description: string, ctaLabel?: string, ctaHref?: string }
   - Used for: empty cart, no search results, empty wishlist, empty orders
   - Warm background, centered layout

All named exports.
Update STATUS.md when done.
```

#### Build Footer
```
@frontend-specialist Build apps/storefront/components/layout/Footer.tsx.
Ref: NewDocs/03-information-architecture.md section 3.3 for exact navigation structure.
Sections:
- Collections column: Living Room, Bedroom, Dining, Office (→ /collections/[slug])
- Customer Service column: Track Your Order, Returns & Refunds, FAQs, Contact Us
- Company column: About Us, Privacy Policy, Terms of Service
- Payment icons strip: UPI, Visa, Mastercard, EMI (use SVG icons or image files from public/)
- Copyright: "© 2026 Shree Furniture. All rights reserved."
Mobile: stack columns vertically with accordion expand/collapse.
Design: warm background (#F5F0E8), subtle top border (#E8E0D0).
Update STATUS.md when done.
```

#### Build MobileMenu
```
@frontend-specialist Build apps/storefront/components/layout/MobileMenu.tsx.
Triggered by hamburger icon in Header on mobile (<md breakpoint).
Full-screen overlay with slide-in animation.
Content (ref NewDocs/03-information-architecture.md section 3.2):
- Primary links: Living Room, Bedroom, Dining, Office
- Divider
- Secondary links: My Account, My Wishlist, My Orders
- Close button (X) top right
- Outside click or escape key closes menu
Uses ui-store: mobileMenuOpen, setMobileMenuOpen.
Update STATUS.md when done.
```

#### Build Breadcrumb Component
```
@frontend-specialist Build apps/storefront/components/layout/Breadcrumb.tsx.
Props: { items: { label: string, href?: string }[] }
- Renders breadcrumb trail: Home > Living Room > Oslo Sofa
- Last item is not a link (current page)
- Includes JSON-LD BreadcrumbList structured data as <script type="application/ld+json">
- Accessible: nav aria-label="Breadcrumb" + ol list structure
Used on PLP and PDP pages.
Ref: NewDocs/03-information-architecture.md section 4 for URL structure.
Update STATUS.md when done.
```

---

### 3c. Homepage

#### Build the Homepage
```
@frontend-specialist Build apps/storefront/app/(store)/page.tsx — the Homepage.
ISR: export const revalidate = 60
Ref: NewDocs/01-product-requirements.md section 4.1 for all requirements.

Fetch in parallel (Promise.all):
- getCollections() — for Featured Collections grid
- getProducts({ limit: 8, order: "-created_at" }) — for Best Sellers + New Arrivals

Sections to build (in order, top to bottom):

1. HeroSection (create components/home/HeroSection.tsx):
   - Full-width banner with background image (from Cloudinary)
   - Headline: "Beautiful Furniture for Every Home"
   - Subheadline: "Free delivery on orders above ₹5,000"
   - Two CTAs: "Shop Now" (→ /collections/living-room) + "Explore Collections" (→ /collections)
   - Mobile: stacked layout, image behind text

2. FeaturedCollections (create components/home/FeaturedCollections.tsx):
   - 4-card grid: Living Room, Bedroom, Dining, Office
   - Each card: full-bleed image, collection name, "Shop Now" link
   - Grid: 2 cols mobile / 4 cols desktop

3. BestSellers (create components/home/BestSellers.tsx):
   - Horizontally scrollable product carousel on mobile (scroll-snap)
   - 4-col grid on desktop
   - Uses ProductCard component

4. TrustSignals (create components/home/TrustSignals.tsx):
   - 4 icons + text: "Free Delivery above ₹5,000" | "Easy Returns" | "EMI Available" | "Genuine Warranty"
   - Subtle background strip (#E8E0D0)

generateMetadata(): title "Shree Furniture — Beautiful Furniture for Every Home", OG image.
Update STATUS.md when done.
```

---

### 3d. Product Pages

#### Build PLP (Product Listing Page)
```
@frontend-specialist Build apps/storefront/app/(store)/collections/[handle]/page.tsx.
ISR: export const revalidate = 60
Ref: NewDocs/01-product-requirements.md section 4.2.

- Fetch collection + products via getProducts({ collection_handle: handle })
- Product grid: 2 cols mobile / 3 tablet / 4 desktop using ProductCard
- Filter panel: sidebar on desktop, bottom sheet on mobile
  Filters: material, colour, price range (₹0–₹10k, ₹10k–₹30k, ₹30k–₹60k, ₹60k+)
- Sort: Price Low→High, High→Low, Newest, Most Popular
  All filter + sort state in URL query params (?material=wood&sort=price_asc)
- "Load More" pagination button (not infinite scroll — SEO)
- Breadcrumb: Home > [Collection Name]
- Empty state with EmptyState component
- loading.tsx: grid of 8 SkeletonCard components
- not-found.tsx for invalid collection handles

generateMetadata(): "[Collection Name] — Shree Furniture"
Update STATUS.md when done.
```

#### Build PDP (Product Detail Page)
```
@frontend-specialist Build apps/storefront/app/(store)/products/[handle]/page.tsx.
ISR: export const revalidate = 60
Ref: NewDocs/01-product-requirements.md section 4.3.

Fetch in parallel:
- getProduct(handle)
- getProducts({ collection_id, limit: 4 }) — for related products

Build these sub-components (create in components/product/):
- ProductGallery.tsx: primary image (priority) + thumbnail strip, pinch-zoom on mobile
- ProductVariants.tsx: colour swatches + size buttons, disabled state for out-of-stock
- ProductBadge.tsx: "New", "Sale", "Only 3 left" badges
- ProductSpecsTable.tsx: L×W×H, weight, material, finish from metadata
- DeliveryEstimate.tsx: PIN code input → show estimated delivery date

Page layout:
- Left: ProductGallery (sticky on desktop)
- Right: name, SKU, PriceDisplay (with original + discount%), GST note,
         ProductVariants, quantity selector (capped to inventory_quantity),
         "Add to Cart" button (sticky on mobile, data-testid="add-to-cart"),
         "Add to Wishlist" button,
         DeliveryEstimate,
         ProductSpecsTable, rich text description
- Below fold: Related Products carousel

SEO:
- generateMetadata(): unique title, description, OG image from thumbnail
- JSON-LD Product schema (name, image, price, availability, brand)
- Breadcrumb: Home > [Collection] > [Product Name]
- Out-of-stock variants: visually disabled (opacity-50, cursor-not-allowed) — NOT hidden

not-found.tsx for invalid product handles.
Update STATUS.md when done.
```

---

### 3e. Search

#### Build Search Feature
```
@frontend-specialist Build the complete search feature.
Ref: NewDocs/01-product-requirements.md section 4.7.

Files to create:
1. components/search/SearchBar.tsx
   - Input in Header, opens SearchOverlay on focus
   - Uses ui-store: searchOverlayOpen, setSearchOverlayOpen
   - data-testid="search-bar"

2. components/search/SearchOverlay.tsx
   - Full-screen overlay on mobile, floating panel on desktop
   - Contains SearchBar + live SearchResults
   - Escape key or outside click closes
   - Uses Algolia InstantSearch from lib/algolia/client.ts

3. components/search/SearchResults.tsx
   - Algolia Hit component: product thumbnail, name, price, collection badge
   - Typo-tolerance is configured in Algolia (no code needed)
   - "No results" state with EmptyState component + 4 collection suggestion links
   - Track query to PostHog: analytics.searchQuery(query)

4. apps/storefront/app/(store)/search/page.tsx (CSR — no revalidate)
   - Full search results page with Algolia InstantSearch
   - URL: /search?q=[query]
   - Faceted filters in sidebar

Update STATUS.md when done.
```

#### Build Cart Page
```
@frontend-specialist Build apps/storefront/app/(store)/cart/page.tsx.
This is the full /cart page — separate from the CartDrawer slide-in.
Rendering: CSR ('use client' or no revalidate)
Content:
- Full cart item list (same CartItem component as CartDrawer)
- CartSummary with subtotal, shipping, GST, total
- "Proceed to Checkout" CTA → /checkout
- Empty cart state: EmptyState component + "Continue Shopping" → /
- Page title: "Your Cart" + item count
- Breadcrumb: Home > Cart
This page is the fallback for users who navigate directly to /cart.
Update STATUS.md when done.
```

---

## 4. Phase 3 — Transactions

### 4a. Cart

#### Build CartDrawer + CartItem + CartSummary
```
@frontend-specialist Build the complete cart drawer UI.
Ref: NewDocs/01-product-requirements.md section 4.4.
Ref: NewDocs/08-cart-pricing-engine-spec.md for pricing rules.

1. components/cart/CartItem.tsx
   - Props: item: LineItem (from @shree/types)
   - Thumbnail (next/image, width=80 height=80), product name, variant title
   - Quantity stepper: − / [qty] / + (cannot go below 1 or above inventory_quantity)
   - Unit price via PriceDisplay, line total
   - Remove button (trash icon), data-testid="remove-item"
   - Optimistic update: call useUpdateCartItem / useRemoveFromCart hooks

2. components/cart/CartSummary.tsx
   - Subtotal in paise via formatPrice()
   - Shipping: "Free" if subtotal >= 500000 (₹5,000), else "₹499"
   - Free shipping progress bar: "Add ₹X more for free shipping"
   - GST note: "Prices include applicable GST"
   - Total (subtotal + shipping)
   - "Proceed to Checkout" button, data-testid="proceed-to-checkout"

3. components/cart/CartDrawer.tsx
   - Triggered by ui-store cartDrawerOpen
   - Slide-in from right (desktop), bottom sheet (mobile) — shadcn/ui Sheet component
   - Header: "Your Cart" + item count + close button
   - Scrollable CartItem list
   - Sticky CartSummary at bottom
   - Empty state: EmptyState component + "Continue Shopping" closes drawer
   - Cart persists across refresh: cart ID stored in cookie (name: "cart_id", 7-day expiry)
   - data-testid="cart-drawer"

Update STATUS.md when done.
```

---

### 4b. Checkout

#### Build CheckoutProgress Component
```
@frontend-specialist Build apps/storefront/components/checkout/CheckoutProgress.tsx.
Used in all 3 checkout steps to show where the user is in the flow.
Ref: NewDocs/03-information-architecture.md section 7 (checkout state machine).

Props:
  interface CheckoutProgressProps {
    currentStep: 'address' | 'shipping' | 'payment'
  }

Steps to show: Address → Shipping → Payment
Visual states:
  - Completed step (before current): checkmark icon + coloured line, clickable (navigate back)
  - Current step: highlighted label + step number, not clickable
  - Future step: greyed out, not clickable

Mobile: compact horizontal strip with step numbers only (1 → 2 → 3)
Desktop: full labels + connecting lines

Colours: completed = Brand Primary (#1A1A1A), current = Brand Accent (#C8A96E), future = muted grey
Named export. data-testid="checkout-progress"
Update STATUS.md when done.
```

#### Build Checkout Step 1 — Address
```
@frontend-specialist Build apps/storefront/app/(checkout)/checkout/page.tsx and AddressForm.
Ref: NewDocs/01-product-requirements.md section 4.5 Step 1.
Ref: NewDocs/06-api-contracts.md section 5 for customer address endpoint.

Zod schema (export addressSchema for testing):
- firstName, lastName: min 2 chars
- email: valid email format
- phone: 10-digit Indian mobile (starts with 6-9)
- address1: required
- address2: optional
- city: required
- state: required (dropdown — all Indian states)
- postalCode: exactly 6 digits
- countryCode: "in" (hidden field)

Behaviour:
- React Hook Form with Zod resolver
- Logged-in users: show saved addresses as selectable cards (pre-fill form)
- On valid submit: call setAddress() + setEmail() in lib/medusa/cart.ts
- On success: navigate to /checkout/shipping
- Guest checkout: no forced login (email field required)
- CheckoutProgress shows: Address (active) → Shipping → Payment

data-testid on all interactive fields (matching E2E test selectors).
Update STATUS.md when done.
```

#### Build Checkout Step 2 — Shipping
```
@frontend-specialist Build apps/storefront/app/(checkout)/checkout/shipping/page.tsx.
Ref: NewDocs/01-product-requirements.md section 4.5 Step 2.
Ref: NewDocs/08-cart-pricing-engine-spec.md section 5 for shipping rules.

Fetch available shipping options for the cart region.
Display options:
- "Standard Shipping" — ₹499 — "7–10 business days" (data-testid="shipping-option-standard")
- "Free Shipping" — ₹0 — "7–10 business days" (only shown if cart total >= ₹5,000)
If eligible for free shipping, show: "🎉 Your order qualifies for free shipping!"
If not eligible: "Add ₹X more to get free shipping"

On selection: call setShippingMethod() in lib/medusa/cart.ts
"Continue to Payment" → /checkout/payment (data-testid="continue-to-payment")
"← Back to Address" link

CheckoutProgress: Address (complete) → Shipping (active) → Payment
Update STATUS.md when done.
```

#### Build Checkout Step 3 — Payment
```
@frontend-specialist Build apps/storefront/app/(checkout)/checkout/payment/page.tsx.
Ref: NewDocs/01-product-requirements.md section 4.5 Step 3.

1. On page load: call initPaymentSession() → gets razorpay_order_id from Medusa
2. Load Razorpay JS SDK dynamically (do NOT bundle — load from https://checkout.razorpay.com/v1/checkout.js)
3. Order summary:
   - Desktop: sticky sidebar
   - Mobile: collapsed accordion showing total
4. "Place Order" button (data-testid="place-order"):
   - Opens Razorpay modal:
     key: NEXT_PUBLIC_RAZORPAY_KEY_ID
     order_id: razorpay_order_id from payment session
     amount: cart.total (in paise)
     currency: "INR"
     name: "Shree Furniture"
     prefill: { email, contact: phone } from cart
     theme: { color: "#C8A96E" } (brand accent)
5. On Razorpay success:
   - Call completeCart() in lib/medusa/cart.ts
   - If response.type === "order": navigate to /order/confirm/[id]
   - Clear cart_id cookie
6. On Razorpay failure:
   - Show inline error message (do NOT redirect)
   - "Try Again" button re-opens Razorpay modal
   - Do NOT expose Razorpay error codes to user

CheckoutProgress: Address (complete) → Shipping (complete) → Payment (active)
Update STATUS.md when done.
```

#### Build Razorpay Webhook Handler
```
@backend-specialist @security-auditor Build apps/storefront/app/api/webhooks/razorpay/route.ts.
Ref: NewDocs/06-api-contracts.md section 7 for exact payload structure.
Ref: NewDocs/KNOWN-ISSUES.md "Duplicate Order Creation" — read this first.

Logic (in this exact order):
1. Read raw body as text (do NOT parse JSON before verification)
2. Extract X-Razorpay-Signature header
3. Compute HMAC-SHA256(RAZORPAY_WEBHOOK_SECRET, rawBody) using crypto.createHmac
4. Use timingSafeEqual for comparison (prevents timing attacks)
5. If mismatch → return 400 immediately, log warning
6. Return 200 OK NOW (before any async processing — prevents Razorpay retry loop)
7. Parse body as JSON
8. Check idempotency: has payment ID already been processed? If yes → return silently
9. Store payment ID as "processing" in Redis (TTL 10 minutes)
10. On payment.captured: call Medusa to capture payment for the order
11. On payment.failed: log for admin visibility
12. Mark payment ID as "processed" in Redis

Ref: NewDocs/08-cart-pricing-engine-spec.md section 2 for cart lifecycle.
Update STATUS.md when done.
```

#### Build Order Confirmation Page
```
@frontend-specialist Build apps/storefront/app/(checkout)/order/confirm/[id]/page.tsx.
Rendering: SSR (export const dynamic = 'force-dynamic') — always fresh, never cached.
Ref: NewDocs/01-product-requirements.md section 4.5 Post-Payment.

Fetch: getOrderByDisplayId(id) from lib/medusa/orders.ts
If order not found → redirect to /

Display (data-testid="order-confirmation"):
- ✅ "Order Confirmed!" hero message
- Order ID: #[display_id] (data-testid="order-confirmation-id")
- Items list: thumbnail, name, variant, quantity, price
- Shipping address
- Payment method: "Paid via Razorpay"
- Order total (breakdown: subtotal, shipping, GST)
- Estimated delivery: getDeliveryEstimate(10) from lib/utils/date.ts
- Email note: "A confirmation email has been sent to [email]"
- "Continue Shopping" CTA → /

Track: analytics.orderComplete({ orderId: display_id, total })
Update STATUS.md when done.
```

---

### 4c. Auth & Account

#### Build Auth Pages (Login / Register / Forgot Password)
```
@frontend-specialist Build the three authentication pages.
Ref: NewDocs/01-product-requirements.md section 4.6.
Ref: NewDocs/06-api-contracts.md section 5 for auth endpoint signatures.

1. apps/storefront/app/(account)/account/login/page.tsx
   - React Hook Form + Zod: email (valid format), password (min 8 chars)
   - On submit: call login() from lib/medusa/customers.ts
   - On success: redirect to /account/orders (or redirect param if present)
   - Show error: "Invalid email or password" (do NOT specify which is wrong)
   - Link to /account/register and /account/forgot-password
   - data-testid="login-submit"

2. apps/storefront/app/(account)/account/register/page.tsx
   - Zod schema: firstName (min 2), lastName (min 2), email, password (min 8), confirmPassword (must match)
   - On submit: call register() from lib/medusa/customers.ts
   - On success: auto-login then redirect to /account/orders
   - data-testid="register-submit"

3. apps/storefront/app/(account)/account/forgot-password/page.tsx
   - Step 1: email form → call requestPasswordReset() → show "Check your email" message
   - Step 2: token + new password form (from email link ?token=xxx) → call resetPassword()
   - On success: redirect to /account/login with success message

All pages: no Header/Footer (use account route group layout).
These three pages are OUTSIDE the auth guard (accessible without login).
Update STATUS.md when done.
```

#### Build Account Layout (Sidebar + Auth Guard)
```
@frontend-specialist Build apps/storefront/app/(account)/layout.tsx — the account layout.
This layout wraps all /account/* pages EXCEPT login/register/forgot-password.

Auth guard:
- Check session via getMe() from lib/medusa/customers.ts (server-side in layout)
- If not authenticated → redirect to /account/login?redirect=[current path]
- If authenticated → render layout with customer data

Sidebar navigation (desktop only — mobile uses bottom nav or dropdown):
- My Orders → /account/orders
- Wishlist → /account/wishlist
- Addresses → /account/addresses
- Account Settings → /account/settings
- Logout button (calls logout() then redirects to /)

Active state: highlight current page link.
Mobile: sidebar collapses to a horizontal tab strip at top.
Update STATUS.md when done.
```

#### Build My Orders Pages
```
@frontend-specialist Build the My Orders list and detail pages.
Ref: NewDocs/01-product-requirements.md section 4.6.
Ref: NewDocs/06-api-contracts.md section 4 for orders endpoint.

1. apps/storefront/app/(account)/account/orders/page.tsx
   Rendering: SSR (dynamic) — always fresh
   - Fetch getOrders() for authenticated customer
   - Order list: each row shows order ID (#display_id), date (formatDate), total (formatPrice),
     payment status badge, fulfillment status badge
   - Clickable rows → /account/orders/[id]
   - Empty state: EmptyState component with "Start Shopping" CTA
   - Pagination: load more (offset-based)

2. apps/storefront/app/(account)/account/orders/[id]/page.tsx
   Rendering: SSR (dynamic)
   - Fetch getOrder(id) for the authenticated customer
   - Verify order belongs to current customer (redirect if not)
   - Show: order ID, date, status, items list (thumbnail, name, variant, qty, price),
     shipping address, payment method, total breakdown
   - If fulfillment_status === "shipped": show tracking link (if tracking_number exists)
   - "Need Help?" link → mailto:support@shree-furniture.in

Update STATUS.md when done.
```

#### Build Wishlist Page
```
@frontend-specialist Build apps/storefront/app/(account)/account/wishlist/page.tsx.
Rendering: SSR (dynamic)
Ref: NewDocs/01-product-requirements.md section 4.6 (US-007).
Ref: NewDocs/06-api-contracts.md section 6 for wishlist endpoint.

- Fetch getWishlist() from lib/medusa/wishlist.ts
- Display saved products as a ProductCard grid (2 cols mobile / 3 cols desktop)
- Each card: ProductCard + "Add to Cart" button + "Remove from Wishlist" ✕ button
- "Add to Cart" → adds default variant to cart via useAddToCart hook
- "Remove" → calls removeFromWishlist(itemId), optimistic update
- Empty state: EmptyState component with "Browse Products" CTA

Also update ProductCard and PDP to handle wishlist toggle:
- Wishlist heart icon on ProductCard and PDP should call addToWishlist/removeFromWishlist
- If guest: heart icon click → redirect to /account/login?redirect=[current path]
Update STATUS.md when done.
```

#### Build Addresses and Settings Pages
```
@frontend-specialist Build the remaining account pages.

1. apps/storefront/app/(account)/account/addresses/page.tsx
   Rendering: SSR (dynamic)
   - Fetch customer addresses from getMe()
   - Show address cards: name, address, city, PIN, phone
   - "Add New Address" → opens AddressForm in a Sheet/modal (reuse from checkout)
   - Edit and Delete actions on each card
   - On add/edit/delete: call appropriate Medusa customer address endpoints
   Ref: NewDocs/06-api-contracts.md section 5 for address endpoints.

2. apps/storefront/app/(account)/account/settings/page.tsx
   Rendering: CSR
   - Update name: firstName + lastName form
   - Update email form (with current password confirmation)
   - Change password form: current password + new password + confirm
   - On submit: call appropriate Medusa customer update endpoints
   - Success/error toast notifications

Update STATUS.md when done.
```

---

## 5. Phase 3 — Backend Subscribers & Workflows

#### Build Order Confirmation Email Subscriber
```
@backend-specialist Build backend/src/subscribers/order-placed.ts.
MedusaJS v2 subscriber pattern (Ref: NewDocs/MEDUSA-V2-PATTERNS.md).
Event: "order.placed"

Steps:
1. Retrieve order with relations: items, shipping_address, customer
2. Format item prices using formatPrice from packages/types (import the shared util)
3. Send via Resend to customer email
4. Create React Email template: backend/src/emails/OrderConfirmation.tsx
   - Shree Furniture header with logo
   - "Your order is confirmed! 🎉"
   - Order #[display_id] box
   - Items table: thumbnail, name, variant, qty, price
   - Shipping address block
   - Order total (subtotal, shipping, GST, total)
   - Estimated delivery: 7-10 business days
   - "View Order" button → [STOREFRONT_URL]/account/orders/[id]

Subject: "Order Confirmed — #[display_id] | Shree Furniture"
Update STATUS.md when done.
```

#### Build Order Shipped Email Subscriber
```
@backend-specialist Build backend/src/subscribers/order-shipped.ts.
Event: "order.shipment_created"
Create React Email template: backend/src/emails/OrderShipped.tsx
Content: "Your order is on its way 🚚", order ID, items, tracking link (if available), estimated delivery.
Subject: "Your Order Has Shipped — #[display_id] | Shree Furniture"
Update STATUS.md when done.
```

#### Build Algolia Sync Subscriber
```
@backend-specialist Build backend/src/subscribers/algolia-sync.ts.
Events: "product.created", "product.updated", "product.status_changed"

On created/updated (status=published, deleted_at=null):
- Upsert to Algolia index with objectID = product.id
- Index these fields: id, title, handle, thumbnail, variants[0].prices[0].amount,
  collection.handle, collection.title, room_type, status, tags

On status_changed to 'draft' or if deleted_at set:
- Remove from Algolia index: index.deleteObject(product.id)

After every sync:
- Call storefront revalidation: POST [STOREFRONT_URL]/api/revalidate
  Body: { secret: REVALIDATION_SECRET, path: /products/[handle] }
  Also revalidate the collection: { path: /collections/[collection.handle] }

Ref: NewDocs/KNOWN-ISSUES.md "Algolia Search Products Not Appearing" before writing event names.
Update STATUS.md when done.
```

#### Build Low Stock Alert Subscriber
```
@backend-specialist Build backend/src/subscribers/low-stock.ts.
Event: check MedusaJS v2 docs for the exact inventory low stock event name.
Document the actual event name in NewDocs/KNOWN-ISSUES.md if different from expected.

Action: Send alert email via Resend to ADMIN_EMAIL env var.
Create template: backend/src/emails/LowStockAlert.tsx
Content: Product name, variant title, SKU, current quantity, "Reorder needed"
Subject: "⚠️ Low Stock Alert: [product title] — [variant title]"
Update STATUS.md when done.
```

#### Build Fulfillment Workflow
```
@backend-specialist Build backend/src/workflows/fulfillment-workflow.ts.
MedusaJS v2 Workflow pattern (Ref: NewDocs/MEDUSA-V2-PATTERNS.md).

Steps in order:
1. validateOrderStep({ orderId }):
   - Check order exists and payment_status === "captured"
   - StepResponse: { valid: true } or throw error with rollback
2. createFulfillmentStep({ orderId }):
   - Create fulfillment record via Medusa fulfillment service
   - StepResponse: { fulfillmentId }
   - Rollback: delete the fulfillment if next step fails
3. notifyCustomerStep({ orderId }):
   - Emit order.shipment_created event to trigger order-shipped subscriber

Register in medusa-config.ts.
Update STATUS.md when done.
```

#### Build Wishlist Module
```
@backend-specialist @database-architect Build the Wishlist custom Medusa module.
Ref: NewDocs/MEDUSA-V2-PATTERNS.md for module structure.

Files:
- backend/src/modules/wishlist/models/wishlist-item.ts
  Fields: id (primary key), customer_id (text), product_id (text), created_at
- backend/src/modules/wishlist/service.ts
  Methods: listByCustomer(customerId), addItem({ customerId, productId }), removeItem(id)
  Check: prevent duplicate (same customer + product)
- backend/src/modules/wishlist/index.ts (Module export, name: "wishlist")
- backend/src/api/store/wishlist/route.ts
  GET: returns wishlist items with expanded product data
  POST: adds item (requires customer session)
- backend/src/api/store/wishlist/[id]/route.ts
  DELETE: removes item (requires customer session, must own the item)

Register module in backend/medusa-config.ts.
Ref: NewDocs/06-api-contracts.md section 6 for exact response shapes.
Update STATUS.md when done.
```

#### Build ISR Revalidation Endpoint
```
@frontend-specialist Build apps/storefront/app/api/revalidate/route.ts.
POST handler for on-demand ISR cache invalidation.
1. Read secret from body or Authorization header
2. Compare with REVALIDATION_SECRET env var (not NEXT_PUBLIC_)
3. If mismatch → return 401
4. Accept body: { path: string } or { paths: string[] }
5. Call revalidatePath(path) from "next/cache" for each path
6. Return: { revalidated: true, paths: [...] }

Also create a revalidateAll helper: revalidate /, all collection paths, all affected product paths.
This endpoint is called by algolia-sync subscriber.
Update STATUS.md when done.
```

---

## 6. Phase 4 — Launch Prep

### 6a. SEO & Indexing

#### Add robots.txt and sitemap.xml
```
@frontend-specialist Add SEO crawling configuration.
Ref: NewDocs/02-user-stories-and-acceptance-criteria.md US-016.

1. Create apps/storefront/app/robots.txt (or use Next.js robots.ts):
   Allow: / (homepage, PLP, PDP)
   Disallow: /checkout, /cart, /account, /api
   Sitemap: https://shree-furniture.in/sitemap.xml

2. Create apps/storefront/app/sitemap.ts (Next.js dynamic sitemap):
   - Fetch all published products → generate /products/[handle] URLs
   - Fetch all collections → generate /collections/[handle] URLs
   - Static pages: /, plus any informational pages
   - Priority: homepage (1.0), collections (0.8), products (0.7)
   - changefreq: weekly for products, monthly for collections

3. Add noindex meta to protected pages:
   In (checkout)/layout.tsx: <meta name="robots" content="noindex, nofollow" />
   In (account)/layout.tsx: <meta name="robots" content="noindex, nofollow" />
   In cart page: export const metadata = { robots: 'noindex' }

Verify: /sitemap.xml returns valid XML, /robots.txt accessible.
Update STATUS.md when done.
```

#### Verify All SEO Meta Tags
```
@seo-specialist Audit SEO meta tags across all indexable pages.
Check these pages:
- / (Homepage)
- /collections/living-room (PLP)
- /products/[any handle] (PDP)

For each page verify:
1. Unique <title> (not duplicate across pages)
2. <meta name="description"> (150-160 chars, includes keywords)
3. Open Graph: og:title, og:description, og:image (Cloudinary URL), og:url
4. Canonical URL: <link rel="canonical" href="..."> (no trailing slashes, no query params)
5. JSON-LD Product schema on PDP: validate at https://search.google.com/test/rich-results
6. JSON-LD BreadcrumbList on PLP and PDP
7. hreflang: not needed (India-only, single language)
8. No noindex on indexable pages

Fix any issues found. Document in KNOWN-ISSUES.md if any structural changes needed.
Update STATUS.md when done.
```

---

### 6b. Analytics & Monitoring

#### Verify PostHog Events End-to-End
```
@frontend-specialist Verify all PostHog events are firing correctly.
Run through the full purchase flow manually and verify these events appear in PostHog Live Events:
- product_viewed: fires on PDP load ✓
- add_to_cart: fires on "Add to Cart" click ✓
- search_query: fires on search input ✓
- checkout_started: fires on "Proceed to Checkout" click ✓
- order_complete: fires on order confirmation page ✓

Also verify PostHog session recordings are working (if enabled in project settings).
Check PostHog dashboard: Funnels → create funnel: product_viewed → add_to_cart → checkout_started → order_complete.

If any events are missing, trace back to the analytics.ts wrapper and fix.
Update STATUS.md when done.
```

#### Verify Sentry is Receiving Errors
```
@frontend-specialist Verify Sentry error monitoring is working in production.
1. Trigger a test error in the storefront (temporary console.error or throw)
2. Verify it appears in Sentry Issues dashboard
3. Verify source maps are uploaded (stack traces should show original TypeScript)
4. Set up these Sentry alerts in dashboard:
   - Payment failure rate > 5% in 1 hour → alert
   - Error rate spike (> 10 new errors in 10 mins) → alert
5. Remove the test error
Confirm: NEXT_PUBLIC_SENTRY_DSN is set in Vercel production env vars.
```

---

### 6c. Security Hardening

#### Run npm Audit and Fix Vulnerabilities
```
@security-auditor Run a dependency vulnerability audit.
Execute:
  cd apps/storefront && pnpm audit --audit-level=high
  cd backend && pnpm audit --audit-level=high

For each high/critical vulnerability:
1. Check if a patch version is available → update in package.json
2. If no patch available → document in KNOWN-ISSUES.md with mitigation
3. Run pnpm audit again to confirm resolved

Also run:
  pnpm outdated (in each workspace)

Create a report of findings.
Do NOT auto-update all packages — update only security patches.
```

#### Scan Git History for Accidentally Committed Secrets
```
@security-auditor Scan the git repository history for any accidentally committed secrets.
Run:
  npx git-secrets --scan-history (or use truffleHog / gitleaks)
  git log --all --full-history -- "*.env*"
  git log --all --full-history -- ".env"

Check for patterns: API keys, passwords, connection strings, JWT secrets.
If any found: document the exact commit, rotate the exposed secret immediately, 
then consider using BFG Repo Cleaner or git filter-branch to remove from history.
Update KNOWN-ISSUES.md with findings.
```

#### Verify Rate Limiting on Auth Endpoints
```
@security-auditor Verify rate limiting is active on sensitive endpoints.
Ref: NewDocs/06-api-contracts.md section 9 for required rate limits.

Test these limits:
1. POST /store/customers/auth: make 6 rapid requests → 6th should return 429
2. POST /store/customers: make 11 requests in 60 mins → should return 429

If MedusaJS v2 does not have built-in rate limiting:
- Add express-rate-limit or @nestjs/throttler to the backend
- Configure: 5 req/15min for /auth, 10 req/60min for /customers

Also verify the Razorpay webhook is IP-restricted to Razorpay's IP ranges.
Document rate limiting config in KNOWN-ISSUES.md.
```

#### Verify HTTP-only Cookie in Production
```
@security-auditor Verify auth token is not accessible via JavaScript in production.
In browser console on shree-furniture.in after logging in:
  document.cookie → should NOT contain any JWT token
  localStorage → should be empty (no tokens)
  sessionStorage → should be empty (no tokens)

Also verify cookie flags via DevTools → Application → Cookies:
  HttpOnly: ✓ (must be checked)
  Secure: ✓ (must be checked on HTTPS)
  SameSite: Strict or Lax

If any of these fail, trace back to the Medusa session configuration.
Ref: NewDocs/DECISIONS.md ADR-009.
```

---

### 6d. Performance Audit

#### Run Lighthouse and Fix Failures
```
@performance-optimizer Run Lighthouse on all critical pages and fix any failures.
Target metrics (from NewDocs/01-product-requirements.md section 5):
  LCP < 2.5s | INP < 200ms | CLS < 0.1 | Lighthouse Score ≥ 90

Run on:
1. / (Homepage) — 4G mobile throttling
2. /collections/living-room (PLP)
3. /products/[handle] (PDP)

For each failure:
- LCP > 2.5s: check priority loading on hero/product images, Cloudinary CDN, ISR cache
- CLS > 0.1: find elements without explicit dimensions (Image width/height, font swap)
- Large bundle: run @next/bundle-analyzer, identify and code-split large imports

Run bundle analyzer:
  cd apps/storefront && ANALYZE=true pnpm build
  (requires @next/bundle-analyzer in next.config.ts)

Target: initial JS bundle < 200KB (gzipped).
Document all fixes in KNOWN-ISSUES.md.
Update STATUS.md when done.
```

#### Run Bundle Analyzer
```
@performance-optimizer Run the Next.js bundle analyzer and identify oversized chunks.
Install @next/bundle-analyzer if not already:
  pnpm add -D @next/bundle-analyzer

Add to apps/storefront/next.config.ts:
  const withBundleAnalyzer = require('@next/bundle-analyzer')({ enabled: process.env.ANALYZE === 'true' })
  module.exports = withBundleAnalyzer({ /* existing config */ })

Run: ANALYZE=true pnpm build

Identify:
- Any chunk > 50KB that can be code-split with dynamic import
- Any library included in the main bundle that should be lazy-loaded
- Duplicate dependencies

Fix the top 3 largest wins.
Update STATUS.md when done.
```

---

### 6e. Go-Live

#### Configure Cloudflare DNS + Custom Domain
```
@devops-engineer Configure the production domain and CDN.
Domain: shree-furniture.in (or configured domain)

Cloudflare DNS records:
1. CNAME: @ → cname.vercel-dns.com (or Vercel A records)
2. CNAME: api → Railway backend URL
3. MX records for email (if hosting email via Cloudflare)

Cloudflare settings:
- SSL/TLS: Full (Strict) — requires valid SSL on origin
- Always Use HTTPS: ON
- Minimum TLS: 1.2
- Caching: Cache Level = Standard
- Page Rules: Cache everything under /products/* and /collections/* (respect Cache-Control)
- WAF: enable managed rules (free tier sufficient for MVP)

Verify:
- https://shree-furniture.in loads the storefront
- https://shree-furniture.in/api/webhooks/razorpay is reachable
- SSL certificate is valid and shows green padlock

Update STATUS.md when done.
```

#### Configure Vercel Production Environment
```
@devops-engineer Configure all Vercel production environment variables.
In Vercel project settings → Environment Variables, add ALL from .env.example.

Critical to verify (check against NewDocs/04-system-architecture.md section 9):
Public (NEXT_PUBLIC_*):
  NEXT_PUBLIC_MEDUSA_BACKEND_URL = https://api.shree-furniture.in
  NEXT_PUBLIC_RAZORPAY_KEY_ID = rzp_live_... (live key, NOT test)
  NEXT_PUBLIC_ALGOLIA_APP_ID = ...
  NEXT_PUBLIC_ALGOLIA_SEARCH_KEY = ... (read-only search key)
  NEXT_PUBLIC_POSTHOG_KEY = ...
  NEXT_PUBLIC_SENTRY_DSN = ...

Server-only (no NEXT_PUBLIC_):
  RAZORPAY_WEBHOOK_SECRET = ...
  REVALIDATION_SECRET = ...
  DATABASE_URL = ... (if any server-side DB calls)

Trigger a production deploy and verify:
- https://shree-furniture.in loads
- Product catalogue shows real products
- Add to cart works
- Sentry receives events
- PostHog receives events

Update STATUS.md when done.
```

#### Configure Railway Production Environment
```
@devops-engineer Configure Railway production environment for the MedusaJS backend.
In Railway project → Variables, add all backend env vars.

Verify these are set with PRODUCTION values (not sandbox/test):
  DATABASE_URL = Neon.tech production connection string
  REDIS_URL = Upstash production connection string
  RAZORPAY_SECRET = rzp_live secret (NOT rzp_test_)
  RAZORPAY_WEBHOOK_SECRET = production webhook secret
  CLOUDINARY_API_SECRET = ...
  ALGOLIA_WRITE_API_KEY = ...
  RESEND_API_KEY = ...
  JWT_SECRET = secure random 64-char string (generate: openssl rand -hex 32)
  COOKIE_SECRET = secure random 64-char string

After deploy:
1. Run npx medusa db:migrate on production
2. GET https://api.shree-furniture.in/store/products → should return products
3. Medusa Admin at https://api.shree-furniture.in/admin → should load

Update STATUS.md when done.
```

#### Verify Razorpay Webhook End-to-End in Production
```
@devops-engineer @security-auditor Verify the Razorpay webhook in production.
1. In Razorpay Dashboard → Settings → Webhooks:
   - URL: https://shree-furniture.in/api/webhooks/razorpay
   - Events: payment.captured, payment.failed
   - Secret: must match RAZORPAY_WEBHOOK_SECRET in Vercel env vars

2. Trigger a test payment via Razorpay test mode (use test card)
3. Verify in Vercel logs: webhook received, HMAC verified, order created
4. Verify no duplicate orders created on retry

Also verify the webhook endpoint is IP-restricted to Razorpay IPs if possible.
Ref: NewDocs/KNOWN-ISSUES.md "Duplicate Order Creation" for idempotency check.
Update STATUS.md when done.
```

#### Pre-Launch Definition of Done Checklist
```
@orchestrator Run the full pre-launch checklist.
Ref: NewDocs/10-development-phases.md section 7.

Verify each item and report ✅ or ❌:
- [ ] Full purchase flow: browse → cart → checkout → Razorpay (UPI + Card) → confirmation email
- [ ] Order confirmation email received within 2 minutes
- [ ] Admin can publish product with images in < 5 minutes
- [ ] LCP < 2.5s on 4G mobile simulation (Lighthouse)
- [ ] CLS < 0.1 (no layout shifts)
- [ ] INP < 200ms
- [ ] Razorpay webhook signature verification confirmed in production logs
- [ ] Duplicate webhook test: no duplicate orders created
- [ ] All secrets in env vars; git log shows no .env files committed
- [ ] TypeScript strict + ESLint + Vitest all pass in CI
- [ ] robots.txt disallows /checkout, /cart, /account
- [ ] sitemap.xml submitted to Google Search Console
- [ ] JSON-LD Product schema validates on a PDP page
- [ ] Custom domain live with SSL
- [ ] Sentry receiving errors in production project
- [ ] PostHog receiving events in production project
- [ ] pnpm audit shows no high/critical vulnerabilities

For any ❌ item: fix it before launch.
```

---

## 7. Testing — Full Suite

#### Write Price Utility Tests
```
@test-engineer Write Vitest tests for apps/storefront/lib/utils/price.ts.
Ref: NewDocs/12-testing-strategy.md section 3.1 for exact test cases.
Must cover:
- formatPrice(4999900) → "₹49,999", formatPrice(100) → "₹1", formatPrice(0) → "₹0"
- getDiscountPercent(5999900, 4999900) → 17, no discount → 0
- calculateGST: 12% inclusive extraction, 18% inclusive extraction, edge cases
File: apps/storefront/lib/utils/price.test.ts
```

#### Write Cart Store Tests
```
@test-engineer Write Vitest tests for apps/storefront/store/cart-store.ts.
Ref: NewDocs/12-testing-strategy.md section 3.2.
Must cover:
- addItemOptimistic: adds item, increments quantity for same variant
- removeItemOptimistic: removes correct item
- updateQuantityOptimistic: updates quantity
- rollbackOptimisticAdd: restores state after API failure
- total recalculation after each mutation
File: apps/storefront/store/cart-store.test.ts
```

#### Write Webhook Signature Verification Tests
```
@test-engineer Write Vitest tests for the Razorpay HMAC verification logic.
Ref: NewDocs/12-testing-strategy.md section 3.3.
Extract verifyRazorpaySignature() as a pure function for testability.
Must cover:
- Valid signature → returns true
- Tampered body → returns false
- Invalid signature string → returns false
- Empty signature → returns false
- Timing-safe comparison (not vulnerable to timing attacks)
File: apps/storefront/app/api/webhooks/razorpay/verify.test.ts
```

#### Write AddressForm Zod Validation Tests
```
@test-engineer Write Vitest tests for the AddressForm Zod schema.
Ref: NewDocs/12-testing-strategy.md section 3.4 for exact test cases.
Must cover:
- Valid address → passes validation
- Invalid PIN code (< 6 digits, > 6 digits, letters) → fails
- Invalid phone (< 10 digits, starts with 1-5) → fails
- Invalid email format → fails
- Missing required fields → fails with correct field path
File: apps/storefront/components/checkout/AddressForm.test.ts
```

#### Write Auth Form Validation Tests
```
@test-engineer Write Vitest tests for the login and register Zod schemas.
Must cover:
Login schema:
- Valid email + password → passes
- Invalid email → fails
- Password < 8 chars → fails

Register schema:
- Valid full registration → passes
- Passwords don't match → fails on confirmPassword
- Weak password → fails
File: apps/storefront/app/(account)/account/register/register.test.ts
```

#### Write Cart API Integration Tests
```
@test-engineer Write integration tests for lib/medusa/cart.ts wrappers.
Ref: NewDocs/12-testing-strategy.md section 4.1.
Tests run only when INTEGRATION_TESTS=true (skip in normal CI).
Must cover:
- createCart() → returns cart with valid ID (prefix: cart_)
- addItem(cartId, { variant_id, quantity }) → cart has 1 item
- updateItem(cartId, lineItemId, { quantity: 2 }) → item quantity is 2
- removeItem(cartId, lineItemId) → cart has 0 items
- Cart total calculation is correct in paise (no floats)
File: apps/storefront/lib/medusa/cart.integration.test.ts
```

#### Write Playwright E2E — Full Purchase Flow
```
@qa-automation-engineer Write Playwright E2E test for the complete purchase flow.
Ref: NewDocs/12-testing-strategy.md section 5.1 for the exact test steps.
Use data-testid selectors exclusively (never CSS class selectors).
Use Page Object Model pattern.
Flow:
1. Visit /collections/living-room
2. Click first product-card → PDP loads
3. Select first available variant (colour swatch)
4. Click add-to-cart → cart-drawer opens, item visible
5. Click proceed-to-checkout → /checkout
6. Fill address form with test data
7. Click continue-to-shipping → /checkout/shipping
8. Select standard shipping → click continue-to-payment → /checkout/payment
9. Verify order total shows GST note
10. Click place-order → mock Razorpay (use Playwright route intercept)
11. Verify redirect to /order/confirm/[id]
12. Verify order-confirmation-id is visible
File: e2e/purchase-flow.spec.ts
```

#### Write Playwright E2E — Auth Flow
```
@qa-automation-engineer Write Playwright E2E test for the auth flow.
Ref: NewDocs/12-testing-strategy.md section 5.2.
Test 1 — Register and login:
- Visit /account/register
- Fill form with unique email (test+[timestamp]@shree-furniture.in)
- Submit → should redirect to /account/orders
- Verify "My Orders" heading visible

Test 2 — Login with existing account:
- Visit /account/login
- Fill valid credentials
- Submit → redirect to /account/orders

Test 3 — Auth guard:
- Visit /account/orders without login
- Verify redirect to /account/login?redirect=/account/orders

File: e2e/auth.spec.ts
```

#### Run Full Test Suite
```
Run the complete test suite and show me a pass/fail summary:
  pnpm type-check
  pnpm lint
  pnpm test --reporter=verbose
  pnpm build

If anything fails, identify the root cause. Do not just show the error message.
For TypeScript errors: fix properly — no `any` unless justified with a comment.
```

---

## 8. Debugging

#### Debug Cart Issue
```
@debugger The cart [describe exact symptom].
Files to check:
  apps/storefront/store/cart-store.ts
  apps/storefront/hooks/useCart.ts
  apps/storefront/lib/medusa/cart.ts
  apps/storefront/components/cart/CartDrawer.tsx
Last change made: [what you last modified]
Error message: [paste full error if any]
Expected: [what should happen]
Actual: [what is happening]
Ref: NewDocs/KNOWN-ISSUES.md first — may already be documented.
```

#### Debug MedusaJS Subscriber Not Firing
```
@debugger A MedusaJS v2 subscriber is not firing.
Subscriber: backend/src/subscribers/[filename].ts
Event it should listen to: [event name]
Expected: [what should happen]
Actual: [nothing happens — check Railway logs first]
Check Railway logs for: "subscriber" errors, migration errors, startup errors
Ref: NewDocs/KNOWN-ISSUES.md "Subscriber Does Not Fire on First Deploy"
```

#### Debug ISR Cache Not Invalidating
```
@debugger Product updated in Medusa Admin is still showing old data on storefront.
Files involved:
  backend/src/subscribers/algolia-sync.ts
  apps/storefront/app/api/revalidate/route.ts
  apps/storefront/app/(store)/products/[handle]/page.tsx
Check Railway logs: did algolia-sync subscriber fire? Did revalidation API call succeed?
Ref: NewDocs/KNOWN-ISSUES.md "ISR Page Shows Stale Product After Admin Update"
```

#### Debug Razorpay Payment Issue
```
@debugger Razorpay payment is [failing / not redirecting / creating duplicate orders].
Files:
  apps/storefront/components/checkout/PaymentStep.tsx
  apps/storefront/app/api/webhooks/razorpay/route.ts
Environment: [sandbox / production]
Error: [paste Razorpay error code or Vercel function log]
Ref: NewDocs/KNOWN-ISSUES.md — check Razorpay section first.
```

#### Debug TypeScript Strict Error
```
@debugger TypeScript error in [filename]:
[paste full error]
Fix this properly without using `any`.
If `any` is genuinely necessary, add: // reason: [explanation]
```

#### Add New Gotcha to Known Issues
```
I discovered a new issue: [describe the symptom, root cause, and fix].
Add it to NewDocs/KNOWN-ISSUES.md using this format:
### [SHORT TITLE]
Symptom: ...
Root Cause: ...
Fix/Workaround: ...
Affects: [files]
Date Discovered: [today]
```

---

## 9. DevOps & CI/CD

#### Deploy to Vercel (First Time)
```
@devops-engineer Set up the Vercel deployment for apps/storefront.
Ref: NewDocs/11-environment-and-devops.md.
Configure:
- Link monorepo root to Vercel project
- Set Root Directory: apps/storefront
- Framework Preset: Next.js
- Build Command: cd ../.. && pnpm build --filter=storefront
- Install Command: pnpm install --frozen-lockfile
- Add all NEXT_PUBLIC_ env vars from .env.example
- Add all server-only env vars from .env.example
- Enable Vercel Analytics (for Core Web Vitals)
Create .github/workflows/deploy.yml for auto-deploy on merge to main.
Update STATUS.md when done.
```

#### Deploy Backend to Railway (First Time)
```
@devops-engineer Set up Railway deployment for the MedusaJS v2 backend.
Configure:
- Service name: shree-furniture-backend
- Root directory: backend/
- Build command: pnpm install && pnpm build
- Start command: node_modules/.bin/medusa start
- Health check path: /health
- Add all backend env vars from .env.example
- Post-deploy command: npx medusa db:migrate
Create .github/workflows/deploy.yml Railway deploy step (after Vercel deploy succeeds).
Update STATUS.md when done.
```

#### Debug CI Pipeline Failure
```
@devops-engineer The GitHub Actions CI pipeline is failing.
Workflow: .github/workflows/ci.yml
Failed step: [which step: tsc / lint / test / build]
Error output: [paste the error log]
Branch: [branch name]
Check: are all required secrets set in GitHub Secrets? (TURBO_TOKEN if using remote cache)
```

#### Add a New Environment Variable
```
@devops-engineer I need to add a new environment variable: [VAR_NAME]
Value type: [public browser-safe / server-only secret]

1. Add to .env.example with a comment explaining what it is
2. If NEXT_PUBLIC_: add prefix; if server-only: no prefix
3. Add to apps/storefront/.env.local (dev) or backend/.env (dev) with placeholder
4. Add to .github/workflows/ci.yml env section if needed for CI
5. Document: add to Vercel dashboard (storefront) or Railway dashboard (backend)
6. Update NewDocs/11-environment-and-devops.md if it documents env vars
```

---

## 10. Session End — Always Last

### Standard Session End
```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark completed items ✅ with a Notes entry describing the implementation
   - Mark partial items 🚧 with exactly what's left to do
   - If anything was started but not in STATUS.md's table, flag it
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I expressed any design preference
6. Update REJECTIONS.md if I rejected anything

Show me a summary of all changes before committing.

```

### Session End With Note for Next Session
```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark [component] as ✅ DONE with a Notes entry
   - Add note: "[what the next session needs to know about this implementation]"
   - Mark partial items 🚧 with exactly what's left to do
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I confirmed a design decision today
6. Update REJECTIONS.md if I rejected an approach today

Show me a summary of all changes before committing.
```

---

## 11. Anti-Patterns & Quick Reference

### Prompts That Get Bad Results — Avoid These

| ❌ Don't say | ✅ Say instead |
|---|---|
| `"Build me a backend"` | `"@backend-specialist Build [specific file]"` |
| `"Make a nice UI"` | `"@frontend-specialist Build [Component] per NewDocs/01 section [X]"` |
| `"Use Prisma for the database"` | Never override the ORM — it's fixed in DECISIONS.md |
| `"Add reviews while you're here"` | Reviews = Phase 3. Check doc 09 before expanding scope |
| `"Just use any type here"` | `"Use unknown + type guard"` |
| `"Store the token in localStorage"` | HTTP-only cookies only. ADR-009. |
| `"Use floats for price"` | Paise integers only. ADR-007. |
| `"Build a custom cart"` | Medusa Cart Module handles this. Never rebuild it. |
| Long vague descriptions | Short + file path + NewDocs reference |
| Forgetting "Update STATUS.md" | Always append it to build prompts |

### Agent Quick Reference

| Say | Gets you |
|---|---|
| `@backend-specialist` | MedusaJS v2, subscribers, workflows, modules, API routes |
| `@frontend-specialist` | Next.js pages, components, Zustand, TanStack Query, Tailwind |
| `@database-architect` | Schema, MikroORM entities, indexes, query optimisation |
| `@devops-engineer` | CI/CD, Railway, Vercel, Docker, GitHub Actions |
| `@test-engineer` | Vitest unit/integration tests, TDD |
| `@qa-automation-engineer` | Playwright E2E, CI test pipelines |
| `@security-auditor` | Webhook security, auth, OWASP audit |
| `@debugger` | Unexpected behavior, root cause analysis |
| `@performance-optimizer` | LCP, CLS, bundle size, Core Web Vitals |
| `@seo-specialist` | generateMetadata, JSON-LD, sitemap (Phase 2+) |
| `@orchestrator` | Tasks spanning multiple files or domains |
| `@explorer-agent` | "What files exist for X?", codebase mapping |

### Key NewDocs References

```
01 — product-requirements.md      Feature specs + acceptance criteria
03 — information-architecture.md  URL structure, page inventory, route groups
04 — system-architecture.md       Rendering strategy (ISR/CSR/SSR), data flows
05 — database-schema.md           All table definitions (check before any DB work)
06 — api-contracts.md             All Medusa endpoints + request/response shapes
07 — monorepo-structure.md        File locations, package structure
08 — cart-pricing-engine-spec.md  Cart lifecycle, pricing, tax, shipping rules
09 — engineering-scope-definition.md  What NOT to build (check before starting any feature)
10 — development-phases.md        Phase milestones, acceptance criteria, DoD checklist
12 — testing-strategy.md          What to test, test file locations, coverage targets
MEDUSA-V2-PATTERNS.md             v2 SDK methods, subscriber/workflow/module patterns
KNOWN-ISSUES.md                   Gotchas log — always check before debugging
CLAUDE.md                         Hard rules (auto-loaded every session)
STATUS.md                         Current build state (read at session start, update at end)
PREFERENCES.md                    Your design taste — UI agents read before generating any UI
REJECTIONS.md                     Never-again log — agents read before suggesting any pattern
STACK-CHANGES.md                  Mid-project tech changes — entries here supersede CLAUDE.md
```

---

*Shree Furniture | Prompt Guide v2.0 — Complete Start-to-Deployment | Q1 2026*
*Place at: `shreefurniture-pro/PROMPT-GUIDE-v2.md` (monorepo root)*