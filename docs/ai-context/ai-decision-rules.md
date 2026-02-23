# AI Decision Rules

When unsure how to proceed, follow these rules in order.

---

## Rule 1 — Check if the decision is already made

Read [decisions.md](./decisions.md) first. If the decision is listed there, follow it. Do not propose alternatives.

---

## Rule 2 — Check the relevant architecture doc

| Situation | Check |
|-----------|-------|
| Schema / database | [database-schema.md](../architecture/database-schema.md) |
| Server action shape | [api-contracts.md](../architecture/api-contracts.md) |
| Business/pricing logic | [computation-logic-engine-spec.md](../engineering/computation-logic-engine-spec.md) |
| File location | [monorepo-structure.md](../engineering/monorepo-structure.md) |
| Security approach | [architecture document — security section](../source-of-truth/04-architecture-document.md) |

---

## Rule 3 — Prioritize security, then performance, then simplicity

1. **Security first** — if two approaches both work, pick the more secure one
2. **Performance second** — prefer RSC over client components, ISR over SSR where suitable
3. **Simplest scalable solution** — avoid over-engineering for MVP; note where future scale will require changes

---

## Rule 4 — When adding a new database field

1. Check `database-schema.md` — does the field exist?
2. If not, **stop and ask** — never invent fields
3. If you believe a field is genuinely missing, flag it explicitly: _"I believe we need a new field X on table Y for reason Z — please confirm before I proceed"_

---

## Rule 5 — When something seems to conflict between docs

The priority order for conflicts is:

1. `/docs/source-of-truth/02-system-design-document.md` (most detailed, most correct schema)
2. `/docs/architecture/database-schema.md`
3. Other docs

Flag the conflict in your response so it can be resolved in the docs.

---

## Rule 6 — Component placement

| Question | Answer |
|----------|--------|
| Does it fetch data from Supabase? | Server Component |
| Does it use `useState`, `useEffect`, or event handlers? | Client Component |
| Does it use a Zustand store? | Client Component |
| Does it only receive props and render UI? | Server Component (prefer) |
| Is it a form? | Client Component (React Hook Form) |

---

## Rule 7 — When in doubt about scope

Check the MVP feature list in [01-mvp-tech-document.md](../source-of-truth/01-mvp-tech-document.md). If a feature is marked P2 or Out of Scope, do not build it. Note it as future scope instead.