# Commerce Logic Engine

## Pricing Logic
final_price =
base_price
+ variant_delta
- discount

---

## Coupon Logic
- validate expiry
- validate min order
- usage limit enforcement

---

## Inventory Logic
- reserve stock at payment success
- prevent overselling

---

## Shipping Logic
- free shipping above ₹X
- flat fee otherwise

---

## Tax Strategy
GST included in price
invoice shows breakdown

---

## Extensibility
- multi-warehouse ready
- dynamic shipping ready