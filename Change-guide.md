## How to Make Any Change and Keep AI Agents in Sync

> **The problem this solves:** You build something, don't like it, ask the AI to change it.
> Next session — same mistake. This guide prevents that from ever happening.

---

## The Golden Rule of AI-Assisted Development

> **Every preference you express verbally is forgotten the next session.
> Every preference you write into a file is remembered forever.**

When you reject something or change direction:
1. Express it verbally to fix it *now*
2. Write it into a file so it's fixed *forever*

The files that hold your preferences:
```
PREFERENCES.md        → Your personal design taste (visual, UX, copy)
REJECTIONS.md         → Things you've seen and explicitly don't want
STACK-CHANGES.md      → Tech/library changes made mid-project
DECISIONS.md          → Original architectural decisions (don't edit — add new ADRs)
CLAUDE.md             → Hard technical rules (update when stack changes)
STATUS.md             → Current build state (update after every session)
```

---

## Part 1 — "I Don't Like How This Looks"

### Scenario: You see a component and hate the visual style

**The simple version (fixes it now + remembers it):**
```
I don't like [component name].
[Specific thing you don't like]: [describe it concisely]
[What you want instead]: [describe it, or link to a reference]

1. Add this to REJECTIONS.md so it never appears again
2. Rebuild [component] with the new direction
3. Update STATUS.md — mark [component] as 🔄 REVISED
```

**Examples of good rejection prompts:**

```
I don't like the CartDrawer.
The slide-in animation from the right feels standard/boring.
I want: bottom sheet on ALL screen sizes (not just mobile), 
sliding up from the bottom with a drag handle at top.

1. Add to REJECTIONS.md: "CartDrawer — no right slide-in, always bottom sheet"
2. Rebuild CartDrawer with bottom-sheet pattern on all viewports
3. Update STATUS.md
```

```
I don't like the ProductCard hover effect.
The image zoom (scale-110) feels cheap and stock-photo-ish.
I want: card lifts (translateY -4px), shadow increases, NO image zoom.

1. Add to REJECTIONS.md: "ProductCard — no image zoom on hover"
2. Rebuild ProductCard hover animation
3. Update STATUS.md
```

```
I don't like the primary button style.
Solid black feels too heavy next to warm product photography.
I want: Brand Accent (#C8A96E) fill, dark text, rounded-full (pill shape).

1. Add to PREFERENCES.md under Buttons
2. Add to REJECTIONS.md: "Buttons — no solid black fill as primary"
3. Update the Button component in components/ui/button.tsx
4. Update STATUS.md
```

---

### Scenario: You want to change the colour palette

```
I want to update the colour palette.
Current accent colour #C8A96E feels too muted — I want something warmer: #D4A853.
Also, I want the background to be slightly cooler: #F7F3EE instead of #F5F0E8.

1. Update packages/ui/src/tokens.ts with new values
2. Update PREFERENCES.md section 2 with confirmed colours
3. Run a search to find any hardcoded hex values that need updating
4. Update STATUS.md — visual review needed on: Homepage, PLP, CartDrawer
```

---

### Scenario: You want to change typography

```
I want to swap Playfair Display for DM Serif Display — it's more modern but still premium.
Keep Inter for body text.

1. Download DM Serif Display and add to public/fonts/
2. Update apps/storefront/app/layout.tsx — change next/font config
3. Update packages/ui/src/tokens.ts — font family tokens
4. Update PREFERENCES.md section 3 — note DM Serif Display as confirmed choice
5. Update STATUS.md — visual review needed on all pages with headings
```

---

### Scenario: You want a completely different layout for a page

```
I don't like the PLP layout.
The 4-column grid feels too dense on desktop — products feel cheap.
I want: 3 columns max on desktop, larger product images (aspect-ratio: 4/5), 
more whitespace between cards.

1. Add to REJECTIONS.md: "PLP — no 4-column grid, max 3 columns"
2. Add to PREFERENCES.md: "Product grid: 2col mobile / 3col desktop, aspect-ratio 4/5"
3. Rebuild PLP page with new grid
4. Also update ProductCard to support the taller 4/5 aspect ratio
5. Update STATUS.md
```

