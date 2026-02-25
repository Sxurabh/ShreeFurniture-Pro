# 12 — Testing Strategy
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Philosophy:** Test business-critical flows. Don't test the framework.

---

## 1. Testing Philosophy

We test what matters to the business and what cannot be caught by TypeScript alone. We do **not** write tests for:
- Third-party library internals (Medusa, Razorpay, Algolia)
- Simple UI rendering (that's what Storybook is for in Phase 3)
- Framework behaviour (Next.js routing, React state)

We **do** write tests for:
- Price computation logic (paise arithmetic, GST, discounts)
- Cart state management (add, remove, update, optimistic rollback)
- Webhook security (HMAC signature verification)
- Idempotency logic (duplicate order prevention)
- Form validation schemas (checkout address, auth forms)
- Utility functions (formatPrice, getDeliveryEstimate, PIN code validation)

---

## 2. Testing Stack

| Tool | Purpose | Runs In |
|---|---|---|
| **Vitest** | Unit and integration tests | CI + local |
| **React Testing Library** | Component behaviour tests (critical flows) | CI + local |
| **Playwright** | End-to-end purchase flow tests | CI (pre-deploy) |
| **Zod** | Schema validation (doubles as runtime tests) | Runtime |

---

## 3. Unit Tests (Vitest)

### 3.1 Price Utilities (`lib/utils/price.test.ts`)

```typescript
import { describe, it, expect } from 'vitest'
import { formatPrice, getDiscountPercent, calculateGST } from './price'

describe('formatPrice', () => {
  it('converts paise to INR string', () => {
    expect(formatPrice(4999900)).toBe('₹49,999')
    expect(formatPrice(100)).toBe('₹1')
    expect(formatPrice(50000)).toBe('₹500')
  })

  it('handles zero', () => {
    expect(formatPrice(0)).toBe('₹0')
  })
})

describe('getDiscountPercent', () => {
  it('calculates discount correctly', () => {
    expect(getDiscountPercent(5999900, 4999900)).toBe(17)
    expect(getDiscountPercent(10000, 8000)).toBe(20)
  })

  it('returns 0 if no discount', () => {
    expect(getDiscountPercent(5000, 5000)).toBe(0)
  })
})

describe('calculateGST', () => {
  it('extracts GST from inclusive price at 12%', () => {
    // ₹49,999 inclusive of 12% GST
    const result = calculateGST(4999900, 0.12)
    expect(result.gstAmount).toBe(535700)   // ~₹5,357
    expect(result.baseAmount).toBe(4464200)  // ~₹44,642
  })
})
```

---

### 3.2 Cart Store (`store/cart-store.test.ts`)

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { useCartStore } from './cart-store'
import { act } from 'react'

describe('Cart Store', () => {
  beforeEach(() => {
    useCartStore.setState({ items: [], total: 0 })
  })

  it('adds an item to the cart (optimistic)', () => {
    act(() => {
      useCartStore.getState().addItemOptimistic({
        id: 'li_001',
        variant_id: 'var_001',
        title: 'Oslo Sofa',
        quantity: 1,
        unit_price: 4999900,
        total: 4999900,
      })
    })
    expect(useCartStore.getState().items).toHaveLength(1)
  })

  it('increments quantity for existing variant', () => {
    // Add same variant twice
    act(() => {
      useCartStore.getState().addItemOptimistic({ variant_id: 'var_001', quantity: 1, unit_price: 100 })
      useCartStore.getState().addItemOptimistic({ variant_id: 'var_001', quantity: 1, unit_price: 100 })
    })
    const items = useCartStore.getState().items
    expect(items).toHaveLength(1)
    expect(items[0].quantity).toBe(2)
  })

  it('rolls back optimistic update on API failure', () => {
    const initialState = { items: [], total: 0 }
    act(() => {
      useCartStore.getState().addItemOptimistic({ variant_id: 'var_001', quantity: 1, unit_price: 100 })
    })
    expect(useCartStore.getState().items).toHaveLength(1)

    act(() => {
      useCartStore.getState().rollbackOptimisticAdd('var_001')
    })
    expect(useCartStore.getState().items).toHaveLength(0)
  })
})
```

---

### 3.3 Webhook Signature Verification (`api/webhooks/razorpay/verify.test.ts`)

```typescript
import { describe, it, expect } from 'vitest'
import { verifyRazorpaySignature } from './verify'

describe('verifyRazorpaySignature', () => {
  const secret = 'test_webhook_secret'

  it('returns true for valid signature', () => {
    const body = JSON.stringify({ event: 'payment.captured' })
    // Pre-computed: HMAC-SHA256(secret, body)
    const validSignature = computeHMAC(secret, body) // test helper
    expect(verifyRazorpaySignature(body, validSignature, secret)).toBe(true)
  })

  it('returns false for tampered body', () => {
    const body = JSON.stringify({ event: 'payment.captured' })
    const tamperedBody = JSON.stringify({ event: 'payment.refunded' })
    const signature = computeHMAC(secret, body)
    expect(verifyRazorpaySignature(tamperedBody, signature, secret)).toBe(false)
  })

  it('returns false for invalid signature string', () => {
    const body = JSON.stringify({ event: 'payment.captured' })
    expect(verifyRazorpaySignature(body, 'invalid_signature', secret)).toBe(false)
  })
})
```

---

### 3.4 Checkout Form Validation (`components/checkout/AddressForm.test.ts`)

```typescript
import { describe, it, expect } from 'vitest'
import { addressSchema } from './AddressForm'

describe('addressSchema (Zod)', () => {
  const validAddress = {
    firstName: 'Arjun',
    lastName: 'Sharma',
    email: 'arjun@example.com',
    phone: '9876543210',
    address1: '42 MG Road',
    city: 'Pune',
    state: 'Maharashtra',
    postalCode: '411001',
    countryCode: 'in',
  }

  it('validates a correct address', () => {
    expect(addressSchema.safeParse(validAddress).success).toBe(true)
  })

  it('rejects invalid PIN code format', () => {
    const result = addressSchema.safeParse({ ...validAddress, postalCode: '4110' })
    expect(result.success).toBe(false)
    expect(result.error?.issues[0].path).toContain('postalCode')
  })

  it('rejects invalid phone number', () => {
    const result = addressSchema.safeParse({ ...validAddress, phone: '12345' })
    expect(result.success).toBe(false)
  })

  it('rejects invalid email', () => {
    const result = addressSchema.safeParse({ ...validAddress, email: 'not-an-email' })
    expect(result.success).toBe(false)
  })
})
```

---

## 4. Integration Tests

### 4.1 Cart API Integration (`lib/medusa/cart.integration.test.ts`)

Tests the Medusa client wrapper functions against the local Medusa dev server.

```typescript
// Requires: local Medusa server running on localhost:9000
// Skip in CI unless INTEGRATION_TESTS=true env var is set
import { describe, it, expect } from 'vitest'
import { createCart, addItemToCart, removeItemFromCart } from './cart'

describe.skipIf(!process.env.INTEGRATION_TESTS)('Cart API Integration', () => {
  it('creates a cart and adds an item', async () => {
    const cart = await createCart()
    expect(cart.id).toMatch(/^cart_/)

    const updatedCart = await addItemToCart(cart.id, {
      variant_id: process.env.TEST_VARIANT_ID!,
      quantity: 1,
    })
    expect(updatedCart.items).toHaveLength(1)
    expect(updatedCart.items[0].quantity).toBe(1)
  })
})
```

---

## 5. End-to-End Tests (Playwright)

Located in `e2e/` at the monorepo root. Run against staging environment before production deploy.

### 5.1 Critical: Purchase Flow (`e2e/purchase-flow.spec.ts`)

```typescript
import { test, expect } from '@playwright/test'

test('complete purchase flow — UPI payment', async ({ page }) => {
  // 1. Browse to a product
  await page.goto('/collections/living-room')
  await page.getByTestId('product-card').first().click()

  // 2. Select variant and add to cart
  await page.getByTestId('variant-option-grey').click()
  await page.getByTestId('add-to-cart').click()
  await expect(page.getByTestId('cart-drawer')).toBeVisible()

  // 3. Proceed to checkout
  await page.getByTestId('proceed-to-checkout').click()
  await expect(page).toHaveURL('/checkout')

  // 4. Fill address form
  await page.fill('[name="firstName"]', 'Test')
  await page.fill('[name="lastName"]', 'User')
  await page.fill('[name="email"]', 'test@shree-furniture.in')
  await page.fill('[name="phone"]', '9999999999')
  await page.fill('[name="address1"]', '1 Test Street')
  await page.fill('[name="city"]', 'Pune')
  await page.selectOption('[name="state"]', 'Maharashtra')
  await page.fill('[name="postalCode"]', '411001')
  await page.getByTestId('continue-to-shipping').click()

  // 5. Select shipping
  await page.getByTestId('shipping-option-standard').click()
  await page.getByTestId('continue-to-payment').click()

  // 6. Place order (Razorpay is mocked in test environment)
  await expect(page).toHaveURL('/checkout/payment')
  await page.getByTestId('place-order').click()

  // 7. Confirm order page
  await expect(page).toHaveURL(/\/order\/confirm\//)
  await expect(page.getByTestId('order-confirmation-id')).toBeVisible()
})
```

### 5.2 Auth Flow (`e2e/auth.spec.ts`)

```typescript
test('register and login', async ({ page }) => {
  const email = `test+${Date.now()}@shree-furniture.in`

  await page.goto('/account/register')
  await page.fill('[name="firstName"]', 'Test')
  await page.fill('[name="lastName"]', 'User')
  await page.fill('[name="email"]', email)
  await page.fill('[name="password"]', 'TestPass123!')
  await page.getByTestId('register-submit').click()

  await expect(page).toHaveURL('/account/orders')
  await expect(page.getByText('My Orders')).toBeVisible()
})
```

---

## 6. Test Coverage Targets

| Area | Target Coverage | Notes |
|---|---|---|
| `lib/utils/` | 100% | Pure functions — must be fully tested |
| `store/cart-store.ts` | 90% | Business-critical state |
| Webhook verify logic | 100% | Security-critical |
| Zod schemas (checkout, auth) | 100% | Validation logic |
| React components | Not targeted | Covered by E2E |
| MedusaJS internals | 0% | Not our code to test |

---

## 7. Running Tests

```bash
# Run all unit tests
pnpm test

# Run with coverage
pnpm test --coverage

# Run in watch mode (development)
pnpm test --watch

# Run E2E tests (requires staging URL)
cd e2e
npx playwright test --project=chromium

# Run integration tests (requires local Medusa)
INTEGRATION_TESTS=true pnpm test
```

---

## 8. CI Test Gates

All of the following must pass before a PR can be merged:

- [ ] `tsc --noEmit` — zero TypeScript errors
- [ ] `eslint` — zero lint errors (warnings allowed)
- [ ] `vitest run` — all unit tests pass, no coverage regression
- [ ] `next build` — build completes without errors

E2E tests (`playwright`) run on merge to `main` (pre-deploy gate), not on every PR, to keep PR feedback fast.

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
