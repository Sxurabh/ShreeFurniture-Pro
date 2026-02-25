# 08 — Cart & Pricing Engine Spec
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Scope:** Cart lifecycle, pricing rules, tax logic, discount handling

---

## 1. Overview

The cart and pricing engine is handled primarily by **MedusaJS v2's Cart Module**. This document defines the business rules layered on top of Medusa's defaults, the price computation logic, tax handling, and the shipping cost rules specific to the Shree Furniture platform.

**Core rule: All monetary values are stored and computed in paise (₹1 = 100 paise). Never use floats for money.**

---

## 2. Cart Lifecycle State Machine

```
                    ┌─────────────────────────────────────┐
                    │              CREATED                 │
                    │  POST /store/carts                   │
                    │  cart.status = 'open'                │
                    └──────────────────┬──────────────────┘
                                       │
                     Items added, address set, shipping selected
                                       │
                    ┌──────────────────▼──────────────────┐
                    │         PAYMENT SESSION INIT        │
                    │  POST /store/carts/:id/payment-     │
                    │       sessions                      │
                    │  cart.payment_sessions populated     │
                    └──────────────────┬──────────────────┘
                                       │
                            Razorpay payment captured
                            Webhook verified + received
                                       │
                    ┌──────────────────▼──────────────────┐
                    │            COMPLETED                 │
                    │  POST /store/carts/:id/complete      │
                    │  cart.completed_at set               │
                    │  Order created                       │
                    └──────────────────┬──────────────────┘
                                       │
                                       │ (Cart is now read-only)
                                       ▼
                               Order lifecycle begins
```

**Cart expiry:** Redis TTL of 7 days. After 7 days without activity, cart data is purged from Redis. PostgreSQL cart record remains for audit but is considered abandoned.

---

## 3. Price Computation

### 3.1 Line Item Price

```
line_item.unit_price = money_amount.amount (paise, snapshot at time of add)
line_item.subtotal   = unit_price × quantity
line_item.tax_total  = subtotal × tax_rate
line_item.total      = subtotal + tax_total - line_discount
```

**Critical:** `unit_price` is a **snapshot** taken at the moment the item is added. Subsequent product price changes do NOT update existing cart line items. This prevents cart total surprises.

### 3.2 Cart Totals

```
cart.subtotal         = Σ line_item.subtotal (pre-tax sum)
cart.discount_total   = Σ discount amounts applied
cart.shipping_total   = selected shipping option price
cart.tax_total        = Σ line_item.tax_total
cart.total            = subtotal + shipping_total + tax_total - discount_total
```

### 3.3 Display Formatting

```typescript
// packages/types/src/utils/price.ts

/**
 * Convert paise to formatted INR string
 * 4999900 → "₹49,999"
 */
export function formatPrice(paise: number): string {
  const inr = paise / 100
  return new Intl.NumberFormat('en-IN', {
    style: 'currency',
    currency: 'INR',
    maximumFractionDigits: 0,
  }).format(inr)
}

/**
 * Calculate discount percentage
 * (originalPaise: 5999900, salePaise: 4999900) → "17% off"
 */
export function getDiscountPercent(original: number, sale: number): number {
  return Math.round(((original - sale) / original) * 100)
}
```

---

## 4. Tax Rules (India / GST)

For the MVP, a simplified flat tax approach is used. Full GST compliance (CGST/SGST/IGST breakdowns, invoice generation) is deferred to Phase 3.

| Product Category | GST Rate | Notes |
|---|---|---|
| Furniture (sofas, tables, beds) | 12% | Standard furniture rate |
| Mattresses | 12% | |
| Storage / Shelving | 18% | |

**MVP Tax Implementation:**
- A single tax rate (12% or 18%) is configured per product collection in MedusaJS
- Tax is **inclusive** in displayed prices (prices shown to customers include GST)
- Tax breakdown shown at checkout: "Incl. 12% GST: ₹5,357"

**Phase 3:** GST-compliant invoicing with CGST/SGST for intra-state, IGST for inter-state, based on customer's PIN code state.

---

## 5. Shipping Rules

### 5.1 Shipping Options

| Tier | Condition | Price | Estimated Delivery |
|---|---|---|---|
| Free Shipping | Order total ≥ ₹5,000 | ₹0 | 7–10 business days |
| Standard Shipping | Order total < ₹5,000 | ₹499 | 7–10 business days |
| Express Shipping | All orders (select cities) | ₹999 | 3–5 business days |

