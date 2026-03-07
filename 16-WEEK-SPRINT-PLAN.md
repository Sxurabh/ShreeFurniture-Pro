# 📅 Shree Furniture V1 — 16-Week Sprint Schedule

**Pace:** ~9-11 hours/week (6h weekends + 1h weekdays).
**Rule of Thumb:** Use weekday 1-hour sessions for running single component prompts, code reviews, and minor styling. Use 6-hour weekend blocks for heavy setups (Infra, DB migrations, Checkout flow, complex UI layouts).

---

## Phase 1: Foundation (Weeks 1–4)
*Goal: Monorepo running, Medusa backend live on Railway, database seeded, integrations connected.*

### Sprint 1 (Week 1): Monorepo & Local Infra
*   **Prompt 1:** Scaffold the Full Monorepo (pnpm workspace, Turborepo, Next.js, Medusa v2 folders).
*   **Prompt 2:** Set Up Root Layout (Fonts, Providers, Global CSS).
*   **Prompt 3:** Set Up Route Group Layouts (`(store)`, `(checkout)`, `(account)`).
*   **Prompt 4:** Set Up Docker Compose for Local Dev (PostgreSQL, Redis).

### Sprint 2 (Week 2): CI/CD & Backend Config
*   **Prompt 5:** Set Up GitHub Repository + Branch Protection.
*   **Prompt 6:** Configure `next.config.ts`, ESLint, Tailwind v4.
*   **Prompt 7:** Set Up CI Pipeline (GitHub Actions).
*   **Prompt 8:** Configure MedusaJS v2 Backend plugins (Razorpay, Cloudinary, Resend, Algolia).

### Sprint 3 (Week 3): Database & Seeding
*   **Prompt 9:** Create Seed Script (India region, INR currency, 4 collections, sample products).
*   **Prompt 10:** Run Migrations and Verify Backend (Smoke tests, Medusa Admin test).
*   *Buffer:* Resolve any database migration or Docker issues. 

### Sprint 4 (Week 4): Third-Party Integrations
*   **Prompt 11:** Configure Algolia Index & initial sync script.
*   **Prompt 12:** Configure Cloudinary upload presets.
*   **Prompt 13:** Set Up Resend Email and DNS records.
*   **Prompts 14 & 15:** Instrument PostHog Analytics and Sentry Error tracking.

---

## Phase 2: Storefront Core (Weeks 5–8)
*Goal: The full customer-facing storefront (browsing, search, product pages).*

### Sprint 5 (Week 5): Data Layer & Shared Hooks
*   **Prompts 16 & 17:** Build typed Medusa v2 API Client & Algolia Client.
*   **Prompt 18:** Build Price & Date Utilities (`formatPrice` for paise, etc.).
*   **Prompt 19:** Build Zustand Stores (`cart-store` with optimistic updates, `ui-store`).
*   **Prompts 20 & 21:** Next.js Cart Cookie Middleware & TanStack Query Hooks.

### Sprint 6 (Week 6): Global UI Components
*   **Prompt 22:** Build Header (Sticky nav, mobile hamburger, cart badge).
*   **Prompt 23:** Build `ProductCard` component (Cloudinary images, INR formatting, hover lift).
*   **Prompts 24 & 25:** Build `PriceDisplay`, `SkeletonCard`, `ErrorBoundary`, and `EmptyState`.
*   **Prompts 26, 27, 28:** Build Footer, `MobileMenu` slide-out, and `Breadcrumb`.

### Sprint 7 (Week 7): Homepage & PLP
*   **Prompt 29:** Build the Homepage (Hero, Featured Collections, Best Sellers, Trust Signals).
*   **Prompt 30:** Build PLP - Product Listing Page (Grid, Filters, Sorting, ISR 60s).
*   *Visual Review Checkpoint:* Ensure typography and `#F5F0E8` warm backgrounds look premium.

### Sprint 8 (Week 8): Product Detail Page (PDP) & Search
*   **Prompt 31:** Build PDP (Gallery, variant swatches, sticky Add to Cart, JSON-LD Schema).
*   **Prompt 32:** Build Algolia Search Feature (Overlay, autocomplete, full results page).
*   **Prompt 33:** Build Cart Page (CSR, item list, CartSummary).

---

## Phase 3: Transactions (Weeks 9–13)
*Goal: Full checkout flow, Razorpay payments, auth, accounts, and backend workflows.*

