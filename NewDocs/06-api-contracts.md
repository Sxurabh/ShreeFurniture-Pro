# 06 — API Contracts
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Base URL:** `https://api.shree-furniture.in` | **Protocol:** REST / JSON

---

## 1. Conventions

- All endpoints are prefixed with `/store/` (storefront) or `/admin/` (admin panel)
- All responses return JSON with `Content-Type: application/json`
- Authentication uses HTTP-only cookies containing a JWT (set by Medusa auth endpoints)
- Protected routes return `401 Unauthorized` if no valid session cookie is present
- Error responses follow the structure: `{ "type": "...", "message": "...", "code": "..." }`
- Monetary values are always in **paise** (integer). Divide by 100 for INR display.
- Pagination uses `limit` and `offset` query params; response includes `count` and `offset`
- Handles (slugs) are lowercase, hyphenated strings

---

## 2. Products

### GET `/store/products`
List products with filtering and pagination.

**Query Params:**
| Param | Type | Description |
|---|---|---|
| `collection_id[]` | string[] | Filter by collection ID(s) |
| `handle` | string | Fetch by URL slug |
| `q` | string | Full-text search query |
| `tags[]` | string[] | Filter by tag values |
| `price_list_id[]` | string[] | Filter by price list |
| `limit` | integer | Results per page (default: 12, max: 100) |
| `offset` | integer | Pagination offset (default: 0) |
| `order` | string | Sort field: `created_at`, `-created_at`, `title`, `price` |

**Response: 200 OK**
```json
{
  "products": [
    {
      "id": "prod_01ABCDEF",
      "title": "Oslo 3-Seater Sofa",
      "handle": "oslo-3-seater-sofa",
      "thumbnail": "https://res.cloudinary.com/.../oslo-sofa.webp",
      "status": "published",
      "collection": {
        "id": "pcol_01ABCDEF",
        "title": "Living Room",
        "handle": "living-room"
      },
      "variants": [
        {
          "id": "variant_01ABCDEF",
          "title": "Grey / L",
          "sku": "SF-SOFA-OSLO-GRY-L",
          "inventory_quantity": 12,
          "prices": [
            {
              "currency_code": "inr",
              "amount": 4999900
            }
          ],
          "options": [
            { "value": "Grey" },
            { "value": "L" }
          ]
        }
      ],
      "images": [
        { "url": "https://res.cloudinary.com/.../oslo-sofa-1.webp" }
      ],
      "tags": [
        { "value": "scandinavian" },
        { "value": "fabric" }
      ]
    }
  ],
  "count": 48,
  "offset": 0,
  "limit": 12
}
```

---

### GET `/store/products/:id`
Retrieve a single product by ID. Also works with handle via `/store/products?handle={handle}`.

**Response: 200 OK** — Single `product` object (same schema as above, fully expanded).

---

### GET `/store/collections`
List all product collections.

**Response: 200 OK**
```json
{
  "collections": [
    {
      "id": "pcol_01ABCDEF",
      "title": "Living Room",
      "handle": "living-room",
      "products": []
    }
  ]
}
```

---

### GET `/store/collections/:id`
Retrieve a single collection by ID or handle.

---

## 3. Cart

### POST `/store/carts`
Create a new cart.

**Request Body:**
```json
{
  "region_id": "reg_01ABCDEF"
}
```

**Response: 200 OK**
```json
{
  "cart": {
    "id": "cart_01ABCDEF",
    "email": null,
    "items": [],
    "total": 0,
    "subtotal": 0,
    "tax_total": 0,
    "shipping_total": 0,
    "region": { "id": "reg_01ABCDEF", "currency_code": "inr" }
  }
}
```

---

### POST `/store/carts/:id/line-items`
Add a product variant to the cart.

**Request Body:**
```json
{
  "variant_id": "variant_01ABCDEF",
  "quantity": 1
}
```

**Response: 200 OK** — Updated `cart` object.

**Errors:**
- `400` if `variant_id` is missing
- `422` if requested quantity exceeds `inventory_quantity`

---

### DELETE `/store/carts/:id/line-items/:line_item_id`
Remove a line item from the cart.

**Response: 200 OK** — Updated `cart` object.

---

### POST `/store/carts/:id/line-items/:line_item_id`
Update a line item quantity.

**Request Body:**
```json
{
  "quantity": 2
}
```

**Response: 200 OK** — Updated `cart` object.

---

### POST `/store/carts/:id/shipping-methods`
Select a shipping method.

**Request Body:**
```json
{
  "option_id": "so_01ABCDEF"
}
```

---

### POST `/store/carts/:id/payment-sessions`
Initialise Razorpay payment session.

**Response: 200 OK**
```json
{
  "cart": {
    "payment_sessions": [
      {
        "provider_id": "razorpay",
        "data": {
          "razorpay_order_id": "order_XXXXXXXXXXXXXXXX",
          "amount": 4999900,
          "currency": "INR",
          "key_id": "rzp_live_XXXXXXXX"
        }
      }
    ]
  }
}
```

---

### POST `/store/carts/:id/complete`
Complete the cart and create an order. Called after Razorpay webhook confirms payment.

**Response: 200 OK**
```json
{
  "type": "order",
  "data": {
    "id": "ord_01ABCDEF",
    "display_id": 1042,
    "status": "pending",
    "payment_status": "captured",
    "total": 4999900
  }
}
```

---

## 4. Orders

### GET `/store/orders`
List orders for the authenticated customer.

**Query Params:** `limit`, `offset`, `fields`

