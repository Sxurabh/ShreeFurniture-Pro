# AI Navigation Map

Quick reference — find the right doc fast.

---

## By Task

| I need to... | Go to |
|-------------|-------|
| Start a new coding session | [START-HERE.md](./START-HERE.md) |
| Check what's been built | [START-HERE.md → Current Status](./START-HERE.md#step-2--current-project-status) |
| Look up a database table or field | [database-schema.md](../architecture/database-schema.md) |
| Add a new server action | [api-contracts.md](../architecture/api-contracts.md) |
| Set up a Supabase client | [supabase-client-setup.md](../architecture/supabase-client-setup.md) |
| Implement pricing/coupon/stock logic | [computation-logic-engine-spec.md](../engineering/computation-logic-engine-spec.md) |
| Place a file in the right folder | [monorepo-structure.md](../engineering/monorepo-structure.md) |
| Check if a feature is in MVP scope | [01-mvp-tech-document.md](../source-of-truth/01-mvp-tech-document.md) |
| Understand the checkout payment flow | [02-system-design-document.md → Section 2.3](../source-of-truth/02-system-design-document.md) |
| Know which env var to use | [.env.example](../../.env.example) |
| Understand a user journey | [03-product-requirements-document.md](../source-of-truth/03-product-requirements-document.md) |
| Review security patterns | [04-architecture-document.md → Section 5](../source-of-truth/04-architecture-document.md) |
| Check component placement rules | [ai-coding-standards.md](./ai-coding-standards.md#component-rules) |
| Resolve conflicting information | [decisions.md](./decisions.md) then check priority order in [README.md](../README.md) |

---

## By Domain

| Domain | Primary Doc | Secondary Doc |
|--------|------------|--------------|
| Auth / sessions | [supabase-client-setup.md](../architecture/supabase-client-setup.md) | [04-architecture-document.md § 5](../source-of-truth/04-architecture-document.md) |
| Products / catalog | [database-schema.md](../architecture/database-schema.md) | [product-requirements.md](../product/product-requirements.md) |
| Cart | [api-contracts.md](../architecture/api-contracts.md) | [decisions.md](./decisions.md) |
| Checkout / payments | [api-contracts.md](../architecture/api-contracts.md) | [computation-logic-engine-spec.md](../engineering/computation-logic-engine-spec.md) |
| Orders | [database-schema.md](../architecture/database-schema.md) | [api-contracts.md](../architecture/api-contracts.md) |
| Admin panel | [user-stories-and-acceptance-criteria.md](../product/user-stories-and-acceptance-criteria.md) | [01-mvp-tech-document.md](../source-of-truth/01-mvp-tech-document.md) |
| SEO / performance | [04-architecture-document.md § 6](../source-of-truth/04-architecture-document.md) | [01-mvp-tech-document.md § 4 Phase 4](../source-of-truth/01-mvp-tech-document.md) |
| DevOps / deploy | [environment-and-devops.md](../devops/environment-and-devops.md) | [04-architecture-document.md § 6](../source-of-truth/04-architecture-document.md) |
| Testing | [testing-strategy.md](../devops/testing-strategy.md) | — |