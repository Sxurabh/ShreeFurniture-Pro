# 10 — Development Phases
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Total Timeline:** 8 weeks to MVP launch

---

## 1. Phase Overview

| Phase | Duration | Goal |
|---|---|---|
| Phase 1 — Foundation | Weeks 1–2 | Monorepo setup, backend live, DB seeded |
| Phase 2 — Storefront Core | Weeks 3–4 | Browsable product catalogue on Vercel |
| Phase 3 — Transactions | Weeks 5–6 | End-to-end purchase flow with Razorpay |
| Phase 4 — Launch Prep | Week 7 | QA, performance audit, security hardening |
| **MVP Launch** | **Week 8** | **Live on Vercel + Railway** |

---

## 2. Phase 1 — Foundation (Weeks 1–2)

**Goal:** Working monorepo, MedusaJS backend live on Railway, PostgreSQL and Redis connected, local dev environment running.

### Week 1 — Monorepo & Dev Environment

**Deliverables:**
- Monorepo initialised: pnpm workspaces + Turborepo
- `apps/storefront/` scaffolded with Next.js 15 (App Router)
- `backend/` scaffolded with MedusaJS v2
- `packages/types/` and `packages/ui/` created with initial content
- Docker Compose running PostgreSQL 16 + Redis 7 locally
- `.env.example` documents all required environment variables
- Root-level `CLAUDE.md`, `CONVENTIONS.md`, `DECISIONS.md` created
- GitHub repository created with branch protection on `main`
- CI pipeline (`.github/workflows/ci.yml`) running: tsc + eslint + build check

**Acceptance:**
- `pnpm dev` starts all apps concurrently
- `pnpm build` completes without errors
- CI passes on first PR

---

### Week 2 — MedusaJS Backend Live

**Deliverables:**
- MedusaJS deployed to Railway.app
- PostgreSQL (Neon.tech) connected and migrations run
- Redis (Upstash) connected for sessions and queue
- Medusa Admin accessible at `/admin`
- Sample products, collections, and regions (India/INR) seeded
- Cloudinary plugin configured; test image uploaded via Admin
- Resend plugin configured; test email sent
- Razorpay plugin configured; sandbox credentials wired
- Algolia index created; test products synced
- All backend env vars configured in Railway

**Acceptance:**
- `GET /store/products` returns products via API
- Medusa Admin accessible and product CRUD works
- Test order confirmation email sends successfully

---

## 3. Phase 2 — Storefront Core (Weeks 3–4)

**Goal:** Customers can browse the full product catalogue. Homepage, PLP, and PDP are live on Vercel with real product data.

### Week 3 — Homepage + PLP

**Deliverables:**
- Cloudflare DNS configured; domain pointing to Vercel
- Homepage (`/`) built with ISR:
  - Hero section (full-width banner, headline, CTA)
  - Featured Collections grid (Living Room, Bedroom, Dining, Office)
  - Best Sellers carousel
  - Trust signals strip
- Product Listing Page (`/collections/[handle]`) built with ISR:
  - Product grid (2/3/4 column responsive)
  - `ProductCard` component with thumbnail, name, price, discount badge
  - Filter panel (sidebar desktop / bottom sheet mobile)
  - Sort options
  - "Load More" pagination
  - Empty state
- Breadcrumb component wired on PLP
- `Header` with navigation links, search icon, wishlist icon, cart icon with badge
- `Footer` with navigation and payment icons
- Responsive design verified at 360px, 768px, 1280px

**Acceptance:**
- PLP loads with ISR, LCP < 2.5s on 4G simulation
- Filter params reflected in URL query string
- All collection links in header and homepage grid resolve correctly

---

### Week 4 — Product Detail Page + Search

**Deliverables:**
- PDP (`/products/[handle]`) built with ISR:
  - `ProductGallery` — primary image + thumbnail strip
  - `ProductVariants` — colour swatches, size selectors
  - Price display with discount percentage
  - Stock status indicator ("In Stock" / "Only n left" / "Out of Stock")
  - Quantity selector with stock cap
  - "Add to Cart" button (sticky on mobile)
  - "Add to Wishlist" button (guest: prompt to login)
  - Product description (rich text)
  - Specifications table
  - Related Products carousel
  - Delivery estimate with PIN code input
  - JSON-LD Product schema + `generateMetadata()` for SEO
  - `not-found.tsx` for invalid handles
- Search overlay built:
  - `SearchBar` in header with 300ms debounce
  - `SearchOverlay` full-screen on mobile
  - Algolia InstantSearch integration
  - "No results" state with category suggestions

**Acceptance:**
- Variant switch updates price, images, stock without page reload
- "Add to Cart" triggers cart drawer (even without cart state yet)
- JSON-LD Product schema validates in Google Rich Results Test
- Search returns results within 200ms

---

## 4. Phase 3 — Transactions (Weeks 5–6)

**Goal:** A customer can add items to cart, check out, and pay via Razorpay. Order confirmation email is sent automatically.

### Week 5 — Cart + State Management

**Deliverables:**
- Zustand cart store (`cart-store.ts`) implemented:
  - Optimistic add item (instant UI update, rollback on error)
  - Remove item, update quantity
  - Cart persistence via Medusa API (Redis backend)
