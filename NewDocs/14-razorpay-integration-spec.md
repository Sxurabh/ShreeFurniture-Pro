# 14 — Razorpay Integration Spec
## Shree Furniture | Payment Gateway Deep-Dive

> **Version:** 1.0 | **Status:** Active | **Scope:** MVP Phase 1
>
> **AI Agents:** Read this before touching `PaymentStep.tsx`, `app/api/webhooks/razorpay/route.ts`,
> or any `backend/src/subscribers/` file that references payment events.
> This document is the authoritative source for all Razorpay-specific behaviour on this project.
>
> **Prerequisite reading:** `NewDocs/06-api-contracts.md` (payment session endpoints) and
> `NewDocs/MEDUSA-V2-PATTERNS.md` (Razorpay plugin config pattern).

---

## 1. Keys & Environment Variables

| Variable | Location | Public? | What It Is |
|---|---|---|---|
| `NEXT_PUBLIC_RAZORPAY_KEY_ID` | Storefront `.env.local` | ✅ Yes | Identifies your Razorpay account to the frontend modal. Razorpay documents this as safe to expose. |
| `RAZORPAY_SECRET` | Backend `.env` | ❌ No | API secret for server-to-server calls (refunds, order verification). Never `NEXT_PUBLIC_`. |
| `RAZORPAY_WEBHOOK_SECRET` | Storefront `.env.local` (webhook handler) | ❌ No | Used to verify HMAC signatures on incoming webhook calls. Never `NEXT_PUBLIC_`. |

**Key ID format:** `rzp_test_XXXXXXXXXXXXXXXX` (sandbox) / `rzp_live_XXXXXXXXXXXXXXXX` (production)

> When switching from sandbox to production, both `NEXT_PUBLIC_RAZORPAY_KEY_ID` and `RAZORPAY_SECRET` must be updated in their respective environments. The webhook secret also changes between test and live dashboards.

---

## 2. Plugin Configuration (Backend)

The Razorpay payment provider is registered via the `medusa-payment-razorpay` plugin. **Do not write custom payment logic — only configure the plugin.**

```typescript
// backend/medusa-config.ts
import { defineConfig } from "@medusajs/framework/utils"

export default defineConfig({
  projectConfig: {
    databaseUrl: process.env.DATABASE_URL,
    redisUrl: process.env.REDIS_URL,
  },
  plugins: [
    {
      resolve: "medusa-payment-razorpay",
      options: {
        key_id: process.env.RAZORPAY_KEY_ID,         // server-side key ID (no NEXT_PUBLIC_)
        key_secret: process.env.RAZORPAY_SECRET,
        razorpay_account: process.env.RAZORPAY_ACCOUNT, // optional: for route-based sub-merchants
        automatic_expiry_period: 30,  // minutes before a Razorpay order expires
        manual_capture: false,        // auto-capture on payment success (MVP setting)
        capture_timeout: 0,           // immediate capture
      },
    },
  ],
})
```

> `manual_capture: false` means payment is captured automatically when the user completes payment. Phase 2 may switch to `true` for pre-authorisation flows (e.g., COD-to-prepaid conversion).

---

## 3. Frontend: Opening the Razorpay Modal

The storefront triggers the Razorpay checkout modal from `components/checkout/PaymentStep.tsx`. The Razorpay JS SDK is loaded via a `<Script>` tag, **not** via npm install.

### 3.1 Loading the SDK

```tsx
// apps/storefront/app/(checkout)/layout.tsx  OR  in PaymentStep.tsx
import Script from "next/script"

// Add alongside the checkout layout — loads once per checkout session
<Script
  src="https://checkout.razorpay.com/v1/checkout.js"
  strategy="lazyOnload"
/>
```

### 3.2 Opening the Modal (PaymentStep.tsx)

