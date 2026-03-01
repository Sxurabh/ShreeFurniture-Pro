# 16 — Email Templates Specification
## Shree Furniture | Transactional Email via Resend + React Email

> **Version:** 1.0 | **Status:** Active
>
> **AI Agents:** Read this before building any file in `backend/src/emails/` or
> modifying `backend/src/subscribers/order-placed.ts`, `order-shipped.ts`, or `low-stock.ts`.
> This document defines the 5 transactional emails this project sends, their variables,
> layout intent, and subject line patterns. Do not invent new emails or email structure.
>
> **Tech stack:** [Resend](https://resend.com) (delivery) + [React Email](https://react.email) (templates).
> Environment variable: `RESEND_API_KEY` (backend only, never `NEXT_PUBLIC_`).
> From address: `orders@shree-furniture.in` (configure in Resend dashboard).

---

## 1. Email Index

| # | Email ID | Trigger | Sent To | Subscriber File |
|---|---|---|---|---|
| 1 | `order-confirmation` | `order.placed` | Customer | `subscribers/order-placed.ts` |
| 2 | `order-shipped` | `order.shipment_created` | Customer | `subscribers/order-shipped.ts` |
| 3 | `low-stock-alert` | `inventory.low_stock` | Admin (internal) | `subscribers/low-stock.ts` |
| 4 | `password-reset` | Customer requests reset | Customer | Medusa auth hook |
| 5 | `refund-confirmation` | `refund.created` webhook | Customer | `app/api/webhooks/razorpay/route.ts` |

---

## 2. Resend Setup (Backend)

```typescript
// backend/src/lib/email.ts — shared Resend client
import { Resend } from "resend"

export const resend = new Resend(process.env.RESEND_API_KEY)

export const FROM_ADDRESS = "Shree Furniture <orders@shree-furniture.in>"
export const REPLY_TO = "support@shree-furniture.in"

// Generic send helper used by all subscriber files
export async function sendEmail({
  to,
  subject,
  react,
}: {
  to: string
  subject: string
  react: React.ReactElement
}) {
  const { error } = await resend.emails.send({
    from: FROM_ADDRESS,
    reply_to: REPLY_TO,
    to,
    subject,
    react,
  })

  if (error) {
    // Log to Sentry — email failures are non-blocking but must be monitored
    console.error("[Resend error]", error)
    throw new Error(`Email send failed: ${JSON.stringify(error)}`)
  }
}
```

---

## 3. Shared Layout Component

All 5 emails use the same outer wrapper for consistent branding.

```tsx
// backend/src/emails/components/EmailLayout.tsx
import { Html, Head, Body, Container, Section, Img, Text, Hr } from "@react-email/components"

interface EmailLayoutProps {
  preview: string       // Short preview text shown in inbox (50–90 chars)
  children: React.ReactNode
}

// Visual spec:
// Background: #F5F0E8 (brand.warm)  — matches storefront page background
// Container:  white (#FFFFFF), max-width 600px, border-radius 8px
// Header:     logo centered, 32px vertical padding, border-bottom 1px #E8E0D0 (brand.subtle)
// Footer:     address, unsubscribe note, "© 2026 Shree Furniture"
// Font:       Inter for body, Playfair Display for h1/h2 headings (via inline styles — email safe)
// Accent colour for links and dividers: #C8A96E (brand.accent)
// Body text:  #1A1A1A (brand.primary), 16px, line-height 1.6

export function EmailLayout({ preview, children }: EmailLayoutProps) {
  return (
    <Html>
      <Head />
      <Body style={{ backgroundColor: "#F5F0E8", fontFamily: "Inter, sans-serif" }}>
        <Container style={{ maxWidth: "600px", margin: "0 auto", backgroundColor: "#FFFFFF", borderRadius: "8px" }}>
          {/* Header with logo */}
          <Section style={{ padding: "32px 0", borderBottom: "1px solid #E8E0D0", textAlign: "center" }}>
            <Img
              src="https://res.cloudinary.com/[CLOUDINARY_CLOUD]/image/upload/shree-furniture/brand/logo-dark.png"
              alt="Shree Furniture"
              width={160}
              height={40}
            />
          </Section>

          {/* Email body */}
          {children}

          {/* Footer */}
          <Section style={{ padding: "24px 40px", borderTop: "1px solid #E8E0D0" }}>
            <Text style={{ color: "#9A9A9A", fontSize: "12px", lineHeight: "1.6", textAlign: "center" }}>
              Shree Furniture · [Your Address, City, State, PIN] · support@shree-furniture.in
            </Text>
            <Text style={{ color: "#9A9A9A", fontSize: "12px", textAlign: "center" }}>
              © 2026 Shree Furniture. All rights reserved.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}
```

---

## 4. Email 1 — Order Confirmation

**Trigger:** `order.placed` event → `backend/src/subscribers/order-placed.ts`
**Sent to:** Customer email (`order.email`)
**Goal:** Confirm the order, make the customer feel reassured they spent money wisely.

### Subject Line
```
Your Shree Furniture order #{{display_id}} is confirmed ✓
```
Example: *"Your Shree Furniture order #1042 is confirmed ✓"*

### Template Variables
```typescript
interface OrderConfirmationProps {
  customer_name: string         // e.g. "Arjun" (first name only)
  display_id: number            // e.g. 1042
  items: {
    title: string               // Product name e.g. "Oslo 3-Seater Sofa"
    variant_title: string       // e.g. "Grey / L"
    quantity: number
    unit_price: number          // paise — format as ₹ in template
    thumbnail: string | null    // Cloudinary URL
  }[]
  subtotal: number              // paise
  shipping_total: number        // paise (0 if free)
  tax_total: number             // paise (GST display — informational only for Phase 1)
  total: number                 // paise
  shipping_address: {
    first_name: string
    last_name: string
    address_1: string
    address_2: string | null
    city: string
    province: string            // State name e.g. "Maharashtra"
    postal_code: string         // 6-digit PIN
    phone: string
  }
  estimated_delivery: string    // e.g. "7–10 business days" — hardcoded for MVP
  order_url: string             // Full URL to order detail page (account/orders/[id])
}
```

### Content Layout (section by section)

**1. Greeting:** "Hi Arjun, your order is confirmed!" (Playfair Display h1, warm tone)

**2. Order summary table:** One row per line item — thumbnail (48×48px), product name + variant, quantity, price. Right-aligned price column.

**3. Order totals block:**
  - Subtotal: ₹XX,XXX
  - Shipping: ₹XXX (or "Free" if 0)
  - GST (18%): ₹XXX *(included in total)*
  - **Total: ₹XX,XXX** (bold)

**4. Delivery address block:** Display the shipping address formatted as it would appear on a label.

**5. Delivery estimate:** "Your order will arrive in {{estimated_delivery}}. We'll send you a shipping confirmation with tracking details once it's on the way."

**6. CTA button:** "View Your Order" → `{{order_url}}` (brand.primary button style)

**7. Trust note:** "Questions? Reply to this email or contact us at support@shree-furniture.in. We're here Mon–Sat, 10am–6pm IST."

### Subscriber Implementation
```typescript
// backend/src/subscribers/order-placed.ts
import { sendEmail } from "../lib/email"
import { OrderConfirmationEmail } from "../emails/OrderConfirmationEmail"

export default async function orderPlacedHandler({ event, container }) {
  const orderService = container.resolve("orderService")
  const order = await orderService.retrieveOrder(event.data.id, {
    relations: ["items", "items.variant", "shipping_address"],
  })

  await sendEmail({
    to: order.email,
    subject: `Your Shree Furniture order #${order.display_id} is confirmed ✓`,
    react: OrderConfirmationEmail({
      customer_name: order.shipping_address.first_name,
      display_id: order.display_id,
      items: order.items.map(item => ({
        title: item.title,
        variant_title: item.variant?.title ?? "",
        quantity: item.quantity,
        unit_price: item.unit_price,
        thumbnail: item.thumbnail,
      })),
      subtotal: order.subtotal,
      shipping_total: order.shipping_total,
      tax_total: order.tax_total,
      total: order.total,
      shipping_address: order.shipping_address,
      estimated_delivery: "7–10 business days",
      order_url: `${process.env.STOREFRONT_URL}/account/orders/${order.id}`,
    }),
  })
}
export const config = { event: "order.placed" }
```

---

## 5. Email 2 — Order Shipped

**Trigger:** `order.shipment_created` event → `backend/src/subscribers/order-shipped.ts`
**Sent to:** Customer email
**Goal:** Let the customer know their furniture is on the way; give them tracking info.

### Subject Line
```
Your Shree Furniture order #{{display_id}} has shipped 🚚
```

### Template Variables
```typescript
interface OrderShippedProps {
  customer_name: string
  display_id: number
  tracking_number: string | null    // From fulfilment record
  tracking_url: string | null       // Courier tracking URL (if available)
  courier_name: string | null       // e.g. "Delhivery", "BlueDart"
  estimated_delivery: string        // e.g. "3–5 business days from today"
  items: {
    title: string
    variant_title: string
    quantity: number
    thumbnail: string | null
  }[]
  shipping_address: ShippingAddress // Same shape as Order Confirmation
  order_url: string
}
```

### Content Layout

**1. Headline:** "Great news, Arjun — your furniture is on its way!" (Playfair Display h1)

**2. Tracking block** (if tracking_number is present):
  - Courier: {{courier_name}}
  - Tracking number: {{tracking_number}} (monospace font)
  - CTA button: "Track Your Shipment" → `{{tracking_url}}`
  - If no tracking URL: "You can track your shipment on the courier's website using the number above."

**3. Items shipped:** Simplified item list (no prices — this is a fulfilment email, not an invoice).

**4. Delivery note:** "Estimated delivery: {{estimated_delivery}}. Someone should be available to receive the package — furniture items may require a signature."

**5. Assembly note:** "Assembly guides for your items are included in the packaging. Need help? Contact us at support@shree-furniture.in."

**6. CTA:** "View Order Details" → `{{order_url}}`

---

## 6. Email 3 — Low Stock Alert (Internal)

**Trigger:** `inventory.low_stock` event → `backend/src/subscribers/low-stock.ts`
**Sent to:** `admin@shree-furniture.in` (internal alert, not customer-facing)
**Goal:** Notify the store owner to restock before going out of stock.

### Subject Line
```
[LOW STOCK] {{product_title}} — {{inventory_quantity}} units remaining
```
Example: *"[LOW STOCK] Oslo 3-Seater Sofa (Grey / L) — 2 units remaining"*

### Template Variables
```typescript
interface LowStockAlertProps {
  product_title: string
  variant_title: string
  sku: string | null
  inventory_quantity: number      // Current stock count
  threshold: number               // The threshold that triggered this alert (e.g. 3)
  admin_url: string               // Direct link to product in Medusa Admin
}
```

### Content Layout

Simple, plain, fast to read — this is an internal ops email, not a brand email.

**1. Alert line:** "⚠️ Low stock alert: {{product_title}} ({{variant_title}}) has only {{inventory_quantity}} units remaining."

**2. Details table:**
  - SKU: {{sku}}
  - Current stock: {{inventory_quantity}}
  - Alert threshold: {{threshold}}

**3. CTA:** "Manage Inventory in Admin" → `{{admin_url}}`

> This email does NOT need the full branded EmailLayout. Use a simplified wrapper with just the logo and no footer decorations — it's an internal ops tool.

### Trigger Threshold

Set the low-stock threshold to `3` units in the Medusa inventory configuration.
The subscriber fires once per variant when stock drops to or below this level.
Implement deduplication using Redis to avoid sending the same alert multiple times per day.

```typescript
// Deduplication using Upstash Redis (already in the stack)
const redisKey = `low_stock_alert:${variantId}:${new Date().toDateString()}`
const alreadyAlerted = await redis.get(redisKey)
if (!alreadyAlerted) {
  await sendEmail({ ... })
  await redis.set(redisKey, "1", { ex: 86400 }) // TTL: 1 day
}
```

---

## 7. Email 4 — Password Reset

**Trigger:** Customer requests password reset via `/account/forgot-password`
**Sent to:** Customer email
**Handled by:** Medusa's built-in auth system — configure the email template via Medusa's notification system.

### Subject Line
```
Reset your Shree Furniture password
```

### Template Variables
```typescript
interface PasswordResetProps {
  customer_name: string
  reset_url: string       // Medusa-generated one-time reset URL (expires in 30 minutes)
}
```

### Content Layout

**1. Headline:** "Password reset request"

**2. Body:** "Hi {{customer_name}}, we received a request to reset your password. Click the button below to choose a new one. This link expires in 30 minutes."

**3. CTA button:** "Reset My Password" → `{{reset_url}}`

**4. Security note:** "If you didn't request this, please ignore this email. Your password won't change unless you click the link above."

> **Do not** build a custom password reset token system. Use Medusa's built-in `customer.password_reset` event.

---

## 8. Email 5 — Refund Confirmation

**Trigger:** `refund.created` Razorpay webhook → `app/api/webhooks/razorpay/route.ts`
**Sent to:** Customer email
**Goal:** Confirm that the refund has been initiated and set timeline expectations.

### Subject Line
```
Your refund of ₹{{amount}} has been initiated — Order #{{display_id}}
```
Example: *"Your refund of ₹5,000 has been initiated — Order #1042"*

### Template Variables
```typescript
interface RefundConfirmationProps {
  customer_name: string
  display_id: number
  refund_amount: number       // paise — display as ₹ in template
  payment_method: string      // e.g. "UPI", "Credit Card" — from Razorpay payment entity
  estimated_credit_days: string  // "5–7 business days" for cards; "2–3 business days" for UPI
  order_url: string
}
```

### Content Layout

**1. Headline:** "Your refund is on its way, {{customer_name}}"

**2. Refund summary:**
  - Amount: **₹{{amount}}** (bold)
  - Refunded to: {{payment_method}}
  - Expected credit: {{estimated_credit_days}}

**3. Note:** "Refund timelines are determined by your bank or payment provider. If you don't see the credit after {{estimated_credit_days}}, please contact your bank with this reference: [Razorpay refund ID]."

**4. CTA:** "View Order" → `{{order_url}}`

**5. Support line:** "Need help? Reply to this email or reach us at support@shree-furniture.in"

---

## 9. Price Formatting in Email Templates

All prices in email templates are formatted from paise to ₹ using the same utility as the storefront.
Import from the shared types package:

```typescript
// Both storefront and backend can import this
import { formatPrice } from "@shree/types/utils/price"

// Usage in React Email component
<Text>{formatPrice(item.unit_price)}</Text>  // → "₹49,999"
```

> Never divide by 100 manually inline in JSX. Always use `formatPrice()` from the shared package.

---

## 10. File Structure

```
backend/src/
├── emails/
│   ├── components/
│   │   ├── EmailLayout.tsx           // Shared wrapper (§3 above)
│   │   ├── OrderItemRow.tsx          // Reusable item row for order/shipped emails
│   │   └── PriceBlock.tsx            // Reusable subtotal/tax/total table
│   ├── OrderConfirmationEmail.tsx
│   ├── OrderShippedEmail.tsx
│   ├── LowStockAlertEmail.tsx
│   ├── PasswordResetEmail.tsx
│   └── RefundConfirmationEmail.tsx
├── lib/
│   └── email.ts                      // Resend client + sendEmail() helper (§2 above)
└── subscribers/
    ├── order-placed.ts
    ├── order-shipped.ts
    └── low-stock.ts
```

---

*Shree Furniture | v1.0 — Q1 2026 | Place at: `NewDocs/16-email-templates-spec.md`*
