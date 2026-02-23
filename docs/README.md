# Documentation Index — Shree Furniture

---

## AI Context (READ FIRST — every session)

| Doc | Purpose |
|-----|---------|
| [START HERE](./ai-context/START-HERE.md) | Entry point — status tracker, navigation guide |
| [AI Project Context](./ai-context/ai-project-context.md) | Stack, routes, goals, constraints |
| [AI Guardrails](./ai-context/ai-guardrails.md) | Hard rules + source-of-truth pointers |
| [AI Decision Rules](./ai-context/ai-decision-rules.md) | How to resolve ambiguity |
| [AI Coding Standards](./ai-context/ai-coding-standards.md) | TypeScript, components, naming conventions |
| [Resolved Decisions](./ai-context/decisions.md) | ⚠️ Finalized decisions — do not re-litigate |
| [AI Navigation Map](./ai-context/ai-navigation-map.md) | Quick doc directory |

---

## Architecture & System Design

| Doc | Purpose |
|-----|---------|
| [System Architecture](./architecture/system-architecture.md) | Component overview, service boundaries |
| [Database Schema](./architecture/database-schema.md) | ⭐ Canonical schema — all tables, RLS, indexes |
| [API Contracts](./architecture/api-contracts.md) | Server actions, edge functions, webhook contracts |
| [Supabase Client Setup](./architecture/supabase-client-setup.md) | The 3 client patterns with code |

---

## Product & UX

| Doc | Purpose |
|-----|---------|
| [Product Requirements](./product/product-requirements.md) | Business goals, MVP scope |
| [User Stories & Acceptance Criteria](./product/user-stories-and-acceptance-criteria.md) | Feature-level AC |
| [Information Architecture](./product/information-architecture.md) | Site map, URL structure, navigation |

---

## Engineering & Logic

| Doc | Purpose |
|-----|---------|
| [Development Phases](./engineering/development-phases.md) | Phase plan and milestones |
| [Engineering Scope Definition](./engineering/engineering-scope-definition.md) | Deliverables and risks |
| [Computation Logic Engine Spec](./engineering/computation-logic-engine-spec.md) | Pricing, coupon, inventory logic |
| [Project Structure](./engineering/monorepo-structure.md) | File/folder structure |

---

## DevOps & Testing

| Doc | Purpose |
|-----|---------|
| [Environment & DevOps](./devops/environment-and-devops.md) | CI/CD, environments |
| [Testing Strategy](./devops/testing-strategy.md) | Unit, integration, E2E, security |

---

## Environment

| File | Purpose |
|------|---------|
| [.env.example](../.env.example) | All required environment variables with documentation |

---

## Source of Truth Documents

> Use these for deep reference. For quick answers, use the architecture docs above.

| Doc | Purpose |
|-----|---------|
| [01 MVP Tech Document](./source-of-truth/01-mvp-tech-document.md) | Feature priority list, phases, infrastructure |
| [02 System Design Document](./source-of-truth/02-system-design-document.md) | Data flows, detailed DB SQL, scalability |
| [03 Product Requirements Document](./source-of-truth/03-product-requirements-document.md) | Personas, journeys, acceptance criteria, metrics |
| [04 Architecture Document](./source-of-truth/04-architecture-document.md) | Full architecture with code examples |

---

## Conflict Resolution

If docs appear to conflict, use this priority order:

1. `/docs/ai-context/decisions.md` (final decisions)
2. `/docs/architecture/database-schema.md` (canonical schema)
3. `/docs/source-of-truth/02-system-design-document.md` (most detailed)
4. Shipping policy details in `/docs/engineering/computation-logic-engine-spec.md`
5. All other docs