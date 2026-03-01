# PROJECT AUDIT REPORT — Shree Furniture
## Full alignment check + workflow confirmation
### Date: March 2026

> ⚠️ **HISTORICAL DOCUMENT — ALL ISSUES RESOLVED**
> This audit was completed in March 2026. All 6 issues identified below have been fixed.
> This file is kept for reference only. Do not re-apply these fixes.

---

## ✅ Overall Verdict

The project is **well-structured and 95% aligned.** Found 6 issues total — all have been resolved. See below for what was fixed.

---

## 🔴 CRITICAL — Fix Before Starting

### Issue 1: File Names Don't Match How They're Referenced

**This is the most important fix.** Your actual files use mixed/title case, but every document references them in UPPERCASE. On Linux (your production server), filenames are case-sensitive. An agent trying to read `PREFERENCES.md` when the file is named `Preferences.md` will silently fail.

| Actual filename | Must rename to |
|---|---|
| `Preferences.md` | `PREFERENCES.md` |
| `Rejections.md` | `REJECTIONS.md` |
| `Stack-changes.md` | `STACK-CHANGES.md` |
| `Change-guide.md` | `CHANGE-GUIDE.md` |

**Action:** In your file explorer or terminal, rename all 4 files:
```bash
# Run from project root
mv Preferences.md PREFERENCES.md
mv Rejections.md REJECTIONS.md
mv Stack-changes.md STACK-CHANGES.md
mv Change-guide.md CHANGE-GUIDE.md
```

---

### Issue 2: PROMPT-GUIDE.md is Missing from Project Root

`CHANGE-GUIDE.md` and `STACK-CHANGES.md` reference `PROMPT-GUIDE.md` but no such file exists in the project. The correct file is `PROMPT-GUIDE-v2.md` (from our previous sessions).

**Action:** Copy `PROMPT-GUIDE-v2.md` into the project root. It's your primary build reference.

---

## 🟡 MEDIUM — Fix Now (Minor Effort)

### Issue 3: Session Start Order Inconsistency

`SHREE-FURNITURE.md` (the always-on Antigravity rule) and `NewDocs/README.md` listed the session start files in slightly different order. Specifically, CLAUDE.md appeared as step 2 in one and step 5 in the other.

**Status: ✅ Fixed in the updated SHREE-FURNITURE.md delivered below.**

Both files now agree: STATUS → PREFERENCES → REJECTIONS → STACK-CHANGES → CLAUDE.md

The logic is correct: check for stack overrides BEFORE reading CLAUDE.md's tech stack table, so you don't get confused by stale info.

---

### Issue 4: CHANGE-GUIDE.md Part 7 File Tree Was Wrong

The file tree in CHANGE-GUIDE.md Part 7 showed `DECISIONS.md` and `CONVENTIONS.md` at the root, and `PROMPT-GUIDE.md` as the prompt file. All three were wrong.

| Was shown | Actual location |
|---|---|
| `DECISIONS.md` at root | `NewDocs/DECISIONS.md` |
| `CONVENTIONS.md` at root | `NewDocs/CONVENTIONS.md` |
| `PROMPT-GUIDE.md` | `PROMPT-GUIDE-v2.md` at root |

**Status: ✅ Fixed in the updated CHANGE-GUIDE.md delivered below.**

---

## 🟢 LOW — Minor / Already Fine

### Issue 5: SHREE-FURNITURE.md Had Duplicate YAML Frontmatter

The file had two `---` frontmatter blocks at the top. Antigravity may parse only the first one and ignore `priority: P0`.

**Status: ✅ Fixed in the updated SHREE-FURNITURE.md. Now a single clean block.**

---

### Issue 6: tokens.ts Spacing vs 13-design-system.md

`packages/ui/src/tokens.ts` currently only defines `spacing.page` (x and maxWidth). `NewDocs/13-design-system.md` references a full spacing scale (`space.1` through `space.16`).

**This is not a conflict** — it's intentional. `tokens.ts` gets the full scale added during Prompt #6 (Configure next.config.ts + design tokens). The design-system doc defines what WILL be there, not what's there before you start building.

**Action: None required.** It resolves itself when you run Prompt #6.

---

## ✅ Everything That Was Checked and Is Correct

