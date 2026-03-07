# 🧪 Shree Furniture – Test Cases Plan
> **Project:** Shree Furniture E-Commerce Platform  
> **Version:** 1.0 — Q1 2026  
> **Stack:** Next.js 15 · MedusaJS v2 · PostgreSQL · Razorpay · Algolia · Cloudinary · Resend  
> **Testing Tools:** Vitest · React Testing Library · Playwright · Zod  
> **Document Type:** Test Plan (not generated test code)  
> **Aligned With:** `NewDocs10-development-phases.md` · `NewDocs12-testing-strategy.md`

---

## 📋 Legend

| Symbol | Meaning |
|--------|---------|
| 🔴 P0 | Critical — must pass before any deploy |
| 🟡 P1 | High — must pass before phase completion |
| 🟢 P2 | Medium — should pass, deferrable to next sprint |
| **Unit** | Vitest — pure logic, no network |
| **Integration** | Vitest + local Medusa dev server |
| **E2E** | Playwright — full browser simulation |
| **Component** | React Testing Library |
| **Security** | Manual + script-assisted |

---

## Phase 1 — Foundation (Weeks 1–2)

### Week 1 · Monorepo & Dev Environment

#### TC-1.1 — Monorepo Build Pipeline

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-1.1.1 | `pnpm dev` starts all apps concurrently without port conflict | Manual | 🔴 P0 | storefront on :3000, backend on :9000, no crash |
| TC-1.1.2 | `pnpm build` completes without TypeScript or ESLint errors | Unit | 🔴 P0 | Exit code 0, zero `tsc` errors in strict mode |
| TC-1.1.3 | CI pipeline triggers on PR and runs `tsc`, `eslint`, `build` | Integration | 🔴 P0 | GitHub Actions workflow passes green |
| TC-1.1.4 | `docker-compose up` starts PostgreSQL 16 and Redis 7 cleanly | Manual | 🔴 P0 | Both containers healthy, no port conflicts |
| TC-1.1.5 | `.env.example` documents all required environment variables | Manual | 🟡 P1 | Every env var used in code is present in `.env.example` |
| TC-1.1.6 | `packages/types` exports compile without errors | Unit | 🟡 P1 | `tsc --noEmit` passes on `packages/types/src/index.ts` |
| TC-1.1.7 | `packages/ui` design tokens are importable in storefront | Unit | 🟢 P2 | Token import resolves without error |
| TC-1.1.8 | Branch protection on `main` blocks direct pushes | Manual | 🟡 P1 | Attempt direct push to `main` is rejected by GitHub |

---

#### TC-1.2 — Dev Environment Integrity

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-1.2.1 | TypeScript strict mode enabled across all workspaces | Unit | 🔴 P0 | `tsconfig.json` has `"strict": true`, CI confirms |
| TC-1.2.2 | No `any` types without explicit justification comment | Unit | 🟡 P1 | ESLint `@typescript-eslint/no-explicit-any` rule passes |
| TC-1.2.3 | Path aliases (`@/`, `features/`, `shared/`) resolve correctly | Unit | 🟡 P1 | Import using aliases compiles without errors |
| TC-1.2.4 | Turborepo task graph resolves correct build order | Manual | 🟢 P2 | `turbo run build --dry` shows correct dependency graph |

---

### Week 2 · MedusaJS Backend Live

#### TC-1.3 — Backend Deployment & Core API

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-1.3.1 | `GET /store/products` returns seeded products from Railway | Integration | 🔴 P0 | 200 OK, array with ≥1 product, correct schema |
| TC-1.3.2 | Medusa Admin UI is accessible at `/admin` | Manual | 🔴 P0 | Dashboard loads, admin login works |
| TC-1.3.3 | Product CRUD works via Medusa Admin | Manual | 🔴 P0 | Create/edit/delete product completes without error |
| TC-1.3.4 | PostgreSQL (Neon.tech) migrations run without errors | Integration | 🔴 P0 | `medusa db:migrate` exits 0, no failed migrations |
| TC-1.3.5 | Redis (Upstash) connects and sessions are stored | Integration | 🔴 P0 | Login creates session, session key visible in Upstash |
| TC-1.3.6 | Region India/INR is configured with correct currency | Manual | 🟡 P1 | `GET /store/regions` returns INR region |
| TC-1.3.7 | Cloudinary plugin configured — test image upload succeeds | Manual | 🟡 P1 | Upload via Admin stores CDN URL, not local path |
| TC-1.3.8 | Resend plugin sends test order confirmation email | Manual | 🟡 P1 | Test email received at verified address |
| TC-1.3.9 | Razorpay sandbox credentials are wired to payment plugin | Manual | 🔴 P0 | `GET /store/payment-providers` returns `pp_razorpay_razorpay` |
| TC-1.3.10 | Algolia index synced — test products appear in Algolia dashboard | Manual | 🟡 P1 | Products visible in Algolia index via dashboard |
| TC-1.3.11 | Health check endpoint `/health` responds 200 | Integration | 🔴 P0 | Railway health check shows healthy |

