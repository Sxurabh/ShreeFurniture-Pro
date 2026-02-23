# AI Project Context

---

## Project

**Name:** Shree Furniture
**Type:** Furniture e-commerce platform
**Market:** India (INR, UPI-first payments, pan-India shipping)
**Status:** Pre-launch, active development

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript (strict mode) |
| Database | Supabase (PostgreSQL + RLS) |
| Auth | Supabase Auth (email + Google OAuth) |
| Storage | Supabase Storage (product images) |
| Edge Functions | Supabase Edge Functions (Deno) |
| Payments | Razorpay (UPI-first) |
| Email | Resend + React Email |
| UI Components | Shadcn/ui |
| Styling | TailwindCSS |
| State (client) | Zustand |
| State (server) | TanStack Query |
| Forms | React Hook Form + Zod |
| Hosting | Vercel (Hobby plan) |
| Error tracking | Sentry (free tier) |
| Analytics | Vercel Analytics + Speed Insights |

---

## Two User Types

### Customers (storefront)
- Browse and filter furniture catalog
- Add to cart, apply coupons, checkout
- Pay via Razorpay (UPI, cards, wallets)
- Track orders
- Manage wishlist and profile

### Admins (admin panel)
- Manage products, categories, inventory
- Process and update orders
- Create discount codes and banners
- View basic analytics dashboard

---

## Primary Goals (in priority order)

1. **Security** — payment verification, RLS, no secret exposure
2. **Performance** — LCP < 2.5s, Lighthouse ≥ 85 mobile
3. **Clean architecture** — scalable patterns from day one
4. **Fast UX** — mobile-first, < 3s load time

---

## Key Constraints

- Single warehouse inventory (no multi-location in MVP)
- GST included in displayed price (no breakdown invoice in MVP)
- Threshold-based shipping (free above ₹5,000, else ₹299 flat fee; no carrier API in MVP)
- INR only, English only
- Manual fulfillment (admin enters tracking numbers)
- No returns/refund workflow in MVP

---

## Routes Overview

```
/ (store)
├── /                        Homepage
├── /products                Product listing (PLP)
├── /products/[slug]         Product detail (PDP)
├── /search?q=               Search results
├── /cart                    Cart
├── /checkout                Checkout
│   └── /confirmation/[orderId]   Order confirmation
└── /account                 (auth required)
    ├── /                    Profile
    ├── /orders              Order history
    ├── /orders/[id]         Order detail
    └── /wishlist            Wishlist

/admin                       (admin role required)
├── /                        Dashboard
├── /products                Product list
├── /products/new            Create product
├── /products/[id]/edit      Edit product
├── /orders                  Order list
├── /orders/[id]             Order detail
├── /categories              Category management
├── /customers               Customer list
├── /promotions              Banners & coupons
└── /inventory               Stock management

/auth
├── /login
├── /signup
└── /callback                OAuth redirect

/api
└── /webhooks/razorpay       Razorpay webhook (backup)
```