```typescript
// The razorpay_order_id comes from the Medusa payment session
// See NewDocs/06-api-contracts.md § POST /store/carts/:id/payment-sessions

interface RazorpayOptions {
  key: string
  amount: number          // in paise (integer) — matches Razorpay's own paise system
  currency: "INR"
  name: string
  description: string
  order_id: string        // razorpay_order_id from Medusa payment session
  prefill: {
    name: string
    email: string
    contact: string       // 10-digit Indian mobile, no country code
  }
  notes: {
    medusa_cart_id: string  // IMPORTANT: used for idempotency in webhook handler
  }
  theme: {
    color: "#1A1A1A"      // brand.primary — matches design system
  }
  modal: {
    ondismiss: () => void   // called when user closes modal without paying
    escape: boolean         // true = clicking outside closes modal
  }
  handler: (response: RazorpayPaymentResponse) => void
}

interface RazorpayPaymentResponse {
  razorpay_payment_id: string   // e.g. "pay_XXXXXXXXXXXXXXXX"
  razorpay_order_id: string     // e.g. "order_XXXXXXXXXXXXXXXX"
  razorpay_signature: string    // HMAC-SHA256 signature for verification
}

// Usage inside PaymentStep component
const openRazorpay = (paymentSession: PaymentSession) => {
  const options: RazorpayOptions = {
    key: process.env.NEXT_PUBLIC_RAZORPAY_KEY_ID!,
    amount: cart.total,         // already in paise from Medusa
    currency: "INR",
    name: "Shree Furniture",
    description: `Order for ${cart.items.length} item(s)`,
    order_id: paymentSession.data.razorpay_order_id,
    prefill: {
      name: `${address.first_name} ${address.last_name}`,
      email: cart.email ?? "",
      contact: address.phone,   // 10 digits, no +91
    },
    notes: {
      medusa_cart_id: cart.id,  // stored on Razorpay order; echoed back in webhook
    },
    theme: { color: "#1A1A1A" },
    modal: {
      ondismiss: () => {
        // Reset payment UI state — do NOT complete the cart
        setPaymentState("idle")
      },
      escape: true,
    },
    handler: async (response) => {
      // Payment succeeded on Razorpay's end
      // Now complete the cart on Medusa to create the order
      await completeCart(cart.id)
      router.push(`/order/confirm/${orderId}`)
    },
  }

  const rzp = new (window as any).Razorpay(options)
  rzp.open()
}
```

### 3.3 Payment Methods Enabled (MVP)

Configure these in the Razorpay Dashboard under Settings → Checkout:

| Method | Enabled | Notes |
|---|---|---|
| UPI | ✅ Yes | Primary for Indian customers |
| Credit / Debit Cards | ✅ Yes | Visa, Mastercard, RuPay |
| Net Banking | ✅ Yes | Major banks |
| EMI | ✅ Yes | 3, 6, 9, 12-month options |
| BNPL (Simpl, LazyPay) | ✅ Yes | Important for high-AOV furniture |
| Wallets (Paytm, PhonePe) | ✅ Yes | |
| International Cards | ❌ No | Phase 2 |
| Cryptocurrency | ❌ No | Not in scope |

---

## 4. Webhook Handler

### 4.1 Webhook Events We Handle

| Event | Trigger | Action |
|---|---|---|
| `payment.captured` | Payment auto-captured on success | Verify signature → complete cart if not already done → idempotency check |
| `payment.failed` | Payment attempt failed | Log to PostHog; no order action needed (user can retry) |
| `refund.created` | Refund initiated via Medusa Admin | Update order payment status; send refund email via Resend |
| `refund.failed` | Razorpay failed to process refund | Alert via Sentry; requires manual intervention |

> Events **not** handled (no action needed on our side):
> `order.paid`, `payment.authorized` (we use auto-capture), `subscription.*` (not in scope)

### 4.2 HMAC Signature Verification

Every incoming webhook **must** be verified before any action is taken. Failure to verify is a critical security flaw.

```typescript
// apps/storefront/app/api/webhooks/razorpay/route.ts
import { createHmac } from "crypto"
import { NextRequest, NextResponse } from "next/server"

export async function POST(req: NextRequest) {
  const rawBody = await req.text()   // Must use raw text — NOT req.json() — for HMAC
  const signature = req.headers.get("x-razorpay-signature")

  if (!signature) {
    return NextResponse.json({ error: "Missing signature" }, { status: 400 })
  }

  // Step 1: Verify HMAC signature
  const expectedSignature = createHmac("sha256", process.env.RAZORPAY_WEBHOOK_SECRET!)
    .update(rawBody)
    .digest("hex")

  if (expectedSignature !== signature) {
    // Log to Sentry as a security event
    return NextResponse.json({ error: "Invalid signature" }, { status: 401 })
  }

  const event = JSON.parse(rawBody)

  // Step 2: Idempotency check — have we already processed this event?
  const eventId = event.payload?.payment?.entity?.id ?? event.payload?.refund?.entity?.id
  if (!eventId) {
    return NextResponse.json({ error: "Unknown event shape" }, { status: 400 })
  }

  // Check your idempotency store (Redis or DB table)
  const alreadyProcessed = await checkIdempotency(eventId)
  if (alreadyProcessed) {
    return NextResponse.json({ status: "already_processed" }, { status: 200 })
  }

  // Step 3: Route to handler
  switch (event.event) {
    case "payment.captured":
      await handlePaymentCaptured(event.payload.payment.entity)
      break
    case "payment.failed":
      await handlePaymentFailed(event.payload.payment.entity)
      break
    case "refund.created":
      await handleRefundCreated(event.payload.refund.entity)
      break
    case "refund.failed":
      await handleRefundFailed(event.payload.refund.entity)
      break
    default:
      // Unknown event — log and return 200 (do not return error to Razorpay)
  }

  await markEventProcessed(eventId)
  return NextResponse.json({ status: "ok" }, { status: 200 })
}
```

### 4.3 Idempotency Pattern

