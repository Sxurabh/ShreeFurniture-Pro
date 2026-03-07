# 🚀 Shree Furniture | E-Commerce Platform V1 
## Project Roadmap, Timeline & Expected Costs
*Prepared for Client Review — March 2026*

---

## 1. Executive Summary

We are building a premium, custom e-commerce storefront and robust backend management system for **Shree Furniture**. 

The platform will be built on modern, secure, and highly scalable technologies:
*   **Customer Storefront:** Next.js (Fast, SEO-optimized, visually stunning).
*   **E-Commerce Engine:** MedusaJS (Powerful admin dashboard, product & order management).
*   **Search & Discovery:** Algolia (Instant search, intelligent filtering).
*   **Payments:** Razorpay (Seamless, secure, local payment methods).

### Project Development Strategy
To optimize costs while ensuring speed and security, we are utilizing a **"Build Serverless, Run Dedicated"** deployment model:
1.  **Months 1-4 (Development & Launch):** We will build and launch using generous free-tier cloud platforms (Vercel, Railway, Neon Database). This keeps development hosting costs at **₹0**.
2.  **Month 5+ (Post-Launch Scale):** Once the site is generating steady traffic and sales, we seamlessly migrate the traffic to a high-performance Dedicated Virtual Private Server (VPS). This guarantees fast loading times and predictable, low fixed costs vs. expensive cloud metering.

---

## 2. The 16-Week Implementation Timeline

The project is broken into a 16-week (approx. 4 months) sprint schedule, working on a sustainable cadence of ~10 hours per week.

### 🏗️ Phase 1: Foundation Setup (Weeks 1–4)
*   **Objective:** Scaffold the system architecture, connect to secure databases, and configure third-party services.
*   **Key Deliverables:** 
    *   Backend connected to PostgreSQL.
    *   "Shree Furniture" brand styling foundations established.
    *   Algolia search engine, Cloudinary image hosting, and Resend email services wired up.
    *   PostHog analytics and Sentry monitoring installed to track customer behavior and catch errors.

### 🏪 Phase 2: The Storefront (Weeks 5–8)
*   **Objective:** Build the public-facing storefront where customers will browse products.
*   **Key Deliverables:** 
    *   Premium Homepage (Hero banners, featured collections, best sellers).
    *   Dynamic Product Listing Pages (Filters, fast loading product grids).
    *   Detailed Product Pages (Image galleries, variant selection, delivery estimates).
    *   High-speed Algolia Search overlay.

### 💳 Phase 3: Transactions & Accounts (Weeks 9–13)
*   **Objective:** Enable customers to securely purchase furniture and manage their accounts.
*   **Key Deliverables:** 
    *   Slide-out Cart drawer with Free Shipping calculation progress bars.
    *   3-Step secure Checkout process (Address validation, Shipping, Payment).
    *   Live Razorpay payment integration and Order Confirmation screens.
    *   Customer Account portals (Order history, Wishlist, Saved addresses).
    *   Automated transactional emails (Order Confirmed, Order Shipped).

### 🚀 Phase 4: QA, Security & Go-Live! (Weeks 14–16)
*   **Objective:** Ensure the site is secure, ranks well on Google, and is deployed flawlessly.
*   **Key Deliverables:** 
    *   Technical SEO implementation (Sitemaps, structured data, Meta tags).
    *   Security hardening (Dependency audits, HTTP-only cookie validation).
    *   Performance optimization (Google Lighthouse audit for maximum speed).
    *   **V1 LAUNCH** on the primary `shree-furniture.in` domain.

---

## 3. Estimated Cost Breakdown (in INR)

We are minimizing overhead by only paying for the exact tools required to write and host the code. 

### A. Development Tooling (Months 1 to 4)
*To build the platform, we utilize an AI-assisted IDE (Cursor) to dramatically accelerate development speed.*
*   **Cursor Pro Subscription:** ~$20 USD (₹1,660) / month.
*   **Total Dev Tooling for 4 Months:** **~₹6,640 INR**

### B. Launch / Month 1 Hosting
*   **Vercel & Railway (Hosting):** Free Tier
*   **Neon Database:** Free Tier
*   **Total Launch Server Cost:** **₹0 INR**

### C. Post-Launch Ongoing Production Costs (Month 5+)
*Once the business scales, we migrate from free cloud services to a cheap, ultra-fast Dedicated Server (VPS) via Hetzner.*
*   **AI Dev Tooling (Cursor):** *Can be canceled post-launch if no active development.*
*   **Hetzner CX31 VPS (Servers):** ~$6.50 USD / month.
*   **Cloudflare DNS / Caching:** Free
*   **Total Ongoing Infrastructure:** **~₹550 INR / month**

### D. Third-Party Service Fees (Usage Based)
*The below services only charge based on volume, and all have generous free tiers that easily cover new businesses.*
*   **Razorpay:** Standard transaction fees (e.g., 2% per transaction). No fixed monthly fee.
*   **Algolia (Search):** Free for 10,000 monthly search requests. 
*   **Resend (Email):** Free for 3,000 emails/month.
*   **Cloudinary (Images):** Free up to 25 monthly credits (approx. 25,000 image transformations).

---

### End Goal Summary
By the end of Week 16, **Shree Furniture** will own a blazing-fast, custom-built e-commerce engine with zero monthly software licensing fees (no Shopify or Magento subscriptions), incredibly low ongoing hosting costs (~₹550/mo), and full ownership of their data and code. 
