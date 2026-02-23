# User Stories & Acceptance Criteria

## Personas
- Buyer (mobile-first)
- Budget shopper
- Design-conscious buyer
- Admin manager

---

## Customer Stories

### Browse Products
**As a** customer  
**I want** to browse categories  
**So that** I can discover furniture

✅ Acceptance:
- Categories visible on homepage
- Filters update results instantly
- URL reflects filter state

Edge Cases:
- No products found → show suggestions

---

### View Product Details
**Acceptance:**
- Gallery loads fast
- Dimensions clearly displayed
- Stock status visible
- Variant selection updates price

Edge:
- Out of stock → disable purchase

---

### Add to Cart
**Acceptance:**
- Cart updates instantly
- Quantity editable
- Cart persists on refresh

Edge:
- Prevent quantity beyond stock

---

### Checkout & Payment
**Acceptance:**
- Guest checkout works
- Razorpay opens correctly
- Order created only after payment verification

Failure:
- Payment failure → retry allowed
- Network error → no duplicate orders

---

### Apply Coupon
**Acceptance:**
- Invalid coupon shows error
- Discount recalculates total

---

## Admin Stories

### Manage Products
- Upload images
- Add variants
- Publish/unpublish

Acceptance:
- Draft products not visible publicly

---

### Manage Orders
- Update status
- Filter by date/status

---

## Failure Scnearios
- Payment success but webhook delay
- Product out of stock during checkout
- Coupon expired mid-checkout