Use a dedicated table in PostgreSQL (or a Redis key with TTL) to track processed webhook events.

**Option A — PostgreSQL table (preferred for auditability):**

```sql
-- Run as a migration in the backend
CREATE TABLE processed_webhook_events (
  event_id   VARCHAR(64) PRIMARY KEY,       -- Razorpay payment_id or refund_id
  event_type VARCHAR(64) NOT NULL,
  processed_at TIMESTAMPTZ DEFAULT now()
);
```

```typescript
async function checkIdempotency(eventId: string): Promise<boolean> {
  const result = await db.query(
    "SELECT 1 FROM processed_webhook_events WHERE event_id = $1",
    [eventId]
  )
  return result.rowCount > 0
}

async function markEventProcessed(eventId: string, eventType: string): Promise<void> {
  await db.query(
    "INSERT INTO processed_webhook_events (event_id, event_type) VALUES ($1, $2) ON CONFLICT DO NOTHING",
    [eventId, eventType]
  )
}
```

### 4.4 payment.captured Handler

```typescript
async function handlePaymentCaptured(payment: RazorpayPaymentEntity) {
  // The cart_id is stored in notes when we opened the Razorpay modal
  const cartId = payment.notes?.medusa_cart_id
  if (!cartId) {
    // Log to Sentry — this should never happen
    throw new Error(`payment.captured missing medusa_cart_id in notes: ${payment.id}`)
  }

  // Complete the Medusa cart → creates the order
  // This call is safe to make even if the cart was already completed by the handler() callback
  // Medusa's cart.complete() is idempotent — it returns the existing order if already created
  const result = await medusaClient.store.cart.complete(cartId)
  if (result.type !== "order") {
    throw new Error(`Cart complete returned unexpected type: ${result.type}`)
  }

  // The order.placed subscriber (backend/src/subscribers/order-placed.ts)
  // will fire automatically and send the confirmation email via Resend
}
```

---

## 5. Payment Failure UX

When the Razorpay modal is dismissed (`ondismiss`) or when `payment.failed` is received:

- **Do NOT call `cart.complete()`** — the cart must remain open for a retry
- Show a user-friendly error toast: *"Payment was not completed. Your cart is saved — try again when you're ready."*
- Keep the user on the Payment step (do not navigate away)
- Log to PostHog: `posthog.capture('payment_failed', { reason: payment.error_reason })`
- Do NOT show Razorpay's internal error codes to the user

**Retry flow:** User clicks "Try Again" → `openRazorpay()` is called again with the same `razorpay_order_id`. Razorpay supports multiple payment attempts on a single order.

---

## 6. Refund Flow (via Medusa Admin)

Refunds are **always initiated from Medusa Admin** (not the storefront). Medusa handles the Razorpay API call to create the refund. The storefront only needs to handle the `refund.created` webhook.

```typescript
// backend/src/subscribers handles this automatically via the Razorpay plugin
// When an admin clicks "Refund" in Medusa Admin:
//   1. Medusa calls Razorpay POST /v1/payments/:id/refund
//   2. Razorpay fires refund.created webhook → our handler
//   3. Our handler sends a refund confirmation email via Resend

// Manual refund amounts: partial refunds in paise
// e.g. ₹5,000 refund = 500000 paise
```

---

## 7. Testing (Sandbox)

Use these test card/UPI credentials in sandbox mode (`rzp_test_*` keys):

| Method | Test Value | Result |
|---|---|---|
| Card (success) | `4111 1111 1111 1111`, any future date, any CVV | Captured |
| Card (failure) | `4000 0000 0000 0002` | Declined |
| UPI (success) | `success@razorpay` | Captured |
| UPI (failure) | `failure@razorpay` | Failed |
| Net Banking | Any bank → test credentials shown on Razorpay test page | Captured |

**Triggering test webhooks:**
In the Razorpay Dashboard (test mode) → Webhooks → trigger a test event to your local webhook URL. Use [ngrok](https://ngrok.com) to expose your local Next.js server during development.

---

## 8. Common Mistakes — Do Not Repeat

| ❌ Mistake | ✅ Correct Approach |
|---|---|
| Using `req.json()` in webhook handler | Use `req.text()` — JSON parsing destroys the raw body needed for HMAC |
| Calling `cart.complete()` on payment failure | Only call on success; keep cart open for retry |
| Hardcoding `rzp_live_` key in source | Always env var: `NEXT_PUBLIC_RAZORPAY_KEY_ID` |
| Sending `amount` in rupees | Amount is always in **paise** (same as Medusa) |
| Returning `4xx` for unknown webhook events | Return `200` — Razorpay will retry on non-2xx |
| Skipping idempotency check | Razorpay sends webhooks multiple times; always deduplicate |

---

*Shree Furniture | v1.0 — Q1 2026 | Place at: `NewDocs/14-razorpay-integration-spec.md`*
