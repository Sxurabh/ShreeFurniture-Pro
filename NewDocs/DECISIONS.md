# DECISIONS.md — Architecture Decision Records
## Shree Furniture | E-Commerce Platform

> This log explains **why** each major technology choice was made.
> AI agents and new engineers: read this before questioning or proposing alternatives.

---

## ADR-001 | MedusaJS v2 over Custom Backend

**Decision:** Use MedusaJS v2 as the headless commerce engine.

**Alternatives Considered:**
- Custom Node.js/Express API from scratch
- Shopify (SaaS)
- WooCommerce

**Reasoning:**
- Building cart, order, customer, and payment logic from scratch is 6–8 weeks of non-differentiating work
- MedusaJS is open-source and self-hostable — zero per-transaction fees (critical for furniture with high AOV)
- Shopify would lock all commerce data behind a SaaS platform; we own nothing
- MedusaJS v2's module system allows replacing individual services (search, email, payment) without rewriting business logic
- The `medusa-payment-razorpay` plugin exists and is webhook-ready

**Trade-offs:**
- MedusaJS v2 is relatively new; API may have breaking changes → mitigation: pin exact version
- Less Google-able than Shopify → mitigation: official docs are comprehensive

---

## ADR-002 | Razorpay over Stripe / PayU

**Decision:** Use Razorpay as the primary payment gateway.

**Alternatives Considered:**
- Stripe (international-first)
- PayU (Indian)
- CCAvenue (Indian)

**Reasoning:**
- Razorpay is the dominant Indian payment gateway with best UPI and EMI coverage
- UPI is critical for high-AOV furniture (customers need familiar, trusted payment methods)
- EMI and BNPL (Simpl, LazyPay) are essential for ₹20,000+ furniture orders
- `medusa-payment-razorpay` plugin reduces integration effort significantly
- Stripe's India support (UPI especially) is inferior and requires additional compliance steps

---

## ADR-003 | Next.js 15 App Router over Pages Router

**Decision:** Use Next.js 15 with the App Router.

**Reasoning:**
- React Server Components allow fetching product data server-side without client JS, reducing bundle size and improving LCP
- ISR (Incremental Static Regeneration) is the optimal strategy for furniture product pages: SEO-friendly, fast, tolerable staleness
- `generateMetadata()` is cleaner than `getServerSideProps` + `<Head>` for SEO
- Parallel data fetching via `Promise.all()` in Server Components is idiomatic

**Trade-offs:**
- App Router has a steeper learning curve than Pages Router
- Some ecosystem libraries don't yet support RSC → mitigation: use `'use client'` directive at the component boundary

---

## ADR-004 | Zustand over Redux / Context API

**Decision:** Use Zustand for client-side state (cart, UI).

**Reasoning:**
- Cart state needs to be accessible from many components without prop drilling
- Zustand has ~1KB bundle overhead vs Redux Toolkit's ~40KB
- No boilerplate: no reducers, actions, or dispatch
- Supports optimistic updates with easy rollback pattern
- TanStack Query handles all *server* state; Zustand is only for *client* state

---

## ADR-005 | TanStack Query v5 over SWR or React Query v4

**Decision:** Use TanStack Query v5 for server state management.

**Reasoning:**
- Background refetch keeps product listings fresh without polling
- Built-in optimistic updates with rollback for cart operations
- v5 has improved TypeScript support and smaller bundle
- SWR is simpler but lacks the mutation + optimistic update patterns needed for cart operations

---

## ADR-006 | Cloudinary over Vercel Blob / S3 + CloudFront

**Decision:** Use Cloudinary for image storage and CDN delivery.

**Reasoning:**
- Automatic WebP/AVIF conversion is critical for Core Web Vitals (LCP)
- On-the-fly responsive image generation (400w, 800w, 1200w) without pre-generating variants
- Generous free tier for MVP (25 credits/month)
- `medusa-file-cloudinary` plugin integrates with Medusa Admin's image upload
- Cloudflare in front of Cloudinary caches edge-delivered images for near-zero origin load

---

## ADR-007 | Paise (Integer) for All Monetary Values

**Decision:** Store and compute all prices as integers in paise.

**Reasoning:**
- Floating-point arithmetic on decimals causes rounding errors in financial calculations
- `₹49,999.00` stored as `4999900` paise is exact
- Standard practice in payment APIs (Razorpay, Stripe both use smallest currency unit)
- Eliminates a whole class of bugs (0.1 + 0.2 ≠ 0.3 in IEEE 754 floats)

**Rule:** Convert paise → INR string only at the rendering layer using `formatPrice()`.

---

## ADR-008 | ISR (60s revalidation) for Product Pages

**Decision:** Use Incremental Static Regeneration with 60-second revalidation for Homepage, PLP, and PDP.

**Alternatives Considered:**
- SSR (per-request): Fresh data but high server load under traffic
- Full SSG: Fast but stale product data and inventory
- CSR: Bad for SEO

**Reasoning:**
- 60-second staleness is acceptable for furniture product pages (prices and stock don't change every second)
- ISR pages serve from CDN cache without hitting PostgreSQL for the vast majority of requests
- On product publish/update, `revalidatePath()` is called to force immediate revalidation
- SEO-friendly: pages are pre-rendered HTML, not client-side rendered

---

## ADR-009 | HTTP-only Cookies for Auth Tokens

**Decision:** Store JWT auth tokens exclusively in HTTP-only cookies.

**Alternatives Considered:**
- localStorage
- sessionStorage
- Memory-only (Zustand)

**Reasoning:**
- HTTP-only cookies are inaccessible to JavaScript, preventing XSS token theft
- `Secure` flag ensures cookies only sent over HTTPS
- `SameSite=Strict` prevents CSRF attacks
- localStorage and sessionStorage are accessible to any JavaScript on the page — including injected scripts

---

## ADR-010 | pnpm + Turborepo Monorepo

**Decision:** Use pnpm workspaces with Turborepo for the monorepo.

**Alternatives Considered:**
- npm workspaces (no task runner)
- Yarn workspaces + Lerna
- Nx

**Reasoning:**
- pnpm has the best disk efficiency for monorepos (hard links vs copies)
- Turborepo provides intelligent build caching; `next build` is only re-run when relevant files change
- Turborepo is simpler to configure than Nx for this project size
- Vercel (the deployment target) has first-class Turborepo support

---

## ADR-011 | Algolia (Phase 1) → MeiliSearch (Phase 2)

**Decision:** Start with Algolia free tier; migrate to self-hosted MeiliSearch on Hetzner VPS in Phase 2.

**Reasoning:**
- Algolia free tier (10K records + 10K ops/month) is sufficient for MVP catalogue size
- Algolia InstantSearch React library provides typo-tolerance, faceting, and instant results out-of-the-box
- Once on VPS, MeiliSearch provides equivalent features with no per-operation costs
- Medusa's search module makes switching providers a configuration change, not a rewrite

---

*Shree Furniture | v1.0 — Q1 2026*
