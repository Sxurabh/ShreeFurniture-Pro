# MY PHASE-BY-PHASE DAILY CODING GUIDE — Shree Furniture
## Roadmap & Milestones | Phase 1 → Phase 4 | Every Prompt Covered
### v1.0 — March 2026 | Aligned with PROMPT-GUIDE-v2.md

> **Who this guide is for & when to use it:**
> - Use this file when you want the **big picture roadmap**: what each phase does, which branch to work on, and which prompts to run in what order.
> - Use `MY-DAILY-CODING-GUIDE.md` for **day-to-day situations** (normal session, rejection, stack change, debugging, PRs); this file tells you *what comes next in the phase*.
> - If anything in this guide ever conflicts with `NewDocs/README.md` or any numbered `NewDocs/0X-*.md` spec, **treat NewDocs as the canonical source of truth**.

---

> **How to use this guide:**
> 1. Find your current phase section below
> 2. Find the next uncompleted prompt (check STATUS.md)
> 3. See the branch to be on, the exact prompt source, what to test, and how to commit
> 4. Work through every prompt in order — never skip
>
> **PROMPT-GUIDE-v2.md is your prompt provider.** This guide tells you WHEN, WHICH, and WHAT TO DO after.

---

## YOUR SESSION LOOP — Every Single Day

```
OPEN THIS GUIDE
    ↓
Find your current phase + next uncompleted prompt
    ↓
git pull + create/switch to the right branch
    ↓
Paste SESSION START prompt (Section 1 of PROMPT-GUIDE-v2.md)
    ↓
Copy build prompt from PROMPT-GUIDE-v2.md → paste → AI generates
    ↓
You test (use the acceptance criteria written inside each prompt)
    ↓
Like it? → commit  |  Don't like it? → rejection flow → rebuild → commit
    ↓
Next prompt in same session OR end of session
    ↓
Paste SESSION END prompt (Section 10 of PROMPT-GUIDE-v2.md) → commit docs → push → PR
```

---

## BEFORE EVERY SESSION — ALWAYS FIRST

### Step 1 — Pull Latest

```bash
git checkout phase/[current-phase]
git pull origin phase/[current-phase]
```

### Step 2 — Paste Session Start Prompt

Open **PROMPT-GUIDE-v2.md → Section 1**.

Use the **"Standard Session Start"** prompt (files 1–5).

> ⚠️ Use this corrected file order — not the order printed in Section 1 of PROMPT-GUIDE-v2:
> `1. STATUS.md → 2. PREFERENCES.md → 3. REJECTIONS.md → 4. STACK-CHANGES.md → 5. CLAUDE.md`

Wait for the summary. Read it. Confirm phase and last completed item match what you expect.

---

## IF YOU WERE INTERRUPTED MID-SESSION

If you had to stop coding mid-session (phone call, power cut, closed laptop):

```bash
git status                    # see what's uncommitted
git stash                     # save uncommitted work temporarily (if needed)
git checkout phase/X-name
git pull origin phase/X-name
git checkout feat/[your-branch]
git stash pop                 # restore your uncommitted work
```

Then paste into Antigravity:

```
Read STATUS.md, PREFERENCES.md, REJECTIONS.md, STACK-CHANGES.md, CLAUDE.md.

I was interrupted mid-session. Tell me:
- What was the last completed item in STATUS.md?
- What was I building when I stopped? (check any uncommitted files)
- What should I do next?
Do not write any code yet.
```

---

## IF YOU STOPPED MID-PROMPT (AI was generating, you had to close)

The AI was building something and you had to close Antigravity without the prompt finishing.

```bash
git status    # were any files actually written?
```

If files were written → inspect them. Are they complete?
- If complete → test them, commit if they pass
- If partial/broken → delete and re-run the prompt from scratch

Paste into Antigravity:
```
The last session was interrupted while building [what you were building].
Read STATUS.md — is [ComponentName] marked as done or in progress?
If in progress: show me the current state of [filepath] and tell me what's missing.
Do not regenerate anything until I confirm.
```

---

---

# ═══════════════════════════════════════
# PHASE 1 — FOUNDATION
# ═══════════════════════════════════════

**Goal:** Monorepo running locally, MedusaJS backend live on Railway, all third-party services connected.
**Duration:** ~2 weeks
**PROMPT-GUIDE-v2 sections:** 2a, 2b, 2c

## Phase 1 Branch Setup

```bash
# Create once at the start of Phase 1:
git checkout develop
git checkout -b phase/1-foundation
git push -u origin phase/1-foundation
```

---

## WEEK 1 — PROMPT-GUIDE-v2 SECTION 2a

---

### PROMPT 1 — Scaffold the Full Monorepo

**Branch:**
```bash
git checkout -b feat/monorepo-scaffold
# Cut from: phase/1-foundation
```

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Scaffold the Full Monorepo"**

**What the AI will create:**
- `pnpm-workspace.yaml`, `turbo.json`, root `package.json`
- `apps/storefront/` (Next.js 15 scaffold)
- `backend/` (MedusaJS v2 scaffold)
- `packages/types/src/index.ts`, `packages/ui/src/tokens.ts`
- `.env.example`, `.gitignore`

**You test (from Acceptance criteria in the prompt):**
```bash
pnpm install          # should install with no errors
pnpm dev              # should start all apps
pnpm build            # should complete without errors
```
- [ ] `apps/storefront` directory exists and has `app/` folder
- [ ] `backend/` directory exists
- [ ] `packages/types/src/index.ts` exists
- [ ] `packages/ui/src/tokens.ts` has brand colours and fonts

**UI to review:** None (no UI yet)

**Commit:**
```bash
git add .
git commit -m "feat: scaffold monorepo with pnpm + turborepo"
```

**PR:** Push and create PR → `phase/1-foundation` now.
```bash
git push -u origin feat/monorepo-scaffold
# GitHub → PR: feat/monorepo-scaffold → phase/1-foundation
```

---

### PROMPT 2 — Set Up Root Layout (Fonts, Providers, Global CSS)

**Branch:**
```bash
git checkout phase/1-foundation && git pull
git checkout -b feat/root-layout-providers
```

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Set Up Root Layout (Fonts, Providers, Global CSS)"**

**What the AI will create:**
- `apps/storefront/app/layout.tsx`
- `apps/storefront/app/globals.css`
- `apps/storefront/components/providers.tsx`
- `apps/storefront/sentry.client.config.ts`

**You test:**
```bash
pnpm dev
```
- [ ] `localhost:3000` loads without a white screen
- [ ] Open browser DevTools → Network tab → confirm Playfair Display and Inter fonts load
- [ ] Check page source: title tag reads `"Shree Furniture"` or similar

**UI to review:** No visible UI yet. This is infrastructure.

**Commit:**
```bash
git add .
git commit -m "feat: add root layout with fonts, TanStack Query provider, Sentry"
```

**PR:** Push → PR → `phase/1-foundation`

---

### PROMPT 3 — Set Up Route Group Layouts

**Branch:** Stay on `feat/root-layout-providers` (same feature) or:
```bash
git checkout -b feat/route-group-layouts
```

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Set Up Route Group Layouts"**

**What the AI will create:**
- `apps/storefront/app/(store)/layout.tsx`
- `apps/storefront/app/(checkout)/layout.tsx`
- `apps/storefront/app/(account)/layout.tsx`

**You test:**
```bash
pnpm type-check    # must pass — no TypeScript errors
```
- [ ] Three layout files exist at the correct paths
- [ ] `(store)` layout has placeholder for Header, Footer, CartDrawer imports
- [ ] `(checkout)` layout has minimal structure (no Header/Footer)
- [ ] `(account)` layout has auth guard logic and sidebar placeholder

**Commit:**
```bash
git add .
git commit -m "feat: add three route group layouts — store, checkout, account"
```

**PR:** Push → PR → `phase/1-foundation`

---

### PROMPT 4 — Set Up Docker Compose for Local Dev

**Branch:**
```bash
git checkout phase/1-foundation && git pull
git checkout -b feat/docker-ci-setup
```

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Set Up Docker Compose for Local Dev"**

**What the AI will create:**
- `infrastructure/docker-compose.yml`
- `.env.docker`

**You test:**
```bash
docker-compose -f infrastructure/docker-compose.yml up -d
```
- [ ] PostgreSQL container starts on port 5432
- [ ] Redis container starts on port 6379
- [ ] `docker ps` shows both containers as "healthy"

**Commit:**
```bash
git add .
git commit -m "chore: add docker-compose for local PostgreSQL + Redis"
```

---

### PROMPT 5 — Set Up GitHub Repository + Branch Protection

**Branch:** Stay on `feat/docker-ci-setup`

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Set Up GitHub Repository + Branch Protection"**

**What the AI will create:**
- `infrastructure/GITHUB-SETUP.md`
- `.github/pull_request_template.md`
- `.github/dependabot.yml`

**You do (manually — in GitHub settings):**
- Go to GitHub → Settings → Branches → Add rule for `main`
- Add rule for `develop`
- Settings exactly as described in the GITHUB-SETUP.md the agent creates

**Commit:**
```bash
git add .
git commit -m "chore: add GitHub setup guide, PR template, Dependabot config"
```

---

### PROMPT 6 — Configure next.config.ts

**Branch:** Stay on `feat/docker-ci-setup`

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Configure next.config.ts"**

**What the AI will create:**
- `apps/storefront/next.config.ts`
- `apps/storefront/.eslintrc.json`
- `apps/storefront/tailwind.config.ts`