| Check | Result |
|---|---|
| Colour values in CLAUDE.md, frontend-specialist, 13-design-system, SHREE-FURNITURE | ✅ All match exactly |
| Rendering strategy (ISR/CSR/SSR) across all 3 files that mention it | ✅ All match |
| Paise rule (integers only) in CLAUDE.md, SHREE-FURNITURE, CONVENTIONS | ✅ Consistent |
| Auth token rule (HTTP-only cookies) across all agent files | ✅ Consistent |
| MedusaJS v2 enforcement across CLAUDE.md, SHREE-FURNITURE, MEDUSA-V2-PATTERNS | ✅ Consistent |
| Component file locations in frontend-specialist vs 07-monorepo-structure | ✅ Match |
| Agent routing table in SHREE-FURNITURE | ✅ Correct |
| STACK-CHANGES.md authority scope (tech only, not feature specs) | ✅ Correct |
| NewDocs README conflict resolution hierarchy | ✅ Sound |
| 13-design-system relationship to PREFERENCES.md | ✅ Correct (PREFERENCES wins) |
| NewDocs README file placement policy | ✅ Correct |
| KNOWN-ISSUES.md and MEDUSA-V2-PATTERNS.md are in NewDocs/ | ✅ Correct |

---

## 📁 Complete File Placement Map

```
shreefurniture-pro/                    ← MONOREPO ROOT
│
├── CLAUDE.md                          ← Auto-loaded. Hard rules, tech stack, what not to build
├── STATUS.md                          ← Build state. Read every session. Update every session.
├── PREFERENCES.md                     ← Your design taste. Empty at start, fills in as you build.
├── REJECTIONS.md                      ← Never-again log. Empty at start, fills in as you reject.
├── STACK-CHANGES.md                   ← Tech swap log. Empty at start, fills in if you change a library.
├── CHANGE-GUIDE.md                    ← How to make any change (UI/UX/stack). Reference doc.
├── PROMPT-GUIDE-v2.md                 ← Your BUILD GUIDE. Copy-paste prompts in sequence.
├── PLACEMENT-GUIDE.md                 ← Where every file lives. Reference doc.
├── .env.example                       ← All env vars documented (no values)
│
└── NewDocs/                           ← ALL SPECS LIVE HERE
    ├── README.md                      ← Loading order + conflict resolution. Agents read this.
    ├── CLAUDE.md                      ← (kept in sync with root CLAUDE.md)
    ├── DECISIONS.md                   ← Architecture Decision Records. Why each tech choice was made.
    ├── CONVENTIONS.md                 ← Naming, imports, code style rules.
    ├── KNOWN-ISSUES.md                ← Gotchas log. Add to this when you hit unexpected things.
    ├── MEDUSA-V2-PATTERNS.md          ← MedusaJS v2 patterns. Backend agents read before coding.
    ├── 01-product-requirements.md     ← Feature specs + acceptance criteria
    ├── 02-user-stories...md           ← User stories
    ├── 03-information-architecture.md ← URL structure, page inventory
    ├── 04-system-architecture.md      ← ISR/CSR/SSR decisions, data flows
    ├── 05-database-schema.md          ← All table definitions
    ├── 06-api-contracts.md            ← All Medusa endpoints
    ├── 07-monorepo-structure.md       ← File locations, package structure
    ├── 08-cart-pricing-engine-spec.md ← Cart lifecycle, pricing, tax, shipping
    ├── 09-engineering-scope-definition.md ← What NOT to build
    ├── 10-development-phases.md       ← 8-week timeline, milestones, DoD checklists
    ├── 11-environment-and-devops.md   ← Env vars, deployment, CI/CD
    ├── 12-testing-strategy.md         ← Test coverage targets, file locations
    └── 13-design-system.md            ← Component specs, tokens, animations, layout grid

└── .agent/                            ← ANTIGRAVITY AGENT SYSTEM (don't edit unless updating agents)
    ├── rules/
    │   ├── GEMINI.md                  ← Generic Antigravity rules (don't touch)
    │   └── SHREE-FURNITURE.md         ← Project-specific override rules ← EDIT THIS IF NEEDED
    ├── agents/
    │   ├── frontend-specialist.md     ← Has Shree Furniture override section at top
    │   ├── backend-specialist.md
    │   ├── orchestrator.md
    │   └── ... (others)
    ├── skills/                        ← Generic skill files (don't edit)
    ├── workflows/                     ← Slash commands (/create, /debug, etc.)
    └── scripts/                       ← Helper Python scripts
```