---

## Part 2 — "I Want to Change a Specific Component's UX"

### Scenario: You don't like the checkout flow UX

```
I want to change the checkout flow.
Currently it's 3 separate pages. I want a single page with collapsible accordion steps
(all on one URL: /checkout). Less navigation, more momentum.

IMPORTANT: Before changing anything, check:
1. Does this conflict with any acceptance criteria in NewDocs/01-product-requirements.md?
2. Does this conflict with any ADR in DECISIONS.md?
If conflicts exist, tell me before proceeding.

If no conflicts:
1. Add to REJECTIONS.md: "Checkout — no multi-page flow, use single-page accordion"
2. Add to PREFERENCES.md under UX Flows
3. Update NewDocs/01-product-requirements.md section 4.5 to reflect new flow
4. Rebuild the checkout as a single-page accordion
5. Update E2E test: e2e/purchase-flow.spec.ts (the navigation steps will change)
6. Update STATUS.md — checkout needs complete rebuild
```

---

### Scenario: You want to change search behaviour

```
I want to change how search works.
Currently the search overlay closes when you click a result. 
I want: clicking a result navigates to the product AND keeps the overlay open
so the user can browse more results.

1. Add to PREFERENCES.md: "Search — overlay stays open after clicking a result"
2. Update SearchOverlay.tsx — change the result click handler
3. Update REJECTIONS.md: "Search — no auto-close on result click"
4. Update STATUS.md
```

---

## Part 3 — "I Want to Change the Tech Stack"

> This is the most impactful type of change. Never rush it.
> Always do: Assessment → Documentation → Migration (in that order).

### Step 1 — Impact Assessment (ALWAYS FIRST)

Before changing any library, understand what you're touching:
```
@orchestrator Impact assessment for replacing [OLD] with [NEW].
I am NOT making this change yet — I want to understand the full scope first.

Please:
1. Search the entire codebase for every import/reference to [OLD]
2. List the files that would need rewriting
3. Check if DECISIONS.md has an ADR for this technology (which ADR number?)
4. Check if any E2E tests would break
5. Estimate: is this a 2-hour change or a 2-day change?
6. Are [OLD] and [NEW] compatible during a gradual migration, or must it be all-at-once?

Output: a simple impact report. No code changes.
```

---

### Step 2 — Documentation Update (BEFORE any code changes)

```
We're replacing [OLD] with [NEW].
Reason: [honest reason]

Please update the documentation before touching any code:
1. STACK-CHANGES.md — add new entry using the template in that file
2. CLAUDE.md — update the tech stack table row for [category]
3. DECISIONS.md — add a new ADR (ADR-0XX) that says:
   "Status: Accepted | Supersedes: ADR-0XX | 
    Decision: Use [NEW] instead of [OLD] because [reason]"
4. .agent/agents/[relevant agent].md — update the technology references
5. PROMPT-GUIDE.md — find any prompts that reference [OLD] and update them
6. STATUS.md — mark affected components as 🔄 NEEDS MIGRATION
7. .env.example — update env variable names if they change

Do NOT touch package.json or any component files yet.
Show me the documentation changes first.
```

---

### Step 3 — Migrate (After documentation is confirmed)

```
Documentation looks good. Now migrate component by component.
Start with the simplest component that uses [OLD] so we can test the pattern.

@[agent] Migrate [simplest component] from [OLD] to [NEW].
Ref: STACK-CHANGES.md entry for [date] — use the migration notes there.
Test that [component] still works after migration.
Do NOT migrate other components yet — confirm this one works first.
```

---

### Common Stack Changes — Specific Prompts

#### Swap shadcn/ui → Custom Components

```
@orchestrator We're removing shadcn/ui and building custom components.
Reason: [your reason]

Impact assessment:
- Which components are shadcn wrappers vs. custom already?
- Which shadcn components are hardest to replace (Dialog? Sheet? Select?)

Migration plan:
1. Keep Radix Primitives (they handle accessibility — don't reinvent this)
2. Replace shadcn's CSS variables with our design tokens in packages/ui/src/tokens.ts
3. Migrate simplest components first: Button → Input → Badge → Card
4. Migrate complex components last: Dialog → Sheet → Select → DatePicker

After assessment, document in STACK-CHANGES.md, then migrate Button first.
```

