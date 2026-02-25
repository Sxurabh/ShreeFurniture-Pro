# 01 — MVP Tech Document
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Date:** Q1 2026 | **Status:** Active Development

---

## 1. Purpose & Scope

This document defines the Minimum Viable Product (MVP) for Shree Furniture — an IKEA-inspired, India-first e-commerce furniture platform. The MVP scope covers the minimum set of features, technology decisions, and infrastructure choices required to go live with a production-ready storefront, checkout, and order management capability.

---

## 2. MVP Goals

The primary goal of the MVP is to validate market demand, enable real transactions, and establish a scalable technical foundation — all without unnecessary over-engineering. Specifically the MVP must:

- Allow customers to browse furniture products by category and room type
- Allow customers to add items to cart and complete checkout with Indian payment methods (UPI, Cards, EMI)
- Allow the business admin to manage products, inventory, and orders via a dashboard
- Be deployable at zero or near-zero infrastructure cost during the validation phase
- Be architected to scale without a full rewrite when traffic grows

---

## 3. MVP Feature Set

### 3.1 Customer-Facing (Storefront)

| Feature | Priority | Notes |
|---|---|---|
| Homepage with Hero + Featured Products | P0 | ISR cached, Framer Motion animations |
| Product Listing Page (PLP) with filters | P0 | Filter by room, material, price, colour |
| Product Detail Page (PDP) | P0 | Gallery, variants, stock status, Add to Cart |
| Shopping Cart (Drawer/Sidebar) | P0 | Zustand state, persisted via Redis |
| Checkout — Address + Payment | P0 | Multi-step: Address → Shipping → Payment |
| Razorpay Payment Integration | P0 | UPI, Cards, NetBanking, EMI, BNPL |
| Order Confirmation Page | P0 | Order summary + email notification |
| Basic Search | P1 | Algolia free tier |
| User Registration & Login | P1 | JWT via Medusa auth, HTTP-only cookies |
| Order History (My Orders) | P1 | Post-login view |
| Wishlist | P1 | Persisted per user account |
| Responsive Design (Mobile-first) | P0 | Tailwind CSS breakpoints |

### 3.2 Admin-Facing (Back-office)

| Feature | Priority | Notes |
|---|---|---|
| Product CRUD (add/edit/delete) | P0 | Medusa Admin Dashboard |
| Variant & Inventory Management | P0 | Size, colour, stock levels per SKU |
| Order Management (view, fulfill, refund) | P0 | Built-in Medusa Admin |
| Customer Management | P1 | View customer profiles, order history |
| Collection / Category Management | P0 | Room types: Living Room, Bedroom, etc. |

### 3.3 Out of Scope for MVP

The following are explicitly excluded and planned for Phase 2–3:

- AR / 3D Product Viewer (React Three Fiber)
- Loyalty & Referral Program
- Multi-currency / GST-compliant invoicing
- SMS & Push Notifications
- Reviews & Ratings
- Marketing email automation

---

## 4. MVP Technology Choices

### 4.1 Frontend

| Concern | Choice | Rationale |
|---|---|---|
| Framework | Next.js 15 (App Router) | RSC + ISR reduce server load and JS bundle; critical for SEO |
| Language | TypeScript | Type safety in checkout and cart logic |
| Styling | Tailwind CSS v4 + shadcn/ui | Zero runtime overhead, full design ownership |
| Animation | Framer Motion | Micro-interactions on product cards and cart drawer |
| State | Zustand | Minimal boilerplate for cart and wishlist |
| Server State | TanStack Query v5 | Background refetch for product listings |
| Forms | React Hook Form + Zod | Performant validated checkout and auth forms |

### 4.2 Backend

| Concern | Choice | Rationale |
|---|---|---|
| Commerce Engine | MedusaJS v2 | Open-source, self-hostable, no per-transaction fees |
| Runtime | Node.js 20 LTS | Stable LTS; runs MedusaJS natively |
| Database | PostgreSQL 16 | Medusa's primary store for all commerce data |
| Cache & Queue | Redis 7 | Sessions, cart, rate limiting, job queue |

### 4.3 Payments

| Concern | Choice | Rationale |
|---|---|---|
| Gateway | Razorpay | UPI, Cards, EMI, BNPL — essential for high-AOV furniture |
| Integration | `medusa-payment-razorpay` | Webhook-ready, plug-and-play |
| Security | HMAC-SHA256 verification | Signature validation on every payment webhook |

### 4.4 External Services (MVP Free Tiers)

| Service | Tool | Limit |
|---|---|---|
| Image CDN | Cloudinary | 25 credits/month free |
| Transactional Email | Resend + React Email | 3,000 emails/month free |
| Search | Algolia | 10K records + 10K ops free |
| Analytics | PostHog | 1M events/month free |
| Error Tracking | Sentry | 5K errors/month free |

---

## 5. MVP Infrastructure — Phase 1 (₹0/month)

| Service | Platform | Free Allowance |
|---|---|---|
| Next.js Frontend | Vercel | 100GB bandwidth, unlimited deploys |
| MedusaJS Backend | Railway.app | $5 credit/month included |
| PostgreSQL | Neon.tech | 512MB, serverless branching |
| Redis | Upstash | 10,000 commands/day |
| CDN + SSL + DNS | Cloudflare | Free |
| CI/CD | GitHub Actions | 2,000 mins/month |
| **Total** | | **~₹0/month** |

---

## 6. MVP Definition of Done

The MVP is complete and ready for launch when:

- A customer can browse, add to cart, and complete checkout via Razorpay (UPI + Card) without errors
- Order confirmation email is triggered automatically via Resend
- Admin can add a product with images, variants, and pricing in under 5 minutes
- Core Web Vitals pass: LCP < 2.5s, CLS < 0.1, INP < 200ms on mobile (4G simulation)
- Razorpay webhook verification is active and tested end-to-end
- All secrets stored as environment variables, never committed to Git
- Codebase passes TypeScript strict mode, ESLint, and Vitest in CI pipeline

---

## 7. MVP Timeline

| Milestone | Duration | Deliverable |
|---|---|---|
| Project Setup & Monorepo | Week 1 | Repo, CI/CD, Docker dev environment |
| MedusaJS Backend + DB | Week 2 | Products, Cart, Order APIs live |
| Storefront — Home + PLP + PDP | Week 3–4 | Browsable product catalogue |
| Cart + Checkout + Razorpay | Week 5–6 | End-to-end purchase flow |
| Admin Configuration | Week 6 | Products and orders manageable |
| QA, Performance & Security Audit | Week 7 | All DoD criteria met |
| **Launch — Phase 1** | **Week 8** | **Live on Vercel + Railway** |

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| MedusaJS v2 breaking API changes | Medium | High | Pin exact version; monitor changelog |
| Razorpay webhook delivery failure | Low | High | Idempotent order processing + retry config |
| Neon.tech DB size limit exceeded | Medium | Medium | Migrate to VPS PostgreSQL at 80% capacity |
| Vercel cold start latency | Low | Medium | ISR + Edge Middleware for critical routes |
| Cloudinary bandwidth exceeded | Low | Low | Cloudflare caching in front of Cloudinary |

---

*Document Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
