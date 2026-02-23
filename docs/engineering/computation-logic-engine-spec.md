# Commerce Logic Engine Spec

> All business logic runs server-side in Server Actions or Edge Functions.
> Never implement pricing, discount, or stock logic in Client Components.

---

## Pricing Logic

```
final_price = base_price + variant_price_delta - discount_amount + shipping_amount
```

| Component | Source |
|-----------|--------|
| `base_price` | `products.base_price` — fetched from DB (never from client) |
| `variant_price_delta` | `product_variants.price_delta` — can be negative (discount variant) |
| `discount_amount` | Calculated from coupon (see below) |
| `shipping_amount` | 0 if order total ≥ free shipping threshold; flat fee otherwise |

**Display price on PDP:**
```
display_price = products.sale_price ?? (products.base_price + selected_variant.price_delta)
```
If `sale_price` is set on the product, it takes precedence over base+variant calculation for display.

**Anti-tamper rule:** The server always recalculates all amounts from DB values during `initiateCheckout`. The client-submitted total is ignored.

---

## Coupon / Discount Logic

Validation order (fail fast):

1. Code exists in `coupons` table — if not: `COUPON_INVALID`
2. `is_active = true` — if not: `COUPON_INVALID`
3. `starts_at <= now()` (if set) — if not: `COUPON_INVALID`
4. `expires_at > now()` (if set) — if not: `COUPON_EXPIRED`
5. `used_count < max_uses` (if `max_uses` is set) — if not: `COUPON_EXHAUSTED`
6. `order_subtotal >= min_order_amount` — if not: `COUPON_MIN_ORDER`

**Discount calculation:**
```typescript
function calculateDiscount(coupon: Coupon, subtotal: number): number {
  if (coupon.discount_type === 'percentage') {
    return Math.round((subtotal * coupon.discount_value) / 100 * 100) / 100
  }
  if (coupon.discount_type === 'fixed') {
    return Math.min(coupon.discount_value, subtotal)  // can't discount more than subtotal
  }
  return 0
}
```

**After successful order:** Increment `coupons.used_count` by 1 in the same transaction as order creation.

---

## Order Total Calculation

```typescript
// Performed inside initiateCheckout Server Action
const subtotal = items.reduce(
  (sum, item) => sum + (item.basePrice + item.priceDelta) * item.quantity, 0
)
const discountAmount = coupon ? calculateDiscount(coupon, subtotal) : 0
const shippingAmount = (subtotal - discountAmount) >= FREE_SHIPPING_THRESHOLD ? 0 : FLAT_SHIPPING_FEE
const taxAmount = 0         // GST included in price for MVP (no breakdown)
const totalAmount = subtotal - discountAmount + shippingAmount + taxAmount
```

**Constants (define in environment or config):**
```typescript
const FREE_SHIPPING_THRESHOLD = 5000   // ₹5,000 — free shipping above this
const FLAT_SHIPPING_FEE = 299          // ₹299 flat rate below threshold
```

---

## Inventory / Stock Logic

### Stock Check at Checkout

Before creating the Razorpay order in `initiateCheckout`:

```typescript
for (const item of items) {
  const variant = await supabase
    .from('product_variants')
    .select('stock_quantity, name')
    .eq('id', item.variantId)
    .single()

  if ((variant.data?.stock_quantity ?? 0) < item.quantity) {
    return { success: false, error: `STOCK_UNAVAILABLE`, code: 'STOCK_UNAVAILABLE' }
  }
}
```

### Stock Reservation (Decrement)

Stock is decremented **only after successful payment verification** (inside `verifyPayment` action), not at checkout initiation:

```typescript
// After HMAC verification succeeds:
for (const item of orderItems) {
  await supabase
    .from('product_variants')
    .update({ stock_quantity: supabase.rpc('decrement', { x: item.quantity }) })
    .eq('id', item.variantId)
}
```

This prevents phantom stock reservation from abandoned payments.

### Low Stock Alert

A product variant is considered low stock when:
```
product_variants.stock_quantity <= products.low_stock_threshold
```

The admin dashboard queries this at render time. No real-time push in MVP.

### Oversell Prevention

If stock reaches 0:
- `product_variants.stock_quantity = 0`
- Product card shows "Out of Stock"
- "Add to Cart" button is disabled
- If item is in cart and stock drops to 0, cart shows warning and blocks checkout

---

## Shipping Logic

```
shipping_amount = order_total >= FREE_SHIPPING_THRESHOLD ? 0 : FLAT_SHIPPING_FEE
```

- **MVP:** Single flat rate — no weight-based, zone-based, or carrier-based calculation
- **Delivery estimate:** Static text "5–7 business days" — no carrier API
- **Tracking number:** Manually entered by admin in order management

---

## Tax Strategy

- **MVP:** GST is included in displayed product prices (tax-inclusive pricing)
- `tax_amount` field in orders is set to `0` for MVP
- No GST invoice breakdown or GSTIN validation in MVP
- Post-MVP: Separate GST line item, GSTIN on invoice, HSN codes per product

---

## Order Number Generation

Format: `SF-{YEAR}-{SEQUENCE}`

Example: `SF-2026-00001`

```sql
-- Generate via Postgres sequence
CREATE SEQUENCE order_number_seq START 1;

-- In application:
-- order_number = 'SF-' || EXTRACT(YEAR FROM now()) || '-' || LPAD(nextval('order_number_seq')::text, 5, '0')
```

---

## Future Extensions

| Feature | When | Approach |
|---------|------|---------|
| Weight-based shipping | Post-MVP | Add `weight_kg` per product, integrate Shiprocket API |
| Zone-based rates | Post-MVP | Pincode → zone mapping table |
| GST-compliant invoices | Post-MVP | HSN codes per product, Postgres function for invoice PDF |
| Multi-warehouse stock | Scale | `inventory_locations` table per warehouse |
| Dynamic shipping quotes | Scale | Shiprocket / Delhivery API at checkout |