---

## Phase 2 — Storefront Core (Weeks 3–4)

### Week 3 · Homepage & Product Listing Page

#### TC-2.1 — Homepage

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-2.1.1 | Homepage renders with ISR — HTML served on first load | Manual | 🔴 P0 | Page source contains rendered HTML, not empty shell |
| TC-2.1.2 | Hero section renders headline, CTA button, full-width banner | Component | 🟡 P1 | All three elements visible in DOM |
| TC-2.1.3 | Featured Collections grid shows Living Room, Bedroom, Dining, Office | Component | 🟡 P1 | 4 collection cards rendered with correct labels |
| TC-2.1.4 | Best Sellers carousel renders at least 4 products | Component | 🟡 P1 | `data-testid="best-sellers"` contains ≥4 product cards |
| TC-2.1.5 | Trust signals strip is visible below hero | Component | 🟢 P2 | Section renders without layout break |
| TC-2.1.6 | Homepage LCP is ≤2.5s on 4G throttle simulation | E2E | 🔴 P0 | Lighthouse/Playwright trace confirms LCP ≤2.5s |
| TC-2.1.7 | All collection links in homepage grid navigate to correct PLP | E2E | 🟡 P1 | Click each collection card → URL changes to `/collections/[handle]` |
| TC-2.1.8 | Homepage renders correctly at 360px, 768px, 1280px | Manual | 🟡 P1 | No layout overflow, text readable, images not clipped |

---

#### TC-2.2 — Header & Navigation

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-2.2.1 | Header renders logo, nav links, search icon, wishlist icon, cart icon | Component | 🟡 P1 | All 5 elements in DOM |
| TC-2.2.2 | Cart icon badge shows correct item count | Component | 🟡 P1 | Badge count matches Zustand `cart.items.length` |
| TC-2.2.3 | Mobile menu opens and closes correctly | Component | 🟡 P1 | Hamburger tap toggles `MobileMenu` visibility |
| TC-2.2.4 | Navigation links resolve to correct routes | E2E | 🟡 P1 | Each nav link navigates to expected URL |
| TC-2.2.5 | Footer renders navigation links and payment icons | Component | 🟢 P2 | Footer links and payment icons in DOM |

---

#### TC-2.3 — Product Listing Page (PLP)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-2.3.1 | PLP loads with ISR and real product data | Integration | 🔴 P0 | Products render on page, data from Medusa |
| TC-2.3.2 | ProductCard shows thumbnail, name, price, and discount badge | Component | 🟡 P1 | All four fields rendered per card |
| TC-2.3.3 | Product grid is responsive — 2/3/4 columns at breakpoints | Manual | 🟡 P1 | Grid adapts at 360px, 768px, 1280px |
| TC-2.3.4 | Filter panel renders on desktop sidebar, bottom sheet on mobile | Component | 🟡 P1 | Sidebar visible ≥768px; bottom sheet visible <768px |
| TC-2.3.5 | Applying a filter updates URL query params | E2E | 🟡 P1 | Filter click changes `?filter=` in URL without page reload |
| TC-2.3.6 | Sort options change product order in grid | E2E | 🟡 P1 | Sort by price shows ascending/descending order |
| TC-2.3.7 | "Load More" fetches next page of products | E2E | 🟡 P1 | Click Load More → new products appended to grid |
| TC-2.3.8 | Empty state renders when no products match filter | Component | 🟢 P2 | "No products found" message appears |
| TC-2.3.9 | Breadcrumb component shows correct path on PLP | Component | 🟢 P2 | `Home > Living Room` breadcrumb rendered |
| TC-2.3.10 | PLP LCP ≤2.5s on 4G simulation | E2E | 🔴 P0 | Lighthouse confirms LCP metric |