**Response: 200 OK**
```json
{
  "orders": [
    {
      "id": "ord_01ABCDEF",
      "display_id": 1042,
      "status": "pending",
      "fulfillment_status": "not_fulfilled",
      "payment_status": "captured",
      "total": 4999900,
      "currency_code": "inr",
      "created_at": "2026-01-15T10:30:00Z",
      "items": [ ... ]
    }
  ]
}
```

---

### GET `/store/orders/:id`
Retrieve a single order by ID.

**Response: 200 OK** — Full `order` object with items, shipping address, payment, fulfilments.

---

### GET `/store/orders?display_id=1042&fields=id,status`
Retrieve order by human-readable display ID (for order confirmation page).

---

## 5. Customers

### POST `/store/customers`
Register a new customer account.

**Request Body:**
```json
{
  "first_name": "Arjun",
  "last_name": "Sharma",
  "email": "arjun@example.com",
  "password": "SecurePass123"
}
```

**Response: 200 OK** — `customer` object (no password field).

---

### POST `/store/customers/auth`
Authenticate and create a session.

**Request Body:**
```json
{
  "email": "arjun@example.com",
  "password": "SecurePass123"
}
```

**Response: 200 OK** — `customer` object. Sets HTTP-only session cookie.

---

### DELETE `/store/customers/auth`
Log out — invalidates session cookie.

**Response: 200 OK**

---

### GET `/store/customers/me`
Get the authenticated customer's profile. Requires valid session cookie.

**Response: 200 OK** — `customer` object with addresses.

---

### POST `/store/customers/password-token`
Request a password reset OTP.

**Request Body:**
```json
{ "email": "arjun@example.com" }
```

**Response: 204 No Content**

---

### POST `/store/customers/password-reset`
Reset password with OTP token.

**Request Body:**
```json
{
  "email": "arjun@example.com",
  "token": "abc123",
  "password": "NewSecurePass456"
}
```

**Response: 200 OK** — Updated `customer` object.

---

### POST `/store/customers/me/addresses`
Add a new delivery address.

**Request Body:**
```json
{
  "first_name": "Arjun",
  "last_name": "Sharma",
  "address_1": "42 MG Road",
  "address_2": "Flat 4B",
  "city": "Pune",
  "province": "Maharashtra",
  "postal_code": "411001",
  "country_code": "in",
  "phone": "9876543210"
}
```

**Response: 200 OK** — Updated `customer` object with new address.

---

## 6. Wishlist (Custom Route)

### GET `/store/wishlist`
Get wishlist items for authenticated customer.

**Response: 200 OK**
```json
{
  "wishlist": [
    {
      "id": "wl_01ABCDEF",
      "product_id": "prod_01ABCDEF",
      "product": { "id": "...", "title": "...", "thumbnail": "...", "variants": [...] }
    }
  ]
}
```

---

### POST `/store/wishlist`
Add a product to wishlist.

**Request Body:**
```json
{ "product_id": "prod_01ABCDEF" }
```

**Response: 200 OK** — Updated wishlist.

---

### DELETE `/store/wishlist/:item_id`
Remove item from wishlist.

**Response: 200 OK** — Updated wishlist.

---

## 7. Webhooks

### POST `/api/webhooks/razorpay`
Razorpay sends payment events to this endpoint.

**Headers Required:**
```
X-Razorpay-Signature: {hmac_sha256_signature}
Content-Type: application/json
```

**Event Payload (payment.captured):**
```json
{
  "entity": "event",
  "event": "payment.captured",
  "payload": {
    "payment": {
      "entity": {
        "id": "pay_XXXXXXXXXXXXXXXX",
        "order_id": "order_XXXXXXXXXXXXXXXX",
        "amount": 4999900,
        "currency": "INR",
        "status": "captured"
      }
    }
  }
}
```

**Processing Logic:**
1. Extract `X-Razorpay-Signature` header
2. Compute `HMAC-SHA256(RAZORPAY_WEBHOOK_SECRET, raw_request_body)`
3. Compare: reject with `400` if mismatch
4. Check if order already processed (idempotency): return `200` if yes, skip processing
5. Call `POST /store/carts/:id/complete` to finalise order
6. Emit `order.placed` event → triggers confirmation email

**Response: 200 OK** (always respond 200 quickly; process async if needed)

---

## 8. Error Response Schema

```json
{
  "type": "invalid_data",
  "message": "variant_id is required",
  "code": "CART_INVALID_DATA"
}
```

**Common Error Types:**
| Type | HTTP Status | Description |
|---|---|---|
| `not_found` | 404 | Resource does not exist |
| `invalid_data` | 400 | Missing or malformed request data |
| `unauthorized` | 401 | No valid session cookie |
| `forbidden` | 403 | Authenticated but not authorised |
| `duplicate_error` | 409 | Unique constraint violation |
| `payment_authorization_error` | 422 | Payment verification failed |
| `insufficient_inventory` | 422 | Requested quantity > available stock |
| `unexpected_state` | 500 | Internal server error |

---

## 9. Rate Limits

| Endpoint | Limit | Window |
|---|---|---|
| `POST /store/customers/auth` | 5 requests | 15 minutes per IP |
| `POST /store/customers` | 10 requests | 60 minutes per IP |
| `POST /store/carts/:id/complete` | 3 requests | 1 minute per customer |
| `POST /api/webhooks/*` | Whitelisted Razorpay IPs only | — |

Rate limit exceeded: `429 Too Many Requests` with `Retry-After` header.

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
