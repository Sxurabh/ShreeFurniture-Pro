# PREFERENCES.md — Shree Furniture
## Owner's Personal Design & UX Taste Layer

> **What this file is:** Your living design diary. Every time you see something the AI built
> and think "I don't like this," write it here. Every future session will read this file
> BEFORE generating any UI, so the same mistake never happens twice.
>
> **What this file is NOT:** Technical specs (those are in NewDocs/). Architecture decisions
> (those are in DECISIONS.md). This is purely your personal taste.
>
> **AI Agents: Read this file before generating any frontend code.**
> These preferences override the generic patterns in your skill files.

---

## How to Update This File

When you don't like something the AI built, use this prompt:
```
Add to PREFERENCES.md:
Component: [name]
What I don't like: [describe it]
What I want instead: [describe it]
Reference: [link to inspiration if you have one]
```

---

## 1. Overall Aesthetic Direction

**The mood I'm going for:**
- Warm, premium, trustworthy — like walking into a well-lit furniture showroom
- IKEA clarity meets Pottery Barn warmth
- NOT: cold, corporate, startup-y, "SaaS blue"
- NOT: maximalist — clean whitespace is intentional, not laziness

**Gut-check question for every UI decision:**
> "Would a first-time buyer trust this enough to spend ₹40,000 here?"
> If the answer is uncertain, the design needs to be more conventional, not less.

---

## 2. Colour Usage Preferences

> Add specific preferences here as you discover them during development.

**What I've decided:**
- [ ] *(empty — fill in as you build)*

**Template to add entries:**
```
### [Date] — [Element]
Tried: [what the AI generated]
Didn't like it because: [reason]
Want instead: [description or hex value]
```

**Example entry:**
```
### Feb 2026 — Primary CTA Button
Tried: Solid black (#1A1A1A) fill with white text
Didn't like it because: Too heavy, overshadows the product images
Want instead: Brand Accent (#C8A96E) fill with dark text, solid black on hover
```

---

## 3. Typography Preferences

> Add as you discover them.

**Defaults to keep:**
- Headings: Playfair Display, serif — keep this, it feels premium
- Body: Inter, sans-serif — keep this, very readable

**Sizes I've confirmed I like:**
- [ ] *(fill in as you test on real devices)*

---

## 4. Component Preferences

> One section per component type. Add entries as you build.

### Buttons
**Preferences:**
- [ ] *(fill in as you review)*

**Template:**
```
Primary CTA: [description — e.g., "filled, rounded-md, 48px height on mobile"]
Secondary: [description]
Destructive: [description]
Disabled: [description]
```

---

### Cards (ProductCard)
**Preferences:**
- [ ] *(fill in as you review)*

---

### Modals & Drawers
**Preferences:**
- [ ] *(fill in as you review)*

---

### Forms
**Preferences:**
- [ ] *(fill in as you review)*

---

### Navigation
**Preferences:**
- [ ] *(fill in as you review)*

---

## 5. Animation & Motion Preferences

**General stance on animation:**
- Subtle > flashy. Every animation should have a functional reason.
- Page transitions: none (instant) — furniture buyers are focused, not browsing a portfolio
- Hover effects: yes, subtle (2–4px lift on cards, colour shift on buttons)
- Loading states: skeleton screens (not spinners) — skeleton feels more premium
- Add to cart: micro-animation on cart badge increment (scale bounce, 150ms)

**What I actively dislike:**
- [ ] *(fill in as you discover)*

---

## 6. Mobile UX Preferences

**Specific to this project:**
- Cart: always bottom sheet on mobile (slide up), never slide-in from right
- Filters: always bottom sheet on mobile (full-screen takeover)
- Product gallery: pinch-to-zoom must feel native (no custom implementation)
- Sticky CTA: "Add to Cart" must be sticky at bottom on PDP — this is non-negotiable
- Touch targets: minimum 44px height on all interactive elements

---

## 7. Content & Copy Preferences

**Tone of voice:**
- Warm but professional — like a knowledgeable showroom staff member
- No startup jargon ("Seamless", "Elevate", "Curated", "Discover")
- Specific over vague: "7–10 business days" not "Ships fast"
- Honest about price: show full price upfront, no hidden fee reveals at checkout

**Error messages:**
- Human, not technical: "Couldn't add to cart" not "POST /store/carts/items 422"
- Always actionable: "Try again" or "Contact us if this keeps happening"

---

## 8. Things I Specifically Don't Want

> This section is the "never again" list. Add to it freely.

| Element | Why I don't want it | What to use instead |
|---|---|---|
| *(fill in as you discover)* | | |

**Example entries (delete once you have real ones):**
| Element | Why I don't want it | What to use instead |
|---|---|---|
| Spinning loader | Feels cheap, makes wait time feel longer | Skeleton screens |
| Modal for add-to-cart confirmation | Interrupts browsing momentum | Cart drawer slide-in |
| Infinite scroll on PLP | Bad for SEO, hard to share position | "Load More" button |
| Purple/violet anywhere | Generic AI design aesthetic | Brand Accent (#C8A96E) |
| Glassmorphism effects | Trendy, not timeless | Clean flat cards |

---

## 9. Inspiration References

> Websites or apps whose UI/UX you like. The AI can reference these stylistically.

| Reference | What I like about it | What NOT to copy from it |
|---|---|---|
| *(add your references)* | | |

**How to add a reference:**
```
@frontend-specialist I want [ComponentName] to feel like [URL] — 
specifically the [aspect] but NOT the [aspect].
Add this to PREFERENCES.md under Inspiration References.
```

---

*Owner: [Your Name] | Last updated: [date] | Place at: shreefurniture-pro/PREFERENCES.md*