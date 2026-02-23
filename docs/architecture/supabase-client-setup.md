# Supabase Client Setup

> This doc defines the three Supabase client patterns used in this project.
> Always use the correct client for the context — using the wrong one is a security risk.

---

## The Three Clients

| Client | File | Used In |
|--------|------|---------|
| Browser client | `src/lib/supabase/client.ts` | Client Components only |
| Server client | `src/lib/supabase/server.ts` | RSC, Server Actions, Route Handlers |
| Middleware client | `src/lib/supabase/middleware.ts` | `src/middleware.ts` only |

---

## Browser Client

```typescript
// src/lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/database'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

**Use in Client Components:**
```typescript
'use client'
import { createClient } from '@/lib/supabase/client'

export function MyClientComponent() {
  const supabase = createClient()
  // ...
}
```

---

## Server Client

```typescript
// src/lib/supabase/server.ts
import { createServerClient as createSSRServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database'

export function createServerClient() {
  const cookieStore = cookies()

  return createSSRServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          cookieStore.set({ name, value, ...options })
        },
        remove(name: string, options: CookieOptions) {
          cookieStore.set({ name, value: '', ...options })
        },
      },
    }
  )
}
```

**Use in Server Components:**
```typescript
// app/(store)/products/page.tsx
import { createServerClient } from '@/lib/supabase/server'

export default async function ProductsPage() {
  const supabase = createServerClient()
  const { data: products } = await supabase.from('products').select('*')
  return <ProductGrid products={products ?? []} />
}
```

**Use in Server Actions:**
```typescript
'use server'
import { createServerClient } from '@/lib/supabase/server'

export async function myAction() {
  const supabase = createServerClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return { success: false, error: 'Unauthorized' }
  // ...
}
```

---

## Middleware Client

```typescript
// src/lib/supabase/middleware.ts
import { createServerClient as createSSRServerClient } from '@supabase/ssr'
import { type NextRequest, NextResponse } from 'next/server'
import type { Database } from '@/types/database'

export function createMiddlewareClient(request: NextRequest) {
  let response = NextResponse.next({ request })

  const supabase = createSSRServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.cookies.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          request.cookies.set({ name, value, ...options })
          response = NextResponse.next({ request })
          response.cookies.set({ name, value, ...options })
        },
        remove(name: string, options: CookieOptions) {
          request.cookies.set({ name, value: '', ...options })
          response = NextResponse.next({ request })
          response.cookies.set({ name, value: '', ...options })
        },
      },
    }
  )

  return { supabase, response }
}
```

**Use in middleware.ts:**
```typescript
// src/middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import { createMiddlewareClient } from '@/lib/supabase/middleware'

export async function middleware(request: NextRequest) {
  const { supabase, response } = createMiddlewareClient(request)

  // Refresh session — IMPORTANT: must be called first
  const { data: { session } } = await supabase.auth.getSession()

  const path = request.nextUrl.pathname

  // Protect /account/* routes
  if (path.startsWith('/account') && !session) {
    return NextResponse.redirect(
      new URL(`/auth/login?redirect=${path}`, request.url)
    )
  }

  // Protect /admin/* routes — check role
  if (path.startsWith('/admin')) {
    if (!session) {
      return NextResponse.redirect(new URL('/auth/login', request.url))
    }

    const { data: profile } = await supabase
      .from('profiles')
      .select('role')
      .eq('id', session.user.id)
      .single()

    if (profile?.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', request.url))
    }
  }

  return response
}

export const config = {
  matcher: ['/account/:path*', '/admin/:path*'],
}
```

---

## Required npm Packages

```bash
npm install @supabase/supabase-js @supabase/ssr
```

---

## Type Generation

After any schema change, regenerate types:

```bash
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > src/types/database.ts
```

**Never edit `src/types/database.ts` manually.** It will be overwritten on the next `gen types` run.