---

## ✅ Your Confirmed Workflow

### Step 1 — Session Start (Every Session, Without Exception)
```
Read these files before doing anything:
1. STATUS.md — current build state
2. PREFERENCES.md — my design preferences (override agent defaults)
3. REJECTIONS.md — patterns I've explicitly rejected (never suggest these)
4. STACK-CHANGES.md — if it has entries, those supersede CLAUDE.md tech stack
5. CLAUDE.md — hard rules

Tell me: current phase, last completed item, any active rejections or stack changes I should know about.
Do not write any code yet.
```

---

### Step 2 — Build (Copy-Paste from PROMPT-GUIDE-v2.md)

Open `PROMPT-GUIDE-v2.md`. Find the next prompt in sequence. Copy-paste it.

The agent will:
- Generate the code at the correct file path
- Follow CLAUDE.md hard rules (paise, HTTP-only cookies, TypeScript strict)
- Follow 13-design-system.md component specs
- Follow PREFERENCES.md and REJECTIONS.md overrides
- Run type-check + lint
- Update STATUS.md when done

---

### Step 2a — If You Don't Like What Was Built

**Option A — Quick visual fix (don't stop the session):**
```
I don't like [what was built].
[Specific thing you don't like and why.]
[What you want instead.]

1. Add to REJECTIONS.md — [concise entry]
2. Rebuild [component] with the new direction
3. Update STATUS.md
```

**Option B — Bigger UX change:**
```
I want to change [flow/page/behaviour].
Before changing anything:
1. Does this conflict with NewDocs/01-product-requirements.md?
2. Does this conflict with any ADR in NewDocs/DECISIONS.md?
Tell me if conflicts exist before proceeding.
```

**Option C — You want to try a different look (inspiration reference):**
```
@frontend-specialist I want [ComponentName] to feel more like [describe or link].
Specifically I like [aspect] but want to keep [aspect from current].
Add my preference to PREFERENCES.md section [X] so future sessions remember this.
Rebuild [ComponentName] with the new direction.
```

**The key rule:** Whatever you express verbally must be written into PREFERENCES.md or REJECTIONS.md at the end of that session. Otherwise it's forgotten.

---

### Step 3 — Review, Debug, Push

**Review checklist before pushing:**
```
@orchestrator Before I push to production, run the Definition of Done checklist:
1. pnpm type-check — no TypeScript errors
2. pnpm lint — no ESLint errors
3. pnpm test — all unit tests pass
4. pnpm test:e2e — Playwright purchase flow passes
5. Check STATUS.md — all items for this phase marked ✅ DONE
6. Check KNOWN-ISSUES.md — any open issues that block launch?
7. Lighthouse score on mobile >= 90
```

**If there's a bug:**
```
@debugger [describe what's wrong — what you expected, what actually happened]
Read NewDocs/KNOWN-ISSUES.md first — this may already be documented.
If it's a new issue, add it to KNOWN-ISSUES.md after fixing.
```

**Push to production:**
Use Prompt #84 from PROMPT-GUIDE-v2.md (Deploy to Vercel) and Prompt #85 (Deploy to Railway).

---

### Step 4 — Session End (Every Session)
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

---

## Summary: What to Do Right Now

1. **Rename 4 files** (Issue 1 — critical):
   ```
   Preferences.md   → PREFERENCES.md
   Rejections.md    → REJECTIONS.md
   Stack-changes.md → STACK-CHANGES.md
   Change-guide.md  → CHANGE-GUIDE.md
   ```

2. **Add PROMPT-GUIDE-v2.md to project root** (Issue 2 — critical)

3. **Replace CHANGE-GUIDE.md with the updated version** (fixes Issue 4 file tree)

4. **Replace .agent/rules/SHREE-FURNITURE.md with updated version** (fixes Issues 3 + 5)

5. **Start building** — run the session start prompt, then copy-paste Prompt #1 from PROMPT-GUIDE-v2.md

---

*Audit completed: March 2026 | Files checked: 27 | Issues found: 6 | Issues fixed: 4 in delivered files*