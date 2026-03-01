# 17 — India-Specific Implementation Guide
## Shree Furniture | Locale, Validation & Market Conventions

> **Version:** 1.0 | **Status:** Active
>
> **AI Agents:** Read this whenever you are building or modifying:
> address forms, phone input, PIN code validation, price display, region/state data,
> GST display, or any user-facing text that involves Indian geography or conventions.
>
> **Most AI training data assumes US/European conventions.** This document corrects for that.
> When there is any conflict between a generic convention and this document, this document wins.

---

## 1. Address Structure

Indian addresses differ significantly from Western formats. The standard structure for this project:

```typescript
// packages/types/src/index.ts — Address interface
interface Address {
  first_name: string
  last_name: string
  phone: string           // 10 digits, stored without country code
  address_1: string       // House/flat number and street — e.g. "Flat 4B, Sunrise Apartments, MG Road"
  address_2: string | null // Locality/area name — e.g. "Baner" — often essential for delivery
  city: string            // e.g. "Pune"
  province: string        // State name — e.g. "Maharashtra" (full name, not abbreviation)
  postal_code: string     // 6-digit PIN code — e.g. "411045"
  country_code: "in"      // Always "in" for Phase 1 — India only
  company: string | null  // Optional — for office deliveries
}
```

### Address Form Field Order

Display address fields in this order (matches how Indians write addresses):

1. Full Name (first + last on same line)
2. Phone Number
3. Address Line 1 (Flat/House No. + Building + Street)
4. Address Line 2 (Area/Locality — clearly labelled "Area / Locality")
5. City
6. State (dropdown — see §4 for full list)
7. PIN Code
8. (Country: hidden, always "India")

### Address Line 2 Is Important

Unlike in Western UX where address_2 is optional and often skipped, **Address Line 2 is functionally important in India** for delivery. The locality/area name helps couriers navigate within large cities. Label it clearly as "Area / Locality" not "Apartment, suite, etc." and mark it as optional in the UI but encourage completion.

---

## 2. Phone Number

### Format Rules

```
Total digits: 10
First digit:  Must be 6, 7, 8, or 9 (Indian mobile numbers only)
Storage:      10 digits only — no +91 prefix, no spaces, no dashes
Display:      "XXXXX XXXXX" (with space after 5th digit) — or raw 10 digits
Razorpay prefill: Use 10 digits without +91 — Razorpay adds country code internally
```

### Zod Validation Schema

```typescript
// apps/storefront/lib/utils/validators.ts
import { z } from "zod"

export const indianPhoneSchema = z
  .string()
  .trim()
  .regex(/^[6-9]\d{9}$/, {
    message: "Please enter a valid 10-digit Indian mobile number",
  })

// Usage in AddressForm schema
const addressSchema = z.object({
  phone: indianPhoneSchema,
  // ... other fields
})
```

### Input Handling

```tsx
// components/checkout/AddressForm.tsx
// Strip +91 prefix and spaces/dashes if user pastes formatted number
const normalizePhone = (input: string): string => {
  let digits = input.replace(/\D/g, "")  // Remove all non-digits
  if (digits.startsWith("91") && digits.length === 12) {
    digits = digits.slice(2)  // Remove country code if pasted with +91
  }
  return digits
}
```

---

## 3. PIN Code

### Format Rules

```
Total digits: 6 (not 5 like US ZIP)
First digit:  1–8 (defines postal zone: 1=Delhi, 4=Maharashtra/Goa, 6=Kerala/Tamil Nadu, etc.)
Storage:      6 digits as string (preserve leading zeros theoretically possible, use string not int)
Validation:   /^[1-8][0-9]{5}$/
```

### Zod Validation Schema

```typescript
// apps/storefront/lib/utils/validators.ts
export const pinCodeSchema = z
  .string()
  .trim()
  .regex(/^[1-8][0-9]{5}$/, {
    message: "Please enter a valid 6-digit PIN code",
  })
```

### PIN Code Serviceability Check (MVP)

For MVP, serviceability is not validated against a real courier API. Display a delivery estimate copy:

> "We deliver across India. Delivery to your PIN code typically takes 7–10 business days."

Phase 2 may integrate a courier API (Delhivery, Shiprocket) for real-time serviceability.
When that is built, add a separate document. Do not build it now.

