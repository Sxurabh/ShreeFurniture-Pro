# System Architecture

## Components

Client (Next.js)
Server Actions
Supabase (DB, Auth, Storage)
Edge Functions (payment verification)
External Services:
- Razorpay
- Resen d
- Vercel CDN

---

## Service Boundaries

Next.js
- UI & server actions

Supabase
- database & auth

Edge Functions
- payment security

---

## Performance Strategy
- Server components for SEO
- CDN caching
- image optimization
- search indexing

---

## Scalability
- stateless architecture
- edge delivery
- horizontal scaling via Vercel