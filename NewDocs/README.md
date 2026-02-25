# NewDocs — Canonical AI Documentation Index
## Shree Furniture | Anti-Drift / Anti-Hallucination Guide

> This folder is the **single source of truth** for planning and implementation decisions.
> If any other document conflicts with this folder, **NewDocs wins**.

---

## 1) Why this file exists

This index is designed so AI agents (including Antigravity agents) can:
- load docs in the correct order,
- avoid conflicting guidance,
- stay aligned with MVP scope,
- avoid inventing out-of-scope work.

---

## 2) Mandatory loading order (do not skip)

1. `CLAUDE.md` — project identity, hard rules, tech stack guardrails
2. `DECISIONS.md` — ADR rationale and non-negotiable technical choices
3. `09-engineering-scope-definition.md` — what to build vs configure vs explicitly avoid
4. `10-development-phases.md` — delivery sequence and phase boundaries
5. `CONVENTIONS.md` — naming, code style, architecture implementation conventions
6. Feature-specific docs (`01`–`08`, `11`, `12`) only when relevant to the task

If a task touches checkout/cart/payments, always read `08-cart-pricing-engine-spec.md` before coding.

---

## 3) Canonical document map

### Core guardrails (always authoritative)
- `CLAUDE.md`
- `DECISIONS.md`
- `CONVENTIONS.md`
- `09-engineering-scope-definition.md`
- `10-development-phases.md`

### Product and architecture specs
- `01-product-requirements.md`
- `02-user-stories-and-acceptance-criteria.md`
- `03-information-architecture.md`
- `04-system-architecture.md`
- `05-database-schema.md`
- `06-api-contracts.md`
- `07-monorepo-structure.md`
- `08-cart-pricing-engine-spec.md`
- `11-environment-and-devops.md`
- `12-testing-strategy.md`

---

## 4) Conflict resolution policy for agents

When two statements appear inconsistent, resolve in this order:
1. `CLAUDE.md` hard rules
2. `DECISIONS.md` ADR decisions
3. `09-engineering-scope-definition.md` scope boundaries
4. Feature spec docs (`01`–`08`, `11`, `12`)
5. Any legacy notes outside `NewDocs/`

Never use assumptions to fill missing requirements; document uncertainty and ask for clarification.

---

## 5) File placement policy (keep agent behavior intact)

- Keep all planning and architecture docs at `NewDocs/` root.
- Keep files numbered and immutable in sequence (`01` to `12`) so references stay stable.
- Keep guardrail files (`CLAUDE.md`, `DECISIONS.md`, `CONVENTIONS.md`, this `README.md`) unnumbered at root.
- Do not duplicate active specs in secondary folders.
- Any deprecated or historical docs must move to a clearly named `archive/` folder and must include `DEPRECATED` in the title.

---

## 6) Maintenance checklist

Before merging doc updates:
- Verify all internal links and paths point to `NewDocs/...`.
- Verify no duplicate “source of truth” copies exist.
- Verify hard rules in `CLAUDE.md` still match `DECISIONS.md` and `09-engineering-scope-definition.md`.
- Verify phase labels in `10-development-phases.md` still match priorities in `01-product-requirements.md`.

---

*Owner: Engineering + Product | Status: Active Canonical Index | Version: 1.0*
