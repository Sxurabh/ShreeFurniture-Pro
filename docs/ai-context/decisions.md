# Resolved Decisions

These decisions are **final**. Do not re-litigate, suggest alternatives, or deviate from them unless explicitly instructed by the project owner.

---

## Architecture

| Decision | Choice | Rejected Alternatives |
|----------|--------|-----------------------|
| **Rendering** | RSC by default, Client Components only for interactivity | Pure CSR, pages router |
| **Mutations** | Next.js Server Actions only | REST API routes, GraphQL, tRPC |
| **External webhooks** | Route Handlers at `/api/webhooks/` only | Server Actions for webhooks |
| **Payment operations** | Supabase Edge Functions (service role isolated) | Server Actions for Razorpay secret ops |
| **Admin panel** | Same Next.js app under `/admin` route group | Separate Next.js app, separate deployment |
| **Monorepo structure** | Single Next.js app in `/src` (not multi-package monorepo for MVP) | Turborepo, nx |

---

## Database

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Shipping address on orders** | JSONB snapshot in `orders.shipping_address` | Address must not change retroactively when user edits profile |
| **Product name/price on order items** | Snapshot into `order_items.product_name` and `order_items.unit_price` | Price changes must not alter historical orders |
| **Product images** | Separate `product_images` table (not `text[]` array in variants) | Allows ordering, alt text, primary flag |
| **Profile fields** | `full_name` as single field (not `first_name` / `last_name`) | Simpler, consistent with Supabase Auth pattern |
| **Cart persistence** | Zustand (localStorage for guests) + Supabase `cart_items` for logged-in users | Cross-device sync for auth users, zero-friction for guests |
| **Cart merge on login** | Merge localStorage cart into Supabase cart on login (combine quantities) | UX: don't lose guest cart on login |
| **Auth** | Supabase Auth with cookie-based sessions | SSR-compatible; no JWT in localStorage |
| **Admin role** | `profiles.role` column with value `'admin'` | Simple, RLS-enforceable |

---

## State Management

| Decision | Choice | Rejected Alternatives |
|----------|--------|-----------------------|
| **Client state** | Zustand | Redux, Context API, Jotai |
| **Server state / caching** | TanStack Query | SWR, manual useState+useEffect |

---

## Payments

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Payment gateway** | Razorpay only | Indian market, UPI support |
| **Order creation timing** | Only after server-side HMAC signature verification | Never create order on client payment success alone |
| **Razorpay order creation** | Supabase Edge Function (server-side) | Razorpay secret never reaches client or Next.js bundle |
| **Amount tampering prevention** | Amount set server-side at order creation, not passed from client | Client cannot manipulate price |

---

## Images

| Decision | Choice | Rejected Alternatives |
|----------|--------|-----------------------|
| **Storage** | Supabase Storage | S3, Cloudinary, Uploadthing |
| **Delivery** | Supabase Storage CDN + Next.js `<Image>` optimization | Separate CDN |
| **Upload optimization** | `browser-image-compression` client-side before upload, target < 200KB | Server-side compression |

---

## Email

| Decision | Choice |
|----------|--------|
| **Provider** | Resend |
| **Templates** | React Email |
| **Trigger mechanism** | Supabase Edge Function: `send-order-email` |

---

## Hosting & Infrastructure

| Decision | Choice |
|----------|--------|
| **Frontend hosting** | Vercel Hobby (upgrade triggers documented in architecture doc) |
| **Database** | Supabase Free (upgrade triggers documented) |
| **Currency** | INR only (no multi-currency in MVP) |
| **Language** | English only (no i18n in MVP) |
| **Shipping** | Free flat rate, no carrier API (manual fulfillment) |
| **Tax** | GST included in displayed price (not GST-invoice-compliant in MVP) |

---

## Rendering Strategy by Page Type

| Page | Strategy | Revalidation |
|------|---------|-------------|
| Homepage | ISR | 300s |
| Category / PLP | ISR | 60s |
| Product Detail (PDP) | ISR + on-demand | 60s + `revalidatePath` on admin update |
| Search results | SSR | Per request |
| Cart / Checkout | CSR | N/A |
| Admin pages | SSR (no-cache) | Per request |
| Account pages | SSR | Per request |