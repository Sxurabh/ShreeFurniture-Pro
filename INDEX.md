# 📁 INDEX — Shree Furniture Pro

> One-line guide to every file in this project. Read this first when you're lost.

---

## 🔴 Session Start (Read These First, In Order)

| File | What It Does |
|------|--------------|
| [STATUS.md](./STATUS.md) | Current build phase + last completed task — **read before touching any code** |
| [PREFERENCES.md](./PREFERENCES.md) | Owner's design & UX preferences — overrides AI defaults |
| [REJECTIONS.md](./REJECTIONS.md) | Patterns already rejected by owner — never suggest these again |
| [STACK-CHANGES.md](./STACK-CHANGES.md) | Mid-development tech stack changes — supersedes CLAUDE.md if entries exist |
| [CLAUDE.md](./CLAUDE.md) | Hard technical rules: money in paise, no `any`, Medusa v2 only, rendering strategy |
| [NewDocs/design-reference.md](./NewDocs/design-reference.md) | IKEA-inspired visual language — read before any UI/component work |

---

## 🗺️ Planning & Roadmap

| File | What It Does |
|------|--------------|
| [MY-PHASE-BY-PHASE-GUIDE.md](./MY-PHASE-BY-PHASE-GUIDE.md) | Master feature roadmap split by Phase 1 / 2 / 3 |
| [16-WEEK-SPRINT-PLAN.md](./16-WEEK-SPRINT-PLAN.md) | Week-by-week schedule mapping sprints to available time slots |
| [MY-DAILY-CODING-GUIDE.md](./MY-DAILY-CODING-GUIDE.md) | Daily workflow checklist — what to do every time you sit down to code |
| [Change-guide.md](./Change-guide.md) | Mid-development stack or architecture change process — use when pivoting |
| [PROMPT-GUIDE-v2.md](./PROMPT-GUIDE-v2.md) | How to write effective prompts to get the best output from the AI |

---

## 📐 Technical Specs (`NewDocs/`)

| File | What It Does |
|------|--------------|
| [01-product-requirements.md](./NewDocs/01-product-requirements.md) | What the product does and who it's for |
| [02-user-stories-and-acceptance-criteria.md](./NewDocs/02-user-stories-and-acceptance-criteria.md) | Feature requirements written as user stories with pass/fail criteria |
| [03-information-architecture.md](./NewDocs/03-information-architecture.md) | Sitemap and page hierarchy |
| [04-system-architecture.md](./NewDocs/04-system-architecture.md) | High-level system design: storefront ↔ MedusaJS ↔ DB |
| [05-database-schema.md](./NewDocs/05-database-schema.md) | PostgreSQL table definitions and relationships |
| [06-api-contracts.md](./NewDocs/06-api-contracts.md) | API endpoint specs with request/response shapes |
| [07-monorepo-structure.md](./NewDocs/07-monorepo-structure.md) | pnpm workspace layout and package responsibilities |
| [08-cart-pricing-engine-spec.md](./NewDocs/08-cart-pricing-engine-spec.md) | Cart logic, price calculation rules, paise handling |
| [09-engineering-scope-definition.md](./NewDocs/09-engineering-scope-definition.md) | What is in/out of scope for engineering this phase |
| [10-development-phases.md](./NewDocs/10-development-phases.md) | Detailed phase breakdown with feature checklists |
| [11-environment-and-devops.md](./NewDocs/11-environment-and-devops.md) | `.env` variables, Railway/Vercel config, CI/CD pipeline |
| [12-testing-strategy.md](./NewDocs/12-testing-strategy.md) | Vitest unit tests + Playwright E2E — what to test and how |
| [13-design-system.md](./NewDocs/13-design-system.md) | Tokens, typography, spacing, component library rules |
| [14-razorpay-integration-spec.md](./NewDocs/14-razorpay-integration-spec.md) | Payment flow, webhook handling, signature verification |
| [15-algolia-schema.md](./NewDocs/15-algolia-schema.md) | Search index structure and sync strategy |
| [16-email-templates-spec.md](./NewDocs/16-email-templates-spec.md) | Transactional email layouts (order confirm, shipping, etc.) |
| [17-india-specific-guide.md](./NewDocs/17-india-specific-guide.md) | GST, INR formatting, Indian address formats, regional rules |
| [18-cloudinary-guide.md](./NewDocs/18-cloudinary-guide.md) | Image upload pipeline, transformations, and CDN usage |

---

## 🧠 Dev Knowledge (`NewDocs/`)

| File | What It Does |
|------|--------------|
| [MEDUSA-V2-PATTERNS.md](./NewDocs/MEDUSA-V2-PATTERNS.md) | Correct MedusaJS v2 API patterns — read before any backend/client code |
| [CONVENTIONS.md](./NewDocs/CONVENTIONS.md) | Naming conventions, import order, file structure rules |
| [DECISIONS.md](./NewDocs/DECISIONS.md) | Architecture decision log — why we chose X over Y |
| [KNOWN-ISSUES.md](./NewDocs/KNOWN-ISSUES.md) | Current bugs and workarounds — check before debugging |
| [README.md](./NewDocs/README.md) | NewDocs folder overview and reading order |

---

## 🎨 Presentation

| File | What It Does |
|------|--------------|
| [CLIENT-PRESENTATION.md](./CLIENT-PRESENTATION.md) | Slides/talking points for showing the project to stakeholders |

---

*Last updated: March 2026 — Add new files here as they're created.*
