# AI Coding Standards

---

## TypeScript

- **Strict mode is required.** No exceptions.
- Never use `any` — use `unknown` and narrow, or generate proper types.
- All Server Actions return `ActionResult<T>` (see type below).
- Types for DB rows come from `/src/types/database.ts` (auto-generated — never edit manually).

### Required tsconfig.json flags

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Standard ActionResult type

```typescript
// /src/types/actions.ts
export type ActionResult<T = void> =
  | { success: true; data: T }
  | { success: false; error: string | Record<string, string[]> }
```

---

## UI & Design System

- **TailwindCSS** for all styling — no inline styles, no CSS modules
- **Shadcn/ui** for all base components (Button, Input, Dialog, Table, etc.)
- **Lucide React** for all icons
- Mobile-first responsive design — design for 375px first
- WCAG 2.1 AA compliance required (contrast ≥ 4.5:1, keyboard navigable, ARIA labels)
- All images use `next/image` — never `<img>` tags

---

## Component Rules

### When to use Server vs Client Components

| Use Server Component (default) | Use Client Component (`'use client'`) |
|-------------------------------|--------------------------------------|
| Data fetching from Supabase | Cart interactions |
| SEO-critical content | Filter UI / search input |
| Static layout and typography | Product variant selector |
| Admin data tables (initial render) | Image gallery with swipe |
| | Forms (React Hook Form) |
| | Zustand store consumers |

### Component file structure

```typescript
// Server Component example
// app/(store)/products/[slug]/page.tsx
import { createServerClient } from '@/lib/supabase/server'

export default async function ProductPage({ params }: { params: { slug: string } }) {
  const supabase = createServerClient()
  const { data: product } = await supabase
    .from('products')
    .select('...')
    .eq('slug', params.slug)
    .single()

  if (!product) notFound()

  return <ProductDetail product={product} />
}
```

```typescript
// Client Component example
'use client'

import { useCartStore } from '@/stores/cartStore'

export function AddToCartButton({ product }: { product: Product }) {
  const addItem = useCartStore((s) => s.addItem)
  return <Button onClick={() => addItem(product)}>Add to Cart</Button>
}
```

---

## Server Actions

Every Server Action must:

1. Call `createServerClient()` — not browser client
2. Authenticate with `supabase.auth.getUser()` — never trust middleware alone
3. Validate input with Zod before touching the database
4. Return `ActionResult<T>`
5. Call `revalidatePath()` after mutations

```typescript
'use server'

import { createServerClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import type { ActionResult } from '@/types/actions'

const schema = z.object({
  // define schema
})

export async function myAction(input: unknown): Promise<ActionResult<{ id: string }>> {
  const supabase = createServerClient()

  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return { success: false, error: 'Unauthorized' }

  const parsed = schema.safeParse(input)
  if (!parsed.success) return { success: false, error: parsed.error.flatten().fieldErrors }

  const { data, error } = await supabase.from('table').insert(parsed.data).select('id').single()
  if (error) return { success: false, error: error.message }

  revalidatePath('/relevant-path')
  return { success: true, data }
}
```

---

## Validation

- **Zod schemas live in `/src/lib/validations/`** — shared between client forms and server actions
- React Hook Form + `@hookform/resolvers/zod` for all forms
- Client-side validation for UX feedback only — server-side is the security layer

---

## Data Fetching

| Pattern | When to Use |
|---------|------------|
| Direct Supabase client in RSC | Server Components reading public or user-scoped data |
| TanStack Query (`useQuery`) | Client Components that need background refetch or caching |
| Server Actions | All mutations |
| Supabase Edge Functions | Payment operations, anything needing service role key |

---

## File & Folder Conventions

- `kebab-case` for all file and folder names
- Components in `/src/components/` grouped by feature
- Server Actions in `/src/actions/` grouped by domain
- Zod schemas in `/src/lib/validations/`
- Zustand stores in `/src/stores/`
- Custom hooks in `/src/hooks/`
- Supabase clients in `/src/lib/supabase/`

---

## Styling Conventions

- Tailwind utility classes only — no arbitrary values unless absolutely necessary
- Use `cn()` utility (from `clsx` + `tailwind-merge`) for conditional classes
- Use Shadcn component variants rather than overriding styles directly
- Skeleton loading states for all async content
- Error boundary required on all page-level components

---

## Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Components | PascalCase | `ProductCard.tsx` |
| Hooks | camelCase with `use` prefix | `useCart.ts` |
| Server Actions | camelCase verbs | `createProduct`, `updateOrderStatus` |
| Stores | camelCase with `Store` suffix | `cartStore.ts` |
| Route files | Next.js convention | `page.tsx`, `layout.tsx`, `route.ts` |
| DB tables | snake_case plural | `order_items` |
| DB columns | snake_case | `created_at`, `unit_price` |