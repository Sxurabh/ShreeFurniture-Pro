# Environment & DevOps

## Environments
local
staging
production

---

## CI/CD
GitHub → Vercel preview deploys
main → production deploy

---

## Secrets
Vercel env variables
Supabase keys server-side only

---

## Cost Efficient Hosting
Vercel hobby
Supabase free tier

---

## Monitoring
Vercel analytics
Supabase logs
Sentry (free tier)

### Sentry Minimum Configuration Checklist
- `SENTRY_DSN` configured in environment variables
- Environment tagging enabled for `local`, `staging`, and `production`
- Next.js app wrapped with Sentry integration and error boundary capture
