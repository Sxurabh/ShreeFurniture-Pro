# AI Guardrails

---

## Single Source of Truth — Always Check These

| What | Where |
|------|-------|
| **Database schema** | `/docs/architecture/database-schema.md` + `/supabase/migrations/` |
| **API contracts & server actions** | `/docs/architecture/api-contracts.md` |
| **TypeScript types** | `/src/types/database.ts` ← auto-generated, never hand-edit |
| **Zod validation schemas** | `/src/lib/validations/` |
| **Environment variables** | `/.env.example` |
| **Resolved arch decisions** | `/docs/ai-context/decisions.md` |

---

## NEVER

| Rule | Why |
|------|-----|
| ❌ Invent database fields | Breaks migrations and type safety |
| ❌ Bypass RLS security | Exposes customer data |
| ❌ Create APIs not defined in api-contracts.md | Breaks contract with frontend |
| ❌ Move business logic to client components | Security and testability risk |
| ❌ Use `any` TypeScript type without justification | Breaks strict mode contract |
| ❌ Prefix server-only secrets with `NEXT_PUBLIC_` | Exposes secrets in client bundle |
| ❌ Edit `src/types/database.ts` manually | Will be overwritten by `supabase gen types` |
| ❌ Add `console.log` to production code | Use structured logging or Sentry |
| ❌ Hardcode values that belong in env vars | Breaks multi-environment setup |

---

## ALWAYS

| Rule | Why |
|------|-----|
| ✅ Validate all inputs with Zod on the server | Defense in depth |
| ✅ Re-validate auth in every Server Action | Middleware alone is not sufficient |
| ✅ Check role from `profiles.role` for admin actions | Defense in depth beyond RLS |
| ✅ Run `revalidatePath` after mutations | Keeps ISR cache fresh |
| ✅ Handle loading and error states in every component | UX and resilience |
| ✅ Use `supabase gen types` after every schema change | Keeps types in sync |
| ✅ Follow file naming conventions (kebab-case) | Consistency |
| ✅ Write Server Actions for all mutations | See decisions.md |
| ✅ Snapshot product name and price into `order_items` | Price changes after order must not affect history |

---

## Supabase Client Usage Rules

| Context | Client to Use | File |
|---------|--------------|------|
| React Server Components (RSC) | `createServerClient` | `/src/lib/supabase/server.ts` |
| Server Actions | `createServerClient` | `/src/lib/supabase/server.ts` |
| Next.js Middleware | `createMiddlewareClient` | `/src/lib/supabase/middleware.ts` |
| Client Components (browser) | `createBrowserClient` | `/src/lib/supabase/client.ts` |
| Edge Functions (Deno) | `createClient` from `@supabase/supabase-js` | inside function |

Never use the browser client in Server Components or Server Actions.