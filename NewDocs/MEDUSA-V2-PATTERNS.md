# MEDUSA-V2-PATTERNS.md — MedusaJS v2 Reference
## Shree Furniture | E-Commerce Platform

> **AI Agent Instruction:** Read this before writing ANY code that touches the MedusaJS backend
> or calls Medusa APIs from the storefront. MedusaJS v2 is a breaking change from v1.
> Most AI training data contains v1 patterns. Using v1 patterns here will break silently.

---

## ⚠️ Critical: v1 vs v2 API Differences

The single most dangerous thing an AI agent will do on this project is use MedusaJS v1 SDK patterns.
Here is the clearest side-by-side reference:

### Storefront Client Initialization

```typescript
// ❌ WRONG — MedusaJS v1 (do not use)
import Medusa from "@medusajs/medusa-js"
const medusa = new Medusa({ baseUrl: "http://localhost:9000" })

// ✅ CORRECT — MedusaJS v2
import Medusa from "@medusajs/js-sdk"
const medusa = new Medusa({
  baseUrl: process.env.NEXT_PUBLIC_MEDUSA_BACKEND_URL!,
  auth: { type: "session" },  // uses HTTP-only cookies — mandatory for this project
})
```

### Fetching Products

```typescript
// ❌ WRONG — MedusaJS v1
const { products } = await medusa.products.list({ limit: 12 })

// ✅ CORRECT — MedusaJS v2
const { products } = await medusa.store.product.list({ limit: 12 })
```

### Cart Operations

```typescript
// ❌ WRONG — v1
await medusa.carts.create()
await medusa.carts.lineItems.create(cartId, { variant_id, quantity })
await medusa.carts.complete(cartId)

// ✅ CORRECT — v2
await medusa.store.cart.create({})
await medusa.store.cart.createLineItem(cartId, { variant_id, quantity })
await medusa.store.cart.complete(cartId)
```

### Customer Auth

```typescript
// ❌ WRONG — v1
await medusa.auth.authenticate({ email, password })
await medusa.customers.retrieve()

// ✅ CORRECT — v2 (email/password login)
await medusa.auth.login("customer", "emailpass", { email, password })
await medusa.store.customer.retrieve()
```

### Retrieving a Single Product

```typescript
// ❌ WRONG — v1
const { product } = await medusa.products.retrieve(id)

// ✅ CORRECT — v2 (by handle, used for SEO-friendly pages)
const { products } = await medusa.store.product.list({ handle })
const product = products[0]
```

---

## Backend: v2 Module System

MedusaJS v2 uses a **module-based architecture** instead of the service-repository pattern from v1.

### Registering a Custom Module (e.g., Wishlist)

```typescript
// backend/medusa-config.ts
import { defineConfig } from "@medusajs/framework/utils"

export default defineConfig({
  projectConfig: {
    databaseUrl: process.env.DATABASE_URL,
    redisUrl: process.env.REDIS_URL,
  },
  modules: [
    {
      resolve: "./src/modules/wishlist",  // path to your module
    },
  ],
})
```

### Defining a Custom Module

```typescript
// backend/src/modules/wishlist/index.ts
import { Module } from "@medusajs/framework/utils"
import WishlistService from "./service"

export const WISHLIST_MODULE = "wishlist"

export default Module(WISHLIST_MODULE, {
  service: WishlistService,
})
```

### Writing a Subscriber (v2 Pattern)

```typescript
// ❌ WRONG — v1 event subscriber pattern
class OrderSubscriber {
  constructor({ eventBusService, orderService }) {
    eventBusService.subscribe("order.placed", this.handleOrderPlaced)
  }
}

// ✅ CORRECT — v2 subscriber
import { SubscriberConfig, SubscriberArgs } from "@medusajs/framework"

export default async function orderPlacedHandler({
  event: { data },
  container,
}: SubscriberArgs<{ id: string }>) {
  const orderId = data.id
  // Use container.resolve() to get services
  const orderService = container.resolve("orderService")
  const order = await orderService.retrieveOrder(orderId, {
    relations: ["items", "shipping_address"],
  })
  // ... send email via Resend
}

export const config: SubscriberConfig = {
  event: "order.placed",
}
```