---

### Week 4 · Product Detail Page & Search

#### TC-2.4 — Product Detail Page (PDP)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-2.4.1 | PDP loads via ISR with correct product data | Integration | 🔴 P0 | Correct product title, price, images rendered |
| TC-2.4.2 | ProductGallery shows primary image and thumbnail strip | Component | 🟡 P1 | Thumbnail click switches primary image |
| TC-2.4.3 | Variant selection updates price, images, and stock | E2E | 🔴 P0 | Click colour/size swatch → price and image update without reload |
| TC-2.4.4 | Stock status shows "In Stock", "Only n left", or "Out of Stock" | Component | 🟡 P1 | Status reflects `inventory_quantity` from Medusa |
| TC-2.4.5 | Quantity selector cannot exceed `inventory_quantity` | Component | 🔴 P0 | Max value capped, increment disabled at max |
| TC-2.4.6 | "Add to Cart" opens CartDrawer | E2E | 🔴 P0 | Click triggers `data-testid="cart-drawer"` visibility |
| TC-2.4.7 | "Add to Wishlist" prompts login for guests | E2E | 🟡 P1 | Click redirects unauthenticated user to login |
| TC-2.4.8 | Product description and specifications table render | Component | 🟢 P2 | Rich text and table present in DOM |
| TC-2.4.9 | Related Products carousel renders ≥3 products | Component | 🟢 P2 | `data-testid="related-products"` has ≥3 cards |
| TC-2.4.10 | Delivery estimate shown for valid PIN code | E2E | 🟡 P1 | Enter serviceable PIN → delivery date appears |
| TC-2.4.11 | Invalid product handle renders `not-found.tsx` (404) | E2E | 🟡 P1 | `/products/invalid-handle` returns 404 page |
| TC-2.4.12 | JSON-LD Product schema present in `<head>` | Manual | 🟡 P1 | Schema validates in Google Rich Results Test |
| TC-2.4.13 | `generateMetadata` produces correct title and description | Unit | 🟡 P1 | Metadata matches product title and description |

---

#### TC-2.5 — Search (Algolia InstantSearch)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-2.5.1 | Search bar in header debounces input at 300ms | Unit | 🟡 P1 | Rapid typing only triggers query after 300ms pause |
| TC-2.5.2 | SearchOverlay opens on search icon click | Component | 🟡 P1 | Overlay is full-screen on mobile, modal on desktop |
| TC-2.5.3 | Typing returns Algolia results within 200ms | E2E | 🔴 P0 | Network trace shows response ≤200ms |
| TC-2.5.4 | "No results" state shows category suggestions | Component | 🟢 P2 | No results state renders with navigation links |
| TC-2.5.5 | Clicking a search result navigates to correct PDP | E2E | 🟡 P1 | Product click → URL matches `/products/[handle]` |
| TC-2.5.6 | Search overlay closes on Escape key or backdrop click | Component | 🟢 P2 | Overlay hidden after dismiss action |

---

## Phase 3 — Transactions (Weeks 5–6)

### Week 5 · Cart State Management

#### TC-3.1 — Zustand Cart Store (Unit Tests)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.1.1 | `addItemOptimistic` adds item to cart immediately | Unit | 🔴 P0 | `items.length` increments before API response |
| TC-3.1.2 | Adding same variant twice increments `quantity`, not item count | Unit | 🔴 P0 | One item with `quantity: 2` in store |
| TC-3.1.3 | `rollbackOptimisticAdd` removes item on API failure | Unit | 🔴 P0 | Store reverts to pre-add state |
| TC-3.1.4 | `removeItem` removes correct item by variant ID | Unit | 🔴 P0 | Item removed, `total` recalculated |
| TC-3.1.5 | `updateQuantity` updates quantity and recalculates total | Unit | 🟡 P1 | `total` reflects updated quantity × unit_price |
| TC-3.1.6 | Cart initialises empty with `total: 0` | Unit | 🟡 P1 | Initial state matches empty cart schema |
| TC-3.1.7 | Cart `total` is always in paise (integer, no decimals) | Unit | 🔴 P0 | `total % 1 === 0` for all operations |