**Shipping total threshold check:**
```
if (cart.subtotal - cart.discount_total) >= 500000:  # ₹5,000 in paise
    apply free_shipping_option
else:
    show standard_shipping_option (₹49,900 paise = ₹499)
```

### 5.2 PIN Code Serviceability
- A PIN code lookup is performed during checkout address entry
- If PIN code is not serviceable, checkout is blocked with a friendly error: "Sorry, we don't deliver to this PIN code yet."
- Serviceability data maintained as a static list initially (Phase 1); API-based lookup in Phase 2.

---

## 6. Discount & Promotion Rules

**MVP: No active discount codes or promotions at launch.** Medusa's built-in discount engine is available but not exposed.

**Phase 2 planned:**
- Percentage discount codes (e.g., `LAUNCH20` → 20% off)
- Fixed amount discount codes (e.g., `SAVE500` → ₹500 off)
- Minimum order value requirement for code activation
- Single-use vs multi-use codes
- Discount displayed in cart as negative line: `Discount (LAUNCH20): -₹1,000`

---

## 7. Inventory Rules

### 7.1 Stock Status Display

```
if variant.inventory_quantity > 10:
    display "In Stock"
elif variant.inventory_quantity > 0 and <= 10:
    display "Only {n} left"
elif variant.inventory_quantity == 0:
    display "Out of Stock" (Add to Cart button disabled)
```

### 7.2 Quantity Cap

```
cart line item quantity cannot exceed variant.inventory_quantity
→ UI enforced: quantity stepper max = inventory_quantity
→ Server enforced: Medusa validates quantity on POST /store/carts/:id/line-items
→ Error returned: { "type": "insufficient_inventory", "message": "..." }
```

### 7.3 Stock Reserved on Cart (Phase 2)

In Phase 1: No stock reservation — inventory is decremented only on order completion. Race condition is accepted (two users can add the same "Only 1 left" item; the second checkout will fail at payment completion).

In Phase 2: Implement soft reservation — inventory decremented when item added to cart, restored if cart expires or item removed.

### 7.4 Low Stock Alert

```
Trigger: after each order.placed event
  if variant.inventory_quantity < LOW_STOCK_THRESHOLD (default: 5):
    emit inventory.low_stock event
      → LowStockSubscriber → email alert to admin
```

---

## 8. Razorpay Order Creation

When `POST /store/carts/:id/payment-sessions` is called:

```
1. Medusa Razorpay plugin calls Razorpay Orders API:
   POST https://api.razorpay.com/v1/orders
   {
     "amount": cart.total,          // in paise
     "currency": "INR",
     "receipt": cart.id,
     "payment_capture": 1           // auto-capture on payment
   }

2. Razorpay returns:
   {
     "id": "order_XXXXXXXXXXXXXXXX",
     "amount": 4999900,
     "currency": "INR",
     "status": "created"
   }

3. Stored in cart.payment_sessions[].data.razorpay_order_id

4. Storefront initialises Razorpay checkout modal:
   new Razorpay({
     key: process.env.NEXT_PUBLIC_RAZORPAY_KEY_ID,
     order_id: razorpay_order_id,
     amount: cart.total,
     currency: "INR",
     name: "Shree Furniture",
     description: `Order #${cart.id}`,
     handler: function(response) {
       // response.razorpay_payment_id
       // response.razorpay_order_id
       // response.razorpay_signature
       // Redirect to /order/confirm — webhook will handle order creation
     }
   })
```

---

## 9. Idempotency & Webhook Safety

The Razorpay webhook handler (`/api/webhooks/razorpay`) must be idempotent:

```typescript
async function handlePaymentCaptured(payload: RazorpayWebhookPayload) {
  const razorpayOrderId = payload.payload.payment.entity.order_id

  // 1. Find cart by razorpay_order_id stored in payment session data
  const cart = await findCartByRazorpayOrderId(razorpayOrderId)
  if (!cart) throw new Error('Cart not found')

  // 2. Idempotency check: already completed?
  if (cart.completed_at !== null) {
    console.log(`Cart ${cart.id} already completed. Skipping.`)
    return // Return 200 without reprocessing
  }

  // 3. Complete cart → creates order
  await medusa.carts.complete(cart.id)
}
```

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