**You test:**
```bash
pnpm build    # must complete with no errors
pnpm lint     # must pass
```
- [ ] `next.config.ts` has Cloudinary remote patterns
- [ ] `next.config.ts` has Sentry + bundle-analyzer wrappers
- [ ] `.eslintrc.json` has `no-any` as error

**Commit:**
```bash
git add .
git commit -m "chore: configure next.config.ts, ESLint, Tailwind v4"
```

---

### PROMPT 7 — Set Up CI Pipeline

**Branch:** Stay on `feat/docker-ci-setup`

**Source:** PROMPT-GUIDE-v2.md → Section 2a → **"Set Up CI Pipeline"**

**What the AI will create:**
- `.github/workflows/ci.yml`

**You test:**
```bash
git push origin feat/docker-ci-setup
# Open GitHub → Actions → see if CI run triggers
```
- [ ] CI workflow file exists at `.github/workflows/ci.yml`
- [ ] Pipeline has: checkout → pnpm setup → install → type-check → lint → test → build

**Commit:**
```bash
git add .
git commit -m "chore: add GitHub Actions CI pipeline"
```

**PR:** Push → PR → `phase/1-foundation` (this closes out Week 1 infra work)
```bash
git push -u origin feat/docker-ci-setup
# GitHub → PR: feat/docker-ci-setup → phase/1-foundation
```

---

## WEEK 2 — PROMPT-GUIDE-v2 SECTION 2b

---

### PROMPT 8 — Configure MedusaJS v2 Backend

**Branch:**
```bash
git checkout phase/1-foundation && git pull
git checkout -b feat/medusa-backend-config
```

**Source:** PROMPT-GUIDE-v2.md → Section 2b → **"Configure MedusaJS v2 Backend"**

**What the AI will create:**
- `backend/medusa-config.ts` with all 4 plugins registered

**You test:**
```bash
cd backend && pnpm build    # must compile without errors
pnpm type-check
```
- [ ] `medusa-config.ts` has Razorpay, Cloudinary, Resend, Algolia plugins configured
- [ ] All plugin config reads from env vars (no hardcoded values)

**Commit:**
```bash
git add .
git commit -m "feat: configure MedusaJS v2 backend with Razorpay, Cloudinary, Resend, Algolia"
```

---

### PROMPT 9 — Create Seed Script

**Branch:** Stay on `feat/medusa-backend-config`

**Source:** PROMPT-GUIDE-v2.md → Section 2b → **"Create Seed Script"**

**What the AI will create:**
- `backend/src/seed.ts`

**You test:**
```bash
pnpm type-check    # seed.ts must compile cleanly
```
- [ ] Seed script creates India region with INR currency
- [ ] Creates 4 collections (living-room, bedroom, dining, office)
- [ ] Creates shipping options with correct paise amounts (Free: ₹0, Standard: ₹49900)
- [ ] All product prices are integers (paise) — no decimals

**Commit:**
```bash
git add .
git commit -m "feat: add seed script with India region, collections, and sample products"
```

---

### PROMPT 10 — Run Migrations and Verify Backend

**Branch:** Stay on `feat/medusa-backend-config`

**Source:** PROMPT-GUIDE-v2.md → Section 2b → **"Run Migrations and Verify Backend"**

