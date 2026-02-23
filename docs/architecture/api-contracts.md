# API Contracts

> Server Actions are the primary mutation mechanism. Route Handlers exist only for external webhooks.
> Edge Functions handle Razorpay operations and email sending.

---

## Architecture Decision Summary

| Mechanism | Used For |
|-----------|---------|
| **Server Actions** | All UI-triggered mutations (cart, checkout, profile, admin CRUD) |
| **Route Handlers** (`/api/*`) | External webhooks from Razorpay only |
| **Supabase Edge Functions** | Razorpay order creation/verification (requires service role), email sending |
| **Supabase JS client in RSC** | All read queries in Server Components |

---

## Standard Types

```typescript
// /src/types/actions.ts

export type ActionResult<T = void> =
  | { success: true; data: T }
  | { success: false; error: string | Record<string, string[]> }

export interface StandardErrorResponse {
  success: false
  error: string
  code: string   // e.g., "UNAUTHORIZED", "VALIDATION_ERROR", "NOT_FOUND"
}
```

---

## Auth Actions

### `signUp`
```typescript
// /src/actions/auth.ts
interface SignUpInput {
  email: string
  password: string
  fullName: string
  phone?: string
}
// Returns: ActionResult<{ userId: string }>
// Side effects: Creates auth.users entry, triggers profile creation via DB trigger
```

### `signIn`
```typescript
interface SignInInput {
  email: string
  password: string
}
// Returns: ActionResult<void>
// Side effects: Sets Supabase session cookie
```

### `signOut`
```typescript
// Returns: ActionResult<void>
// Side effects: Clears session cookie
```

---

## Cart Actions

### `syncCartToSupabase`
```typescript
// /src/actions/cart.ts
interface CartSyncInput {
  items: {
    productId: string
    variantId: string | null
    quantity: number
  }[]
}
// Returns: ActionResult<void>
// Called: On login, to merge localStorage cart into Supabase cart_items
// Logic: For each item, upsert into cart_items. On conflict (user_id, product_id, variant_id), add quantities (capped at 10).
```

### `updateCartItem`
```typescript
interface UpdateCartItemInput {
  productId: string
  variantId: string | null
  quantity: number   // set to 0 to remove
}
// Returns: ActionResult<void>
```

### `clearCart`
```typescript
// Returns: ActionResult<void>
// Called after successful order creation
```

---

## Checkout Actions

### `validateCoupon`
```typescript
// /src/actions/checkout.ts
interface ValidateCouponInput {
  code: string
  orderSubtotal: number
}
interface ValidateCouponData {
  couponId: string
  discountType: 'percentage' | 'fixed'
  discountValue: number
  discountAmount: number   // calculated discount to apply
}
// Returns: ActionResult<ValidateCouponData>
// Validates: code exists, is_active, not expired, min_order_amount met, max_uses not exceeded
```

### `initiateCheckout`
```typescript
interface InitiateCheckoutRequest {
  items: {
    variantId: string
    quantity: number
  }[]
  shippingAddressId?: string    // for logged-in users with saved address
  shippingAddress?: {           // for guest or new address
    fullName: string
    phone: string
    line1: string
    line2?: string
    city: string
    state: string
    pincode: string
  }
  couponCode?: string
  guestEmail?: string           // required for guest checkout
}

interface InitiateCheckoutResponse {
  success: boolean
  data?: {
    orderId: string             // internal order ID (status: pending_payment)
    razorpayOrderId: string
    amount: number              // in paise (smallest unit)
    currency: string            // "INR"
    keyId: string               // NEXT_PUBLIC_RAZORPAY_KEY_ID
  }
  error?: string
}
```

**Flow:**
1. Validate all cart items (prices from DB, not client — anti-tamper)
2. Validate stock availability for each item
3. Apply and validate coupon if provided
4. Calculate totals server-side
5. Create order record with status `pending_payment`
6. Call Edge Function `create-razorpay-order`
7. Return Razorpay order details to client

### `verifyPayment`
```typescript
interface VerifyPaymentRequest {
  orderId: string               // internal order ID
  razorpay_payment_id: string
  razorpay_order_id: string
  razorpay_signature: string
}

interface VerifyPaymentResponse {
  success: boolean
  data?: {
    orderId: string
    orderNumber: string
  }
  error?: string
}
```

**Flow:**
1. Call Edge Function `verify-razorpay-payment` (HMAC signature check)
2. On success: update order status to `confirmed`, record `razorpay_payment_id`
3. Decrement stock for each ordered variant
4. Clear user's cart
5. Call Edge Function `send-order-email`
6. Return orderId for redirect to confirmation page

---

## Profile Actions

### `updateProfile`
```typescript
// /src/actions/profile.ts
interface UpdateProfileInput {
  fullName?: string
  phone?: string
  avatarUrl?: string
}
// Returns: ActionResult<void>
```

### `addAddress`
```typescript
interface AddAddressInput {
  label: string
  fullName: string
  phone: string
  line1: string
  line2?: string
  city: string
  state: string
  pincode: string
  isDefault?: boolean
}
// Returns: ActionResult<{ id: string }>
// If isDefault: true, sets all other addresses to is_default = false first
```