---

#### TC-3.2 — Cart UI Components

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.2.1 | CartDrawer slides in when item is added | E2E | 🔴 P0 | Drawer opens within 500ms of "Add to Cart" click |
| TC-3.2.2 | CartItem renders thumbnail, name, variant, quantity stepper, remove | Component | 🟡 P1 | All 5 elements in each cart item |
| TC-3.2.3 | Quantity stepper in CartItem increments and decrements | E2E | 🟡 P1 | `+` / `−` buttons update quantity and subtotal |
| TC-3.2.4 | Remove button removes item from drawer | E2E | 🟡 P1 | Item disappears from CartDrawer after click |
| TC-3.2.5 | CartSummary shows subtotal, shipping estimate, GST note | Component | 🟡 P1 | All three elements present |
| TC-3.2.6 | Free shipping progress bar shows correct threshold | Component | 🟢 P2 | Progress reflects cart total vs. free shipping threshold |
| TC-3.2.7 | Empty cart state renders in CartDrawer | Component | 🟢 P2 | "Your cart is empty" message with CTA |
| TC-3.2.8 | Header cart badge count updates after add/remove | E2E | 🟡 P1 | Badge reflects live cart item count |

---

#### TC-3.3 — Cart Persistence

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.3.1 | Cart persists across browser refresh | E2E | 🔴 P0 | Refresh page → cart items still present |
| TC-3.3.2 | `cartId` stored in cookie, not localStorage | Manual | 🔴 P0 | `document.cookie` contains `cartId`, localStorage is empty |
| TC-3.3.3 | Cart syncs with Medusa API via Redis backend | Integration | 🔴 P0 | Cart ID from API matches cookie, items match DB |
| TC-3.3.4 | Quantity cannot exceed `inventory_quantity` | Unit | 🔴 P0 | Add request rejected when quantity > stock |

---

### Week 6 · Checkout & Razorpay

#### TC-3.4 — Address Step (Step 1)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.4.1 | `addressSchema` validates a correct Indian address | Unit | 🔴 P0 | `safeParse` returns `success: true` |
| TC-3.4.2 | `addressSchema` rejects invalid PIN code (not 6 digits) | Unit | 🔴 P0 | Error on `postalCode` field |
| TC-3.4.3 | `addressSchema` rejects invalid phone number (<10 digits) | Unit | 🔴 P0 | Error on `phone` field |
| TC-3.4.4 | `addressSchema` rejects invalid email format | Unit | 🔴 P0 | Error on `email` field |
| TC-3.4.5 | Guest email field is visible and required for unauthenticated users | Component | 🔴 P0 | Guest email field present, empty submit blocked |
| TC-3.4.6 | Saved addresses dropdown populates for logged-in user | Integration | 🟡 P1 | `GET /store/customers/me` addresses shown in dropdown |
| TC-3.4.7 | Selecting saved address pre-fills form | E2E | 🟡 P1 | Dropdown selection populates all address fields |
| TC-3.4.8 | "Continue to Shipping" is blocked until all required fields valid | E2E | 🔴 P0 | Button disabled with invalid form state |
| TC-3.4.9 | Progress indicator shows Step 1 active | Component | 🟢 P2 | Step 1 highlighted in checkout progress bar |

---

#### TC-3.5 — Shipping Step (Step 2)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.5.1 | Shipping options load from Medusa after address submission | Integration | 🔴 P0 | `GET /store/shipping-options` returns options |
| TC-3.5.2 | Each shipping option shows name, price, and delivery estimate | Component | 🟡 P1 | Three fields visible per option |
| TC-3.5.3 | Free shipping threshold is shown below option list | Component | 🟢 P2 | Threshold message renders correctly |
| TC-3.5.4 | Selecting a shipping option enables "Continue to Payment" | E2E | 🟡 P1 | CTA enabled only after option is selected |

---

