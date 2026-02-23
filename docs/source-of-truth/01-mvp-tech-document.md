# Shree Furniture — MVP Tech Document

> **Project:** Shree Furniture E-Commerce Platform
> **Document Type:** MVP Tech Document
> **Version:** 1.0
> **Date:** February 2026
> **Stack:** Next.js · TypeScript · Supabase · ShadCN UI · Tailwind CSS

---

## Table of Contents

1. [MVP Vision & Goals](#1-mvp-vision--goals)
2. [MVP Feature Scope](#2-mvp-feature-scope)
3. [Prioritized Feature List](#3-prioritized-feature-list)
4. [Development Phases](#4-development-phases)
5. [Cost-Efficient Infrastructure Setup](#5-cost-efficient-infrastructure-setup)
6. [Deployment Approach](#6-deployment-approach)
7. [Risks & Mitigation](#7-risks--mitigation)
8. [Definition of Done](#8-definition-of-done)

---

## 1. MVP Vision & Goals

The Shree Furniture MVP is designed to deliver a **fully functional, production-ready e-commerce experience** that allows customers to discover, filter, and purchase furniture online, while providing admins with complete control over catalog, inventory, and order management.

### Primary Goals

- Launch a live, revenue-generating storefront within **10–12 weeks**
- Serve real customers with a fully functional buying journey from browse → checkout → order confirmation
- Keep monthly infrastructure costs **under ₹3,000/month** during early growth
- Lay a clean architectural foundation that supports future scaling without significant rewrites
- Achieve a Lighthouse performance score of **≥ 85** on mobile

### Non-Goals for MVP

- Mobile native apps (iOS / Android)
- Multi-vendor / marketplace features
- AI-powered product recommendations
- Live chat / customer support widget
- Advanced loyalty or reward programs
- Multi-language / multi-currency support
- ERP or warehouse management integration

---

## 2. MVP Feature Scope

### ✅ Customer Features — In Scope

| # | Feature | Priority |
|---|---------|----------|
| C1 | Homepage with hero banner, featured products, category grid | P0 |
| C2 | Category browsing with pagination | P0 |
| C3 | Advanced product filters (price range, category, material, in-stock, rating) | P0 |
| C4 | Product detail page (images gallery, specs, dimensions, materials, variants) | P0 |
| C5 | Full-text search with debounced input | P0 |
| C6 | Cart management (add, remove, update quantity, persist across sessions) | P0 |
| C7 | Wishlist (add/remove, requires login) | P1 |
| C8 | Guest checkout + Registered checkout | P0 |
| C9 | Razorpay payment integration (Cards, UPI, Net Banking, Wallets) | P0 |
| C10 | Order confirmation page + email notification | P0 |
| C11 | Order history & basic order tracking | P1 |
| C12 | Customer account registration & login (Email + Google OAuth) | P0 |
| C13 | Customer profile management (address book, account details) | P1 |
| C14 | Promotional banner display on homepage | P1 |
| C15 | Discount / coupon code application at checkout | P1 |
| C16 | Product image zoom on detail page | P2 |
| C17 | Related products section on PDP | P2 |
| C18 | Recently viewed products | P2 |

### ✅ Admin Features — In Scope

| # | Feature | Priority |
|---|---------|----------|
| A1 | Secure admin login with role-based access | P0 |
| A2 | Product CRUD (create, edit, delete, publish/draft) | P0 |
| A3 | Bulk image upload with optimization via Supabase Storage | P0 |
| A4 | Category & subcategory management | P0 |
| A5 | Product attribute management (materials, dimensions, colors) | P0 |
| A6 | Inventory / stock quantity management | P0 |
| A7 | Low-stock alerts (threshold-based notifications) | P1 |
| A8 | Order listing, filtering, and status updates | P0 |
| A9 | Customer list & basic profile view | P1 |
| A10 | Discount code creation & management | P1 |
| A11 | Homepage banner CRUD (upload image, set link, toggle active) | P1 |
| A12 | Basic analytics dashboard (revenue, GMV, orders, top products) | P1 |

### ❌ Out of Scope (Post-MVP / V2)

- Product reviews & ratings system
- Advanced analytics (cohort analysis, funnel analysis)
- Email marketing automation / drip campaigns
- Real-time inventory sync with physical store
- Abandoned cart recovery emails
- Returns & refund management workflow
- Multiple shipping provider integration
- A/B testing framework
- Progressive Web App (PWA) support
- Advanced SEO tooling (XML sitemap auto-generation is in scope)

---

## 3. Prioritized Feature List

### Priority Tiers Explained

| Priority | Meaning |
|----------|---------|
| **P0 — Must Have** | Non-negotiable. MVP is not shippable without these. |
| **P1 — Should Have** | High business value; include if timeline permits. |
| **P2 — Nice to Have** | Enhances experience; defer to post-MVP if needed. |

### P0 Features — Core Commerce Flow

```
Authentication → Browse Catalog → Search & Filter → View Product →
Add to Cart → Checkout → Payment (Razorpay) → Order Confirmation
```

1. Supabase Auth (email + Google OAuth)
2. Homepage layout (hero, featured, categories)
3. Category listing page with filter sidebar
4. Product listing page (grid view with pagination)
5. Product detail page (gallery, specs, variants, add to cart)
6. Full-text search (`pg_trgm` via Supabase)
7. Cart (Zustand local state + Supabase persisted for logged-in users)
8. Checkout form (address, shipping method, order summary)
9. Razorpay payment gateway integration
10. Order creation & confirmation
11. Admin: Product CRUD + image upload
12. Admin: Category management
13. Admin: Order management (view + update status)
14. Admin: Inventory stock management
15. SEO: metadata, Open Graph, structured JSON-LD, sitemap

### P1 Features — Business Completeness

1. Wishlist (save products for later)
2. Customer profile & address book
3. Order history + status tracking page
4. Discount / coupon codes at checkout
5. Homepage promotional banners (admin-managed)
6. Admin dashboard analytics (revenue KPIs)
7. Low-stock threshold alerts
8. Customer management in admin
9. Transactional emails (order confirmation, shipping update) via Resend

### P2 Features — Enhancement Layer

1. Product image zoom/lightbox
2. Related products carousel on PDP
3. Recently viewed products (localStorage)
4. Sort options on listing (Price: Low→High, Newest, Best Seller)
5. Breadcrumb navigation

---

## 4. Development Phases

### Phase 0 — Project Foundation (Week 1)

**Objective:** Set up development environment, repository, and CI/CD baseline.

**Deliverables:**
- Next.js 14 project scaffold with TypeScript, Tailwind CSS, ShadCN UI
- ESLint + Prettier + Husky pre-commit hooks configured
- Supabase project created (dev + prod environments)
- Database schema migrations baseline (users, products, categories, orders, carts)
- Row Level Security (RLS) policies defined and applied
- GitHub repository with branch protection rules (`main`, `develop`)
- Vercel project linked to GitHub for preview deployments
- Environment variable management (`.env.local`, Vercel env config)

**Team Focus:** Full-stack setup, DevOps

---

### Phase 1 — Core Customer Experience (Weeks 2–4)

**Objective:** Build the entire customer-facing shopping flow.

**Deliverables:**

**Week 2 — Discovery & Browsing**
- Homepage layout (hero banner, featured products, category grid)
- Category listing page with URL-based filter state
- Product listing grid with pagination
- Filter sidebar (price range, category, in-stock, material)
- Basic search with Supabase full-text search
- Responsive mobile layout

**Week 3 — Product Detail & Cart**
- Product Detail Page (PDP) with image gallery, specs, variants selector
- Add-to-cart functionality (Zustand state)
- Cart drawer/page with quantity controls
- Cart persistence (Supabase for logged-in, localStorage for guests)
- Wishlist functionality

**Week 4 — Authentication & Checkout**
- Supabase Auth integration (email + Google OAuth)
- Signup/Login UI with ShadCN components
- Checkout page (address form, order summary)
- Guest checkout flow
- Address book for registered users

---

### Phase 2 — Payments & Orders (Week 5)

**Objective:** Complete the purchase lifecycle with Razorpay integration.

**Deliverables:**
- Razorpay payment integration (frontend SDK + Supabase Edge Function for order verification)
- Payment success/failure handling
- Order creation in database on payment success
- Order confirmation page with order summary
- Order confirmation email via Resend
- Webhook handler for Razorpay payment events (Supabase Edge Function)

---

### Phase 3 — Admin Dashboard (Weeks 6–7)

**Objective:** Build the complete admin panel for store management.

**Deliverables:**

**Week 6 — Catalog & Inventory**
- Admin authentication with role-based access (admin role in Supabase)
- Product CRUD interface (create, edit, delete, publish/draft toggle)
- Multi-image upload with Supabase Storage + image reordering
- Category & subcategory management
- Product attribute management (materials, dimensions, colors)
- Inventory stock quantity editor with low-stock flag

**Week 7 — Orders, Customers & Promotions**
- Order management table (filter by status, date, customer)
- Order status update workflow (pending → processing → shipped → delivered)
- Customer list view with order history
- Discount code creation, management, and usage tracking
- Homepage banner management (image upload, link, active/inactive toggle)
- Basic analytics dashboard (total revenue, order count, average order value, top 5 products)

---

### Phase 4 — SEO, Performance & Polish (Week 8)

**Objective:** Optimize for production quality, SEO, and performance.

**Deliverables:**
- Dynamic metadata per page (title, description, Open Graph, Twitter Card)
- JSON-LD structured data for products (Product schema)
- Auto-generated XML sitemap (`/sitemap.xml`)
- `robots.txt` configuration
- Next.js Image component optimization across all image instances
- Font optimization (next/font with subset)
- Lazy loading for below-the-fold content
- Core Web Vitals audit and fixes (LCP, CLS, FID)
- Accessibility audit (ARIA labels, keyboard nav, color contrast)
- Error boundary implementation
- 404, 500 custom error pages
- Loading skeleton states throughout app

---

### Phase 5 — QA, Security Audit & Launch (Weeks 9–10)

**Objective:** Harden, test, and deploy to production.

**Deliverables:**
- Full cross-browser testing (Chrome, Firefox, Safari, Edge)
- Mobile device testing (iOS Safari, Android Chrome)
- End-to-end checkout flow testing with Razorpay test mode
- RLS policy security audit
- Dependency vulnerability scan (`npm audit`)
- Environment secrets audit (no secrets in client-side code)
- Performance benchmark (Lighthouse ≥ 85 mobile)
- Razorpay production credentials integration
- Vercel production deployment
- Domain configuration, SSL verification
- Supabase backup policy enabled
- Monitoring setup (Vercel Analytics + Supabase Dashboard)
- Post-launch smoke testing checklist

---

### Phase 6 — Post-Launch Stabilization (Weeks 11–12)

**Objective:** Monitor production, fix issues, and ship P2 features.

**Deliverables:**
- Bug fixing based on real user feedback
- P2 feature implementation (product zoom, related products, recently viewed)
- Performance monitoring review
- Conversion funnel analysis (drop-off points)
- SEO indexing verification in Google Search Console

---

## 5. Cost-Efficient Infrastructure Setup

### Monthly Cost Breakdown

| Service | Plan | Monthly Cost (INR) |
|---------|------|--------------------|
| **Supabase** | Free tier (500MB DB, 1GB storage, 50MB bandwidth) | ₹0 |
| **Vercel** | Hobby (100GB bandwidth, unlimited deployments) | ₹0 |
| **Razorpay** | 2% per transaction (no monthly fee) | Transaction-based |
| **Resend** (email) | Free tier (3,000 emails/month) | ₹0 |
| **Domain** | .in / .com from Namecheap or GoDaddy | ~₹800/year |
| **Total Fixed** | | **~₹67/month** |

> **When to upgrade:**
> - **Supabase Pro (₹2,100/month):** When DB exceeds 500MB or you need PITR backups — typically at ~5,000+ products or 10,000+ orders
> - **Vercel Pro (₹1,700/month):** When you need team collaboration features or bandwidth exceeds 100GB/month
> - **Resend Pro ($20/month):** When transactional emails exceed 3,000/month

### Why This Stack Is Cost-Optimal

**Supabase Free Tier covers:**
- PostgreSQL database with full relational capabilities
- Authentication (unlimited users on free tier)
- Storage (1GB for product images — sufficient for ~500–1,000 products at optimized sizes)
- Edge Functions (500,000 invocations/month free)
- Realtime subscriptions

**Vercel Hobby covers:**
- Unlimited Next.js deployments
- Automatic preview deployments per PR
- Built-in CDN for static assets
- Serverless function execution
- Automatic HTTPS

**Image CDN Strategy (Zero Additional Cost):**
- Upload to Supabase Storage → serve via Supabase CDN
- Use Next.js `<Image>` with `remotePatterns` configured for Supabase Storage URL
- Generate WebP variants via Next.js Image Optimization API
- Add `quality={80}` and appropriate `sizes` attribute for responsive delivery

---

## 6. Deployment Approach

### Environments

```
Local Dev → Feature Branch → PR Preview (Vercel) → Staging → Production
```

| Environment | URL | Branch | Purpose |
|-------------|-----|--------|---------|
| Local | `localhost:3000` | Any feature branch | Developer testing |
| Preview | `*.vercel.app` | Any PR branch | Stakeholder review, QA |
| Staging | `staging.shreefurniture.in` | `develop` | Pre-production testing |
| Production | `shreefurniture.in` | `main` | Live customers |

### CI/CD Pipeline

```
Push to branch
     │
     ▼
GitHub Actions
  ├── Type check (tsc --noEmit)
  ├── Lint check (ESLint)
  ├── Unit tests (Vitest)
  └── Build check (next build)
     │
     ▼ (on PR)
Vercel Preview Deploy
  └── PR comment with preview URL
     │
     ▼ (on merge to main)
Vercel Production Deploy
  └── Automatic with zero-downtime
```

### Environment Variable Management

```bash
# .env.local (never committed)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=         # Server-only
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=               # Server-only
NEXT_PUBLIC_RAZORPAY_KEY_ID=
RESEND_API_KEY=                    # Server-only
NEXT_PUBLIC_APP_URL=
```

**Rules:**
- `NEXT_PUBLIC_*` variables are safe for client-side exposure
- Service role keys, Razorpay secrets, and email API keys must **never** be prefixed with `NEXT_PUBLIC_`
- All secrets are stored in Vercel's encrypted environment variable store

### Rollback Strategy

- Vercel maintains full deployment history — instant rollback to any previous deployment via dashboard
- Database migrations use `supabase db diff` → versioned migration files in `/supabase/migrations/`
- Never run destructive migrations in production without a backup

---

## 7. Risks & Mitigation

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| R1 | Razorpay webhook failures cause missed order confirmations | Medium | High | Idempotent webhook handler with retry logic; manual order reconciliation dashboard |
| R2 | Supabase free tier storage exceeded (product images) | Medium | Medium | Compress images to WebP < 200KB before upload; upgrade Storage as needed |
| R3 | RLS misconfiguration exposes customer data | Low | Critical | Comprehensive RLS policy testing suite; read back as anon role in tests |
| R4 | Scope creep extends timeline beyond 12 weeks | High | Medium | Strict P0/P1/P2 prioritization; weekly scope review meetings |
| R5 | Next.js cold starts on serverless functions impact checkout UX | Low | Medium | Move critical APIs to Supabase Edge Functions; add loading states |
| R6 | Search performance degrades with large product catalog | Low | Medium | Add `pg_trgm` index; implement search result caching with SWR stale-while-revalidate |
| R7 | Admin panel accessible without proper auth | Low | Critical | Middleware-enforced auth on all `/admin/*` routes; Supabase RLS `admin` role check |
| R8 | SEO indexing delayed due to client-side rendering | Low | Medium | Use Next.js Server Components for all public pages; verify SSR in production |
| R9 | Payment double-charge on network retry | Low | High | Idempotency key on every Razorpay order creation |
| R10 | Dependency vulnerability in third-party packages | Medium | Medium | Weekly `npm audit`; Dependabot alerts enabled on GitHub |

---

## 8. Definition of Done

A feature is considered **done** when it meets ALL of the following:

- [ ] Code reviewed and approved by at least one other developer
- [ ] TypeScript types defined (no `any` without justification)
- [ ] Responsive across mobile (375px), tablet (768px), and desktop (1280px)
- [ ] Accessibility: keyboard navigable, ARIA labels present, color contrast ≥ 4.5:1
- [ ] Loading and error states implemented
- [ ] RLS policies applied and verified for any new Supabase tables
- [ ] Environment variables documented in `.env.example`
- [ ] No console errors or warnings in production build
- [ ] Lighthouse score ≥ 85 on mobile for affected pages
- [ ] Tested on Chrome (desktop), Safari (iOS), and Chrome (Android)
- [ ] Deployed to preview environment and approved by client

---

*Document Version 1.0 — Shree Furniture MVP Tech Document*