#### Swap Algolia → Typesense

```
@orchestrator We're replacing Algolia with self-hosted Typesense.
Reason: Cost. Typesense will run on our Hetzner VPS (Phase 2).

1. Impact assessment (don't change anything yet)
2. Key question: does typesense-instantsearch-adapter work with our current
   SearchOverlay.tsx implementation? (It should — same InstantSearch API)
3. Document in STACK-CHANGES.md
4. Migration order:
   a. Update backend algolia-sync subscriber → typesense-sync subscriber
   b. Update .env.example with new TYPESENSE_ vars
   c. Update lib/algolia/client.ts → lib/typesense/client.ts
   d. Update SearchOverlay.tsx to use new client
   e. Remove Algolia index setup from infrastructure docs
```

#### Swap Railway → Fly.io (backend)

```
@devops-engineer We're moving the MedusaJS backend from Railway to Fly.io.
Reason: [your reason]

1. What changes in deployment:
   - .github/workflows/deploy.yml
   - fly.toml (new file needed)
   - Health check path (same: /health)
   - Environment variable management (Fly secrets vs Railway vars)
2. What stays the same:
   - The backend code itself (no code changes)
   - DATABASE_URL and REDIS_URL (just update values)
   - The build command
3. Document in STACK-CHANGES.md
4. Create fly.toml and update deployment workflow
5. Do NOT change Railway setup until Fly deployment is confirmed working
```

#### Add a new library (not replacing anything)

```
We need to add [library name] to the project.
Purpose: [what it does]
Installs in: [apps/storefront / backend / packages/ui]

1. Is this library already provided by something we have? (check before adding)
2. Check the bundle impact: what's the gzip size?
3. Is there a lighter alternative?
4. If proceeding: 
   - Install with pnpm add [library] --filter=[workspace]
   - Add to CLAUDE.md tech stack table if it's significant
   - No need for STACK-CHANGES.md (that's for replacements)
```

---

## Part 4 — "I Want to Change Something Across the Whole Site"

### Global colour change

```
I want to change the Brand Accent from #C8A96E to #B8860B (darker gold).
This is a global change — it touches every component that uses Brand Accent.

1. Update packages/ui/src/tokens.ts — change the Brand Accent value
2. Update PREFERENCES.md section 2 with the new confirmed value
3. Search for any hardcoded #C8A96E hex values in the codebase:
   grep -r "C8A96E" apps/storefront/
4. If CSS custom property is used correctly (var(--brand-accent)), 
   then changing tokens.ts should cascade automatically — verify this
5. Update STATUS.md: "Global colour change — visual review needed on all pages"
```

### Global font change

```
I want to swap Playfair Display for [new font] globally.
1. Download [new font] files, add to public/fonts/
2. Update apps/storefront/app/layout.tsx — next/font config
3. Update packages/ui/src/tokens.ts — fontFamily.heading
4. Verify the font loads at the same weights (400, 700) we use
5. Update PREFERENCES.md section 3
6. Run Lighthouse — check for CLS regression from font swap
```

### Change the design system scale (spacing/sizes)

```
Our spacing feels too tight. I want to increase the base spacing scale by 20%.
Specifically: component padding should feel more generous.

1. Review current spacing values in packages/ui/src/tokens.ts
2. Propose specific changes to: card padding, section spacing, button padding
3. Show me the changes before applying
4. After I approve: apply changes + update STATUS.md "spacing system updated — review all components"
```

---

## Part 5 — Keeping Everything in Sync

### The "sync check" prompt — run when in doubt

```
@orchestrator Something changed recently and I want to make sure all files are in sync.
Changed: [what changed]

Please verify:
1. Is CLAUDE.md up to date? (tech stack table, hard rules)
2. Is STATUS.md accurate? (does it reflect the current build state?)
3. Are there any conflicts between REJECTIONS.md and what's currently in the codebase?
4. Are there any references to OLD libraries/patterns from STACK-CHANGES.md still in the code?
5. Does PREFERENCES.md capture the current design direction?

Report any conflicts. Don't make changes — just identify what needs updating.
```