---

## 4. Indian States — Medusa Region Seeding

Use these state names exactly when seeding Medusa regions or populating the state dropdown.
The `province` field in the Address interface stores the full state name (not abbreviation).

```typescript
// backend/src/seed.ts — state list for address form dropdown
export const INDIAN_STATES = [
  "Andaman and Nicobar Islands",
  "Andhra Pradesh",
  "Arunachal Pradesh",
  "Assam",
  "Bihar",
  "Chandigarh",
  "Chhattisgarh",
  "Dadra and Nagar Haveli and Daman and Diu",
  "Delhi",
  "Goa",
  "Gujarat",
  "Haryana",
  "Himachal Pradesh",
  "Jammu and Kashmir",
  "Jharkhand",
  "Karnataka",
  "Kerala",
  "Ladakh",
  "Lakshadweep",
  "Madhya Pradesh",
  "Maharashtra",
  "Manipur",
  "Meghalaya",
  "Mizoram",
  "Nagaland",
  "Odisha",
  "Puducherry",
  "Punjab",
  "Rajasthan",
  "Sikkim",
  "Tamil Nadu",
  "Telangana",
  "Tripura",
  "Uttar Pradesh",
  "Uttarakhand",
  "West Bengal",
] as const

export type IndianState = typeof INDIAN_STATES[number]
```

### Medusa Region Seed

```typescript
// backend/src/seed.ts
// Seed one region covering all of India with INR currency
const region = await regionService.createRegions([{
  name: "India",
  currency_code: "inr",
  tax_rate: 18,           // 18% GST (standard furniture rate)
  countries: ["in"],
  payment_providers: ["razorpay"],
  fulfillment_providers: ["manual"],
}])
```

---

## 5. Currency & Price Display

### Paise Rule (Enforced Project-Wide)

All prices are stored and passed through the system as **paise integers**.
See `NewDocs/CLAUDE.md` and `NewDocs/CONVENTIONS.md` for the full paise rule.
This section covers the India-specific display conventions.

### `formatPrice()` Utility

```typescript
// packages/types/src/utils/price.ts

/**
 * Formats paise integer to Indian Rupee display string.
 *
 * Examples:
 *   formatPrice(4999900)  → "₹49,999"
 *   formatPrice(500000)   → "₹5,000"
 *   formatPrice(99900)    → "₹999"
 *   formatPrice(5000)     → "₹50"
 *
 * Uses Indian numbering system (lakhs/crores) via Intl.NumberFormat.
 * ₹1,00,000 for ₹1 lakh — not ₹100,000.
 */
export function formatPrice(paise: number): string {
  const rupees = paise / 100
  return new Intl.NumberFormat("en-IN", {
    style: "currency",
    currency: "INR",
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(rupees)
}

/**
 * Formats paise to rupees number (for Algolia range filter, calculations).
 * Do not use this for display — use formatPrice().
 */
export function paiseToRupees(paise: number): number {
  return paise / 100
}
```

### Indian Numbering System

The `en-IN` locale uses the Indian numbering convention:
- ₹1,000 (one thousand)
- ₹10,000 (ten thousand)
- ₹1,00,000 (one lakh — not ₹100,000)
- ₹10,00,000 (ten lakhs — not ₹1,000,000)
- ₹1,00,00,000 (one crore)

`Intl.NumberFormat("en-IN")` handles this automatically. Do not implement custom comma-placement logic.

---

## 6. GST Display (Phase 1 — Informational Only)

For Phase 1, GST is **included in the displayed price** and shown as an informational line item.
Full GST-compliant invoicing (CGST/SGST breakdown) is deferred to Phase 2.

### Display Pattern

In `CartSummary.tsx` and order confirmation email:

```
Subtotal:          ₹49,999
Shipping:          Free
──────────────────────────
Total:             ₹49,999
  (GST @ 18% included: ₹7,627)   ← smaller text, muted colour, informational only
```

### GST Calculation Utility