### Writing a Workflow (v2 Pattern)

```typescript
// backend/src/workflows/fulfillment-workflow.ts
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

const fulfillOrderStep = createStep(
  "fulfill-order-step",
  async (input: { orderId: string }, { container }) => {
    // step logic
    return new StepResponse({ fulfilled: true })
  }
)

export const fulfillmentWorkflow = createWorkflow(
  "fulfillment-workflow",
  function (input: { orderId: string }) {
    const result = fulfillOrderStep(input)
    return new WorkflowResponse(result)
  }
)
```

---

## Custom API Routes (v2 Pattern)

```typescript
// ❌ WRONG — v1 Express-style route
router.post("/store/wishlist", authenticate(), async (req, res) => { ... })

// ✅ CORRECT — v2 route file
// backend/src/api/store/wishlist/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(req: MedusaRequest, res: MedusaResponse) {
  const wishlistService = req.scope.resolve("wishlist")
  const items = await wishlistService.listItems(req.auth_context.actor_id)
  res.json({ items })
}

export async function POST(req: MedusaRequest, res: MedusaResponse) {
  const { variant_id } = req.body
  const wishlistService = req.scope.resolve("wishlist")
  const item = await wishlistService.addItem({
    customer_id: req.auth_context.actor_id,
    variant_id,
  })
  res.status(201).json({ item })
}
```

---

## Storefront API Layer — Project Convention

All Medusa API calls in the storefront go through typed wrappers in `lib/medusa/`.
**Never call `medusa.store.*` directly from a component or page.**

```typescript
// ✅ CORRECT — how to add a new API call
// apps/storefront/lib/medusa/products.ts

import { medusaClient } from "./client"
import type { Product } from "@shree/types"

export async function getProduct(handle: string): Promise<Product | null> {
  try {
    const { products } = await medusaClient.store.product.list({
      handle,
      fields: "+variants.prices,+images,+collection",
    })
    return (products[0] as unknown as Product) ?? null
  } catch {
    return null
  }
}
```

---

## ISR + Revalidation Pattern

```typescript
// apps/storefront/app/(store)/products/[handle]/page.tsx

import { getProduct } from "@/lib/medusa/products"

// ISR: revalidate every 60 seconds
export const revalidate = 60

export default async function ProductPage({
  params,
}: {
  params: { handle: string }
}) {
  const product = await getProduct(params.handle)
  if (!product) notFound()
  return <ProductDetail product={product} />
}
```

```typescript
// backend/src/subscribers/algolia-sync.ts
// After syncing to Algolia, also revalidate Next.js ISR cache
import { revalidatePath } from "next/cache"  // ← server action or separate endpoint

// In production: call the storefront's revalidation API endpoint
// POST /api/revalidate?secret=REVALIDATION_SECRET&path=/products/{handle}
await fetch(`${process.env.STOREFRONT_URL}/api/revalidate`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    secret: process.env.REVALIDATION_SECRET,
    path: `/products/${product.handle}`,
  }),
})
```

---

## Environment Variables — Medusa Specific

```bash
# .env (backend)
DATABASE_URL=postgresql://...                 # Never NEXT_PUBLIC_
REDIS_URL=redis://...                         # Never NEXT_PUBLIC_
MEDUSA_ADMIN_SECRET=...                       # Never NEXT_PUBLIC_
JWT_SECRET=...                                # Never NEXT_PUBLIC_
COOKIE_SECRET=...                             # Never NEXT_PUBLIC_

# .env.local (storefront)
NEXT_PUBLIC_MEDUSA_BACKEND_URL=https://...    # Safe: backend URL only, no secrets
REVALIDATION_SECRET=...                       # Server-only, never NEXT_PUBLIC_
```

---

*Shree Furniture | v1.0 — Q1 2026*