- TanStack Query hooks for cart operations
- `CartDrawer` component — slide-in, shows all items, subtotal, free shipping progress
- `CartItem` component — thumbnail, name, variant, quantity stepper, remove
- `CartSummary` — subtotal, shipping estimate, GST note, CTA
- Cart badge in header updates on add/remove
- Cart persists across browser refresh (cart_id stored in cookie)
- Empty cart state

**Acceptance:**
- Add item → drawer opens in < 500ms (optimistic)
- Cart survives browser refresh
- Quantity cannot exceed inventory_quantity

---

### Week 6 — Checkout + Razorpay

**Deliverables:**
- Multi-step checkout layout (`(checkout)/layout.tsx`) with progress indicator
- Step 1 — Address (`/checkout`):
  - React Hook Form + Zod validation
  - Guest email field
  - Full address form with PIN code validation
  - Saved addresses dropdown for logged-in users
- Step 2 — Shipping (`/checkout/shipping`):
  - Shipping options list with prices and delivery estimates
  - Free shipping threshold shown
- Step 3 — Payment (`/checkout/payment`):
  - Order summary
  - "Place Order" → Razorpay modal opens
  - `POST /store/carts/:id/payment-sessions` called before modal opens
  - Razorpay JS SDK integrated
  - Payment failure: inline error with retry (no page redirect)
- Webhook handler (`/api/webhooks/razorpay/route.ts`):
  - HMAC-SHA256 signature verification
  - Idempotency check
  - `carts.complete()` on success
- Order confirmation page (`/order/confirm/[id]`) — SSR, shows full order summary
- `order.placed` subscriber triggers Resend confirmation email
- User authentication pages: `/account/login`, `/account/register`, `/account/forgot-password`
- My Orders list and detail pages (post-login)
- Wishlist module (backend) + wishlist API routes + Wishlist page

**Acceptance:**
- Full purchase flow: browse → cart → checkout → Razorpay → confirmation email
- Duplicate webhook events do not create duplicate orders
- Order confirmation email received within 2 minutes
- Guest checkout works without account creation

---

## 5. Phase 4 — Launch Prep (Week 7)

**Goal:** All Definition of Done criteria met. Platform is production-ready.

### QA & Testing

- End-to-end purchase flow tested on:
  - Mobile (iOS Safari, Android Chrome)
  - Desktop (Chrome, Safari, Firefox)
  - 4G throttling simulation
- Razorpay webhook verified end-to-end in production mode
- All P0 and P1 features smoke tested by product stakeholder
- Edge cases tested: out-of-stock variants, PIN code not serviceable, payment failure + retry, duplicate webhook

### Performance Audit

- Lighthouse run on Homepage, PLP, PDP — target: LCP < 2.5s, INP < 200ms, CLS < 0.1
- Next.js `@next/bundle-analyzer` — initial JS bundle < 200KB
- Core Web Vitals verified via Vercel Analytics

### Security Hardening

- All secrets confirmed in env vars; `git log` audit for any accidental commits
- `npm audit` run; no high/critical vulnerabilities unaddressed
- Razorpay webhook signature verification confirmed in production logs
- Rate limiting tested: auth endpoint rejects after 5 attempts
- HTTP-only cookie confirmed: JWT not accessible via `document.cookie` in browser console

### SEO Checks

- All PLP and PDP pages crawlable (Screaming Frog / manual check)
- `robots.txt` excludes `/checkout`, `/cart`, `/account`
- `sitemap.xml` generated and submitted to Google Search Console
- Canonical tags correct on all pages

### Go-Live

- Custom domain live on Vercel with Cloudflare CDN
- MedusaJS production URL configured
- PostHog, Sentry verified receiving events in production
- Admin team trained on product and order management
- First real product catalogue uploaded

---

## 6. Post-MVP Roadmap

### Phase 2 — Commerce Complete (Months 3–4)
- Reviews & Ratings (basic star + text)
- Loyalty program foundation
- Migration to Hetzner VPS (MedusaJS + PostgreSQL + Redis + MeiliSearch)
- Bulk CSV product import
- Inventory reservation on cart add
- Advanced discount codes
- SMS notifications (MSG91 / Exotel)

### Phase 3 — Growth (Months 5–6)
- AR / 3D product viewer (React Three Fiber)
- GST-compliant invoicing (CGST/SGST/IGST)
- Multi-currency (future expansion)
- Marketing email automation (Klaviyo or Customer.io)
- Referral program
- SEO enhancements: FAQ schema, review schema

---

## 7. Definition of Done (MVP Launch Checklist)

- [ ] Customer can browse, add to cart, and complete checkout via Razorpay (UPI + Card) without errors
- [ ] Order confirmation email is triggered automatically via Resend
- [ ] Admin can publish a product with images, variants, and pricing in under 5 minutes
- [ ] Core Web Vitals pass: LCP < 2.5s, CLS < 0.1, INP < 200ms on 4G mobile simulation
- [ ] Razorpay webhook signature verification active and tested end-to-end in production
- [ ] Duplicate webhook events confirmed as idempotent (no duplicate orders)
- [ ] All secrets stored as environment variables, no secrets in Git history
- [ ] Codebase passes TypeScript strict mode, ESLint, and Vitest in CI pipeline
- [ ] `robots.txt` and `sitemap.xml` present and correctly configured
- [ ] JSON-LD Product schema validates on all PDP pages
- [ ] Custom domain live with Cloudflare CDN + SSL
- [ ] Sentry and PostHog confirmed receiving events in production

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
