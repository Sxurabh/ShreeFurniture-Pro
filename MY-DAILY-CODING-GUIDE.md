# MY DAILY CODING GUIDE — Shree Furniture
## Personal Step-by-Step | Open This Before Opening Your IDE
### v2.0 — Fully Aligned with PROMPT-GUIDE-v2.md

---

> **How this guide works:**
> Every situation you will ever face has its own section below.
> Each section tells you exactly which prompts to use from **PROMPT-GUIDE-v2.md** — by section name.
> You never write prompts from scratch. You always copy from PROMPT-GUIDE-v2.md.
>
> **Your daily loop in one line:**
> Open this guide → pick your situation → go to PROMPT-GUIDE-v2.md section shown → copy prompt → paste into Antigravity → AI generates → YOU test → commit → push → PR.

---

## PROMPT-GUIDE-v2.md — Section Map (Memorise This)

| Section | What it covers | When you use it |
|---|---|---|
| **Section 1** | Session Start prompts | Every single session — always first |
| **Section 2a** | Phase 1 — Monorepo & Dev Environment | Week 1 scaffolding |
| **Section 2b** | Phase 1 — Backend Live | Week 2 Medusa setup |
| **Section 2c** | Phase 1 — Third-Party Services | Algolia, Cloudinary, Resend, PostHog, Sentry |
| **Section 3a** | Phase 2 — Lib, Hooks & API Client | Medusa client, Zustand stores, hooks |
| **Section 3b** | Phase 2 — Shared Components | Header, Footer, ProductCard, MobileMenu, Breadcrumb |
| **Section 3c** | Phase 2 — Homepage | Homepage page |
| **Section 3d** | Phase 2 — Product Pages | PLP + PDP |
| **Section 3e** | Phase 2 — Search & Cart | Search feature + Cart page |
| **Section 4a** | Phase 3 — Cart UI | CartDrawer, CartItem, CartSummary |
| **Section 4b** | Phase 3 — Checkout | Address, Shipping, Payment steps |
| **Section 4c** | Phase 3 — Auth & Account | Login, Register, Orders, Wishlist, Addresses |
| **Section 5** | Phase 3 — Backend | Subscribers, Workflows, Wishlist Module, ISR endpoint |
| **Section 6a** | Phase 4 — SEO | Sitemap, robots.txt, JSON-LD |
| **Section 6b** | Phase 4 — Analytics & Monitoring | PostHog verification, Sentry verification |
| **Section 6c** | Phase 4 — Security | npm audit, secret scan, rate limiting, cookie check |
| **Section 6d** | Phase 4 — Performance | Lighthouse audit, bundle analyzer |
| **Section 6e** | Phase 4 — Go-Live | DNS, Vercel prod env, Railway prod env, Razorpay webhook, pre-launch checklist |
| **Section 7** | Testing | All Vitest unit tests + Playwright E2E |
| **Section 8** | Debugging | Cart bugs, subscriber not firing, ISR issues, Razorpay issues, TypeScript errors |
| **Section 9** | DevOps & CI/CD | Deploy to Vercel, deploy to Railway, CI pipeline failures, new env vars |
| **Section 10** | Session End prompts | Every single session — always last |
| **Section 11** | Anti-Patterns & Quick Reference | Agent names, what not to say |

---

## TABLE OF CONTENTS

