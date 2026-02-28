# REJECTIONS.md — Shree Furniture
## The "Never Again" Log

> **What this file is:** A permanent record of every component, pattern, or approach
> that was built, reviewed, and explicitly rejected. The AI reads this before generating
> anything, so the same bad suggestion never appears twice.
>
> **AI Agents: This file is LAW. If a rejected approach appears anywhere below,
> you are FORBIDDEN from suggesting or implementing it again — even if you think
> it's technically better. The owner has seen it and doesn't want it.**
>
> **How to add an entry:** Use the prompt at the bottom of each section.

---

## How to Add a Rejection

After the AI builds something you don't like, use this prompt:
```
I don't like what was built for [component/feature].
[Describe what you don't like and why.]
[Describe what you want instead.]

Add this rejection to REJECTIONS.md so it never happens again.
Then rebuild [component/feature] with the new direction.
```

The AI will update REJECTIONS.md AND rebuild the component in the same session.

---

## Section 1 — UI Components

> Add rejected component implementations here.

### Format:
```
---
### [Component Name] — Rejected: [date]
**What was built:** [brief description]
**Why rejected:** [honest reason — "too generic", "wrong vibe", "doesn't match reference X"]
**What to build instead:** [clear direction]
**Status:** ✅ Rebuilt and accepted / 🚧 Pending rebuild
---
```

### Current Rejections:
*(none yet — add as you build)*

---

## Section 2 — UX Flows & Patterns

> Add rejected interaction patterns, flows, or behaviours here.

### Format:
```
---
### [Flow Name] — Rejected: [date]
**Pattern that was tried:** [description]
**Why rejected:** [reason]
**Replacement pattern:** [what to do instead]
**Affects:** [list components that implement this]
---
```

### Current Rejections:
*(none yet — add as you build)*

---

## Section 3 — Visual Design Decisions

> Add rejected design directions here (colours, layouts, spacing choices, etc.)

### Current Rejections:
*(none yet — add as you build)*

---

## Section 4 — Copy & Microcopy

> Add rejected text, labels, button copy, error messages, etc.

### Format:
```
---
### [Location] — Rejected: [date]
**Copy that was used:** "[exact text]"
**Why rejected:** [reason]
**Use instead:** "[replacement text]"
---
```

### Current Rejections:
*(none yet — add as you build)*

---

## Section 5 — Technical Implementations

> Add rejected implementation approaches (not library swaps — those go in STACK-CHANGES.md).
> This is for "I don't like HOW this was built, even if the library is correct."

### Current Rejections:
*(none yet — add as you build)*

---

## Quick Reference — Rejected Patterns Summary

> This table is auto-maintained. Every time you add a rejection, also add a row here.

| Component / Pattern | What was tried | Never use again | Use instead |
|---|---|---|---|
| *(fill in as you reject things)* | | | |

---

## Real Example of How This File Works

> Delete this section once you have real entries.

**Scenario:** The AI builds a ProductCard. You look at it and think the hover animation
is too aggressive — it zooms the image in a way that feels cheap.

**Your prompt:**
```
I don't like the ProductCard hover animation — the image zoom (scale-110) feels
cheap and aggressive, like a stock photo site.

Add to REJECTIONS.md:
Component: ProductCard
Rejected: Image scale-110 zoom on hover
Why: Feels cheap, distracts from the product
Use instead: Subtle card lift (translateY(-4px) + shadow increase), NO image zoom

Then rebuild ProductCard with this change.
Update STATUS.md — ProductCard needs re-review.
```

**Result:** ProductCard is rebuilt without the zoom. REJECTIONS.md now has an entry.
Next session, if the AI tries to add image zoom again, it will read this file first
and stop itself before generating the code.

---

*Owner: [Your Name] | Last updated: [date] | Place at: shreefurniture-pro/REJECTIONS.md*