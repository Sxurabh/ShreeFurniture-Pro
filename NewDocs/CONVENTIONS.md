# CONVENTIONS.md — Shree Furniture Platform
## Naming, Coding & Folder Conventions

---

## 1. File & Folder Naming

| Context | Convention | Example |
|---|---|---|
| React components | PascalCase | `ProductCard.tsx`, `CartDrawer.tsx` |
| Utility functions | camelCase | `formatPrice.ts`, `validators.ts` |
| Zustand stores | kebab-case | `cart-store.ts`, `ui-store.ts` |
| TanStack Query hooks | camelCase with `use` prefix | `useCart.ts`, `useProducts.ts` |
| Next.js pages | lowercase (`page.tsx`) | `app/(store)/page.tsx` |
| Test files | `.test.ts` / `.spec.ts` suffix | `price.test.ts`, `purchase-flow.spec.ts` |
| Type files | camelCase | `index.ts` in `packages/types/` |

---

## 2. TypeScript Conventions

```typescript
// ✅ Use explicit return types on exported functions
export function formatPrice(paise: number): string { ... }

// ✅ Use Zod for all runtime validation; infer TS types from schemas
const addressSchema = z.object({ ... })
type AddressInput = z.infer<typeof addressSchema>

// ✅ Use type imports for type-only imports
import type { Product } from '@shree/types'

// ❌ Never use `any`
const price: any = ... // FORBIDDEN without explicit comment

// ✅ Use unknown + type narrowing when type is genuinely unknown
const raw: unknown = JSON.parse(body)
```

---

## 3. Component Conventions

```tsx
// ✅ Always use named exports for components (not default)
export function ProductCard({ product }: { product: Product }) { ... }

// ✅ Define props with an interface when > 2 props
interface ProductCardProps {
  product: Product
  showWishlist?: boolean
  onAddToCart?: (variantId: string) => void
}

// ✅ Use `data-testid` attributes for E2E test hooks
<button data-testid="add-to-cart" onClick={handleAdd}>Add to Cart</button>

// ✅ Use Next.js <Image> for all images (never <img> for product images)
<Image src={product.thumbnail} width={400} height={400} alt={product.title} />

// ✅ Explicit width + height ALWAYS on <Image> to prevent CLS
```

---

## 4. Price & Money Conventions

```typescript
// All prices are stored and passed as paise (integer)
// ₹49,999 = 4999900 paise

// ✅ Format for display only at the render layer
<PriceDisplay amount={variant.prices[0].amount} />

// ❌ Never do arithmetic on formatted strings
const total = "₹49,999" + "₹5,000" // FORBIDDEN

// ✅ Do arithmetic on raw paise integers
const total = lineItem.unit_price * lineItem.quantity
```

---

## 5. API Call Conventions

```typescript
// ✅ All Medusa API calls go through lib/medusa/ wrappers (never raw fetch in components)
import { getProduct } from '@/lib/medusa/products'

// ✅ Server Components fetch directly (no TanStack Query)
// app/(store)/products/[handle]/page.tsx
const product = await getProduct(handle)

// ✅ Client Components use TanStack Query hooks
// components/product/ProductVariants.tsx
const { data: cart } = useCart()
```

---

## 6. Styling Conventions

```tsx
// ✅ Tailwind utility classes — always use design token values from packages/ui
// ❌ Never hardcode colour hex values in className

// ✅ Use shadcn/ui components as base; extend with Tailwind
import { Button } from '@/components/ui/button'
<Button variant="default" size="lg">Add to Cart</Button>

// ✅ Mobile-first breakpoints: base (mobile) → sm → md → lg → xl
<div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
```

---

## 7. Environment Variable Conventions

```bash
# ✅ NEXT_PUBLIC_ prefix: only for values safe to expose to the browser
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_live_...   # Key ID is public (Razorpay docs confirm)
NEXT_PUBLIC_ALGOLIA_APP_ID=...              # App ID is public

# ✅ Server-only (no NEXT_PUBLIC_): anything that grants write access or is a secret
RAZORPAY_WEBHOOK_SECRET=...                 # Secret — NEVER expose to browser
RAZORPAY_SECRET=...                         # API secret — NEVER expose to browser
DATABASE_URL=...                            # NEVER expose to browser
```

---

## 8. Git Conventions

```bash
# Commit message format: type(scope): description
feat(cart): add optimistic item removal
fix(checkout): prevent duplicate webhook processing  
chore(deps): update medusa to v2.x.x
docs(api): update cart endpoints
test(price): add edge cases for GST calculation
```

**Branch naming:**
- `feat/cart-drawer` — new features
- `fix/razorpay-webhook-idempotency` — bug fixes
- `chore/update-deps` — maintenance

---

## 9. Import Order

```typescript
// 1. Node built-ins (if any)
import { createHmac } from 'crypto'

// 2. External packages
import { z } from 'zod'
import type { NextRequest } from 'next/server'

// 3. Internal packages (workspace)
import type { Product } from '@shree/types'

// 4. App-level aliases
import { medusaClient } from '@/lib/medusa/client'
import { useCartStore } from '@/store/cart-store'

// 5. Relative imports
import { ProductCard } from './ProductCard'
```

---

*Shree Furniture | v1.0 — Q1 2026*
