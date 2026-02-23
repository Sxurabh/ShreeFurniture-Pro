# Product Requirements — Shree Furniture

## Product Goals

### Business Goals
- Launch revenue-generating storefront in 10–12 weeks
- Achieve ₹5L monthly GMV within 6 months
- Enable online expansion beyond local geography
- Reduce manual order handling

### Experience Goals
- Mobile-first UX
- < 3s load time
- Conversion rate ≥ 2.5%
- Trust-first purchase flow

### Success Metrics
- Conversion rate
- Average order value
- Cart abandonment rate
- Repeat purchase rate
- Page load performance (LCP < 2.5s)

---

## MVP Scope

### Customer
- Browse & filter catalog
- Product detail with specs & dimensions
- Search
- Cart & checkout
- Guest & account checkout
- Razorpay payments (UPI-first)
- Order confirmation & tracking
- Wishlist

### Admin
- Product & category management
- Inventory control
- Order management
- Discount & banner management

---

## Future Phases
- Reviews & ratings
- Pincode delivery estimation
- WhatsApp notifications
- Returns & refunds workflow
- Advanced analytics
- AI recommendations

---

## Functional Requirements

### Catalog
- Category hierarchy
- Variant support
- High-quality images

### Checkout
- Guest checkout
- Coupon support
- Address saving

### Orders
- Order lifecycle tracking
- Email confirmation

---

## Non-Functional Requirements

### Performance
- Lighthouse ≥ 85 mobile
- CDN image delivery
- Lazy loading & caching

### Security
- Supabase RLS enforcement
- Secure payment verification
- Admin role access control

### Scalability
- Handle 10k+ products
- Handle peak sale traffic

---

## Constraints & Assumptions

- Single warehouse inventory
- GST inclusive pricing
- Pan-India shipping (free above threshold)
- Static delivery estimate (5–7 days)
- Email notifications only (MVP)