# 11 — Environment & DevOps
## Shree Furniture | E-Commerce Platform

> **Version:** 1.0 | **Scope:** Environment variables, local dev setup, CI/CD, deployment

---

## 1. Environment Overview

| Environment | Storefront | Backend | Database | Redis |
|---|---|---|---|---|
| Local Dev | `localhost:3000` | `localhost:9000` | Docker PostgreSQL | Docker Redis |
| Staging | Vercel preview URL | Railway staging service | Neon.tech dev branch | Upstash dev DB |
| Production | Vercel (custom domain) | Railway production service | Neon.tech production | Upstash production |

---

## 2. Environment Variables

### 2.1 Storefront (`apps/storefront/.env.local`)

```bash
# ── Medusa Backend ────────────────────────────────────────────
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
# Production: https://api.shree-furniture.in

# ── Razorpay (Public key only — safe to expose) ───────────────
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_test_XXXXXXXXXXXXXXXX
# Production: rzp_live_XXXXXXXXXXXXXXXX

# ── Algolia (Public keys — safe to expose) ───────────────────
NEXT_PUBLIC_ALGOLIA_APP_ID=XXXXXXXXXX
NEXT_PUBLIC_ALGOLIA_SEARCH_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_ALGOLIA_INDEX_NAME=products

# ── PostHog (Public key — safe to expose) ────────────────────
NEXT_PUBLIC_POSTHOG_KEY=phc_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# ── Sentry ────────────────────────────────────────────────────
NEXT_PUBLIC_SENTRY_DSN=https://XXXXXXXX@o0.ingest.sentry.io/0
SENTRY_AUTH_TOKEN=sntrys_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# (SENTRY_AUTH_TOKEN is server-only; used for source map upload at build time)

# ── Razorpay Webhook Secret (SERVER ONLY — never NEXT_PUBLIC_) ─
RAZORPAY_WEBHOOK_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

---

### 2.2 Backend (`backend/.env`)

```bash
# ── Database ──────────────────────────────────────────────────
DATABASE_URL=postgresql://user:password@localhost:5432/shree_furniture
# Production (Neon.tech): postgresql://user:password@ep-xxx.ap-south-1.aws.neon.tech/shree_furniture?sslmode=require

# ── Redis ─────────────────────────────────────────────────────
REDIS_URL=redis://localhost:6379
# Production (Upstash): rediss://default:XXXXXXXX@xxxxx.upstash.io:6379

# ── MedusaJS Core ─────────────────────────────────────────────
JWT_SECRET=a-very-long-random-secret-string-at-least-64-chars
COOKIE_SECRET=another-very-long-random-secret-at-least-64-chars
MEDUSA_ADMIN_ONBOARDING_TYPE=nextjs
MEDUSA_ADMIN_ONBOARDING_NEXTJS_PORT=3000

# ── Razorpay ──────────────────────────────────────────────────
RAZORPAY_ID=rzp_test_XXXXXXXXXXXXXXXX
RAZORPAY_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
RAZORPAY_WEBHOOK_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# ── Cloudinary ────────────────────────────────────────────────
CLOUDINARY_CLOUD_NAME=shree-furniture
CLOUDINARY_API_KEY=XXXXXXXXXXXXXXXX
CLOUDINARY_API_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# ── Resend (Email) ────────────────────────────────────────────
RESEND_API_KEY=re_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
RESEND_FROM_EMAIL=orders@shree-furniture.in

# ── Algolia ───────────────────────────────────────────────────
ALGOLIA_APP_ID=XXXXXXXXXX
ALGOLIA_ADMIN_API_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
ALGOLIA_INDEX_NAME=products

# ── Storefront URL (for CORS + email links) ───────────────────
STORE_CORS=http://localhost:3000
# Production: https://shree-furniture.in
ADMIN_CORS=http://localhost:7001
STORE_URL=http://localhost:3000
# Production: https://shree-furniture.in
```

---

### 2.3 Secret Rotation Rules

| Secret | Rotation Frequency | Action on Rotation |
|---|---|---|
| `JWT_SECRET` / `COOKIE_SECRET` | On suspicion of compromise | Rotates invalidates all active sessions |
| `RAZORPAY_SECRET` | Annually or on compromise | Update in Razorpay dashboard + env vars |
| `RAZORPAY_WEBHOOK_SECRET` | On compromise | Regenerate in Razorpay dashboard |
| `RESEND_API_KEY` | Annually | Regenerate in Resend dashboard |
| `ALGOLIA_ADMIN_API_KEY` | Never in client code | Backend only; regenerate annually |
| Database password | Quarterly | Update `DATABASE_URL` in all environments |

---

## 3. Local Development Setup

### Prerequisites
- Node.js 20 LTS
- pnpm 9+
- Docker Desktop (for PostgreSQL + Redis)

### First-Time Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-org/shree-furniture.git
cd shree-furniture

# 2. Install dependencies
pnpm install

# 3. Start local infrastructure (PostgreSQL + Redis)
docker compose up -d

# 4. Configure environment variables
cp .env.example apps/storefront/.env.local
cp .env.example backend/.env
# Fill in all required values (see Section 2 above)

# 5. Run database migrations
pnpm db:migrate

# 6. Seed initial data (products, collections, regions)
pnpm db:seed

# 7. Start all apps in development mode
pnpm dev
```

### Local Service URLs

