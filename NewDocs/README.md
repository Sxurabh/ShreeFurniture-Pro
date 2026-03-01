# NewDocs — Canonical AI Documentation Index
## Shree Furniture | Anti-Drift / Anti-Hallucination Guide

> This folder is the **single source of truth** for planning and implementation decisions.
> If any other document conflicts with this folder, **NewDocs wins.**
> Exception: root-level owner preference files (PREFERENCES.md, REJECTIONS.md) govern
> visual taste and rejected patterns — these are additive, not conflicting with specs here.

---

## 1) Why this file exists

This index is designed so AI agents (including Antigravity agents) can:
- load docs in the correct order,
- avoid conflicting guidance,
- stay aligned with MVP scope,
- avoid inventing out-of-scope work.

---

## 2) Mandatory loading order (do not skip)

**Root files (read first — project root, not NewDocs/):**
1. `STATUS.md` — what is already built (prevents regenerating existing code)
2. `PREFERENCES.md` — owner's design taste (overrides UI agent defaults)
3. `REJECTIONS.md` — patterns explicitly rejected (never suggest these)
4. `STACK-CHANGES.md` — if entries exist, they update the tech stack table in CLAUDE.md

**Then NewDocs/ (in this order):**
5. Root `CLAUDE.md` — project identity, hard rules, tech stack guardrails (auto-loaded by Antigravity)
6. `DECISIONS.md` — ADR rationale and non-negotiable technical choices
7. `09-engineering-scope-definition.md` — what to build vs configure vs explicitly avoid
8. `10-development-phases.md` — delivery sequence and phase boundaries
9. `CONVENTIONS.md` — naming, code style, architecture implementation conventions
10. Feature-specific docs (`01`–`08`, `11`–`13`) only when relevant to the task

If a task touches checkout/cart/payments, always read `08-cart-pricing-engine-spec.md` before coding.
If a task touches UI/components, always read `13-design-system.md` before coding.

---

## 3) Canonical document map

### Owner preference files (root level — additive layer)
- `PREFERENCES.md` — visual taste and UX preferences (fills in as project progresses)
- `REJECTIONS.md` — patterns explicitly rejected after review
- `STACK-CHANGES.md` — mid-project technology substitutions

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
- `13-design-system.md` ← design tokens, component specs, interaction states

---

## 4) Conflict resolution policy for agents

When two statements appear inconsistent, resolve in this order:
1. `CLAUDE.md` hard rules (security, money, TypeScript)
2. `DECISIONS.md` ADR decisions (architecture choices and their reasons)
3. `09-engineering-scope-definition.md` scope boundaries
4. `PREFERENCES.md` / `REJECTIONS.md` (owner's visual taste — additive, not conflicting with above)
5. `STACK-CHANGES.md` (tech substitutions — updates stack table only, not feature specs)
6. Feature spec docs (`01`–`08`, `11`–`13`)
7. Any legacy notes outside `NewDocs/`

**Scope of each layer:**
- `CLAUDE.md` governs: what is forbidden, security rules, money handling
- `DECISIONS.md` governs: which technology and why (cannot be overridden without a new ADR)
- `PREFERENCES.md` governs: how something looks and feels (does not conflict with above)
- `STACK-CHANGES.md` governs: which technology replaces which (updates DECISIONS.md in parallel)

Never use assumptions to fill missing requirements; document uncertainty and ask for clarification.

---

## 5) File placement policy (keep agent behavior intact)

- Keep all planning and architecture docs at `NewDocs/` root.
- Keep files numbered and immutable in sequence (`01` to `13`) so references stay stable.
- Keep guardrail files (`CLAUDE.md`, `DECISIONS.md`, `CONVENTIONS.md`, this `README.md`) unnumbered at root.
- Owner preference files (`PREFERENCES.md`, `REJECTIONS.md`, `STACK-CHANGES.md`) live at **project root** (not NewDocs/).
- Do not duplicate active specs in secondary folders.
- Any deprecated or historical docs must move to a clearly named `archive/` folder and must include `DEPRECATED` in the title.

---

## 6) Maintenance checklist

Before merging doc updates:
- Verify all internal links and paths point to `NewDocs/...`.
- Verify no duplicate "source of truth" copies exist.
- Verify hard rules in `CLAUDE.md` still match `DECISIONS.md` and `09-engineering-scope-definition.md`.
- Verify phase labels in `10-development-phases.md` still match priorities in `01-product-requirements.md`.
- Verify `13-design-system.md` component specs don't contradict `packages/ui/src/tokens.ts`.
- Verify `STACK-CHANGES.md` entries are reflected in `CLAUDE.md` tech stack table.

---

*Owner: Engineering + Product | Status: Active Canonical Index | Version: 1.1 — updated for preference files*