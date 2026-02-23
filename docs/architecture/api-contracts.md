# API Contracts

## Naming

/api/{resource}
/api/webhooks/razorpay

---

## Auth Flow
Supabase session cookies  
JWT validated server-side  

---

## Checkout

**Server Action:** `initiateCheckout`

**Request Payload:**
```typescript
interface InitiateCheckoutRequest {
  items: {
    variantId: string;
    quantity: number;
  }[];
  shippingAddressId: string;
  couponCode?: string;
}
```

**Response Payload:**
```typescript
interface InitiateCheckoutResponse {
  success: boolean;
  data?: {
    orderId: string;
    razorpayOrderId: string;
    amount: number; // in smallest currency unit (paise)
    currency: string;
  };
  error?: string;
}
```

---

## Payment Verify

**Server Action:** `verifyPayment`

**Request Payload:**
```typescript
interface VerifyPaymentRequest {
  orderId: string; // Internal Order ID
  razorpay_payment_id: string;
  razorpay_order_id: string;
  razorpay_signature: string;
}
```

**Response Payload:**
```typescript
interface VerifyPaymentResponse {
  success: boolean;
  message?: string;
}
```

---

## Error Format

```typescript
interface StandardErrorResponse {
  success: false;
  error: string; // "Message"
  code: string;  // "ERROR_CODE"
}
```