### `deleteAddress`
```typescript
interface DeleteAddressInput {
  addressId: string
}
// Returns: ActionResult<void>
// Cannot delete if only one address exists
```

---

## Admin — Product Actions

### `createProduct`
```typescript
// /src/actions/admin/products.ts
interface CreateProductInput {
  categoryId: string
  name: string
  slug: string
  description?: string
  shortDesc?: string
  basePrice: number
  salePrice?: number
  sku?: string
  material?: string
  weightKg?: number
  dimensionsCm?: { length: number; width: number; height: number }
  isPublished: boolean
  isFeatured: boolean
  stockQuantity: number
  lowStockThreshold: number
  metaTitle?: string
  metaDescription?: string
  images: { url: string; altText?: string; displayOrder: number; isPrimary: boolean }[]
  variants?: { name: string; sku?: string; priceDelta: number; stockQuantity: number; attributes: Record<string, string> }[]
}
// Returns: ActionResult<{ id: string }>
// Auth: admin role required
```

### `updateProduct`
```typescript
// Partial of CreateProductInput + productId
// Returns: ActionResult<void>
// Side effects: revalidatePath('/products'), revalidatePath(`/products/${slug}`)
```

### `deleteProduct`
```typescript
interface DeleteProductInput { productId: string }
// Returns: ActionResult<void>
// Behavior: Sets is_published = false (soft delete) — product still exists in order history
// Does NOT hard-delete — use a separate admin action for that if ever needed
```

### `toggleProductPublished`
```typescript
interface TogglePublishedInput { productId: string; isPublished: boolean }
// Returns: ActionResult<void>
```

---

## Admin — Order Actions

### `updateOrderStatus`
```typescript
// /src/actions/admin/orders.ts
interface UpdateOrderStatusInput {
  orderId: string
  status: 'confirmed' | 'processing' | 'shipped' | 'out_for_delivery' | 'delivered' | 'cancelled'
  trackingNumber?: string    // required when status = 'shipped'
}
// Returns: ActionResult<void>
// Side effects: Sets shipped_at / delivered_at timestamps as appropriate
```

---

## Admin — Promotions Actions

### `createCoupon`
```typescript
// /src/actions/admin/promotions.ts
interface CreateCouponInput {
  code: string
  description?: string
  discountType: 'percentage' | 'fixed'
  discountValue: number
  minOrderAmount?: number
  maxUses?: number
  isActive: boolean
  startsAt?: string
  expiresAt?: string
}
// Returns: ActionResult<{ id: string }>
```

### `createBanner`
```typescript
interface CreateBannerInput {
  title: string
  subtitle?: string
  imageUrl: string
  linkUrl?: string
  displayOrder: number
  isActive: boolean
  startsAt?: string
  endsAt?: string
}
// Returns: ActionResult<{ id: string }>
```

---

## Route Handlers (Webhooks Only)

### POST `/api/webhooks/razorpay`

Backup mechanism. Handles events Razorpay retries if our primary flow fails.

**Headers:**
```
x-razorpay-signature: {hmac_sha256_signature}
```

**Handled event types:**
```typescript
'payment.captured'    // ensure order exists and is confirmed
'payment.failed'      // update order status to payment_failed
'refund.created'      // update order status to refunded
```

**Security:** HMAC-SHA256 signature verified against `RAZORPAY_WEBHOOK_SECRET` before any processing.

**Idempotency:** Check `razorpay_payment_id` before processing — skip if already handled.

**Response:** Always return `200 OK` even on no-op (Razorpay retries on non-200).

---

## Supabase Edge Functions

### `create-razorpay-order`
```typescript
// Called from: Server Action initiateCheckout
// Input: { amount: number, currency: string, receipt: string (orderId) }
// Output: { id: string, amount: number, currency: string }  — Razorpay order object
// Secrets used: RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET (never exposed outside Edge Function)
```

### `verify-razorpay-payment`
```typescript
// Called from: Server Action verifyPayment
// Input: { razorpay_order_id, razorpay_payment_id, razorpay_signature }
// Output: { verified: boolean }
// Logic: HMAC-SHA256(razorpay_order_id + "|" + razorpay_payment_id, RAZORPAY_KEY_SECRET)
```

### `send-order-email`
```typescript
// Called from: Server Action verifyPayment (after successful verification)
// Input: { orderId: string }
// Logic: Fetches order + items from DB, renders OrderConfirmation React Email template, sends via Resend
// Output: { success: boolean }
```

---

## Error Codes

| Code | Meaning |
|------|---------|
| `UNAUTHORIZED` | No session found |
| `FORBIDDEN` | Authenticated but wrong role |
| `VALIDATION_ERROR` | Zod schema failed |
| `NOT_FOUND` | Resource doesn't exist |
| `STOCK_UNAVAILABLE` | Requested quantity exceeds stock |
| `COUPON_INVALID` | Coupon not found or not applicable |
| `COUPON_EXPIRED` | Coupon past expiry date |
| `COUPON_MIN_ORDER` | Order subtotal below coupon minimum |
| `COUPON_EXHAUSTED` | Max uses reached |
| `PAYMENT_VERIFICATION_FAILED` | HMAC signature mismatch |
| `DUPLICATE_PAYMENT` | Payment ID already processed |