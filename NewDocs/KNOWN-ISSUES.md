# KNOWN-ISSUES.md — Gotchas & Hard-Won Knowledge
## Shree Furniture | E-Commerce Platform

> **AI Agent Instruction:** Read this before debugging unexpected behavior.
> **Engineers:** Add to this file immediately when you hit a gotcha, a silent failure, or an
> undocumented MedusaJS/Razorpay/Vercel behavior. This file prevents the same bug from
> being investigated twice.

---

## How to Add an Entry

```md
### [SHORT TITLE]
**Symptom:** What the broken behavior looked like.
**Root Cause:** Why it happened.
**Fix/Workaround:** What to do instead.
**Affects:** Which files/layers are involved.
**Date Discovered:** YYYY-MM-DD
```

---

## MedusaJS v2

### MedusaJS v2 — Module Container Scope in Subscribers
**Symptom:** `container.resolve("some-service")` throws `Service not found` inside a subscriber, even though the module is registered.
**Root Cause:** Subscribers receive a **request-scoped** container. If a module was registered after the initial server boot without a restart, it won't be available in the container.
**Fix:** Always restart the MedusaJS server after adding or modifying a module registration in `medusa-config.ts`.
**Affects:** `backend/src/subscribers/`, `backend/medusa-config.ts`
**Date Discovered:** (fill when encountered)

---

### MedusaJS v2 — Subscriber Does Not Fire on First Deploy
**Symptom:** `order.placed` subscriber sends no email after first Railway deployment.
**Root Cause:** MedusaJS requires database migrations to register event subscribers. If you deploy code without running `medusa db:migrate`, subscribers may silently not register.
**Fix:** Always run `npx medusa db:migrate` after deploying backend changes.
**Affects:** All backend subscribers
**Date Discovered:** (fill when encountered)

---

### MedusaJS v2 — `carts/:id/complete` Returns `{ type: "cart" }` Instead of Order
**Symptom:** After calling complete, the response has `type: "cart"` not `type: "order"`.
**Root Cause:** Payment was not captured. Razorpay payment session must be in `authorized` or `captured` state before calling complete. This usually means the webhook hasn't fired yet OR the payment modal was closed before completion.
**Fix:** Verify Razorpay webhook is correctly configured and HMAC signature check passes. Add a payment status poll/check before allowing the user to proceed to confirmation.
**Affects:** `components/checkout/PaymentStep.tsx`, `app/api/webhooks/razorpay/route.ts`
**Date Discovered:** (fill when encountered)

---

## Next.js / Vercel

### ISR Page Shows Stale Product After Admin Update
**Symptom:** A product price updated in Medusa Admin still shows the old price on the storefront for more than 60 seconds.
**Root Cause:** `revalidatePath()` was not called, OR the `algolia-sync` subscriber failed silently (network error calling the storefront revalidation endpoint).
**Fix:**
1. Check Railway logs for the `algolia-sync` subscriber for errors.
2. Verify `REVALIDATION_SECRET` is correctly set in both Railway (backend) and Vercel (storefront).
3. For immediate fix: manually call `POST /api/revalidate` from the server.
**Affects:** `backend/src/subscribers/algolia-sync.ts`, `apps/storefront/app/api/revalidate/route.ts`
**Date Discovered:** (fill when encountered)

---

### Next.js App Router — Server Component Cannot Use `useRouter`
**Symptom:** `useRouter is not a function` or hydration errors on a page that uses router navigation.
**Root Cause:** `useRouter`, `useSearchParams`, `usePathname` are Client Component hooks. Server Components cannot use them.
**Fix:** Add `'use client'` directive at the top of the component file. If the component mixes server data with client interactivity, split it: a Server Component parent that passes data as props to a Client Component child.
**Affects:** Any component in `apps/storefront/`
**Date Discovered:** (fill when encountered)

---

### Vercel Build Fails — `Cannot find module '@shree/types'`
**Symptom:** Vercel deployment fails with module resolution error for workspace packages.
**Root Cause:** Turborepo build cache from a previous version. The `packages/types` build artifact is stale.
**Fix:** In Vercel project settings, clear the build cache and redeploy. Locally: `pnpm clean && pnpm build`.
**Affects:** `apps/storefront/`, CI/CD pipeline
**Date Discovered:** (fill when encountered)

---

## Razorpay

### Razorpay Webhook — Duplicate Order Creation
**Symptom:** Two orders appear for one payment in the Medusa Admin.
**Root Cause:** Razorpay can fire the same webhook event multiple times if your endpoint doesn't respond with `200` within 5 seconds. If the cart completion takes > 5s, Razorpay retries and a second order is created.
**Fix:** The webhook handler at `/api/webhooks/razorpay/route.ts` must:
1. Respond `200 OK` immediately after HMAC verification.
2. Store the processed `razorpay_payment_id` in the database before taking action.
3. Check for this ID at the top of every invocation and return early if already processed.
See `NewDocs/08-cart-pricing-engine-spec.md` for the full idempotency spec.
**Affects:** `app/api/webhooks/razorpay/route.ts`
**Date Discovered:** (fill when encountered)

---

### Razorpay — EMI Options Not Showing in Modal
**Symptom:** Razorpay modal loads but the EMI tab is not present.
**Root Cause:** EMI requires a minimum order value (typically ₹3,000) AND must be explicitly enabled in your Razorpay dashboard for the specific payment methods (HDFC EMI, etc.).
**Fix:** Log into Razorpay Dashboard → Settings → Payment Methods → Enable EMI for each bank.
**Affects:** Checkout UX only — no code change needed
**Date Discovered:** (fill when encountered)

---

## Algolia

### Algolia Search — Products Not Appearing After Publish
**Symptom:** A product published in Medusa Admin does not appear in Algolia search results.
**Root Cause:** The `algolia-sync` subscriber only fires on `product.updated` event. If MedusaJS fires `product.published` as a separate event (v2 behaviour varies), the subscriber may miss it.
**Fix:** Register the subscriber for both `product.created`, `product.updated`, AND `product.status_changed` events.
**Affects:** `backend/src/subscribers/algolia-sync.ts`
**Date Discovered:** (fill when encountered)

---

## Cloudinary

### Cloudinary Images — `400 Bad Request` on Upload from Medusa Admin
**Symptom:** Image upload in Medusa Admin fails with a `400` error.
**Root Cause:** Cloudinary upload preset is set to `signed` mode. The `medusa-file-cloudinary` plugin requires an `unsigned` upload preset for direct uploads.
**Fix:** In Cloudinary Dashboard → Settings → Upload Presets → set the preset used by Medusa to `Unsigned`.
**Affects:** `backend/medusa-config.ts` (Cloudinary plugin config)
**Date Discovered:** (fill when encountered)

---

## General

### TypeScript Strict — `Object is possibly 'null'` on Medusa Response Fields
**Symptom:** TypeScript errors on fields like `product.thumbnail` or `variant.sku`.
**Root Cause:** MedusaJS v2 types are accurately `string | null` for optional fields. TypeScript strict mode correctly catches usage without null checks.
**Fix:** Use optional chaining: `product.thumbnail ?? "/images/placeholder.jpg"`. Do not use `!` non-null assertion unless you've explicitly verified the field is always set.
**Affects:** Any code consuming Medusa API responses
**Date Discovered:** (fill when encountered)

---

*Shree Furniture | v1.0 — Q1 2026 | Add entries as you discover them — don't lose this knowledge*