#### TC-3.6 — Payment Step & Razorpay (Step 3)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.6.1 | Order summary on payment step matches cart items and totals | Component | 🔴 P0 | Line items, shipping cost, total match cart state |
| TC-3.6.2 | `POST /store/carts/:id/payment-sessions` called before modal opens | Integration | 🔴 P0 | API called, payment session created |
| TC-3.6.3 | Razorpay modal opens on "Place Order" click | E2E | 🔴 P0 | Razorpay iframe visible after button click |
| TC-3.6.4 | Payment failure shows inline error, no page redirect | E2E | 🔴 P0 | Error message in DOM, URL remains `/checkout/payment` |
| TC-3.6.5 | Payment failure allows retry without losing cart state | E2E | 🔴 P0 | Cart intact after failed payment attempt |
| TC-3.6.6 | Successful payment redirects to `/orders/confirm/:id` | E2E | 🔴 P0 | URL matches order confirmation pattern |
| TC-3.6.7 | Order confirmation page shows full order summary (SSR) | E2E | 🔴 P0 | Items, address, total visible on confirmation page |

---

#### TC-3.7 — Razorpay Webhook (Security Critical)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.7.1 | `verifyRazorpaySignature` returns `true` for valid HMAC | Unit | 🔴 P0 | Computed HMAC matches header |
| TC-3.7.2 | `verifyRazorpaySignature` returns `false` for tampered body | Unit | 🔴 P0 | Body mismatch → `false` returned |
| TC-3.7.3 | `verifyRazorpaySignature` returns `false` for invalid signature string | Unit | 🔴 P0 | Invalid signature → `false` |
| TC-3.7.4 | Webhook handler rejects requests with missing signature header | Integration | 🔴 P0 | Returns `400` when header absent |
| TC-3.7.5 | Idempotency check prevents duplicate order creation | Integration | 🔴 P0 | Second identical webhook → `200` but no duplicate order |
| TC-3.7.6 | `carts/:id/complete` called only after signature is verified | Integration | 🔴 P0 | No order created for tampered webhook |
| TC-3.7.7 | `order.placed` event triggers Resend confirmation email | Integration | 🔴 P0 | Email received at test address after webhook fires |
| TC-3.7.8 | Webhook endpoint whitelisted to Razorpay IPs only | Security | 🔴 P0 | Request from non-Razorpay IP returns `403` |

---

#### TC-3.8 — Price & Utility Functions (Unit)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.8.1 | `formatPrice(4999900)` returns `"₹49,999"` | Unit | 🔴 P0 | Exact string match |
| TC-3.8.2 | `formatPrice(100)` returns `"₹1"` | Unit | 🔴 P0 | Exact string match |
| TC-3.8.3 | `formatPrice(0)` returns `"₹0"` | Unit | 🟡 P1 | Zero case handled |
| TC-3.8.4 | `getDiscountPercent(5999900, 4999900)` returns `17` | Unit | 🔴 P0 | Correct percentage |
| TC-3.8.5 | `getDiscountPercent(5000, 5000)` returns `0` | Unit | 🟡 P1 | No discount case |
| TC-3.8.6 | `calculateGST(4999900, 0.12)` returns correct base and GST amounts | Unit | 🔴 P0 | `gstAmount: 535700`, `baseAmount: 4464200` |
| TC-3.8.7 | All monetary values stored and returned as integers (paise) | Unit | 🔴 P0 | No float values in price computations |

---

#### TC-3.9 — Authentication & Account

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.9.1 | Register creates new customer and redirects to `/account/orders` | E2E | 🔴 P0 | New account created, redirect confirmed |
| TC-3.9.2 | Login with valid credentials sets HTTP-only session cookie | E2E | 🔴 P0 | Cookie set, `document.cookie` does NOT expose JWT |
| TC-3.9.3 | Login with invalid credentials shows error, no redirect | E2E | 🔴 P0 | Error message visible, URL unchanged |
| TC-3.9.4 | Logout clears session cookie | E2E | 🔴 P0 | Cookie absent after logout |
| TC-3.9.5 | "Forgot Password" sends OTP to provided email | Integration | 🟡 P1 | API returns `204`, email sent via Resend |
| TC-3.9.6 | Password reset with valid OTP updates password | Integration | 🟡 P1 | Login with new password succeeds |
| TC-3.9.7 | Auth-guarded account pages redirect guests to login | E2E | 🔴 P0 | `/account/orders` → redirect to `/account/login` for guests |
| TC-3.9.8 | My Orders page lists correct orders for logged-in user | Integration | 🟡 P1 | Orders match `GET /store/orders` response |
| TC-3.9.9 | Order detail page shows correct order data | Integration | 🟡 P1 | Line items, status, total displayed correctly |