```typescript
// packages/types/src/utils/price.ts
/**
 * Calculates the GST component included in a GST-inclusive price.
 * Most furniture in India is taxed at 18% GST.
 *
 * Formula: GST = total × (rate / (100 + rate))
 *
 * Example: ₹49,999 total at 18% GST
 *   GST included = 4999900 × (18/118) ≈ 762696 paise ≈ ₹7,627
 */
export function calculateIncludedGST(totalPaise: number, gstRate: 5 | 12 | 18 | 28 = 18): number {
  return Math.round(totalPaise * (gstRate / (100 + gstRate)))
}

// GST rates for reference (furniture falls under 18% as standard)
// 5%  — some basic/essential goods
// 12% — some processed goods
// 18% — standard rate (applies to most furniture)
// 28% — luxury goods
```

> **Do not** generate GST invoices (GSTIN, CGST/SGST split, HSN codes) in Phase 1.
> When the business registers for GST, Phase 2 will add invoice generation.
> See `NewDocs/09-engineering-scope-definition.md` for scope boundaries.

---

## 7. Date & Time

### Date Display

```typescript
// packages/types/src/utils/date.ts

// Order dates display in Indian convention: DD MMM YYYY
// e.g. "15 Jan 2026" (not Jan 15, 2026 or 15/01/2026)
export function formatOrderDate(isoString: string): string {
  return new Intl.DateTimeFormat("en-IN", {
    day: "numeric",
    month: "short",
    year: "numeric",
  }).format(new Date(isoString))
}

// Delivery estimate (static for MVP)
export function getDeliveryEstimate(orderDate: Date): string {
  return "7–10 business days"  // Phase 2: calculate from courier API
}
```

### Time Zone

All timestamps stored in Medusa/PostgreSQL are UTC. When displaying times to users (e.g., order placed at), convert to **IST (UTC+5:30)**:

```typescript
export function formatOrderTime(isoString: string): string {
  return new Intl.DateTimeFormat("en-IN", {
    timeZone: "Asia/Kolkata",
    hour: "2-digit",
    minute: "2-digit",
    hour12: true,
  }).format(new Date(isoString))
}
// Output: "02:30 PM IST"
```

---

## 8. Delivery Copy Conventions

These are the standard copy strings for delivery messaging across the storefront.
Do not improvise delivery text — use these exact strings for consistency:

| Context | Copy |
|---|---|
| Free shipping threshold | "Free delivery on orders above ₹5,000" |
| Standard shipping cost | "₹250 shipping" |
| Estimated delivery | "7–10 business days" |
| Delivery note on PDP | "Estimated delivery in 7–10 business days after dispatch" |
| PIN code note | "Delivering across India" |
| Large item note | "Assembly guide included. Installation services available in select cities." |

> **No vague copy.** See `PREFERENCES.md` §7: "Specific over vague: '7–10 business days' not 'Ships fast'."

---

## 9. Trust Signals (India-Specific)

These trust signals are specifically chosen for Indian furniture buyers and should appear on
the Homepage and PDP as per `NewDocs/01-product-requirements.md §4.1`:

| Signal | Text | Icon |
|---|---|---|
| Free delivery | "Free Delivery above ₹5,000" | `Truck` (Lucide) |
| Easy returns | "7-Day Easy Returns" | `RefreshCw` (Lucide) |
| EMI | "No-Cost EMI Available" | `CreditCard` (Lucide) |
| Warranty | "1-Year Genuine Warranty" | `Shield` (Lucide) |
| Secure payments | "Secure Payments via Razorpay" | `Lock` (Lucide) |

---

## 10. Common Mistakes — Do Not Repeat

| ❌ Mistake | ✅ Correct Approach |
|---|---|
| 5-digit ZIP code validation | PIN code is 6 digits: `/^[1-8][0-9]{5}$/` |
| Storing phone as integer | Store as string — leading zeros must be preserved conceptually |
| `+91` prefix stored in DB | Store 10 digits only; add `+91` only for SMS (Phase 2) |
| `Intl.NumberFormat("en-US")` for prices | Always `"en-IN"` for Indian numbering system |
| "Zip Code" label in forms | Use **"PIN Code"** (all-caps PIN) |
| "State" dropdown with US states | Use INDIAN_STATES list from §4 |
| Tax shown as separate charge | GST is included in price for Phase 1 — display as informational |
| Date format "Jan 15, 2026" | Use "15 Jan 2026" (Indian convention) |

---

*Shree Furniture | v1.0 — Q1 2026 | Place at: `NewDocs/17-india-specific-guide.md`*