### Sprint 9 (Week 9): Cart Drawer & Checkout Infra
*   **Prompt 34:** Build `CartDrawer`, `CartItem`, and `CartSummary` components.
*   **Prompt 35:** Build `CheckoutProgress` indicator.
*   **Prompt 36:** Build Checkout Step 1 — Address Form (React Hook Form + Zod).
*   **Prompt 37:** Build Checkout Step 2 — Shipping Options (Free shipping logic).

### Sprint 10 (Week 10): Payments & Post-Purchase
*   *(Heavy Weekend Recommended)*
*   **Prompt 38:** Build Checkout Step 3 — Payment (Razorpay sandbox integration).
*   **Prompt 39:** Build Razorpay Webhook Handler (Security: HMAC verify, idempotency).
*   **Prompt 40:** Build Order Confirmation Page (SSR, analytic snippets).

### Sprint 11 (Week 11): Auth & Account Pages
*   **Prompt 41:** Build Auth Pages (Login, Register, Forgot Password).
*   **Prompt 42:** Build Account Layout (Sidebar + Auth Guard).
*   **Prompt 43:** Build "My Orders" list and detail pages.
*   **Prompts 44 & 45:** Build Wishlist Page, Addresses, and Settings Pages.

### Sprint 12 (Week 12): Backend Workflows & Subscribers
*   **Prompts 46 & 47:** Build Order Confirmation & Order Shipped Email Subscribers (React Email).
*   **Prompt 48:** Build Algolia Sync Subscriber (Upserts on publish).
*   **Prompt 49:** Build Low Stock Alert Subscriber.
*   **Prompt 50:** Build Fulfillment Workflow (Validate → Create → Notify).

### Sprint 13 (Week 13): Custom Modules & ISR
*   **Prompt 51:** Build Wishlist Custom Module in Medusa (GET/POST/DELETE API).
*   **Prompt 52:** Build ISR Revalidation Endpoint (Webhook for content updates).
*   *Milestone:* Full Sandbox Purchase Flow end-to-end manual test. 

---

## Phase 4: Launch Prep (Weeks 14–16)
*Goal: SEO, Performance, Security Hardening, and Production Deployment.*

### Sprint 14 (Week 14): SEO, Analytics & Security
*   **Prompts 53 & 54:** Add `robots.txt`, `sitemap.xml`, and verify all SEO Meta Tags/JSON-LD.
*   **Prompt 55:** Verify PostHog Events End-to-End (`add_to_cart`, `order_complete`).
*   **Prompt 56:** Verify Sentry is receiving errors and maps source correctly.
*   **Prompts 57, 58, 59:** Security Hardening (npm Audit, Git Secret Scan, verify HTTP-only cookies).

### Sprint 15 (Week 15): Performance & QA
*   **Prompt 60:** Run Lighthouse Audit & Fix Failures (Target LCP < 2.5s, Score > 90).
*   *Testing Catch-up:* Run Playwright E2E checkout tests and Vitest unit tests (Price logic, Razorpay HMAC).
*   *Buffer:* Final UI polish and addressing any items in `KNOWN-ISSUES.md`.

### Sprint 16 (Week 16): Go-Live!
*   *(Heavy Weekend Recommended)*
*   **Prompt 61:** Configure Cloudflare DNS + Custom Domain.
*   **Prompt 62:** Configure/Deploy Vercel Production Environment (Storefront).
*   **Prompt 63:** Configure/Deploy Railway Production Environment (Backend).
*   **Prompt 64:** Verify Razorpay Webhook End-to-End in Production (Live Mode test).
*   **Prompt 65:** Pre-Launch Definition of Done Checklist.
*   **MERGE TO MAIN & LAUNCH 🚀** 

---

## Phase 5: Post-Launch (Month 5+)
*Goal: Cost optimization. Migrate from serverless free-tiers to a dedicated, low-cost VPS once traffic grows.*

### Sprint 17+ (Ongoing): VPS Migration
*   **Step 1:** Provision Hetzner CX31 (or similar) Ubuntu 22.04 VPS.
*   **Step 2:** Secure Server (UFW Firewall, Certbot SSL).
*   **Step 3:** Deploy Apps (Install Node, PM2, and Nginx reverse proxy).
*   **Step 4:** Migrate Databases if needed, or keep external (Neon/Upstash).
*   **Step 5:** Point Cloudflare DNS to new VPS IP.
*   *Reference:* Follow `NewDocs/11-environment-and-devops.md` Section 7.