---

#### TC-3.10 — Wishlist

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-3.10.1 | `GET /store/wishlist` returns wishlist for authenticated user | Integration | 🟡 P1 | 200 OK with wishlist array |
| TC-3.10.2 | `POST /store/wishlist` adds product to wishlist | Integration | 🟡 P1 | Product appears in next `GET /store/wishlist` |
| TC-3.10.3 | `DELETE /store/wishlist/:itemId` removes item from wishlist | Integration | 🟡 P1 | Item absent in subsequent GET |
| TC-3.10.4 | Wishlist page renders items with image, name, price | E2E | 🟡 P1 | Correct product data displayed |
| TC-3.10.5 | Unauthenticated wishlist action prompts login | E2E | 🟡 P1 | Login modal or redirect triggered |

---

## Phase 4 — Launch Prep (Week 7)

### TC-4.1 — End-to-End Purchase Flow (Full Smoke Test)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.1.1 | Full flow: browse → PDP → add to cart → checkout → pay → confirm | E2E | 🔴 P0 | All steps complete, order confirmation displayed |
| TC-4.1.2 | Full flow on iOS Safari (mobile) | E2E | 🔴 P0 | No layout breaks, Razorpay modal works |
| TC-4.1.3 | Full flow on Android Chrome (mobile) | E2E | 🔴 P0 | No layout breaks, payment completes |
| TC-4.1.4 | Full flow on Desktop Chrome | E2E | 🔴 P0 | Full desktop checkout experience |
| TC-4.1.5 | Full flow on Desktop Safari | E2E | 🔴 P0 | No Safari-specific rendering issues |
| TC-4.1.6 | Full flow on Desktop Firefox | E2E | 🟡 P1 | No Firefox-specific issues |
| TC-4.1.7 | Full flow on 4G throttling simulation | E2E | 🔴 P0 | Completes without timeout, LCP ≤2.5s on PDP |
| TC-4.1.8 | Guest checkout completes without account creation | E2E | 🔴 P0 | Order placed with guest email only |

---

### TC-4.2 — Edge Case Testing

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.2.1 | Add out-of-stock variant shows error, does not add to cart | E2E | 🔴 P0 | "Out of Stock" shown, cart unchanged |
| TC-4.2.2 | PIN code not serviceable shows delivery unavailable message | E2E | 🟡 P1 | Error message on PIN code check |
| TC-4.2.3 | Payment failure with retry completes successfully | E2E | 🔴 P0 | Retry after failure leads to confirmed order |
| TC-4.2.4 | Duplicate Razorpay webhook does not create duplicate order | Integration | 🔴 P0 | Idempotency check confirmed in logs |
| TC-4.2.5 | Session expiry during checkout prompts re-login | E2E | 🟡 P1 | Expired token → redirect to login |
| TC-4.2.6 | Rage-clicking "Add to Cart" does not create multiple cart lines | E2E | 🔴 P0 | Quantity increments, not duplicate items |
| TC-4.2.7 | Rage-clicking "Place Order" does not trigger duplicate orders | E2E | 🔴 P0 | Button disabled after first click |
| TC-4.2.8 | XSS payload in address fields is sanitised | Security | 🔴 P0 | `<script>alert(1)</script>` not executed |
| TC-4.2.9 | SQL injection attempt in search input is handled | Security | 🔴 P0 | No DB error, graceful empty result |
| TC-4.2.10 | Slow network (Slow 3G) — cart drawer still opens within acceptable time | E2E | 🟡 P1 | Drawer opens with loading state, does not crash |

---

### TC-4.3 — Performance Audit

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.3.1 | Homepage LCP ≤2.5s (Lighthouse, 4G mobile) | Manual | 🔴 P0 | Lighthouse score confirms |
| TC-4.3.2 | PLP LCP ≤2.5s | Manual | 🔴 P0 | Lighthouse score confirms |
| TC-4.3.3 | PDP LCP ≤2.5s | Manual | 🔴 P0 | Lighthouse score confirms |
| TC-4.3.4 | INP ≤200ms on all key pages | Manual | 🔴 P0 | Lighthouse INP metric |
| TC-4.3.5 | CLS ≤0.1 on all key pages | Manual | 🔴 P0 | No layout shift on image or font load |
| TC-4.3.6 | Initial JS bundle ≤200KB (next-bundle-analyzer) | Manual | 🔴 P0 | Bundle report shows ≤200KB |
| TC-4.3.7 | All `<Image>` components have explicit `width` and `height` | Unit | 🔴 P0 | ESLint/code scan passes, zero CLS from images |
| TC-4.3.8 | Core Web Vitals verified in Vercel Analytics | Manual | 🟡 P1 | Vercel dashboard shows green CWV |