- [Part 1 — GitHub: Branch Strategy & Setup](#part-1--github-branch-strategy--setup)
- [Part 2 — Situation A: Normal Coding Session](#part-2--situation-a-normal-coding-session)
- [Part 3 — Situation B: I Don't Like What Was Built](#part-3--situation-b-i-dont-like-what-was-built)
- [Part 4 — Situation C: I Want to Save a Design Preference](#part-4--situation-c-i-want-to-save-a-design-preference)
- [Part 5 — Situation D: Mid-Project Stack Change](#part-5--situation-d-mid-project-stack-change)
- [Part 6 — Situation E: Something Is Broken](#part-6--situation-e-something-is-broken)
- [Part 7 — Situation F: Resuming After Days Away](#part-7--situation-f-resuming-after-days-away)
- [Part 8 — Pushing Code, PRs & GitHub Management](#part-8--pushing-code-prs--github-management)
- [Part 9 — End of Session (Every Day)](#part-9--end-of-session-every-day)
- [Part 10 — Quick Cheat Sheet](#part-10--quick-cheat-sheet)

---

# PART 1 — GITHUB: BRANCH STRATEGY & SETUP

## Your Complete Branch Structure

```
main                       ← Production only. CI must be green. Never push directly.
  └── develop              ← Always deployable. Integration branch. Never push directly.
        ├── phase/1-foundation        ← All Phase 1 work merges here
        ├── phase/2-storefront        ← All Phase 2 work merges here
        ├── phase/3-transactions      ← All Phase 3 work merges here
        ├── phase/4-launch-prep       ← All Phase 4 work merges here
        └── feat/[name]               ← One feature or one prompt group = one branch
            fix/[name]                ← Bug fixes
            chore/[name]              ← Config, tooling, stack changes
```

## The Rule That Stops Pages Breaking Each Other

**One branch = one logical unit of work.**

If you build Header, ProductCard, and Cart all in one branch — and Cart has a bug — you can't merge just Header. You're blocked. Split them.

**One prompt or one group of tightly related prompts = one feature branch.**

| What you're building | Branch name |
|---|---|
| Monorepo scaffold | `feat/monorepo-scaffold` |
| Docker + CI | `feat/docker-ci-setup` |
| Root layout + providers | `feat/root-layout-providers` |
| Header + Footer + Nav (one unit) | `feat/layout-header-footer-nav` |
| ProductCard only | `feat/product-card` |
| PLP page | `feat/product-listing-page` |
| PDP page | `feat/product-detail-page` |
| All search components | `feat/search-feature` |
| Cart store + CartDrawer + CartItem + CartSummary | `feat/cart-ui` |
| Full checkout flow | `feat/checkout-flow` |
| Login + Register + Auth guard | `feat/auth-pages` |
| All account pages | `feat/account-pages` |
| All backend subscribers | `feat/backend-subscribers` |
| Algolia → Typesense swap | `chore/swap-algolia-to-typesense` |
| Bug: subscriber not firing | `fix/order-subscriber-not-firing` |

## One-Time Setup Commands

```bash
# Clone the repo
git clone https://github.com/Sxurabh/ShreeFurniture-Pro.git
cd ShreeFurniture-Pro

# Create develop
git checkout -b develop
git push -u origin develop

# Create Phase 1 branch (do this when you start Phase 1)
git checkout -b phase/1-foundation develop
git push -u origin phase/1-foundation
```

**Branch protection rules to set in GitHub once:**
- Go to Settings → Branches → Add protection rule
- Apply to `main`: require PR, require CI to pass, no direct push
- Apply to `develop`: require PR, require CI to pass

## How Code Flows (Never Skip a Step)

```
feat/your-work
  ↓  (PR when feature is done, CI green)
phase/X-current-phase
  ↓  (PR when ALL features in the phase are done, full flow tested manually)
develop
  ↓  (PR when you're ready to deploy to production)
main
```

---

# PART 2 — SITUATION A: NORMAL CODING SESSION

**Use this every day when you're building something.**

---

## STEP 1 — Pull Latest Code

```bash
git checkout phase/1-foundation    # replace with your current phase branch
git pull origin phase/1-foundation
```

---

## STEP 2 — Create Your Feature Branch

```bash
git checkout -b feat/[what-youre-building-today]
```

Example: `git checkout -b feat/layout-header-footer-nav`

---

## STEP 3 — Open PROMPT-GUIDE-v2.md → Section 1 → Copy Session Start Prompt

Open `PROMPT-GUIDE-v2.md`. Go to **Section 1 — Session Start**.

Copy the **"Standard Session Start"** prompt (the one that reads 5 files):

```
Read these files before doing anything:
1. STATUS.md — current build state and what's already been built
2. PREFERENCES.md — my design preferences (override your UI defaults for all components)
3. REJECTIONS.md — patterns I've explicitly rejected (never suggest these again)
4. STACK-CHANGES.md — if it has entries, they supersede the CLAUDE.md tech stack table
5. CLAUDE.md — hard technical rules

Tell me:
- Current phase and what was last completed
- Any active rejections or preference overrides I should know about
- Next 2–3 tasks from NewDocs/10-development-phases.md
Do not write any code yet.
```

Paste this into Antigravity. **Wait for the summary.** Read it. Confirm it matches what you expect.

> ⚠️ **IMPORTANT:** The session start prompt in this guide has the files in the **correct order**. PROMPT-GUIDE-v2.md Section 1 has a known error — it reads CLAUDE.md at step 2 before STACK-CHANGES.md. Use the order above, not the one printed in Section 1 of PROMPT-GUIDE-v2.

---

## STEP 4 — Open PROMPT-GUIDE-v2.md → Find Your Build Prompt

Use the section map at the top of this guide to find your current situation:

**Example:** You're in Phase 1 Week 2, ready to configure MedusaJS.
→ Go to **PROMPT-GUIDE-v2.md Section 2b — Backend Live**
→ Find **"Configure MedusaJS v2 Backend"**
→ Copy that entire prompt block

**Paste it into Antigravity.**

The AI will:
- Generate code at the exact file paths specified in the prompt
- Follow all CLAUDE.md rules (paise, HTTP-only cookies, TypeScript strict)
- Follow PREFERENCES.md and REJECTIONS.md overrides
- Update STATUS.md when done (because every prompt ends with "Update STATUS.md when done")

---

## STEP 5 — AI GENERATES — While It's Running

While Antigravity generates code, do NOT paste another prompt. One at a time.

---

## STEP 6 — YOU TEST (This Is Your Job, Not the AI's)

Every prompt in PROMPT-GUIDE-v2.md ends with an **Acceptance** line or a **Verify** line. That is your test checklist.

**After the AI finishes, run these in order:**

### 6a. Auto checks (paste this into Antigravity):
```
Before I review, run:
1. pnpm type-check — paste the output
2. pnpm lint — paste the output
If either fails, fix it now before I review anything.
```

### 6b. Manual checks — you do these yourself:

For **every** prompt, check:
- [ ] The file exists at the path the agent said it created
- [ ] Open the file — does it look correct?

For **frontend** prompts, also check:
- [ ] Open `localhost:3000` (or wherever the dev server runs)
- [ ] Does it render without a white screen?
- [ ] Does it match the design tokens (warm gold, near-black, Playfair Display)?
- [ ] Does it work on mobile screen size? (drag the browser to 375px width)

For **backend** prompts, also check:
- [ ] Run the smoke test commands shown in the prompt (e.g., `GET /store/products → should return products`)

For prompts with a specific **Acceptance** line — test that exact thing:
```
Example from "Scaffold Monorepo" prompt:
Acceptance: pnpm dev starts all apps, pnpm build completes without errors.
→ You run: pnpm dev  and  pnpm build  and check both pass.
```

---

## STEP 7 — Do You Like It?

- **Yes** → go to Step 8 (commit)
- **No** → go to **Situation B** (Part 3 of this guide), then come back to Step 8

---

## STEP 8 — Commit What Was Built

```bash
# Stage everything
git add .

# Commit with a clear message
git commit -m "feat: add Header, Footer, and Navigation components"
```

**Commit message format:**
```
feat: [what was added]
fix: [what was fixed]
chore: [config, tooling, deps, stack changes]
test: [adding tests]
docs: [docs only — STATUS.md, PREFERENCES.md, etc.]
```

Commit after **every working, tested chunk** — not only at the end of the day.
If the AI built 3 files and they all work: commit. If they don't work: fix first, then commit.

---

## STEP 9 — Repeat Steps 4–8 for the Next Prompt

Go back to PROMPT-GUIDE-v2.md. Find the next prompt in the same section. Copy, paste, test, commit.

**How many prompts per session?**
Aim for 2–4 completed prompts per session. Each prompt is a meaningful chunk. Reviewing properly is more important than rushing.

---

## STEP 10 — When Today's Feature Is Complete → Push and Create PR

See **Part 8** of this guide for the full push + PR flow.

---

## STEP 11 — End of Session

See **Part 9** of this guide. Do this before closing your IDE.

---

# PART 3 — SITUATION B: I DON'T LIKE WHAT WAS BUILT

**Use this mid-session when the AI generates something and you want to change it.**

---

## Option B1 — You Know Exactly What You Want Instead (Quick Fix)

Stay in your current session. Paste this prompt into Antigravity:

```
I don't like what was built for [ComponentName].
Specifically: [describe what you don't like — be precise about the element]
What I want instead: [describe exactly what you want]

Do these two things:
1. Add to REJECTIONS.md — Section [1=UI / 2=UX Flow / 3=Visual / 4=Copy / 5=Technical]:
   ### [ComponentName] — [today's date]
   What was built: [short description]
   Why rejected: [your reason]
   What to build instead: [the correct direction]
   Status: 🚧 Pending rebuild

2. Rebuild [ComponentName] with the new direction.
   Update STATUS.md when done.
```

**Real example:**
```
I don't like what was built for ProductCard.
Specifically: The hover animation zooms the image using scale-110. It feels cheap.
What I want instead: A card lift only — shadow increases and the card moves up by 4px. No image zoom.

1. Add to REJECTIONS.md Section 1:
   ### ProductCard image zoom hover — [today's date]
   What was built: scale-110 on hover
   Why rejected: Looks cheap, like a stock photo site
   What to build instead: translateY(-4px) + deeper shadow. Never scale the image.
   Status: 🚧 Pending rebuild

2. Rebuild ProductCard with the card lift hover.
   Update STATUS.md when done.
```

Then go back to Step 6 and test the rebuild.

---

## Option B2 — Bigger Change (Flow, Page Behaviour, Layout)

Check for conflicts before changing anything:

```
I want to change [flow / page / behaviour].
Before making any changes, check:
1. Does this conflict with NewDocs/01-product-requirements.md acceptance criteria?
2. Does this conflict with any ADR in NewDocs/DECISIONS.md?
Show me the relevant sections. Do not touch any code yet.
```

Read the agent's response. If no conflict → tell it to proceed:
```
No conflict. Go ahead with the change.
Also add to REJECTIONS.md what was replaced and why.
Update STATUS.md when done.
```

---

## Option B3 — You Saw a Reference You Like (Inspiration)

```
@frontend-specialist I want [ComponentName] to feel more like [describe the reference or paste a URL].
I specifically like [aspect of the reference] but want to keep [aspect from the current version].
Before rebuilding:
1. Add my preference to PREFERENCES.md under Section [2=Colours / 3=Typography / 4=Components / 5=Animations]
2. Rebuild [ComponentName] with the new direction.
3. Update STATUS.md.
```

---

## After Any Rejection → Commit Correctly

```bash
git add .
git commit -m "fix: rebuild [ComponentName] — [brief reason for change]"
```

Example: `git commit -m "fix: rebuild ProductCard hover — card lift instead of image zoom"`

---

# PART 4 — SITUATION C: I WANT TO SAVE A DESIGN PREFERENCE

**Use this any time you see something and think "I want future components to do this too."**

Paste into Antigravity:

```
Add to PREFERENCES.md under Section [2/3/4/5]:

Element: [component name or UI element]
What I saw: [what was generated]
What I liked and want to keep: [description]
What I want different: [description — or "nothing, keep exactly like this"]
Date: [today]
```

**Example:**
```
Add to PREFERENCES.md under Section 4 (Components):

Element: Primary CTA button
What I saw: Solid black (#1A1A1A) fill with white text
What I liked: Clean and authoritative
What I want different: Use Brand Accent (#C8A96E) fill with dark text.
                        On hover: flip to solid black fill with white text.
Date: March 2026
```

Then commit the doc update:

```bash
git add PREFERENCES.md
git commit -m "docs: add CTA button colour preference to PREFERENCES.md"
```

---

## Applying a Preference to Already-Built Components

```
I've added a new preference to PREFERENCES.md: [describe it briefly].
List every component already marked ✅ DONE in STATUS.md that might need updating.
Do NOT change anything yet. Just list them so I can decide.
```

---

# PART 5 — SITUATION D: MID-PROJECT STACK CHANGE

**Use this when you want to swap a library, service, or tool.**

This is a 5-step process. Never skip a step. Stack changes on their own branch.

---

## Step D1 — Create a Stack Change Branch

```bash
git checkout develop
git pull origin develop
git checkout -b chore/swap-[old-tech]-to-[new-tech]
```

Example: `git checkout -b chore/swap-algolia-to-typesense`

---

## Step D2 — Impact Assessment (Never Change Blind)

Paste into Antigravity:

```
@explorer-agent We are considering replacing [OLD TECH] with [NEW TECH].
Before making any changes:
1. List every file that imports from or references [OLD TECH]
2. List every env variable related to [OLD TECH] in .env.example
3. List every NewDocs document that mentions [OLD TECH]
4. Does [NEW TECH] have a compatibility adapter that reduces migration effort?
Do NOT change anything. Just report.
```

Read the report. Understand the scope. Then decide.

---

## Step D3 — Document the Change in STACK-CHANGES.md First

```
Before we change any code, add this entry to STACK-CHANGES.md:

## CHANGE: [OLD TECH] → [NEW TECH]
Date: [today]
Reason: [why — cost / deprecation / performance / preference]

### Before
Technology: [OLD TECH]
Env vars: [list from .env.example]
Key files: [list from impact assessment]

### After
Technology: [NEW TECH]
Env vars: [new vars needed]
Key files: [same files, updated]

### Migration Notes
[anything special — e.g. "Typesense has an InstantSearch adapter so the UI layer doesn't change"]

### Files to Update
- [ ] CLAUDE.md — update tech stack table
- [ ] .env.example — remove old vars, add new vars
- [ ] .agent/rules/SHREE-FURNITURE.md — if it mentions old tech
- [ ] NewDocs/MEDUSA-V2-PATTERNS.md — if backend change
- [ ] [every file from the impact assessment]
```

Commit the documentation before touching source code:

```bash
git add STACK-CHANGES.md
git commit -m "docs: document [old] → [new] stack change before migration"
```

---

## Step D4 — Migrate One Component at a Time

```
@orchestrator Begin migration from [OLD TECH] to [NEW TECH].
STACK-CHANGES.md has the full change log.
Start with [SIMPLEST FILE ONLY].
After migrating it:
- Run pnpm type-check
- Run pnpm test
- Confirm it works
- Then show me the next file to migrate.
Do NOT migrate everything at once.
```

After each file migrates and is verified:

```bash
git add [migrated-file]
git commit -m "chore: migrate [filename] from [old tech] to [new tech]"
```

---

## Step D5 — Update All Reference Files

```
The [OLD TECH] → [NEW TECH] migration is complete.
Now update all reference files — no source code changes, documentation only:
1. CLAUDE.md — update the tech stack table
2. .env.example — remove [old vars], add [new vars] with comments
3. STACK-CHANGES.md — tick off every checkbox in the "Files to Update" list
4. .agent/rules/SHREE-FURNITURE.md — update any mentions
5. STATUS.md — update affected components
```

Final commit and push:

```bash
git add .
git commit -m "chore: complete [old] → [new] migration, update all reference files"
git push origin chore/swap-[old]-to-[new]
```

Create PR: `chore/swap-[old]-to-[new]` → `develop` (stack changes go directly to develop, not a phase branch)

---

## Common Stack Change Scenarios

### Swap UI Library (e.g., shadcn/ui → Radix or Headless UI)

Impact assessment prompt:
```
@explorer-agent List every file that imports from shadcn/ui (@/components/ui/*).
Also list every CSS variable coming from shadcn's theme system.
Do not change anything.
```

### Swap Search Provider (Algolia → Typesense or MeiliSearch)

Key files that change:
- `backend/medusa-config.ts` — plugin config
- `backend/src/subscribers/algolia-sync.ts` — rename and rewrite the subscriber
- `apps/storefront/lib/algolia/client.ts` — new client
- `.env.example` — old Algolia vars out, new vars in

Tip: Typesense has an InstantSearch adapter — the search UI components don't need to change at all.

### Swap Payment Provider (Razorpay → Stripe or PayU)

Key files that change:
- `backend/medusa-config.ts` — swap plugin
- `apps/storefront/app/api/webhooks/razorpay/route.ts` — rewrite HMAC logic
- `apps/storefront/components/checkout/PaymentStep.tsx` — new SDK

This is a significant ADR-level change. Also run:
```
Add to NewDocs/DECISIONS.md:
ADR-[N]: Payment Provider Change — [old] → [new]
Date: [today]
Status: Accepted
Decision: [one sentence]
Rationale: [why]
Consequences: PaymentStep, webhook handler, and backend plugin all change.
```

---

# PART 6 — SITUATION E: SOMETHING IS BROKEN

**Use this when something isn't working as expected.**

---

## Step E1 — Run These Yourself First

```bash
pnpm type-check     # most errors are TypeScript
pnpm lint           # ESLint
```

Check the browser console for client-side errors.
Check Railway logs for backend errors.
Check `NewDocs/KNOWN-ISSUES.md` — it may already have the fix.

---

## Step E2 — Open PROMPT-GUIDE-v2.md → Section 8 → Find Your Debug Scenario

Section 8 has pre-written debug prompts for:
- Cart issues (state, optimistic update, sync)
- MedusaJS subscriber not firing
- ISR cache not invalidating after product update
- Razorpay payment failing or creating duplicates
- TypeScript strict errors

Copy the matching prompt. Fill in the `[brackets]` with your specifics. Paste into Antigravity.

**Always paste the full error message, not a summary.** The agent needs the exact text.

---

## Step E3 — After the Fix

```bash
git add .
git commit -m "fix: [short description of what was broken and how it's fixed]"
```

Then add it to KNOWN-ISSUES.md so it never has to be debugged again:

```
Add to NewDocs/KNOWN-ISSUES.md:

### [SHORT DESCRIPTIVE TITLE]
**Symptom:** [what you or a user would see]
**Root Cause:** [why it happened technically]
**Fix / Workaround:** [exact steps]
**Affects:** [file paths]
**Date Discovered:** [today]
```

```bash
git add NewDocs/KNOWN-ISSUES.md
git commit -m "docs: add [issue title] to KNOWN-ISSUES.md"
```

---

# PART 7 — SITUATION F: RESUMING AFTER DAYS AWAY

**Use this when you haven't coded for 3 or more days.**

---

## Step F1 — Pull Latest

```bash
git checkout phase/X-your-current-phase
git pull origin phase/X-your-current-phase
```

Check GitHub → Pull Requests — is there anything waiting to be merged?

---

## Step F2 — Open PROMPT-GUIDE-v2.md → Section 1 → Copy "Resuming After a Gap" Prompt

```
Read these files before doing anything:
STATUS.md, PREFERENCES.md, REJECTIONS.md, STACK-CHANGES.md, CLAUDE.md, NewDocs/KNOWN-ISSUES.md

Summarise:
- Current phase and last completed item from STATUS.md
- Any preferences I've saved (briefly)
- Any active rejections I should know about
- Any stack changes that have been made
- Any known issues to watch out for
- What the next 2–3 tasks are from NewDocs/10-development-phases.md

Do not write any code. Just the situation report.
```

Read the report carefully before proceeding. Then continue with Situation A (Part 2) from Step 2.

---

# PART 8 — PUSHING CODE, PRS & GITHUB MANAGEMENT

## When to Push and Create a PR

Push and create a PR when:
- A logical feature is complete and tested (matches the Acceptance criteria in its prompt)
- All your commits for that feature are in the branch
- `pnpm type-check` and `pnpm lint` pass

You do **not** need to wait until the end of the day. Push when the feature is done.

---

## The Push and PR Flow

### Step 1 — Run Quality Checks

Paste this into Antigravity:
```
Before I push:
1. Run pnpm type-check — show me the output
2. Run pnpm lint — show me the output
3. Run pnpm test — show me the output
Fix any failures before I push.
```

### Step 2 — Push Your Branch

```bash
git push -u origin feat/[your-branch-name]
```

### Step 3 — Create the PR on GitHub

Go to GitHub → your repo. You'll see a yellow banner: **"Your recently pushed branches"** → click **"Compare & pull request"**.

Fill in:

**Title:** `feat: [what this adds]`
Examples:
- `feat: add Header, Footer, and Navigation components`
- `feat: build ProductCard with Cloudinary image and INR price`
- `fix: resolve subscriber not firing on first deploy`

**Base branch (where you're merging INTO):**
- Your feature branches → `phase/1-foundation` (or current phase)
- Your phase branch → `develop` (only when the whole phase is done)
- `develop` → `main` (only when you're ready to deploy to production)

> ⚠️ **NEVER** set base branch to `main` for feature branches. Always target the phase branch.

**Body — Use this template:**
```
## What This PR Does
[1-2 sentences]

## Files Changed
- `path/to/file.tsx` — [what changed]

## How I Tested This
- [ ] pnpm type-check passes
- [ ] pnpm lint passes
- [ ] pnpm test passes
- [ ] Manually tested in browser at localhost:3000
- [ ] Checked mobile screen size (375px)
- [ ] Verified acceptance criteria from the prompt

## STATUS.md updated?
- [ ] Yes

## Screenshots (if UI change)
[paste before/after or just the after]
```

### Step 4 — Wait for CI

GitHub Actions will run automatically on every PR. It runs:
1. Type check
2. ESLint
3. Vitest tests
4. Build

If any step fails → fix it locally, commit, push again. CI re-runs automatically. Do **not** merge until CI is green.

### Step 5 — Merge the PR

Once CI is green and you're satisfied:
- Click **"Squash and Merge"** — keeps the branch history clean
- After merging, click **"Delete branch"** — GitHub offers this button automatically

---

## Handling Merge Conflicts

When GitHub shows a merge conflict warning on your PR:

```bash
# On your local machine:
git checkout phase/2-storefront    # pull the latest target branch
git pull origin phase/2-storefront

git checkout feat/product-card     # go back to your feature branch
git merge phase/2-storefront       # this will show you the conflicts

# Fix the conflicts in the files listed (look for <<<<<<< and >>>>>>> markers)
git add .
git commit -m "merge: resolve conflicts with phase/2-storefront"
git push origin feat/product-card
```

If you're unsure how to resolve a conflict, paste it into Antigravity:
```
I have a merge conflict in [filename]. Here is the conflicted section:
<<<<<<< HEAD
[your code]
=======
[incoming code]
>>>>>>> phase/2-storefront
Which version is correct? Write me the resolved version.
```

---

## When to Merge a Phase Branch into Develop

Only merge `phase/X-name → develop` when:
- [ ] Every item for that phase is marked ✅ in STATUS.md
- [ ] You've manually walked through the full user flow (browse → cart → checkout)
- [ ] CI is green on the phase branch
- [ ] No open PRs from feature branches are still pending

---

## When to Merge Develop into Main (Deploy to Production)

Only merge `develop → main` when:
- [ ] All Phase items are complete through the current release
- [ ] Open PROMPT-GUIDE-v2.md → Section 6e → **"Pre-Launch Definition of Done Checklist"**
- [ ] Every item in that checklist is ✅
- [ ] Playwright E2E purchase flow passes (Section 7 → "Write Playwright E2E — Full Purchase Flow")
- [ ] Lighthouse mobile score ≥ 90

---

# PART 9 — END OF SESSION (EVERY DAY)

**Do this before closing Antigravity. Every single session.**

---

## Step 1 — Open PROMPT-GUIDE-v2.md → Section 10 → Copy Session End Prompt

Use **"Standard Session End"**:

```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark completed items ✅ with a Notes entry describing the implementation
   - Mark partial items 🚧 with exactly what's left to do
   - If anything was started but not in STATUS.md's table, flag it
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I expressed any design preference
6. Update REJECTIONS.md if I rejected anything

Show me a summary of all changes before committing.
```

Or use **"Session End With Note for Next Session"** if something is half-done:

```
End of session. Before updating docs, do a full audit:

1. List every file you created or modified this session (full paths)
2. For each file: is it complete and functional, or partial?
3. Now update STATUS.md:
   - Mark [component] as ✅ DONE with a Notes entry
   - Add note: "[what the next session needs to know about this implementation]"
   - Mark partial items 🚧 with exactly what's left to do
4. Update KNOWN-ISSUES.md if any unexpected behavior was found
5. Update PREFERENCES.md if I confirmed a design decision today
6. Update REJECTIONS.md if I rejected an approach today

Show me a summary of all changes before committing.
```

---

## Step 2 — Commit the Doc Updates

```bash
git add STATUS.md PREFERENCES.md REJECTIONS.md NewDocs/KNOWN-ISSUES.md
git commit -m "docs: end of session — STATUS, PREFERENCES, REJECTIONS, KNOWN-ISSUES"
git push
```

---

## Step 3 — Create PR If Your Feature Is Complete

If the feature you worked on today is finished and tested → go to Part 8 and create the PR now.
If it's not finished yet → leave the branch open and continue tomorrow.

---

## What a Complete Session Looks Like

```
09:00  Pull latest → create feat/branch
09:05  Paste session start prompt → read summary
09:10  Open PROMPT-GUIDE-v2.md → copy build prompt
09:30  AI generates → you run type-check + lint
09:35  You test in browser — works, looks right
09:40  Commit: "feat: add ProductCard component"
09:45  Copy next build prompt from PROMPT-GUIDE-v2.md
10:15  AI generates → you test → small rejection → rebuild → test again
10:25  Commit: "feat: add ProductGallery with mobile pinch zoom"
10:30  Paste session end prompt → AI updates STATUS, PREFERENCES, REJECTIONS
10:35  Commit: "docs: end of session updates"
10:36  Push → create PR on GitHub
10:38  Done for the day
```

---

# PART 10 — QUICK CHEAT SHEET

## Daily Sequence in Short

```
1. git pull → git checkout -b feat/name
2. PROMPT-GUIDE-v2 Section 1 → Session Start prompt
3. PROMPT-GUIDE-v2 [current section] → Build prompt
4. AI generates → pnpm type-check + lint → you test in browser
5. Like it? → commit  |  Don't like it? → rejection flow (Part 3) → commit
6. Repeat steps 3–5 for next prompt
7. PROMPT-GUIDE-v2 Section 10 → Session End prompt → commit docs → push → PR
```

## PROMPT-GUIDE-v2 Sections by Situation

| Situation | Section to open |
|---|---|
| Starting any session | **Section 1** |
| Building Phase 1 (monorepo, backend, services) | **Sections 2a / 2b / 2c** |
| Building Phase 2 (lib, components, pages) | **Sections 3a / 3b / 3c / 3d / 3e** |
| Building Phase 3 (cart, checkout, auth, subscribers) | **Sections 4a / 4b / 4c / 5** |
| Building Phase 4 (SEO, performance, go-live) | **Sections 6a–6e** |
| Writing tests | **Section 7** |
| Debugging something broken | **Section 8** |
| CI/CD or deployment | **Section 9** |
| Ending any session | **Section 10** |

## File Reading Order (Session Start — Never Change This)

```
1. STATUS.md
2. PREFERENCES.md
3. REJECTIONS.md
4. STACK-CHANGES.md
5. CLAUDE.md
```

## Branch → Where It Merges

```
feat/X    →  phase/X-name  (PR for each feature)
phase/X   →  develop        (PR when whole phase done)
develop   →  main           (PR when ready to deploy)
```

## Agent Quick Reference

| You need | Use |
|---|---|
| Next.js, components, UI, Zustand, TanStack Query | `@frontend-specialist` |
| MedusaJS backend, subscribers, workflows, modules | `@backend-specialist` |
| PostgreSQL schema, migrations, indexes | `@database-architect` |
| CI/CD, Railway, Vercel, Docker, GitHub Actions | `@devops-engineer` |
| Vitest unit tests, Playwright E2E | `@test-engineer` |
| Cart bugs, TypeScript errors, unexpected behavior | `@debugger` |
| LCP, CLS, bundle size, Core Web Vitals | `@performance-optimizer` |
| Webhook security, auth, OWASP | `@security-auditor` |
| Features spanning multiple files | `@orchestrator` |
| "What files exist for X?" | `@explorer-agent` |

## The 5 Non-Negotiable Rules

```
1. All prices = integers in paise.  ₹49,999 = 4999900.  formatPrice() at render only.
2. Auth tokens = HTTP-only cookies only.  Never localStorage.  Never Zustand.
3. No any type without // reason: [explanation] comment.
4. Never hardcode hex colors in JSX.  Always packages/ui/src/tokens.ts.
5. Every build prompt ends with "Update STATUS.md when done" — that's the AI's job.
```

## What NOT to Build (MVP — Scope Check)

If you're about to build any of these, **stop and re-read NewDocs/09-engineering-scope-definition.md:**

Reviews & Ratings | Loyalty/Referral | AR/3D viewer | GST invoice breakdown | SMS | Multi-currency | Bulk CSV import | Discount codes | MeiliSearch | VPS migration

---

*Shree Furniture | MY-DAILY-CODING-GUIDE.md | v2.0 — March 2026*
*Place at: `shreefurniture-pro/MY-DAILY-CODING-GUIDE.md` (monorepo root)*