| Service | URL |
|---|---|
| Storefront | http://localhost:3000 |
| Medusa Admin | http://localhost:7001 |
| MedusaJS API | http://localhost:9000 |
| PostgreSQL | localhost:5432 |
| Redis | localhost:6379 |

### Docker Compose (`infrastructure/docker-compose.yml`)

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: shree_furniture
      POSTGRES_USER: shree
      POSTGRES_PASSWORD: localpassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --save 60 1

volumes:
  postgres_data:
```

---

## 4. CI Pipeline (`.github/workflows/ci.yml`)

Runs on every pull request targeting `main`.

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v3
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: TypeScript check
        run: pnpm type-check

      - name: Lint
        run: pnpm lint

      - name: Unit tests
        run: pnpm test

      - name: Build check (storefront)
        run: cd apps/storefront && pnpm build
        env:
          NEXT_PUBLIC_MEDUSA_BACKEND_URL: http://localhost:9000
          NEXT_PUBLIC_RAZORPAY_KEY_ID: rzp_test_placeholder
          NEXT_PUBLIC_ALGOLIA_APP_ID: placeholder
          NEXT_PUBLIC_ALGOLIA_SEARCH_API_KEY: placeholder
          NEXT_PUBLIC_ALGOLIA_INDEX_NAME: products

      - name: npm audit (no high/critical)
        run: pnpm audit --audit-level=high
```

---

## 5. Deploy Pipeline (`.github/workflows/deploy.yml`)

Runs on every merge to `main`.

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-storefront:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-backend:
    runs-on: ubuntu-latest
    needs: deploy-storefront
    steps:
      - uses: actions/checkout@v4
      # Railway auto-deploys on push to main via GitHub integration
      # Manual deploy trigger if needed:
      - name: Trigger Railway deploy
        run: |
          curl -X POST \
            -H "Authorization: Bearer ${{ secrets.RAILWAY_API_TOKEN }}" \
            "https://backboard.railway.app/graphql/v2" \
            -d '{"query": "mutation { deploymentTrigger(input: {serviceId: \"${{ secrets.RAILWAY_SERVICE_ID }}\"}) { id } }"}'

  smoke-test:
    runs-on: ubuntu-latest
    needs: [deploy-storefront, deploy-backend]
    steps:
      - name: Health check — Medusa API
        run: |
          curl --fail https://api.shree-furniture.in/health || exit 1
      - name: Health check — Storefront
        run: |
          curl --fail https://shree-furniture.in || exit 1
```

---

## 6. GitHub Secrets Required

| Secret | Used In | Value From |
|---|---|---|
| `VERCEL_TOKEN` | deploy.yml | Vercel → Account Settings → Tokens |
| `VERCEL_ORG_ID` | deploy.yml | Vercel → `.vercel/project.json` |
| `VERCEL_PROJECT_ID` | deploy.yml | Vercel → `.vercel/project.json` |
| `RAILWAY_API_TOKEN` | deploy.yml | Railway → Account Settings → Tokens |
| `RAILWAY_SERVICE_ID` | deploy.yml | Railway → Service → Settings |

---

## 7. Phase 2 — VPS Deploy (Hetzner)

When migrating to Hetzner CX31 VPS (Ubuntu 22.04):

### Initial Server Setup
```bash
# UFW Firewall
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw enable

# Install Node.js 20 via nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20

# Install pnpm
npm install -g pnpm

# Install PM2
npm install -g pm2

# Install Certbot
apt install certbot python3-certbot-nginx -y
```

### Nginx Config (`infrastructure/nginx/shree-furniture.conf`)
```nginx
server {
    listen 443 ssl http2;
    server_name shree-furniture.in www.shree-furniture.in;

    ssl_certificate /etc/letsencrypt/live/shree-furniture.in/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/shree-furniture.in/privkey.pem;

    # Storefront
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # MedusaJS API
    location /api/ {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Medusa Admin
    location /admin/ {
        # Optional: restrict to known IPs in production
        # allow 103.x.x.x;
        # deny all;
        proxy_pass http://localhost:7001;
    }
}

server {
    listen 80;
    server_name shree-furniture.in www.shree-furniture.in;
    return 301 https://$host$request_uri;
}
```

### PM2 Ecosystem File
```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'storefront',
      cwd: './apps/storefront',
      script: 'node_modules/.bin/next',
      args: 'start',
      env: { NODE_ENV: 'production', PORT: 3000 }
    },
    {
      name: 'medusa',
      cwd: './backend',
      script: 'node_modules/.bin/medusa',
      args: 'start',
      env: { NODE_ENV: 'production', PORT: 9000 }
    }
  ]
}
```

---

## 8. Monitoring Checklist

| Tool | What to Monitor | Alert Condition |
|---|---|---|
| Uptime Kuma | `https://shree-furniture.in` HTTP 200 | Alert if down > 2 minutes |
| Uptime Kuma | `https://api.shree-furniture.in/health` | Alert if down > 2 minutes |
| Sentry | JS errors, API 500s | Any new error pattern |
| Vercel Analytics | LCP, INP, CLS per route | LCP > 3s on any route |
| PostHog | Checkout funnel drop-off | Drop-off > 60% at any step |
| Netdata (Phase 2 VPS) | CPU > 80%, RAM > 85%, Disk > 80% | Alert via Netdata Cloud |

---

*Owner: Engineering Lead | Shree Furniture | v1.0 — Q1 2026*