---

### TC-4.4 — Security Hardening

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.4.1 | No secrets committed to Git history | Security | 🔴 P0 | `git log` audit and `gitleaks` scan pass clean |
| TC-4.4.2 | `npm audit` returns no high/critical vulnerabilities | Security | 🔴 P0 | Zero unaddressed high/critical CVEs |
| TC-4.4.3 | Razorpay webhook signature verification active in production logs | Security | 🔴 P0 | Production log shows signature verified per webhook |
| TC-4.4.4 | Auth rate limiting: 5 failed attempts triggers 429 response | Security | 🔴 P0 | `POST /store/customers/auth` — 6th attempt returns 429 |
| TC-4.4.5 | HTTP-only cookie confirmed — JWT not accessible via `document.cookie` | Security | 🔴 P0 | Browser console: `document.cookie` does not contain JWT |
| TC-4.4.6 | CORS is restricted to production domain | Security | 🟡 P1 | Cross-origin request from unknown origin returns 403 |
| TC-4.4.7 | `security` headers present (helmet): `X-Frame-Options`, `CSP`, `HSTS` | Security | 🟡 P1 | Response headers inspected, all present |
| TC-4.4.8 | `.env` files are not in Git repository | Security | 🔴 P0 | `.gitignore` includes `.env.local`, `.env` |

---

### TC-4.5 — SEO & Structured Data

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.5.1 | `robots.txt` excludes `/checkout`, `/cart`, `/account` | Manual | 🔴 P0 | Screaming Frog or curl confirms exclusion |
| TC-4.5.2 | `sitemap.xml` generated and includes PLP and PDP URLs | Manual | 🔴 P0 | Sitemap accessible and valid |
| TC-4.5.3 | Sitemap submitted to Google Search Console | Manual | 🟡 P1 | GSC shows sitemap submitted without errors |
| TC-4.5.4 | Canonical tags correct on all PLP and PDP pages | Manual | 🟡 P1 | `<link rel="canonical">` points to self |
| TC-4.5.5 | JSON-LD Product schema validates on all PDP pages | Manual | 🔴 P0 | Google Rich Results Test: valid schema |
| TC-4.5.6 | BreadcrumbList JSON-LD schema valid on PDP | Manual | 🟡 P1 | Schema test confirms breadcrumb structured data |
| TC-4.5.7 | Meta title ≤60 chars, meta description ≤160 chars on all pages | Manual | 🟡 P1 | Screaming Frog confirms |
| TC-4.5.8 | All collection pages are crawlable | Manual | 🟡 P1 | Screaming Frog shows 200 status on collection URLs |

---

### TC-4.6 — Go-Live Verification

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-4.6.1 | Custom domain live on Vercel with Cloudflare CDN | Manual | 🔴 P0 | `https://shreefurniture.in` resolves and loads |
| TC-4.6.2 | SSL/HTTPS active, no mixed content warnings | Manual | 🔴 P0 | Padlock shown, console has no mixed content errors |
| TC-4.6.3 | MedusaJS production URL configured in storefront env | Manual | 🔴 P0 | API calls go to production Railway URL |
| TC-4.6.4 | Sentry receiving events in production | Manual | 🔴 P0 | Trigger test error → appears in Sentry dashboard |
| TC-4.6.5 | PostHog receiving events in production | Manual | 🔴 P0 | Browse → event visible in PostHog Live View |
| TC-4.6.6 | Razorpay production webhook URL configured | Manual | 🔴 P0 | Razorpay dashboard shows correct webhook URL |
| TC-4.6.7 | Razorpay webhook end-to-end verified in production | Manual | 🔴 P0 | Production test purchase triggers confirmed order + email |
| TC-4.6.8 | Order confirmation email received within 2 minutes | Manual | 🔴 P0 | Test purchase → email in inbox ≤2 min |
| TC-4.6.9 | Admin can publish a product in under 5 minutes | Manual | 🟡 P1 | Timed run: product live on storefront within 5 min |