### The "what changed since last week" prompt

```
Read STATUS.md, REJECTIONS.md, and STACK-CHANGES.md.
Summarise:
- What was built and confirmed working?
- What was rejected and rebuilt?
- Any stack changes made?
- What's the current state of PREFERENCES.md?
I want a 30-second briefing on project state before continuing.
```

---

## Part 6 — When to Update Which File

| Situation | Update these files |
|---|---|
| You don't like a visual detail | PREFERENCES.md + REJECTIONS.md |
| You reject a whole component | REJECTIONS.md + STATUS.md |
| You change a UX flow | PREFERENCES.md + REJECTIONS.md + NewDocs/01 spec + STATUS.md |
| You swap a UI library | STACK-CHANGES.md + CLAUDE.md + agent file + DECISIONS.md + STATUS.md |
| You swap a backend service | STACK-CHANGES.md + CLAUDE.md + agent file + DECISIONS.md + .env.example |
| You change deployment platform | STACK-CHANGES.md + CLAUDE.md + devops-engineer.md + deploy workflow |
| You add a new library | CLAUDE.md (if significant) |
| You change design tokens | PREFERENCES.md + packages/ui/src/tokens.ts + STATUS.md |
| You change copy/microcopy | PREFERENCES.md section 7 + REJECTIONS.md section 4 |

---

## Part 7 — File Placement

```
shreefurniture-pro/               ← monorepo root
├── CLAUDE.md                     ← Auto-loaded by AI — hard rules + tech stack
├── STATUS.md                     ← Current build state (update every session)
├── PREFERENCES.md                ← Your design taste layer — fill in as you build
├── REJECTIONS.md                 ← Never-again log — fill in as you reject things
├── STACK-CHANGES.md              ← Tech change log — fill in if you swap a library
├── CHANGE-GUIDE.md               ← This file — how to make any change
├── PROMPT-GUIDE-v2.md            ← Copy-paste prompts for every build task
├── PLACEMENT-GUIDE.md            ← Where every file lives (reference)
├── .env.example                  ← All required environment variables
└── NewDocs/                      ← All architecture + product specs live here
    ├── README.md                 ← Loading order + conflict resolution policy
    ├── CLAUDE.md                 ← (same as root CLAUDE.md — kept in sync)
    ├── DECISIONS.md              ← Architecture Decision Records (ADRs)
    ├── CONVENTIONS.md            ← Naming, code style, import order
    ├── KNOWN-ISSUES.md           ← Gotchas log
    ├── MEDUSA-V2-PATTERNS.md     ← MedusaJS v2 patterns reference
    ├── 01-product-requirements.md
    ├── 02–12 ...
    └── 13-design-system.md       ← Component specs, tokens, interaction states
```

**Note:** `DECISIONS.md` and `CONVENTIONS.md` live in `NewDocs/` — NOT at the root.
The three change-management files (`PREFERENCES.md`, `REJECTIONS.md`, `STACK-CHANGES.md`)
live at the root so AI agents auto-load them alongside `CLAUDE.md`.

---

## Part 8 — Update Your Session Start Prompt

Your current session start prompt:
```
Read STATUS.md and CLAUDE.md first.
```

**Your session start prompt (copy this into Antigravity/Cursor/Claude Code):**
```
Read these files before doing anything:
1. STATUS.md — current build state
2. PREFERENCES.md — my design preferences (these override agent defaults)
3. REJECTIONS.md — patterns I've explicitly rejected (never suggest these)
4. STACK-CHANGES.md — if it has entries, those supersede CLAUDE.md tech stack
5. CLAUDE.md — hard rules

Tell me: current phase, last completed item, any active rejections or stack changes I should know about.
Do not write any code yet.
```

When you're ready to build, open `PROMPT-GUIDE-v2.md` and copy-paste the next prompt.

This one change to your session start makes EVERY session aware of your preferences from minute one.

---

*Owner: [Your Name] | Last updated: [date] | Place at: shreefurniture-pro/CHANGE-GUIDE.md*


================================================