# Project Structure

> **MVP Decision:** Single Next.js application with a flat `/src` structure. No multi-package monorepo for MVP.
> A monorepo (Turborepo) is documented in the future scalability roadmap if a mobile app is added later.

---

## Root Directory

```
shree-furniture/
├── src/                          # All application source code
├── supabase/                     # Supabase config, migrations, edge functions
├── emails/                       # React Email templates
├── public/                       # Static assets (icons, og images)
├── docs/                         # Project documentation
├── .env.example                  # Environment variable template
├── .env.local                    # Local secrets (never committed)
├── next.config.ts
├── tailwind.config.ts
├── components.json               # Shadcn/ui config
├── tsconfig.json
└── package.json
```

---

## /src Structure

```
src/
├── app/                          # Next.js App Router
│   ├── (store)/                  # Storefront route group
│   │   ├── page.tsx              # / — Homepage
│   │   ├── products/
│   │   │   ├── page.tsx          # /products — PLP
│   │   │   └── [slug]/page.tsx   # /products/[slug] — PDP
│   │   ├── search/page.tsx       # /search
│   │   ├── cart/page.tsx         # /cart
│   │   ├── checkout/
│   │   │   ├── page.tsx
│   │   │   └── confirmation/[orderId]/page.tsx
│   │   └── account/              # Auth-protected via middleware
│   │       ├── layout.tsx
│   │       ├── page.tsx          # Profile
│   │       ├── orders/
│   │       │   ├── page.tsx
│   │       │   └── [id]/page.tsx
│   │       └── wishlist/page.tsx
│   │
│   ├── admin/                    # Admin panel — admin role required
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Dashboard
│   │   ├── products/
│   │   │   ├── page.tsx
│   │   │   ├── new/page.tsx
│   │   │   └── [id]/edit/page.tsx
│   │   ├── orders/
│   │   │   ├── page.tsx
│   │   │   └── [id]/page.tsx
│   │   ├── categories/page.tsx
│   │   ├── customers/page.tsx
│   │   ├── promotions/page.tsx
│   │   └── inventory/page.tsx
│   │
│   ├── auth/
│   │   ├── login/page.tsx
│   │   ├── signup/page.tsx
│   │   └── callback/route.ts     # Google OAuth callback
│   │
│   ├── api/
│   │   └── webhooks/
│   │       └── razorpay/route.ts # ONLY route handler in the app
│   │
│   ├── layout.tsx                # Root layout — Providers, fonts
│   ├── not-found.tsx             # 404 page
│   └── error.tsx                 # Error boundary
│
├── components/
│   ├── ui/                       # Shadcn/ui base components (auto-generated, don't modify)
│   ├── layout/
│   │   ├── header/
│   │   │   ├── header.tsx
│   │   │   ├── nav-menu.tsx
│   │   │   └── cart-icon.tsx
│   │   ├── footer/
│   │   │   └── footer.tsx
│   │   └── admin-sidebar/
│   │       └── admin-sidebar.tsx
│   ├── products/
│   │   ├── product-card.tsx
│   │   ├── product-grid.tsx
│   │   ├── product-image-gallery.tsx
│   │   ├── product-filters.tsx
│   │   ├── product-variant-selector.tsx
│   │   └── add-to-cart-button.tsx
│   ├── cart/
│   │   ├── cart-drawer.tsx
│   │   ├── cart-item.tsx
│   │   └── cart-summary.tsx
│   ├── checkout/
│   │   ├── address-form.tsx
│   │   ├── order-summary.tsx
│   │   ├── coupon-input.tsx
│   │   └── razorpay-button.tsx
│   ├── home/
│   │   ├── hero-banner.tsx
│   │   ├── featured-products.tsx
│   │   └── category-grid.tsx
│   └── admin/
│       ├── product-form.tsx
│       ├── image-uploader.tsx
│       ├── orders-table.tsx
│       └── revenue-chart.tsx
│
├── actions/                      # Next.js Server Actions (server-side only)
│   ├── auth.ts
│   ├── cart.ts
│   ├── checkout.ts
│   ├── profile.ts
│   └── admin/
│       ├── products.ts
│       ├── orders.ts
│       └── promotions.ts
│
├── stores/                       # Zustand client stores
│   ├── cart-store.ts
│   ├── wishlist-store.ts
│   └── ui-store.ts
│
├── hooks/                        # Custom React hooks
│   ├── use-cart.ts
│   ├── use-wishlist.ts
│   ├── use-products.ts
│   └── use-debounce.ts
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts             # Browser client (createBrowserClient)
│   │   ├── server.ts             # RSC / Server Action client (createServerClient)
│   │   └── middleware.ts         # Middleware client (createMiddlewareClient)
│   ├── validations/              # Zod schemas — shared client + server
│   │   ├── product.ts
│   │   ├── checkout.ts
│   │   ├── auth.ts
│   │   └── coupon.ts
│   ├── razorpay.ts               # Razorpay client utilities
│   └── utils.ts                  # cn(), formatCurrency(), etc.
│
├── types/
│   ├── database.ts               # AUTO-GENERATED — run: supabase gen types typescript
│   ├── actions.ts                # ActionResult<T> and other shared action types
│   └── index.ts                  # Re-exports and app-level types
│
└── middleware.ts                 # Next.js Edge Middleware — auth guard
```

---

## /supabase Structure

```
supabase/
├── functions/
│   ├── create-razorpay-order/
│   │   └── index.ts
│   ├── verify-razorpay-payment/
│   │   └── index.ts
│   └── send-order-email/
│       └── index.ts
├── migrations/                   # Versioned SQL — never edit existing files
│   ├── 20260201000001_create_profiles.sql
│   ├── 20260201000002_create_categories.sql
│   ├── 20260201000003_create_products.sql
│   ├── 20260201000004_create_cart_orders.sql
│   ├── 20260201000005_create_promotions.sql
│   └── 20260201000006_create_indexes_rls.sql
└── seed.sql                      # Dev seed data (not run in production)
```

---

## /emails Structure

```
emails/
├── order-confirmation.tsx        # React Email template
└── shipping-update.tsx           # React Email template
```

---

## Key Rules

- **kebab-case** for all file and folder names
- **No barrel files** (`index.ts` re-exports) except in `/src/types/`
- **Never add files to `/src/types/database.ts`** — auto-generated
- **Never add route handlers** except `/api/webhooks/razorpay/route.ts`
- **All mutations go in `/src/actions/`** — not in components or API routes
- **Shadcn/ui components** go in `/src/components/ui/` and are not manually edited

---

## Future Scalability (Post-MVP)

If a React Native mobile app is added, the project can be restructured into a Turborepo monorepo:

```
apps/
  web/          ← current Next.js app
  mobile/       ← future React Native / Expo app
packages/
  types/        ← shared TypeScript types
  validations/  ← shared Zod schemas
  utils/        ← shared utilities
```

The Supabase backend (DB, Auth, Edge Functions) is already mobile-compatible with no changes needed.