---

## Regression Suite — CI/CD Gates

### TC-R.1 — PR Gate (Every Pull Request)

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-R.1.1 | `tsc --noEmit` passes with zero errors | Unit | 🔴 P0 | CI step exits 0 |
| TC-R.1.2 | ESLint passes with zero errors | Unit | 🔴 P0 | CI step exits 0 |
| TC-R.1.3 | `pnpm build` completes without build errors | Unit | 🔴 P0 | Build step exits 0 |
| TC-R.1.4 | All Vitest unit tests pass | Unit | 🔴 P0 | Zero failing tests |
| TC-R.1.5 | Code coverage on critical paths ≥80% | Unit | 🟡 P1 | Vitest coverage report shows ≥80% on `lib/utils` and cart store |

---

### TC-R.2 — Nightly / Pre-Deploy Suite

| ID | Test Case | Type | Priority | Pass Condition |
|----|-----------|------|----------|----------------|
| TC-R.2.1 | Playwright purchase flow E2E passes on staging | E2E | 🔴 P0 | All steps in `e2e/purchase-flow.spec.ts` pass |
| TC-R.2.2 | Playwright auth flow E2E passes on staging | E2E | 🔴 P0 | All steps in `e2e/auth.spec.ts` pass |
| TC-R.2.3 | `securityscan.py` returns no critical issues | Security | 🔴 P0 | Script exits 0 |
| TC-R.2.4 | `dependencyanalyzer.py` returns no high/critical CVEs | Security | 🟡 P1 | Script exits 0 |
| TC-R.2.5 | `lintrunner.py` passes | Unit | 🔴 P0 | Script exits 0 |
| TC-R.2.6 | `lighthouseaudit.py` reports LCP ≤2.5s on key pages | Manual | 🔴 P0 | All three key pages pass |
| TC-R.2.7 | `bundleanalyzer.py` confirms bundle ≤200KB | Manual | 🟡 P1 | Report below threshold |

---

## Test Coverage Targets

| Module | Target Coverage | Tool |
|--------|----------------|------|
| `lib/utils/price.ts` | 100% | Vitest |
| `store/cart-store.ts` | 100% | Vitest |
| `api/webhooks/razorpay/verify.ts` | 100% | Vitest |
| `components/checkout/AddressForm.tsx` | 80% | Vitest + RTL |
| `lib/medusa/` client wrappers | 70% | Vitest (integration) |
| `components/` UI components | As needed | RTL |
| E2E Purchase Flow | Fully automated | Playwright |
| E2E Auth Flow | Fully automated | Playwright |

---

## Test File Location Map

```
ShreeFurniture-Pro/
├── apps/storefront/
│   ├── lib/utils/price.test.ts                      # TC-3.8.*
│   ├── store/cart-store.test.ts                      # TC-3.1.*
│   ├── api/webhooks/razorpay/verify.test.ts          # TC-3.7.1–3
│   ├── components/checkout/AddressForm.test.ts       # TC-3.4.1–4
│   └── lib/medusa/cart.integration.test.ts           # TC-3.3.*
├── e2e/
│   ├── purchase-flow.spec.ts                         # TC-4.1.*
│   ├── auth.spec.ts                                  # TC-3.9.*
│   ├── cart.spec.ts                                  # TC-3.2.*
│   ├── pdp.spec.ts                                   # TC-2.4.*
│   ├── search.spec.ts                                # TC-2.5.*
│   └── wishlist.spec.ts                              # TC-3.10.*
└── .github/workflows/
    ├── ci.yml                                        # TC-R.1.*
    └── deploy.yml                                    # TC-R.2.*
```

---

## Out of Scope (MVP)

> These features are deferred to Phase 2/3 per `NewDocs09-engineering-scope-definition.md`. No test cases required for MVP.

- Reviews & Ratings
- Loyalty / Referral program
- AR / 3D product viewer
- GST-compliant invoicing
- SMS notifications
- Multi-currency support
- Bulk CSV product import
- Advanced discount engine
- Inventory reservation on cart add

---

*Owner: Engineering Lead — Shree Furniture v1.0 · Q1 2026*  
*Last Updated: March 2026*