**You run (follow the agent's instructions exactly):**
```bash
npx medusa db:create
npx medusa db:migrate
npx medusa exec ./src/seed.ts
pnpm dev    # start backend
```

**You test (all 4 smoke tests from the prompt):**
- [ ] `GET localhost:9000/store/products` → returns seeded products ✅
- [ ] `GET localhost:9000/store/collections` → returns 4 collections ✅
- [ ] `GET localhost:9000/health` → returns 200 ✅
- [ ] `localhost:9000/app/admin` → Medusa Admin loads ✅

If any smoke test fails → paste the error to `@debugger` with: `Check NewDocs/KNOWN-ISSUES.md first.`

**Commit:**
```bash
git add .
git commit -m "feat: database migrations complete, backend verified with smoke tests"
```

**PR:** Push → PR → `phase/1-foundation`
```bash
git push -u origin feat/medusa-backend-config
```

---

## THIRD-PARTY SERVICES — PROMPT-GUIDE-v2 SECTION 2c

---

### PROMPT 11 — Configure Algolia Index

**Branch:**
```bash
git checkout phase/1-foundation && git pull
git checkout -b feat/third-party-services
```

**Source:** PROMPT-GUIDE-v2.md → Section 2c → **"Configure Algolia Index"**

**What the AI will create:**
- `backend/src/scripts/algolia-init.ts`
- `backend/src/scripts/algolia-sync-all.ts`

**You test (from Verify line in prompt):**
```bash
npx ts-node src/scripts/algolia-init.ts
npx medusa exec ./src/scripts/algolia-sync-all.ts
```
- [ ] Go to Algolia Dashboard → search for "sofa" → returns results

**Commit:**
```bash
git add .
git commit -m "feat: configure Algolia index with searchable attributes and facets"
```

---

### PROMPT 12 — Configure Cloudinary

**Branch:** Stay on `feat/third-party-services`

**Source:** PROMPT-GUIDE-v2.md → Section 2c → **"Configure Cloudinary"**

**What the AI will create:**
- `infrastructure/CLOUDINARY-SETUP.md`

**You do (manually — in Cloudinary dashboard):**
- Create upload preset `"shree_furniture_products"` with settings from the doc

**You test:**
- [ ] Upload a test image via Medusa Admin → image URL contains `res.cloudinary.com` and has WebP format

**Commit:**
```bash
git add .
git commit -m "chore: add Cloudinary setup documentation"
```

---

### PROMPT 13 — Set Up Resend Email

**Branch:** Stay on `feat/third-party-services`

**Source:** PROMPT-GUIDE-v2.md → Section 2c → **"Set Up Resend Email"**

**What the AI will create:**
- `infrastructure/RESEND-SETUP.md`

**You do (manually — in Resend and DNS):**
- Add domain, configure DKIM and SPF

**You test:**
- [ ] Send a test email → received within 2 minutes

**Commit:**
```bash
git add .
git commit -m "chore: add Resend email setup documentation"
```

---

### PROMPT 14 — Instrument PostHog

**Branch:** Stay on `feat/third-party-services`

**Source:** PROMPT-GUIDE-v2.md → Section 2c → **"Instrument PostHog"**

**What the AI will create:**
- `apps/storefront/lib/analytics.ts`
- PostHog tracking calls added to: PDP, cart-store, SearchBar, CartSummary, order confirmation

**You test:**
- [ ] Open PostHog dashboard → Live Events
- [ ] Browse a product → `product_viewed` event appears
- [ ] Add to cart → `add_to_cart` event appears

**Commit:**
```bash
git add .
git commit -m "feat: instrument PostHog analytics — 5 key events tracked"
```

---

### PROMPT 15 — Instrument Sentry

**Branch:** Stay on `feat/third-party-services`

**Source:** PROMPT-GUIDE-v2.md → Section 2c → **"Instrument Sentry"**

**What the AI will create:**
- `apps/storefront/sentry.server.config.ts`
- `apps/storefront/sentry.edge.config.ts`
- ErrorBoundary wrapped with Sentry

**You test (from Verify line):**
- [ ] Trigger a test error in the browser console
- [ ] Go to Sentry dashboard → Issues → error appears

**Commit:**
```bash
git add .
git commit -m "feat: configure Sentry client, server, and edge error monitoring"
```

**PR:** Push → PR → `phase/1-foundation`
```bash
git push -u origin feat/third-party-services
```

---

## PHASE 1 COMPLETE — MERGE TO DEVELOP

When all 15 prompts are done and all STATUS.md Phase 1 items are ✅:

```bash
# On GitHub: PR → phase/1-foundation → develop
# Manually test: backend health check passes, Medusa Admin loads, Algolia has data
```

---

## PHASE 1 — INTERRUPTION & RESUME GUIDE

### "I stopped mid-week and don't know where I am"

```bash
git log --oneline -10    # see your recent commits
cat STATUS.md            # see what's marked ✅
```

Paste into Antigravity:
```
Read STATUS.md and tell me exactly which Phase 1 prompt I completed last.
List the remaining Phase 1 items I haven't started yet.
Do not write any code.
```

### "My backend migration failed"

Open **PROMPT-GUIDE-v2.md → Section 8 → "Debug MedusaJS Subscriber Not Firing"**
Also check `NewDocs/KNOWN-ISSUES.md` first — migration errors are frequently documented there.

### "I need to stop right now mid-prompt"

```bash
git add .
git commit -m "wip: [what you were building] — incomplete, needs continuation"
git push
```

Add to STATUS.md manually: mark the item as 🚧 IN PROGRESS with note "WIP — stopped mid-prompt"

---

---

# ═══════════════════════════════════════
# PHASE 2 — STOREFRONT CORE
# ═══════════════════════════════════════

**Goal:** The full customer-facing storefront: browsing, search, product pages.
**Duration:** ~3 weeks
**PROMPT-GUIDE-v2 sections:** 3a, 3b, 3c, 3d, 3e

## Phase 2 Branch Setup

```bash
# After Phase 1 is merged to develop:
git checkout develop && git pull
git checkout -b phase/2-storefront
git push -u origin phase/2-storefront
```

---

## SECTION 3a — LIB, HOOKS & API CLIENT

Build the data layer before any UI. These files have no visual output — test with type-check only.

---

### PROMPT 16 — Build the Medusa v2 API Client

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/medusa-api-client
```

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build the Medusa v2 API Client"**

**What the AI will create:**
- `apps/storefront/lib/medusa/client.ts`
- `lib/medusa/products.ts`, `cart.ts`, `customers.ts`, `orders.ts`, `wishlist.ts`

**You test:**
```bash
pnpm type-check    # must pass
```
- [ ] All 5 wrapper files exist
- [ ] Methods match `NewDocs/06-api-contracts.md`
- [ ] No v1 patterns (`medusa.products.list()`) — only v2 SDK

**Commit:**
```bash
git add .
git commit -m "feat: build typed Medusa v2 API client with product, cart, customer, order wrappers"
```

---

### PROMPT 17 — Build Algolia Client

**Branch:** Stay on `feat/medusa-api-client`

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build Algolia Client"**

**What the AI will create:**
- `apps/storefront/lib/algolia/client.ts`

**You test:**
```bash
pnpm type-check
```
- [ ] Client reads from `NEXT_PUBLIC_ALGOLIA_APP_ID` and `NEXT_PUBLIC_ALGOLIA_SEARCH_KEY`

**Commit:**
```bash
git add .
git commit -m "feat: add Algolia InstantSearch client"
```

---

### PROMPT 18 — Build Price and Date Utilities

**Branch:** Stay on `feat/medusa-api-client`

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build Price and Date Utilities"**

**What the AI will create:**
- `lib/utils/price.ts`, `lib/utils/date.ts`, `lib/utils/validators.ts`

**You test:**
```bash
pnpm type-check
```
- [ ] `formatPrice(4999900)` returns `"₹49,999"`
- [ ] `isPinCode("400001")` returns `true`
- [ ] `isIndianPhone("9876543210")` returns `true`
- [ ] Manually check: `formatPrice(0)` → `"₹0"`, `formatPrice(100)` → `"₹1"`

**Commit:**
```bash
git add .
git commit -m "feat: add price, date, and validator utility functions"
```

**PR:** Push → PR → `phase/2-storefront`
```bash
git push -u origin feat/medusa-api-client
```

---

### PROMPT 19 — Build Zustand Stores

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/state-management
```

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build Zustand Stores"**

**What the AI will create:**
- `store/cart-store.ts` (with optimistic updates + rollback)
- `store/ui-store.ts` (cart drawer, search overlay, mobile menu)

**You test:**
```bash
pnpm type-check
```
- [ ] `cart-store.ts` has `addItemOptimistic`, `rollbackOptimisticAdd`, `_snapshot`
- [ ] `ui-store.ts` has `cartDrawerOpen`, `searchOverlayOpen`, `mobileMenuOpen`
- [ ] cartId is persisted in cookie via `zustand/middleware persist`
- [ ] No prices stored as floats

**Commit:**
```bash
git add .
git commit -m "feat: add Zustand cart and UI stores with optimistic update pattern"
```

---

### PROMPT 20 — Build Next.js Middleware for Cart Persistence

**Branch:** Stay on `feat/state-management`

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build Next.js Middleware for Cart Persistence"**

**What the AI will create:**
- `apps/storefront/middleware.ts`
- `apps/storefront/lib/cart-id.ts`

**You test:**
```bash
pnpm type-check
pnpm dev    # middleware must not break page loads
```
- [ ] Middleware matcher excludes `/api/*`, `/_next/*`, `/favicon.ico`
- [ ] `getCartId()`, `setCartId()`, `clearCartId()` functions exist

**Commit:**
```bash
git add .
git commit -m "feat: add Next.js middleware for cart cookie persistence"
```

---

### PROMPT 21 — Build TanStack Query Hooks

**Branch:** Stay on `feat/state-management`

**Source:** PROMPT-GUIDE-v2.md → Section 3a → **"Build TanStack Query Hooks"**

**What the AI will create:**
- `hooks/useCart.ts`, `hooks/useProducts.ts`, `hooks/useCustomer.ts`

**You test:**
```bash
pnpm type-check
```
- [ ] `useAddToCart` has optimistic update + rollback using cart-store
- [ ] All hooks use v2 client methods
- [ ] Query keys are stable (not inline objects)

**Commit:**
```bash
git add .
git commit -m "feat: add TanStack Query hooks for cart, products, and customer"
```

**PR:** Push → PR → `phase/2-storefront`
```bash
git push -u origin feat/state-management
```

---

## SECTION 3b — SHARED COMPONENTS

> ⚠️ **PREFERENCES CHECK:** Every component built from here onward has visual output.
> After each AI generation, open `localhost:3000` and ask yourself:
> "Does this feel warm, premium, trustworthy — like a furniture showroom?"
> "Would a first-time buyer trust this to spend ₹40,000 here?"
> If no → use the rejection flow below before committing.

**Rejection flow (run after testing any UI component):**
```
I don't like [what was built for ComponentName].
Specifically: [precise description]
What I want instead: [precise direction]
1. Add to REJECTIONS.md Section 1 with today's date
2. Rebuild [ComponentName] with the new direction
3. Update STATUS.md
```

---

### PROMPT 22 — Build Header

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/layout-header-footer-nav
```

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build Header"**

**What the AI will create:**
- `components/layout/Header.tsx`

**You test:**
```bash
pnpm dev    # open localhost:3000
```
- [ ] Desktop: Logo | Nav links | Search | Wishlist | Account | Cart icons
- [ ] Mobile (drag browser to 375px): Logo | Search | Cart | Hamburger
- [ ] Header sticks to top when you scroll
- [ ] Cart icon badge is hidden when cart is empty
- [ ] Background: white with subtle bottom border

**PREFERENCES CHECK:**
- Does the typography feel premium? (Playfair Display for logo)
- Does the spacing feel right?
- Does it feel like a furniture brand, not a SaaS product?

**Commit:**
```bash
git add .
git commit -m "feat: build Header with sticky nav, mobile hamburger, cart badge"
```

---

### PROMPT 23 — Build ProductCard Component

**Branch:** Stay on `feat/layout-header-footer-nav` OR:
```bash
git checkout -b feat/product-card    # separate branch — ProductCard is used everywhere
```

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build ProductCard Component"**

**What the AI will create:**
- `components/product/ProductCard.tsx`

**You test:**
```bash
pnpm dev    # go to any page that renders ProductCard with mock data
```
- [ ] Thumbnail renders with explicit width=400 height=400 (no layout shift)
- [ ] Discount badge appears in Brand Accent gold when sale price < original
- [ ] "New" badge appears for recent products
- [ ] Product name truncates at 2 lines
- [ ] Wishlist heart icon visible
- [ ] Card is fully clickable (links to `/products/[handle]`)

**PREFERENCES CHECK:**
- Hover effect: is it a subtle card lift (translateY -2px) with shadow? Or did it do something else?
- Image: does it crop nicely? Does it zoom on hover? (It should NOT zoom)
- Price: does ₹49,999 display correctly?

**Commit:**
```bash
git add .
git commit -m "feat: build ProductCard with Cloudinary image, INR price, hover lift"
```

---

### PROMPT 24 — Build PriceDisplay Component

**Branch:** Stay on current branch

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build PriceDisplay Component"**

**What the AI will create:**
- `components/shared/PriceDisplay.tsx`

**You test:**
- [ ] `formatPrice(4999900)` shows as `₹49,999`
- [ ] If original price > sale price: original is struck through
- [ ] GST note `"Incl. 12% GST"` appears below price
- [ ] `data-testid="price-display"` is on the root element

**PREFERENCES CHECK:**
- Font size appropriate?
- Gold discount badge visible?

**Commit:**
```bash
git add .
git commit -m "feat: build PriceDisplay with INR formatting, discount badge, GST note"
```

---

### PROMPT 25 — Build Skeleton, ErrorBoundary, EmptyState

**Branch:** Stay on current branch

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build Skeleton, ErrorBoundary, EmptyState"**

**What the AI will create:**
- `components/shared/SkeletonCard.tsx`
- `components/shared/ErrorBoundary.tsx`
- `components/shared/EmptyState.tsx`

**You test:**
```bash
pnpm type-check
```
- [ ] SkeletonCard has `animate-pulse` Tailwind class
- [ ] ErrorBoundary is a class component (not function — React requirement)
- [ ] EmptyState accepts `ctaLabel` and `ctaHref` props

**PREFERENCES CHECK:**
- SkeletonCard: does the size match ProductCard's proportions?
- EmptyState: does it feel warm, not cold/corporate?

**Commit:**
```bash
git add .
git commit -m "feat: build SkeletonCard, ErrorBoundary, and EmptyState shared components"
```

---

### PROMPT 26 — Build Footer

**Branch:** Stay on `feat/layout-header-footer-nav`

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build Footer"**

**What the AI will create:**
- `components/layout/Footer.tsx`

**You test:**
```bash
pnpm dev    # scroll to bottom of any page
```
- [ ] 3 link columns: Collections, Customer Service, Company
- [ ] Payment icons strip (UPI, Visa, Mastercard, EMI)
- [ ] Copyright line present
- [ ] Mobile: columns stack vertically
- [ ] Background: `#F5F0E8` (warm off-white)

**PREFERENCES CHECK:**
- Does the footer feel finished? Premium? Right spacing?
- Does mobile accordion work?

**Commit:**
```bash
git add .
git commit -m "feat: build Footer with 3 columns, payment icons, mobile accordion"
```

---

### PROMPT 27 — Build MobileMenu

**Branch:** Stay on `feat/layout-header-footer-nav`

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build MobileMenu"**

**What the AI will create:**
- `components/layout/MobileMenu.tsx`

**You test (on mobile screen size — drag browser to 375px):**
- [ ] Hamburger icon in Header opens the menu
- [ ] Full-screen overlay slides in
- [ ] Primary links: Living Room, Bedroom, Dining, Office
- [ ] Secondary links: My Account, My Wishlist, My Orders
- [ ] Close button works
- [ ] Escape key closes menu

**PREFERENCES CHECK:**
- Does the slide-in animation feel smooth?
- Is the menu background colour correct (white or warm)?

**Commit:**
```bash
git add .
git commit -m "feat: build MobileMenu with full-screen overlay and escape key handling"
```

---

### PROMPT 28 — Build Breadcrumb Component

**Branch:** Stay on current branch

**Source:** PROMPT-GUIDE-v2.md → Section 3b → **"Build Breadcrumb Component"**

**What the AI will create:**
- `components/layout/Breadcrumb.tsx`

**You test:**
```bash
pnpm type-check
```
- [ ] Renders `Home > Collection > Product Name`
- [ ] Last item is not a link
- [ ] Contains JSON-LD `BreadcrumbList` script tag
- [ ] `nav aria-label="Breadcrumb"` present (accessibility)

**Commit:**
```bash
git add .
git commit -m "feat: build Breadcrumb with JSON-LD structured data"
```

**PR:** Push → PR → `phase/2-storefront`
```bash
git push -u origin feat/layout-header-footer-nav
```

---

## SECTION 3c — HOMEPAGE

---

### PROMPT 29 — Build the Homepage

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/homepage
```

**Source:** PROMPT-GUIDE-v2.md → Section 3c → **"Build the Homepage"**

**What the AI will create:**
- `app/(store)/page.tsx`
- `components/home/HeroSection.tsx`
- `components/home/FeaturedCollections.tsx`
- `components/home/BestSellers.tsx`
- `components/home/TrustSignals.tsx`

**You test:**
```bash
pnpm dev    # open localhost:3000
```
- [ ] Hero section renders with headline "Beautiful Furniture for Every Home"
- [ ] "Free delivery on orders above ₹5,000" subheadline visible
- [ ] Two CTA buttons: "Shop Now" + "Explore Collections"
- [ ] Featured Collections: 4 cards in a 2-col (mobile) / 4-col (desktop) grid
- [ ] Best Sellers: horizontal scroll on mobile, 4-col grid on desktop
- [ ] Trust Signals strip at the bottom
- [ ] `revalidate = 60` is set (ISR)

**PREFERENCES CHECK — This is your most important visual review:**
- Hero section: does it feel like a premium furniture brand or a stock template?
- Background colours: is it using `#F5F0E8` (warm) not plain white?
- Typography: is Playfair Display rendering for headings?
- Collection cards: do they feel aspirational?

**Commit:**
```bash
git add .
git commit -m "feat: build Homepage with Hero, FeaturedCollections, BestSellers, TrustSignals (ISR)"
```

**PR:** Push → PR → `phase/2-storefront`

---

## SECTION 3d — PRODUCT PAGES

---

### PROMPT 30 — Build PLP (Product Listing Page)

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/product-listing-page
```

**Source:** PROMPT-GUIDE-v2.md → Section 3d → **"Build PLP (Product Listing Page)"**

**What the AI will create:**
- `app/(store)/collections/[handle]/page.tsx`
- `app/(store)/collections/[handle]/loading.tsx`
- `app/(store)/collections/[handle]/not-found.tsx`

**You test:**
```bash
pnpm dev    # open localhost:3000/collections/living-room
```
- [ ] Product grid renders: 2 cols mobile / 3 tablet / 4 desktop
- [ ] Filter panel: sidebar on desktop, accessible on mobile
- [ ] Sort dropdown works (Price Low→High, etc.)
- [ ] URL updates with filter params: `?material=wood&sort=price_asc`
- [ ] "Load More" button present (not infinite scroll)
- [ ] Breadcrumb: `Home > Living Room`
- [ ] Loading state: 8 SkeletonCard components
- [ ] Visit `/collections/invalid-handle` → shows not-found page
- [ ] `revalidate = 60` is set

**PREFERENCES CHECK:**
- Grid spacing feels right?
- Filter panel design feels clean?

**Commit:**
```bash
git add .
git commit -m "feat: build PLP with filter panel, URL-based sort/filter, loading skeleton (ISR)"
```

**PR:** Push → PR → `phase/2-storefront`

---

### PROMPT 31 — Build PDP (Product Detail Page)

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/product-detail-page
```

**Source:** PROMPT-GUIDE-v2.md → Section 3d → **"Build PDP (Product Detail Page)"**

**What the AI will create:**
- `app/(store)/products/[handle]/page.tsx`
- `components/product/ProductGallery.tsx`
- `components/product/ProductVariants.tsx`
- `components/product/ProductBadge.tsx`
- `components/product/ProductSpecsTable.tsx`
- `components/product/DeliveryEstimate.tsx`

**You test:**
```bash
pnpm dev    # open localhost:3000/products/[any-seeded-handle]
```
- [ ] Gallery: primary image loads with `priority` (fast LCP)
- [ ] Thumbnail strip works, selecting thumbnail changes primary image
- [ ] Colour variants (swatches) selectable
- [ ] Out-of-stock variants are visually disabled (opacity-50), NOT hidden
- [ ] "Add to Cart" button sticky on mobile
- [ ] `data-testid="add-to-cart"` present
- [ ] PIN code input in DeliveryEstimate
- [ ] JSON-LD Product schema in page source
- [ ] Breadcrumb: `Home > Collection > Product Name`

**PREFERENCES CHECK — Critical review:**
- Gallery layout: does it feel premium?
- "Add to Cart" button: colour? Size? Position on mobile?
- Price display: does the INR amount look right?
- Overall feel: would you buy a ₹40,000 sofa from this page?

**Commit:**
```bash
git add .
git commit -m "feat: build PDP with gallery, variants, delivery estimate, JSON-LD schema (ISR)"
```

**PR:** Push → PR → `phase/2-storefront`

---

## SECTION 3e — SEARCH & CART PAGE

---

### PROMPT 32 — Build Search Feature

**Branch:**
```bash
git checkout phase/2-storefront && git pull
git checkout -b feat/search-feature
```

**Source:** PROMPT-GUIDE-v2.md → Section 3e → **"Build Search Feature"**

**What the AI will create:**
- `components/search/SearchBar.tsx`
- `components/search/SearchOverlay.tsx`
- `components/search/SearchResults.tsx`
- `app/(store)/search/page.tsx`

**You test:**
```bash
pnpm dev    # click search icon in Header
```
- [ ] Clicking search icon opens SearchOverlay
- [ ] Typing "sofa" → results appear from Algolia (requires Algolia data from Phase 1)
- [ ] "No results" state shows EmptyState + 4 collection links
- [ ] Escape key closes overlay
- [ ] `/search?q=sofa` URL loads full search results page

**PREFERENCES CHECK:**
- Overlay: desktop floating panel vs mobile full-screen — both correct?
- Results: product thumbnail, name, price visible in results?

**Commit:**
```bash
git add .
git commit -m "feat: build Algolia search feature with overlay, results, and full search page"
```

---

### PROMPT 33 — Build Cart Page

**Branch:** Stay on `feat/search-feature`

**Source:** PROMPT-GUIDE-v2.md → Section 3e → **"Build Cart Page"**

**What the AI will create:**
- `app/(store)/cart/page.tsx`

**You test:**
```bash
pnpm dev    # open localhost:3000/cart
```
- [ ] Empty cart shows EmptyState with "Continue Shopping" link to `/`
- [ ] Cart with items shows CartItem list + CartSummary
- [ ] "Proceed to Checkout" button → navigates to `/checkout`
- [ ] CSR (no `revalidate`)

**PREFERENCES CHECK:**
- Empty cart state: feels encouraging (not punishing)?
- Cart layout: clean and functional?

**Commit:**
```bash
git add .
git commit -m "feat: build Cart page (CSR) with item list and CartSummary"
```

**PR:** Push → PR → `phase/2-storefront`

---

## PHASE 2 COMPLETE — MERGE TO DEVELOP

When all Phase 2 STATUS.md items are ✅ and you've manually walked through:
- Browse homepage → click collection → view product → search → go back

```bash
# GitHub → PR: phase/2-storefront → develop
```

## PHASE 2 — INTERRUPTION & RESUME GUIDE

### "I built some components but can't remember which ones"

```bash
cat STATUS.md | grep -A 50 "Storefront — Components"
```

Paste into Antigravity:
```
Read STATUS.md and list all Phase 2 components that are still ⬜ NOT STARTED.
List them in the order they should be built, per NewDocs/10-development-phases.md.
```

### "I don't like the design direction after several components are built"

This is a PREFERENCES update that applies globally. Paste:
```
I want to change the overall visual direction of the storefront.
My new preference: [describe the change — e.g. "more whitespace", "darker CTAs"]
1. Add this to PREFERENCES.md as a confirmed global preference
2. List every ✅ DONE component in STATUS.md that needs to be updated
3. Do NOT update anything yet — just show me the impact list
```

---

---

# ═══════════════════════════════════════
# PHASE 3 — TRANSACTIONS
# ═══════════════════════════════════════

**Goal:** Cart drawer, full checkout, payments, auth, account, backend subscribers.
**Duration:** ~3 weeks
**PROMPT-GUIDE-v2 sections:** 4a, 4b, 4c, 5

## Phase 3 Branch Setup

```bash
git checkout develop && git pull
git checkout -b phase/3-transactions
git push -u origin phase/3-transactions
```

---

## SECTION 4a — CART UI

---

### PROMPT 34 — Build CartDrawer + CartItem + CartSummary

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/cart-ui
```

**Source:** PROMPT-GUIDE-v2.md → Section 4a → **"Build CartDrawer + CartItem + CartSummary"**

**What the AI will create:**
- `components/cart/CartItem.tsx`
- `components/cart/CartSummary.tsx`
- `components/cart/CartDrawer.tsx`

**You test:**
```bash
pnpm dev    # click cart icon in Header
```
- [ ] Cart drawer slides in from right (desktop) / bottom (mobile)
- [ ] Empty state shows EmptyState component
- [ ] Add a product to cart via PDP → item appears in drawer
- [ ] Quantity stepper: `−` and `+` buttons work, can't go below 1
- [ ] Remove button works (optimistic — item disappears instantly)
- [ ] CartSummary: subtotal in INR
- [ ] Free shipping bar: "Add ₹X more for free shipping" when below ₹5,000
- [ ] "Proceed to Checkout" button navigates to `/checkout`
- [ ] `data-testid="cart-drawer"` and `data-testid="proceed-to-checkout"` present

**PREFERENCES CHECK:**
- Drawer animation: feels smooth, not jarring?
- Cart item layout: thumbnail size, text hierarchy?
- Free shipping progress bar: design feel right?

**Commit:**
```bash
git add .
git commit -m "feat: build CartDrawer with CartItem, CartSummary, optimistic updates, free shipping bar"
```

**PR:** Push → PR → `phase/3-transactions`

---

## SECTION 4b — CHECKOUT

---

### PROMPT 35 — Build CheckoutProgress Component

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/checkout-flow
```

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build CheckoutProgress Component"**

**What the AI will create:**
- `components/checkout/CheckoutProgress.tsx`

**You test:**
```bash
pnpm type-check
```
- [ ] 3 states: `address`, `shipping`, `payment`
- [ ] Completed steps clickable (navigate back)
- [ ] Current step highlighted in Brand Accent gold
- [ ] Mobile: step numbers only (compact)
- [ ] `data-testid="checkout-progress"` present

**Commit:**
```bash
git add .
git commit -m "feat: build CheckoutProgress with 3 steps, completed/active/future states"
```

---

### PROMPT 36 — Build Checkout Step 1 — Address

**Branch:** Stay on `feat/checkout-flow`

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build Checkout Step 1 — Address"**

**What the AI will create:**
- `app/(checkout)/checkout/page.tsx`
- `components/checkout/AddressForm.tsx` (with exported Zod schema)

**You test:**
```bash
pnpm dev    # navigate to /checkout
```
- [ ] Form renders all required fields: firstName, lastName, email, phone, address1, city, state, PIN
- [ ] State field is a dropdown with all Indian states
- [ ] Phone validation: 10 digits starting with 6-9
- [ ] PIN validation: exactly 6 digits
- [ ] On invalid submit: inline error messages appear
- [ ] On valid submit: navigates to `/checkout/shipping`
- [ ] CheckoutProgress shows "Address" as active step

**PREFERENCES CHECK:**
- Form layout on mobile? Easy to fill?
- Input field styling feels right?
- Error message colour and placement?

**Commit:**
```bash
git add .
git commit -m "feat: build Checkout Step 1 — AddressForm with React Hook Form + Zod validation"
```

---

### PROMPT 37 — Build Checkout Step 2 — Shipping

**Branch:** Stay on `feat/checkout-flow`

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build Checkout Step 2 — Shipping"**

**What the AI will create:**
- `app/(checkout)/checkout/shipping/page.tsx`

**You test:**
```bash
pnpm dev    # complete address step → should land on /checkout/shipping
```
- [ ] "Standard Shipping — ₹499 — 7–10 business days" option visible
- [ ] "Free Shipping" option only visible when cart ≥ ₹5,000
- [ ] "Continue to Payment" button present with `data-testid="continue-to-payment"`
- [ ] "← Back to Address" link works
- [ ] CheckoutProgress shows "Shipping" as active

**PREFERENCES CHECK:**
- Shipping option layout: radio buttons? Cards?
- Free shipping eligibility message: clear and encouraging?

**Commit:**
```bash
git add .
git commit -m "feat: build Checkout Step 2 — ShippingOptions with free shipping logic"
```

---

### PROMPT 38 — Build Checkout Step 3 — Payment

**Branch:** Stay on `feat/checkout-flow`

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build Checkout Step 3 — Payment"**

**What the AI will create:**
- `app/(checkout)/checkout/payment/page.tsx`

**You test:**
```bash
pnpm dev    # complete shipping step → should land on /checkout/payment
```
- [ ] Razorpay JS loads dynamically from `checkout.razorpay.com` (NOT bundled)
- [ ] "Place Order" button opens Razorpay modal
- [ ] Modal shows brand name "Shree Furniture" and gold theme `#C8A96E`
- [ ] In sandbox mode: complete test payment → redirects to `/order/confirm/[id]`
- [ ] Cart cookie is cleared after successful payment

> ⚠️ Test in **sandbox mode** only. Never use real Razorpay keys during development.

**PREFERENCES CHECK:**
- Order summary sidebar: visible on desktop, collapsed on mobile?
- "Place Order" button: prominent? Correct colour?

**Commit:**
```bash
git add .
git commit -m "feat: build Checkout Step 3 — Razorpay payment with sandbox test"
```

---

### PROMPT 39 — Build Razorpay Webhook Handler

**Branch:** Stay on `feat/checkout-flow` OR:
```bash
git checkout -b feat/razorpay-webhook    # security-sensitive — own branch recommended
```

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build Razorpay Webhook Handler"**

> ⚠️ Read `NewDocs/KNOWN-ISSUES.md "Duplicate Order Creation"` BEFORE the AI starts. Paste this first.

**What the AI will create:**
- `app/api/webhooks/razorpay/route.ts`

**You test:**
```bash
pnpm type-check    # must pass
```
- [ ] HMAC-SHA256 verification using `timingSafeEqual` (not `===`)
- [ ] 200 OK returned BEFORE async processing (prevents Razorpay retry)
- [ ] Idempotency check: duplicate webhook → silently ignored
- [ ] `crypto.createHmac` used (not any library)
- [ ] Raw body read as text before any JSON parsing

**Commit:**
```bash
git add .
git commit -m "feat: build Razorpay webhook — HMAC verify, idempotency, 200 before processing"
```

---

### PROMPT 40 — Build Order Confirmation Page

**Branch:** Stay on `feat/checkout-flow`

**Source:** PROMPT-GUIDE-v2.md → Section 4b → **"Build Order Confirmation Page"**

**What the AI will create:**
- `app/(checkout)/order/confirm/[id]/page.tsx`

**You test:**
```bash
pnpm dev    # complete a test purchase → should land here
```
- [ ] ✅ "Order Confirmed!" hero message visible
- [ ] Order ID `#[display_id]` shown with `data-testid="order-confirmation-id"`
- [ ] Items list, shipping address, total breakdown correct
- [ ] Estimated delivery date formatted correctly (e.g. "22–25 Apr 2026")
- [ ] `"force-dynamic"` export → never cached (SSR)
- [ ] `analytics.orderComplete()` call present

**PREFERENCES CHECK:**
- Confirmation page: does it feel celebratory but not over-the-top?

**Commit:**
```bash
git add .
git commit -m "feat: build Order Confirmation page (SSR) with order details and analytics"
```

**PR:** Push → PR → `phase/3-transactions`
```bash
git push -u origin feat/checkout-flow
```

---

## SECTION 4c — AUTH & ACCOUNT

---

### PROMPT 41 — Build Auth Pages (Login / Register / Forgot Password)

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/auth-pages
```

**Source:** PROMPT-GUIDE-v2.md → Section 4c → **"Build Auth Pages (Login / Register / Forgot Password)"**

**What the AI will create:**
- `app/(account)/account/login/page.tsx`
- `app/(account)/account/register/page.tsx`
- `app/(account)/account/forgot-password/page.tsx`

**You test:**
```bash
pnpm dev    # visit /account/login
```
- [ ] Login form: email + password, `data-testid="login-submit"`
- [ ] Error on wrong credentials: "Invalid email or password" (not specific about which)
- [ ] Successful login → redirects to `/account/orders`
- [ ] Register: all fields with validation, confirms passwords match
- [ ] Forgot password: email form → shows "Check your email" message
- [ ] All 3 pages accessible WITHOUT login (outside auth guard)

**PREFERENCES CHECK:**
- Auth page layout: centred form? Background colour?
- Form fields: consistent with checkout form styling?

**Commit:**
```bash
git add .
git commit -m "feat: build Login, Register, ForgotPassword pages with React Hook Form + Zod"
```

**PR:** Push → PR → `phase/3-transactions`

---

### PROMPT 42 — Build Account Layout (Sidebar + Auth Guard)

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/account-pages
```

**Source:** PROMPT-GUIDE-v2.md → Section 4c → **"Build Account Layout (Sidebar + Auth Guard)"**

**What the AI will create:**
- `app/(account)/layout.tsx` (the protected layout)

**You test:**
```bash
pnpm dev    # visit /account/orders WITHOUT being logged in
```
- [ ] Redirects to `/account/login?redirect=/account/orders` ← auth guard works
- [ ] After logging in → redirects back to `/account/orders`
- [ ] Desktop: sidebar with My Orders, Wishlist, Addresses, Settings, Logout
- [ ] Mobile: horizontal tab strip at top
- [ ] Active link highlighted

**Commit:**
```bash
git add .
git commit -m "feat: build Account layout with auth guard, sidebar nav, mobile tab strip"
```

---

### PROMPT 43 — Build My Orders Pages

**Branch:** Stay on `feat/account-pages`

**Source:** PROMPT-GUIDE-v2.md → Section 4c → **"Build My Orders Pages"**

**What the AI will create:**
- `app/(account)/account/orders/page.tsx`
- `app/(account)/account/orders/[id]/page.tsx`

**You test:**
```bash
pnpm dev    # log in → navigate to /account/orders
```
- [ ] Orders list: ID, date, total, status badges
- [ ] Clicking a row → navigates to order detail page
- [ ] Detail page: items list, shipping address, total breakdown
- [ ] Empty state for no orders

**Commit:**
```bash
git add .
git commit -m "feat: build My Orders list and detail pages (SSR)"
```

---

### PROMPT 44 — Build Wishlist Page

**Branch:** Stay on `feat/account-pages`

**Source:** PROMPT-GUIDE-v2.md → Section 4c → **"Build Wishlist Page"**

**What the AI will create:**
- `app/(account)/account/wishlist/page.tsx`
- Updates to `ProductCard.tsx` and PDP wishlist toggle

**You test:**
```bash
pnpm dev
```
- [ ] Heart icon on ProductCard → adds to wishlist when logged in
- [ ] Heart icon when guest → redirects to login with redirect param
- [ ] Wishlist page: grid of saved products with "Add to Cart" + "Remove" buttons
- [ ] Remove: optimistic (item disappears instantly)
- [ ] Empty wishlist: EmptyState with "Browse Products" CTA

**Commit:**
```bash
git add .
git commit -m "feat: build Wishlist page with optimistic remove, update ProductCard heart toggle"
```

---

### PROMPT 45 — Build Addresses and Settings Pages

**Branch:** Stay on `feat/account-pages`

**Source:** PROMPT-GUIDE-v2.md → Section 4c → **"Build Addresses and Settings Pages"**

**What the AI will create:**
- `app/(account)/account/addresses/page.tsx`
- `app/(account)/account/settings/page.tsx`

**You test:**
```bash
pnpm dev    # log in → /account/addresses
```
- [ ] Saved addresses show as cards
- [ ] "Add New Address" opens AddressForm in a Sheet (reuses checkout AddressForm)
- [ ] Edit and Delete work
- [ ] Settings: name, email, password change forms all work
- [ ] Toast notifications on success/error

**Commit:**
```bash
git add .
git commit -m "feat: build Addresses and Settings account pages"
```

**PR:** Push → PR → `phase/3-transactions`
```bash
git push -u origin feat/account-pages
```

---

## SECTION 5 — BACKEND SUBSCRIBERS & WORKFLOWS

---

### PROMPT 46 — Build Order Confirmation Email Subscriber

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/backend-subscribers
```

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Order Confirmation Email Subscriber"**

**What the AI will create:**
- `backend/src/subscribers/order-placed.ts`
- `backend/src/emails/OrderConfirmation.tsx`

**You test:**
```bash
# Place a test order → check your email within 2 minutes
```
- [ ] Email arrives with subject: `"Order Confirmed — #[id] | Shree Furniture"`
- [ ] Email shows items table, shipping address, total breakdown
- [ ] "View Order" button in email links correctly

**Commit:**
```bash
git add .
git commit -m "feat: build order.placed subscriber + OrderConfirmation React Email template"
```

---

### PROMPT 47 — Build Order Shipped Email Subscriber

**Branch:** Stay on `feat/backend-subscribers`

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Order Shipped Email Subscriber"**

**What the AI will create:**
- `backend/src/subscribers/order-shipped.ts`
- `backend/src/emails/OrderShipped.tsx`

**Commit:**
```bash
git add .
git commit -m "feat: build order.shipment_created subscriber + OrderShipped email template"
```

---

### PROMPT 48 — Build Algolia Sync Subscriber

**Branch:** Stay on `feat/backend-subscribers`

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Algolia Sync Subscriber"**

**What the AI will create:**
- `backend/src/subscribers/algolia-sync.ts`

**You test:**
- [ ] Update a product in Medusa Admin
- [ ] Search for it in Algolia → updated data appears
- [ ] Storefront PDP refreshes (ISR revalidation triggered)

**Commit:**
```bash
git add .
git commit -m "feat: build algolia-sync subscriber — upserts on publish, calls ISR revalidation"
```

---

### PROMPT 49 — Build Low Stock Alert Subscriber

**Branch:** Stay on `feat/backend-subscribers`

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Low Stock Alert Subscriber"**

**What the AI will create:**
- `backend/src/subscribers/low-stock.ts`
- `backend/src/emails/LowStockAlert.tsx`

**Commit:**
```bash
git add .
git commit -m "feat: build low-stock subscriber + LowStockAlert email template"
```

**PR:** Push → PR → `phase/3-transactions`
```bash
git push -u origin feat/backend-subscribers
```

---

### PROMPT 50 — Build Fulfillment Workflow

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/fulfillment-workflow
```

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Fulfillment Workflow"**

**What the AI will create:**
- `backend/src/workflows/fulfillment-workflow.ts`

**You test:**
```bash
pnpm type-check
```
- [ ] 3 steps: validateOrderStep, createFulfillmentStep, notifyCustomerStep
- [ ] Rollback logic on `createFulfillmentStep`
- [ ] Registered in `medusa-config.ts`

**Commit:**
```bash
git add .
git commit -m "feat: build fulfillment workflow with validate → create → notify steps"
```

**PR:** Push → PR → `phase/3-transactions`

---

### PROMPT 51 — Build Wishlist Module

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/wishlist-module
```

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build Wishlist Module"**

**What the AI will create:**
- `backend/src/modules/wishlist/models/wishlist-item.ts`
- `backend/src/modules/wishlist/service.ts`
- `backend/src/modules/wishlist/index.ts`
- `backend/src/api/store/wishlist/route.ts`
- `backend/src/api/store/wishlist/[id]/route.ts`

**You test:**
```bash
# Using Postman or browser:
# POST /store/wishlist  (with session cookie) → adds item
# GET /store/wishlist   → returns items
# DELETE /store/wishlist/[id] → removes item
```
- [ ] Duplicate prevention: add same product twice → returns 409 or silently ignores second

**Commit:**
```bash
git add .
git commit -m "feat: build Wishlist custom Medusa module with GET, POST, DELETE API"
```

**PR:** Push → PR → `phase/3-transactions`

---

### PROMPT 52 — Build ISR Revalidation Endpoint

**Branch:**
```bash
git checkout phase/3-transactions && git pull
git checkout -b feat/isr-revalidation
```

**Source:** PROMPT-GUIDE-v2.md → Section 5 → **"Build ISR Revalidation Endpoint"**

**What the AI will create:**
- `app/api/revalidate/route.ts`

**You test:**
```bash
curl -X POST localhost:3000/api/revalidate \
  -H "Content-Type: application/json" \
  -d '{"secret":"[REVALIDATION_SECRET]","path":"/products/oslo-sofa"}'
# Should return: { "revalidated": true, "paths": ["/products/oslo-sofa"] }
```
- [ ] Wrong secret → returns 401
- [ ] Correct secret → `revalidatePath()` called

**Commit:**
```bash
git add .
git commit -m "feat: build ISR revalidation endpoint with secret validation"
```

**PR:** Push → PR → `phase/3-transactions`

---

## PHASE 3 COMPLETE — FULL PURCHASE FLOW TEST

Before merging Phase 3 to develop, manually walk through the **entire purchase flow**:

```
Browse homepage → click Living Room → click product → select variant → Add to Cart
→ Cart drawer opens → Proceed to Checkout → fill address → continue → select shipping
→ continue → Place Order (sandbox) → see confirmation page → check email
```

Every step must work. If anything fails → go to Part 6 of this guide (debugging).

```bash
# Only when all flows pass:
# GitHub → PR: phase/3-transactions → develop
```

## PHASE 3 — INTERRUPTION & RESUME GUIDE

### "I need to stop mid-checkout build"

```bash
git add .
git commit -m "wip: checkout [step] partial — stopped at [where]"
git push
```

Resume prompt:
```
Read STATUS.md.
I was building the checkout flow. Tell me exactly which step I completed last:
- CheckoutProgress ✅/⬜
- AddressForm ✅/⬜
- ShippingOptions ✅/⬜
- PaymentStep ✅/⬜
- RazorpayWebhook ✅/⬜
- OrderConfirmation ✅/⬜
What should I build next?
```

### "Payment is not working mid-session"

Open **PROMPT-GUIDE-v2.md → Section 8 → "Debug Razorpay Payment Issue"**

---

---

# ═══════════════════════════════════════
# PHASE 4 — LAUNCH PREP
# ═══════════════════════════════════════

**Goal:** SEO, analytics verification, security hardening, performance, production deploy.
**PROMPT-GUIDE-v2 sections:** 6a, 6b, 6c, 6d, 6e

## Phase 4 Branch Setup

```bash
git checkout develop && git pull
git checkout -b phase/4-launch-prep
git push -u origin phase/4-launch-prep
```

---

### PROMPT 53 — Add robots.txt and sitemap.xml

**Branch:**
```bash
git checkout phase/4-launch-prep && git pull
git checkout -b feat/seo-setup
```

**Source:** PROMPT-GUIDE-v2.md → Section 6a → **"Add robots.txt and sitemap.xml"**

**You test:**
- [ ] `localhost:3000/sitemap.xml` → valid XML with product and collection URLs
- [ ] `localhost:3000/robots.txt` → Disallow: /checkout, /cart, /account, /api
- [ ] `/checkout` has `noindex` meta tag
- [ ] `/account` layout has `noindex` meta tag

**Commit:**
```bash
git add .
git commit -m "feat: add dynamic sitemap.xml, robots.txt, noindex on protected routes"
```

---

### PROMPT 54 — Verify All SEO Meta Tags

**Branch:** Stay on `feat/seo-setup`

**Source:** PROMPT-GUIDE-v2.md → Section 6a → **"Verify All SEO Meta Tags"**

**You test:**
- [ ] Each page has unique `<title>`
- [ ] `<meta name="description">` on all indexable pages
- [ ] OG tags present
- [ ] Canonical URL correct
- [ ] PDP: paste URL into [Google Rich Results Test](https://search.google.com/test/rich-results) → Product schema valid

**Commit:**
```bash
git add .
git commit -m "feat: verify and fix SEO meta tags across homepage, PLP, PDP"
```

**PR:** Push → PR → `phase/4-launch-prep`

---

### PROMPT 55 — Verify PostHog Events End-to-End

**Branch:**
```bash
git checkout phase/4-launch-prep && git pull
git checkout -b feat/analytics-security-perf
```

**Source:** PROMPT-GUIDE-v2.md → Section 6b → **"Verify PostHog Events End-to-End"**

**You test (manually in PostHog Live Events):**
- [ ] `product_viewed` fires on PDP
- [ ] `add_to_cart` fires on button click
- [ ] `search_query` fires on search
- [ ] `checkout_started` fires on "Proceed to Checkout"
- [ ] `order_complete` fires on confirmation page

**Commit:**
```bash
git add .
git commit -m "fix: verified all 5 PostHog events firing correctly"
```

---

### PROMPT 56 — Verify Sentry Is Receiving Errors

**Branch:** Stay on `feat/analytics-security-perf`

**Source:** PROMPT-GUIDE-v2.md → Section 6b → **"Verify Sentry is Receiving Errors"**

**You test:**
- [ ] Trigger test error → appears in Sentry Issues
- [ ] Source maps show TypeScript line numbers (not compiled JS)
- [ ] Two alerts configured: payment failure rate > 5%, error spike

**Commit:**
```bash
git add .
git commit -m "chore: verify Sentry in production, configure payment failure alert"
```

---

### PROMPTS 57–59 — Security Hardening

**Source:** PROMPT-GUIDE-v2.md → Section 6c → **3 security prompts**:
- **"Run npm Audit and Fix Vulnerabilities"**
- **"Scan Git History for Accidentally Committed Secrets"**
- **"Verify HTTP-only Cookie in Production"**

Run each in sequence. Fix every `high` or `critical` finding before continuing.

**Commits:**
```bash
git commit -m "fix: resolve high/critical npm audit vulnerabilities"
git commit -m "chore: verify git history clean — no secrets committed"
git commit -m "chore: verify auth token in HTTP-only cookie — not accessible via JS"
```

---

### PROMPT 60 — Run Lighthouse and Fix Failures

**Source:** PROMPT-GUIDE-v2.md → Section 6d → **"Run Lighthouse and Fix Failures"**

**Target scores (from NewDocs/01-product-requirements.md):**
- LCP < 2.5s on 4G mobile
- INP < 200ms
- CLS < 0.1
- Lighthouse score ≥ 90

Run on: Homepage, PLP `/collections/living-room`, any PDP

**Commit:**
```bash
git add .
git commit -m "perf: fix LCP, CLS, bundle size issues from Lighthouse audit"
```

---

### PROMPTS 61–65 — Go-Live

**Source:** PROMPT-GUIDE-v2.md → Section 6e:
- **"Configure Cloudflare DNS + Custom Domain"**
- **"Configure Vercel Production Environment"**
- **"Configure Railway Production Environment"**
- **"Verify Razorpay Webhook End-to-End in Production"**
- **"Pre-Launch Definition of Done Checklist"**

> ⚠️ The **Pre-Launch Definition of Done Checklist** (last prompt in Section 6e) must have every item ✅ before you merge to `main`.

**Final merge:**
```bash
# GitHub → PR: phase/4-launch-prep → develop
# GitHub → PR: develop → main      ← production deploy
```

---

## PHASE 4 — INTERRUPTION & RESUME GUIDE

### "Performance fixes are taking longer than expected"

```
@performance-optimizer We fixed [describe fixes] but Lighthouse on PDP still shows [score].
The remaining issue is [describe symptom].
Files involved: [list]
What is the fastest fix to get to ≥ 90?
```

---

---

# ═══════════════════════════════════════
# TESTING — ACROSS ALL PHASES
# ═══════════════════════════════════════

**Source:** PROMPT-GUIDE-v2.md → **Section 7 — Testing**

Write tests alongside or immediately after the feature they test. Don't leave all tests until Phase 4.

**Recommended timing:**
| Test | Write after |
|---|---|
| Price utility tests | After Prompt 18 (Build Price Utilities) |
| Cart store tests | After Prompt 19 (Build Zustand Stores) |
| Webhook signature tests | After Prompt 39 (Build Razorpay Webhook) |
| AddressForm Zod tests | After Prompt 36 (Build Checkout Step 1) |
| Auth form validation tests | After Prompt 41 (Build Auth Pages) |
| Cart API integration tests | After Prompt 16 (Build Medusa API Client) |
| Playwright — Purchase flow | After Prompt 40 (Order Confirmation) |
| Playwright — Auth flow | After Prompt 41 (Build Auth Pages) |

**Branch for all tests:**
```bash
git checkout -b feat/tests-[unit or e2e]
```

**Run tests:**
```bash
pnpm test                          # unit tests (Vitest)
pnpm test:e2e                      # Playwright
pnpm test --reporter=verbose       # with output
```

---

---

# ═══════════════════════════════════════
# ALL-SITUATION REFERENCE
# ═══════════════════════════════════════

## Section 9 — DevOps & CI/CD Prompts

These come from **PROMPT-GUIDE-v2.md → Section 9**. Use them whenever you're dealing with deployments, CI failures, or environment variables — at any phase.

---

### Deploy Storefront to Vercel (First Time — Phase 4)

**Branch:** Inside `feat/analytics-security-perf` or its own `feat/vercel-deploy`

**Source:** PROMPT-GUIDE-v2.md → Section 9 → **"Deploy to Vercel (First Time)"**

Paste into Antigravity:
```
@devops-engineer Set up the Vercel deployment for apps/storefront.
Ref: NewDocs/11-environment-and-devops.md.
Configure:
- Link monorepo root to Vercel project
- Set Root Directory: apps/storefront
- Framework Preset: Next.js
- Build Command: cd ../.. && pnpm build --filter=storefront
- Install Command: pnpm install --frozen-lockfile
- Add all NEXT_PUBLIC_ env vars from .env.example
- Add all server-only env vars from .env.example
- Enable Vercel Analytics (for Core Web Vitals)
Create .github/workflows/deploy.yml for auto-deploy on merge to main.
Update STATUS.md when done.
```

**You test:**
- [ ] Vercel preview URL loads the storefront
- [ ] No build errors in Vercel dashboard
- [ ] `NEXT_PUBLIC_MEDUSA_BACKEND_URL` points to Railway backend URL

**Commit:**
```bash
git add .
git commit -m "chore: configure Vercel deployment with monorepo root dir and env vars"
```

---

### Deploy Backend to Railway (First Time — Phase 4)

**Source:** PROMPT-GUIDE-v2.md → Section 9 → **"Deploy Backend to Railway (First Time)"**

Paste into Antigravity:
```
@devops-engineer Set up Railway deployment for the MedusaJS v2 backend.
Configure:
- Service name: shree-furniture-backend
- Root directory: backend/
- Build command: pnpm install && pnpm build
- Start command: node_modules/.bin/medusa start
- Health check path: /health
- Add all backend env vars from .env.example
- Post-deploy command: npx medusa db:migrate
Create .github/workflows/deploy.yml Railway deploy step (after Vercel deploy succeeds).
Update STATUS.md when done.
```

**You test:**
- [ ] `GET https://[railway-url]/health` → returns 200
- [ ] `GET https://[railway-url]/store/products` → returns products
- [ ] Medusa Admin at `https://[railway-url]/app/admin` → loads

**Commit:**
```bash
git add .
git commit -m "chore: configure Railway deployment for MedusaJS v2 backend"
```

---

### When CI Pipeline Fails

**Source:** PROMPT-GUIDE-v2.md → Section 9 → **"Debug CI Pipeline Failure"**

Paste into Antigravity (filling in the brackets):
```
@devops-engineer The GitHub Actions CI pipeline is failing.
Workflow: .github/workflows/ci.yml
Failed step: [which step: tsc / lint / test / build]
Error output: [paste the error log from GitHub Actions]
Branch: [branch name]
Check: are all required secrets set in GitHub Secrets? (TURBO_TOKEN if using remote cache)
```

**Common fixes:**
- `tsc` fails → TypeScript error in a file. Fix and push.
- `lint` fails → ESLint error. Run `pnpm lint` locally to see the exact line.
- `test` fails → a Vitest test broke. Run `pnpm test` locally.
- `build` fails → Next.js build error. Run `pnpm build` locally.

---

### Adding a New Environment Variable (Any Phase)

**Source:** PROMPT-GUIDE-v2.md → Section 9 → **"Add a New Environment Variable"**

Paste into Antigravity:
```
@devops-engineer I need to add a new environment variable: [VAR_NAME]
Value type: [public browser-safe / server-only secret]

1. Add to .env.example with a comment explaining what it is
2. If NEXT_PUBLIC_: add prefix; if server-only: no prefix
3. Add to apps/storefront/.env.local (dev) or backend/.env (dev) with placeholder
4. Add to .github/workflows/ci.yml env section if needed for CI
5. Document: add to Vercel dashboard (storefront) or Railway dashboard (backend)
6. Update NewDocs/11-environment-and-devops.md if it documents env vars
```

**Commit:**
```bash
git add .env.example .github/workflows/ci.yml
git commit -m "chore: add [VAR_NAME] env var to .env.example, CI, and docs"
```

---

## Section 10 — Session End (Exact Prompts)

**Source:** PROMPT-GUIDE-v2.md → Section 10. Use one of these every single session before closing.

### Standard Session End

Paste this at end of every session:
```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark completed items ✅ with a Notes entry describing the implementation
   - Mark partial items 🚧 with exactly what's left to do
   - If anything was started but not in STATUS.md's table, flag it
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I expressed any design preference
6. Update REJECTIONS.md if I rejected anything

Show me a summary of all changes before committing.
```

Then commit:
```bash
git add STATUS.md PREFERENCES.md REJECTIONS.md NewDocs/KNOWN-ISSUES.md
git commit -m "docs: end of session — STATUS, PREFERENCES, REJECTIONS, KNOWN-ISSUES"
git push
```

### Session End With a Note for Next Session (Use When Something Is Half-Done)

```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark [component] as ✅ DONE with a Notes entry
   - Add note: "[what the next session needs to know about this implementation]"
   - Mark partial items 🚧 with exactly what's left to do
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I confirmed a design decision today
6. Update REJECTIONS.md if I rejected an approach today

Show me a summary of all changes before committing.
```

**Example note in STATUS.md:**
```
ProductCard ✅ DONE
Note: "Uses optimistic update from cart-store. Hover uses translateY(-2px) shadow — see PREFERENCES.md Section 4."
```

---

## Section 11 — Anti-Patterns (What NOT to Say to the AI)

**Source:** PROMPT-GUIDE-v2.md → Section 11

These are prompts that will get you bad or wrong results. Avoid all of them.

| ❌ Never say this | ✅ Say this instead |
|---|---|
| `"Build me a backend"` | `"@backend-specialist Build [specific file] per NewDocs/06-api-contracts.md section [X]"` |
| `"Make a nice UI"` | `"@frontend-specialist Build [ComponentName] per NewDocs/01 section [X]"` |
| `"Use Prisma for the database"` | Never override the ORM — fixed in DECISIONS.md |
| `"Add reviews while you're here"` | Reviews are OUT of scope. Check NewDocs/09-engineering-scope-definition.md first |
| `"Just use any type here"` | `"Use unknown + type guard"` |
| `"Store the token in localStorage"` | HTTP-only cookies only. ADR-009. |
| `"Use floats for price"` | Paise integers only. ADR-007. `4999900` not `49999.00` |
| `"Build a custom cart"` | Medusa Cart Module handles this. Never rebuild it. |
| `"Fix everything"` | One file, one problem at a time |
| Long vague descriptions | Short + file path + NewDocs section reference |
| Forgetting "Update STATUS.md" | Always append it to every build prompt |

### Agent Names — Complete Reference

| Agent | Use for |
|---|---|
| `@backend-specialist` | MedusaJS v2, subscribers, workflows, modules, API routes |
| `@frontend-specialist` | Next.js pages, components, Zustand, TanStack Query, Tailwind |
| `@database-architect` | Schema, MikroORM entities, indexes, query optimisation |
| `@devops-engineer` | CI/CD, Railway, Vercel, Docker, GitHub Actions |
| `@test-engineer` | Vitest unit/integration tests, TDD |
| `@qa-automation-engineer` | Playwright E2E, CI test pipelines |
| `@security-auditor` | Webhook security, auth, OWASP audit |
| `@debugger` | Unexpected behavior, root cause analysis |
| `@performance-optimizer` | LCP, CLS, bundle size, Core Web Vitals |
| `@seo-specialist` | generateMetadata, JSON-LD, sitemap |
| `@orchestrator` | Tasks spanning multiple files or domains |
| `@explorer-agent` | "What files exist for X?", codebase mapping |

### Key NewDocs References — What Each File Is For

```
01 — product-requirements.md         Feature specs + acceptance criteria per page
03 — information-architecture.md     URL structure, page inventory, route groups
04 — system-architecture.md          Rendering strategy (ISR/CSR/SSR), data flows
05 — database-schema.md              All table definitions (check before any DB work)
06 — api-contracts.md                All Medusa endpoints + request/response shapes
07 — monorepo-structure.md           File locations, package structure, folder tree
08 — cart-pricing-engine-spec.md     Cart lifecycle, pricing, tax, shipping rules
09 — engineering-scope-definition.md What NOT to build — check before any new feature
10 — development-phases.md           Phase milestones, acceptance criteria, DoD checklist
12 — testing-strategy.md             What to test, test file locations, coverage targets
MEDUSA-V2-PATTERNS.md                v2 SDK methods, subscriber/workflow/module patterns
KNOWN-ISSUES.md                      Gotchas log — always check before debugging
CLAUDE.md                            Hard rules — auto-loaded every session
STATUS.md                            Current build state — read at start, update at end
PREFERENCES.md                       Your design taste — all UI agents read this first
REJECTIONS.md                        Never-again log — agents read before any suggestion
STACK-CHANGES.md                     Mid-project tech changes — supersede CLAUDE.md entries
14 — razorpay-integration-spec.md   Razorpay config, modal options, webhook events, HMAC, refunds
15 — algolia-schema.md              Index record schema, sync subscriber, index settings, replicas
16 — email-templates-spec.md        All 5 email templates — variables, layout, subject lines
17 — india-specific-guide.md        Phone/PIN validation, states list, GST display, Indian date format
18 — cloudinary-guide.md            Folder naming, transformation presets, Next.js image loader
design-reference.md                 IKEA-inspired visual language — read before any frontend work
```

---

## Session Start Prompt (Corrected Order)

```
Read these files before doing anything:
1. STATUS.md
2. PREFERENCES.md
3. REJECTIONS.md
4. STACK-CHANGES.md
5. CLAUDE.md

Tell me: current phase, last completed item, next 2–3 tasks.
Do not write any code yet.
```

## Session End Prompt

Open **PROMPT-GUIDE-v2.md → Section 10** → copy **"Standard Session End"**.

Then commit:
```bash
git add STATUS.md PREFERENCES.md REJECTIONS.md NewDocs/KNOWN-ISSUES.md
git commit -m "docs: end of session — STATUS, PREFERENCES, REJECTIONS, KNOWN-ISSUES"
git push
```

## If You Don't Like What Was Built

```
I don't like [ComponentName]: [what specifically]
I want instead: [what]
1. Add to REJECTIONS.md Section [1/2/3/4/5]
2. Rebuild [ComponentName]
3. Update STATUS.md
```

## If You Want to Save a Preference

```
Add to PREFERENCES.md Section [2/3/4/5]:
Element: [name]
What I liked: [description]
What I want different: [description]
Date: [today]
```

## Debugging

Open **PROMPT-GUIDE-v2.md → Section 8** → find matching scenario.

## Stack Change

See CHANGE-GUIDE.md for the full protocol. Short version:
```bash
git checkout -b chore/swap-[old]-to-[new]
```
Then: impact assessment → document in STACK-CHANGES.md → migrate one component at a time → update all reference files → PR to `develop`.

---

## Complete Branch & Prompt Reference

| Phase | Prompt | Branch |
|---|---|---|
| 1 | Scaffold Monorepo | `feat/monorepo-scaffold` |
| 1 | Root Layout + Providers | `feat/root-layout-providers` |
| 1 | Route Group Layouts | `feat/route-group-layouts` |
| 1 | Docker + GitHub + CI + next.config | `feat/docker-ci-setup` |
| 1 | MedusaJS Backend + Seed + Migrations | `feat/medusa-backend-config` |
| 1 | Algolia + Cloudinary + Resend + PostHog + Sentry | `feat/third-party-services` |
| 2 | Medusa API Client + Algolia Client + Utils | `feat/medusa-api-client` |
| 2 | Zustand + Middleware + TanStack Query | `feat/state-management` |
| 2 | Header + Footer + MobileMenu + Breadcrumb | `feat/layout-header-footer-nav` |
| 2 | ProductCard + PriceDisplay + Skeleton + EmptyState | `feat/product-card` |
| 2 | Homepage | `feat/homepage` |
| 2 | PLP | `feat/product-listing-page` |
| 2 | PDP | `feat/product-detail-page` |
| 2 | Search + Cart Page | `feat/search-feature` |
| 3 | CartDrawer + CartItem + CartSummary | `feat/cart-ui` |
| 3 | Checkout flow (Progress → Address → Shipping → Payment → Confirmation) | `feat/checkout-flow` |
| 3 | Razorpay Webhook | `feat/razorpay-webhook` |
| 3 | Login + Register + Forgot Password | `feat/auth-pages` |
| 3 | Account Layout + Orders + Wishlist + Addresses + Settings | `feat/account-pages` |
| 3 | All 4 Backend Subscribers | `feat/backend-subscribers` |
| 3 | Fulfillment Workflow | `feat/fulfillment-workflow` |
| 3 | Wishlist Module | `feat/wishlist-module` |
| 3 | ISR Revalidation Endpoint | `feat/isr-revalidation` |
| 4 | robots.txt + sitemap + SEO meta tags | `feat/seo-setup` |
| 4 | PostHog + Sentry + Security + Performance + Go-Live | `feat/analytics-security-perf` |
| All | Unit tests (Vitest) | `feat/tests-unit` |
| All | E2E tests (Playwright) | `feat/tests-e2e` |

---

*Shree Furniture | MY-PHASE-BY-PHASE-GUIDE.md | v1.0 — March 2026*
*Place at: `shreefurniture-pro/MY-PHASE-BY-PHASE-GUIDE.md` (monorepo root)*
