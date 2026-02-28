# STACK-CHANGES.md — Shree Furniture
## Mid-Project Technology Change Log

> **What this file is:** The authoritative record of every technology decision that changed
> AFTER the project started. Works alongside DECISIONS.md (which records original decisions).
>
> **AI Agents:** Read this file early in your session start sequence.
> If this file has entries in the "Active Overrides" table, those entries supersede
> the **tech stack table in CLAUDE.md** for the specific technologies listed.
> This file does NOT override NewDocs/ feature specs, acceptance criteria, or ADR reasoning —
> it only updates WHICH technology is used, not WHAT is built or HOW it behaves.
>
> **Conflict resolution:** NewDocs/README.md governs feature specs and architecture.
> STACK-CHANGES.md governs technology substitutions only.
>
> **For engineers:** Every stack change also requires updating CLAUDE.md and the
> relevant agent file. This file tracks WHAT changed. Those files say HOW to implement.

---

## ⚡ Active Overrides (Read First)

> This table shows technologies that replaced something else.
> AI agents must use the NEW technology, never the old one.

| Replaced | With | Effective Date | Impact |
|---|---|---|---|
| *(empty — this project hasn't made any stack changes yet)* | | | |

---

## How to Make a Stack Change

### Step 1 — Decide
Before changing anything, use this prompt to understand the full impact:
```
@orchestrator I'm thinking of replacing [OLD] with [NEW].
Before we do anything:
1. List every file in the codebase that references [OLD]
2. List every agent file or skill that mentions [OLD]
3. Estimate the number of components/files that would need rewriting
4. Identify any data migration risks
5. Are there any places where [OLD] and [NEW] can't coexist?
Do NOT make any changes yet. Just give me the impact assessment.
```

### Step 2 — Document
After you decide to proceed, use this prompt:
```
We're replacing [OLD] with [NEW].
1. Add an entry to STACK-CHANGES.md under "Change Log" using the template
2. Update CLAUDE.md — change the tech stack table entry
3. Update the relevant agent file ([agent name]) — replace the old technology references
4. Update DECISIONS.md — add a new ADR that supersedes [old ADR number]
5. Update STATUS.md — mark affected components as 🔄 NEEDS MIGRATION

Do NOT touch any actual code yet. Just update the documentation.
```

### Step 3 — Migrate
Once docs are updated, proceed component by component:
```
@[agent] Migrate [component/file] from [OLD] to [NEW].
Ref: STACK-CHANGES.md entry [date] for context on why this changed.
Ref: CLAUDE.md for the updated tech stack constraints.
Test that [component] still works after migration.
Update STATUS.md when done.
```

### Step 4 — Verify
```
@orchestrator Verify the [OLD] → [NEW] migration is complete.
Search the entire codebase for any remaining references to [OLD]:
- Import statements
- package.json dependencies
- Environment variable names
- Config files
- Comments and README files
Report: any remaining references that need updating.
```

---

## Change Log

> One entry per technology change. Most recent first.

---

### Template (copy this for each change)

```
---
## CHANGE: [OLD TECHNOLOGY] → [NEW TECHNOLOGY]
**Date:** [date]
**Reason:** [why you made this change — be honest, even if "I just didn't like it"]
**Decided by:** [you / team / external constraint]

### Before
Technology: [OLD]
Where it was used: [list files/components]
Config location: [where the old config lived]

### After
Technology: [NEW]
Where it is used: [list files/components]
Config location: [where the new config lives]
Key differences: [2–3 important behaviour differences to know]

### Migration Notes
- [Any gotcha discovered during migration]
- [Any pattern that changed]
- [Any data that needed transforming]

### Files Updated
- [ ] CLAUDE.md — tech stack table
- [ ] DECISIONS.md — new ADR added (ADR-0XX)
- [ ] .agent/agents/[relevant agent].md — references updated
- [ ] PROMPT-GUIDE.md — affected prompts updated
- [ ] STATUS.md — affected components marked for re-review
- [ ] package.json — old dependency removed, new one added
- [ ] .env.example — env var names updated

### Affected Components
| Component | Status |
|---|---|
| [component name] | ✅ Migrated / 🚧 In Progress / ⬜ Pending |
---
```

---

## Worked Examples

> These show you how real changes would look. Delete once you have real entries.

---

### EXAMPLE A: UI Library Change (shadcn → Custom Components)

```
## CHANGE: shadcn/ui → Custom Tailwind Components
Date: [hypothetical]
Reason: Wanted full design control without shadcn's opinionated defaults.
        shadcn's Dialog and Sheet components were hard to customise for the
        bottom-sheet-on-mobile pattern we needed.

### Before
Technology: shadcn/ui
Where used: components/ui/*, CartDrawer, SearchOverlay, all form inputs

### After
Technology: Custom Tailwind CSS components + Radix Primitives for accessibility
Where used: components/ui/* (rebuilt)
Key differences:
- No more `npx shadcn@latest add [component]` — components are owned code now
- Radix Primitives used for Dialog, Popover, Select (accessibility handled)
- All styling is our Tailwind tokens — no shadcn CSS variables conflict

### Migration Notes
- Radix Primitive components (Dialog.Root, etc.) replace shadcn Sheet/Dialog
- Our Button component uses exact same API as shadcn's but with custom styles
- No `cn()` utility needed — replaced with plain Tailwind classes

### Files Updated
- [x] CLAUDE.md — tech stack table: "shadcn/ui" → "Custom + Radix Primitives"
- [x] DECISIONS.md — ADR-015 added (supersedes ADR-003)
- [x] .agent/agents/frontend-specialist.md — updated component library guidance
- [x] STATUS.md — all shadcn components marked 🔄 NEEDS REBUILD
```

---

### EXAMPLE B: Search Provider Change (Algolia → Typesense)

```
## CHANGE: Algolia → Typesense (self-hosted)
Date: [hypothetical]
Reason: Algolia costs escalated at scale. Typesense is free self-hosted on Hetzner.

### Before
Technology: Algolia InstantSearch
Config: ALGOLIA_APP_ID, ALGOLIA_SEARCH_KEY, ALGOLIA_WRITE_KEY env vars
Search component: react-instantsearch-hooks-web

### After
Technology: Typesense + typesense-instantsearch-adapter
Config: TYPESENSE_HOST, TYPESENSE_PORT, TYPESENSE_API_KEY env vars
Search component: Same InstantSearch UI, different adapter

### Migration Notes
- Index name format changes: Algolia "products" → Typesense collection "products"
- Typesense runs on Hetzner VPS (same server as Phase 2 PostgreSQL migration)
- typesense-instantsearch-adapter makes the UI migration almost zero-code
- Algolia subscriber in backend must be replaced with Typesense subscriber

### Files Updated
- [x] CLAUDE.md — tech stack: Algolia → Typesense
- [x] .agent/agents/backend-specialist.md — subscriber pattern updated
- [x] NewDocs/MEDUSA-V2-PATTERNS.md — sync subscriber updated
- [x] .env.example — new env vars, old ones removed
```

---

## Common Stack Change Scenarios & Prompts

### "I want to swap the UI library"

**Impact assessment first:**
```
@explorer-agent List every file that imports from shadcn/ui (or whichever library).
Also check: any CSS variables that come from shadcn's theme system.
Don't change anything. Just report.
```

**Then document and migrate:**
```
We're replacing shadcn/ui with [NEW LIBRARY].
Reason: [your reason]
Follow the STACK-CHANGES.md protocol: document first, then migrate component by component.
Start with the simplest component (Button) to test the pattern before touching Cart or Checkout.
```

---

### "I want to switch search providers"

```
@orchestrator We're switching from Algolia to [Typesense / MeiliSearch / other].
1. List all files that reference Algolia (imports, env vars, config)
2. Check if [new provider] has an InstantSearch adapter (keeps UI the same)
3. Document in STACK-CHANGES.md
4. Update backend-specialist.md to replace the algolia-sync subscriber pattern
5. Update CLAUDE.md tech stack table
6. Migrate component by component
```

---

### "I want to switch deployment platforms"

```
@devops-engineer We're moving from [Railway] to [Fly.io / Hetzner / Render].
1. What changes in the deployment pipeline?
2. Are there any Railway-specific configs to remove?
3. What environment variables change format?
Document in STACK-CHANGES.md, then update .github/workflows/deploy.yml.
```

---

### "I want to change the payment provider"

```
@orchestrator We're replacing Razorpay with [Stripe / CCAvenue / PayU].
This affects: frontend PaymentStep, webhook handler, and backend plugin config.
1. Impact assessment first — list all files
2. Document in STACK-CHANGES.md (this is a significant ADR change)
3. Update DECISIONS.md with a new ADR
4. Migrate in this order: plugin config → webhook handler → frontend PaymentStep
```

---

*Owner: [Your Name] | Last updated: [date] | Place at: shreefurniture-pro/STACK-CHANGES.md*