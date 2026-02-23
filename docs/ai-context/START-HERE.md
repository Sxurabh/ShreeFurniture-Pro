# START HERE — Shree Furniture

You are working on **Shree Furniture**, a furniture e-commerce platform.

---

## Step 1 — Read These First (in order)

1. [AI Project Context](./ai-project-context.md)
2. [AI Guardrails](./ai-guardrails.md)
3. [AI Decision Rules](./ai-decision-rules.md)
4. [Resolved Decisions](./decisions.md) ← never re-litigate these

---

## Step 2 — Current Project Status

> **Update this section at the start of every coding session.**

| Field | Value |
|-------|-------|
| **Active Phase** | Phase 0 — Project Foundation |
| **Current Week** | Week 1 |

### Phase Completion

| Phase | Name | Status |
|-------|------|--------|
| 0 | Setup & Schema | 🟡 In Progress |
| 1 | Core Customer Experience | ⬜ Not Started |
| 2 | Payments & Orders | ⬜ Not Started |
| 3 | Admin Dashboard | ⬜ Not Started |
| 4 | SEO & Performance | ⬜ Not Started |
| 5 | QA & Launch | ⬜ Not Started |
| 6 | Post-Launch | ⬜ Not Started |

### Completed This Session
- [ ] _(fill in before starting)_

### In Progress
- [ ] _(fill in before starting)_

### Blocked / Needs Decision
- _(none)_

---

## Step 3 — Navigate to the Right Doc Before Making Changes

| Change Type | Read This First |
|-------------|-----------------|
| Database / schema | [Database Schema](../architecture/database-schema.md) |
| API / server actions | [API Contracts](../architecture/api-contracts.md) |
| Business logic | [Computation Logic Engine Spec](../engineering/computation-logic-engine-spec.md) |
| Architecture decisions | [System Architecture](../architecture/system-architecture.md) |
| File/folder structure | [Monorepo Structure](../engineering/monorepo-structure.md) |
| UI components | [AI Coding Standards](./ai-coding-standards.md) |
| Environment variables | [/.env.example](../../.env.example) |
| Auth/security patterns | [Architecture Doc — Security](../source-of-truth/04-architecture-document.md#5-security-architecture) |
| Deployment | [Environment & DevOps](../devops/environment-and-devops.md) |

---

## NEVER

- Invent database fields not in [Database Schema](../architecture/database-schema.md)
- Create API endpoints not in [API Contracts](../architecture/api-contracts.md)
- Bypass RLS security
- Move business logic to client components
- Edit `/src/types/database.ts` manually — it is auto-generated via `supabase gen types`
- Re-litigate decisions listed in [decisions.md](./decisions.md)