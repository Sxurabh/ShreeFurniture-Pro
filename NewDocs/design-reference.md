# NewDocs/design-reference.md — IKEA-Inspired Design Language
## Shree Furniture | Visual Design & UX Reference for AI Agents

> **Version:** 1.0 | **Status:** Active | **Authority:** This document is the primary visual reference.
>
> **Reading hierarchy for all design decisions:**
> `design-reference.md` (this file — IKEA baseline) →
> `NewDocs/13-design-system.md` (component specs) →
> `PREFERENCES.md` (owner overrides, highest priority)
>
> **AI Agents:** Read this file before generating ANY frontend component or page.
> When you are unsure about a visual decision, ask: "Would IKEA.com do it this way?"
> Then verify it matches the Shree Furniture colour tokens and Indian market context.
>
> **This is not a copy instruction.** Shree Furniture is IKEA-inspired, not IKEA.
> The goal is the same *clarity and confidence* that makes IKEA's website navigable
> by anyone, anywhere — applied to an Indian furniture audience spending real money.

---

## Part 1 — The IKEA Design DNA: What We're Learning From

### 1.1 Why IKEA's Web Design Works

IKEA.com succeeds because it is ruthlessly functional. Every design decision serves one purpose:
help the customer find the furniture, understand it fully, trust the price, and complete the purchase.
There is no decoration for its own sake. Every element earns its space.

**The 5 core IKEA design behaviours to emulate:**

1. **Radical clarity** — The user always knows exactly where they are, what the item costs, and what to do next. No ambiguity.
2. **Breathable whitespace** — Products need room to be seen. Crowding is the enemy.
3. **Photography as the hero** — UI chrome is minimal so products can be maximally prominent.
4. **Flat, confident hierarchy** — One primary action per screen. No competing priorities.
5. **Trust through completeness** — Dimensions, materials, weight, care instructions — everything is shown. Hidden information destroys trust.

### 1.2 IKEA Design Traits to Adopt

| Trait | What IKEA Does | How We Apply It |
|---|---|---|
| Clean product grids | Equal-height cards, generous gap, no border clutter | 2/3/4 col grid with 12/16/24px gaps, subtle border only |
| Sticky add-to-cart | CTA never leaves the viewport on mobile PDP | Sticky bottom bar with price + "Add to Cart" |
| Room context shots | Show furniture in a room, not just on white | Cloudinary lifestyle images alongside studio shots |
| Clear price hierarchy | Price is large, unavoidable, honest | Price at minimum 24px, bold, near the product name |
| Breadcrumb navigation | Always visible path: Home > Living Room > Sofas | Breadcrumb on PLP and PDP, every page |
| Filter left sidebar | Filter panel persistent on desktop, bottom sheet mobile | Matches our FilterPanel spec |
| "Load More" not infinite scroll | Respects user intent, SEO-friendly | As specified in product requirements |
| Stock transparency | "Only 3 left", "Out of stock" — never hidden | Low-stock and OOS states on every card and PDP |
| Dimensions are prominent | L×W×H always visible before deciding | ProductSpecsTable above the fold on PDP |
| Minimal header | Logo, search, cart — nothing else at mobile | Mobile header: logo (centre), hamburger (left), cart (right) |

### 1.3 IKEA Traits to Specifically NOT Copy

These are IKEA-specific conventions that don't translate to Shree Furniture's context:

| IKEA Does This | Why We Don't | What We Do Instead |
|---|---|---|
| Blue + yellow brand identity | Not our brand | Warm gold (#C8A96E) + near-black (#1A1A1A) |
| All-caps product name style (BILLY, KALLAX) | Names are descriptive Indian products | Normal title case: "Oslo 3-Seater Sofa" |
| Assembly instruction complexity | We acknowledge but don't feature it | Simple "Assembly guide included" note |
| Flat pricing with no warmth | IKEA is value-first; we are quality-first | Warmer language, premium presentation |
| Swedish meatballs in cafeteria | Different business | N/A |
| Store locator as primary nav | We are online-only for Phase 1 | No store nav |
| Complex co-worker design system | Enterprise internal system | We use shadcn/ui + our tokens |

---

## Part 2 — Layout System

### 2.1 Page Shell

Every page uses the same outer shell. This is non-negotiable.

```
┌─────────────────────────────────────────────────────────┐
│  HEADER (sticky, 64px desktop / 56px mobile)            │
│  [Logo]          [Search Bar]          [Wish][Cart][Nav] │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  BREADCRUMB (on all interior pages, not homepage)       │
│  Home > Living Room > Sofas                             │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  MAIN CONTENT AREA                                      │
│  max-width: 1280px, centred, horizontal padding:        │
│  24px mobile / 48px desktop                             │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  FOOTER                                                 │
└─────────────────────────────────────────────────────────┘
```

### 2.2 The IKEA PLP (Product Listing Page) Layout

IKEA's PLP is the reference for our PLP. The key is the filter sidebar + product grid split.

```
DESKTOP (≥ 1024px):
┌─────────────┬────────────────────────────────────────────┐
│             │  Sort: [Newest ▾]         48 results found │
│  FILTERS    ├────────────────────────────────────────────┤
│  (260px     │  [Card][Card][Card][Card]                  │
│  fixed,     │  [Card][Card][Card][Card]                  │
│  sticky)    │  [Card][Card][Card][Card]                  │
│             │                                            │
│  Room type  │              [Load More]                   │
│  Material   │                                            │
│  Colour     │                                            │
│  Price      │                                            │
│  Style      │                                            │
└─────────────┴────────────────────────────────────────────┘

MOBILE (< 768px):
┌────────────────────────────────────────┐
│ [Filter ≡]          Sort [Newest ▾]    │
├────────────────────────────────────────┤
│  [Card]  [Card]                        │
│  [Card]  [Card]                        │
│  [Card]  [Card]                        │
│  [Card]  [Card]                        │
│       [Load More]                      │
└────────────────────────────────────────┘

→ Tapping [Filter ≡] opens full-screen bottom sheet
→ Filters apply without page reload
→ Active filter count shown on the filter button: [Filter ≡ (3)]
```

### 2.3 The IKEA PDP (Product Detail Page) Layout

IKEA's PDP puts imagery first, information second, action third — and keeps the action visible.

```
DESKTOP (≥ 768px):
┌───────────────────────────────┬────────────────────────┐
│                               │ Collection tag          │
│   PRODUCT GALLERY             │ Product Name (H1)       │
│   (Large primary image +      │ ₹49,999 (large bold)   │
│    thumbnail strip below)     │ GST included note       │
│                               │                        │
│   Image occupies 50% width    │ ───────────────────    │
│                               │ Colour: [Grey ●][Beige ●]│
│                               │ Size: [2-Seat][3-Seat]  │
│                               │                        │
│                               │ Stock: ● In Stock       │
│                               │                        │
│                               │ Qty: [- 1 +]           │
│                               │                        │
│                               │ [ADD TO CART] ← primary│
│                               │ [♡ Add to Wishlist]    │
│                               │                        │
│                               │ ───────────────────    │
│                               │ 🚚 7–10 business days  │
│                               │ 📍 [Check PIN code]    │
└───────────────────────────────┴────────────────────────┘

Below the fold (full width):
┌──────────────────────────────────────────────────────────┐
│ [Description] [Dimensions] [Materials] [Care]  ← tabs  │
├──────────────────────────────────────────────────────────┤
│  Specifications table: L×W×H | Weight | Material | SKU   │
├──────────────────────────────────────────────────────────┤
│  RELATED PRODUCTS (horizontal scroll on mobile)          │
└──────────────────────────────────────────────────────────┘

MOBILE (< 768px):
- Gallery: Full width, swipeable
- Sticky bottom bar: [₹49,999] [ADD TO CART] (always visible)
- All content stacks vertically in this order:
  Image → Name → Price → Variants → Stock → Delivery → CTA → Specs → Related
```

### 2.4 The Homepage Layout

IKEA's homepage is a sequence of clearly-defined zones. Each zone has one purpose.

```
ZONE 1: HERO
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   Full-width lifestyle image (furniture in a room)       │
│   Headline: "Make it home."                             │
│   Subheadline: "Quality furniture for every room."      │
│   CTA: [Shop Collections]                               │
│                                                          │
│   Height: 560px desktop / 400px mobile                  │
│   Image: Cloudinary, WebP, priority loading             │
└──────────────────────────────────────────────────────────┘

ZONE 2: COLLECTION GRID (4 cards desktop / 2×2 mobile)
┌──────────┬──────────┬──────────┬──────────┐
│ Living   │ Bedroom  │ Dining   │ Office   │
│ Room     │          │          │          │
└──────────┴──────────┴──────────┴──────────┘
→ Each card: category image + label + "Explore →"
→ Clicking goes to /collections/{handle}

ZONE 3: BEST SELLERS (product card carousel)
Heading: "Best Sellers"  [View All →]
[Card][Card][Card][Card]  ← horizontal scroll on mobile

ZONE 4: TRUST SIGNALS (full-width band)
[🚚 Free Delivery ₹5,000+] [↩ 7-Day Returns] [💳 EMI] [🛡 Warranty]

ZONE 5: NEW ARRIVALS (product card grid)
Heading: "New Arrivals"  [View All →]
[Card][Card][Card][Card]  ← 2 col mobile, 4 col desktop

ZONE 6: FOOTER
```

---

## Part 3 — Header Design (IKEA Reference)

### 3.1 Desktop Header

IKEA's header is confident and restrained. Navigation is clear, not overwhelming.

```
HEIGHT: 64px
BACKGROUND: #FFFFFF
BORDER-BOTTOM: 1px solid #E8E0D0
POSITION: sticky top-0, z-index 40
BOX-SHADOW on scroll: shadow.xl (see 13-design-system.md §5)

LAYOUT (left to right):
[LOGO]  ←→ space ←→  [SEARCH BAR (centre, 420px wide)]  ←→ space ←→  [♡ Wishlist] [🛒 Cart (n)] [Account]

LOGO:
- Shree Furniture wordmark or logotype
- Height: 32px
- Click: navigates to /

SEARCH BAR:
- Placeholder: "Search for furniture, room type..."
- Clicking expands into SearchOverlay (full-screen modal)
- Has search icon (left inside input), Lucide Search, 20px

CART ICON:
- ShoppingCart (Lucide, 20px, stroke 1.5)
- Badge: item count, positioned top-right of icon
- Badge colour: brand.primary (#1A1A1A), text: white, min-width 18px, border-radius 50%

NAVIGATION (below header bar — separate nav row, NOT a mega-menu for MVP):
[Living Room]  [Bedroom]  [Dining]  [Office]  [All Products]  [New Arrivals]  [Sale]
- Font: Inter Medium 14px, text.secondary (#5A5A5A)
- Hover: text.primary (#1A1A1A), underline animation
- Active: brand.accent (#C8A96E) underline, 2px, text.primary
```

### 3.2 Mobile Header

```
HEIGHT: 56px
LAYOUT:
[☰ Menu]   ←→ space ←→   [LOGO (centred)]   ←→ space ←→   [🔍][🛒]

→ ☰ opens MobileMenu (slide from left, full height)
→ 🔍 opens SearchOverlay (full screen)
→ 🛒 opens CartDrawer (slide up from bottom)

MobileMenu contents (slide-in from left):
┌────────────────────────────┐
│ [×] Close                  │
├────────────────────────────┤
│ Living Room          [>]   │
│ Bedroom              [>]   │
│ Dining               [>]   │
│ Office               [>]   │
│ All Products         [>]   │
│ New Arrivals         [>]   │
├────────────────────────────┤
│ My Account                 │
│ My Orders                  │
│ Wishlist                   │
│ Track Order                │
└────────────────────────────┘
```

---

## Part 4 — Product Card Design (IKEA Reference)

### 4.1 Card Anatomy

IKEA's product card is deceptively simple. Every element has a fixed role.

```
┌──────────────────────────────┐
│ ████████████████████████████ │ ← Image: aspect-ratio 1:1, object-fit cover
│ ████████████████████████████ │   No padding around image — edge to edge
│ ██████████████ [-17%] ██████ │ ← Discount badge: top-right of image
│ ██████████████ [♡]    ██████ │ ← Wishlist: top-left of image, 32px tap target
│ ████████████████████████████ │
├──────────────────────────────┤
│ Living Room              ↑   │ ← Collection tag: Inter 12px, text.muted
│ Oslo 3-Seater Sofa           │ ← Product name: Inter Semibold 16px, line-clamp-2
│ ₹49,999                      │ ← Price: Inter Bold 18px, text.primary
│ ~~₹59,999~~                  │ ← Original price: strikethrough, text.muted 14px
│ ⚠ Only 3 left                │ ← Low stock: semantic.warning, 12px (only if qty ≤ 3)
└──────────────────────────────┘

TEXT AREA PADDING: 12px all sides
CARD BORDER-RADIUS: 12px
CARD BORDER: 1px solid #E8E0D0
CARD SHADOW (default): shadow.sm
CARD SHADOW (hover): shadow.md + translateY(-4px)  ← NO image zoom, NO image scale
TRANSITION: transform 200ms ease, box-shadow 200ms ease
```

### 4.2 Card Behaviour States

```
DEFAULT:
  - Border: 1px solid #E8E0D0
  - Shadow: shadow.sm (very subtle)

HOVER (desktop only):
  - Shadow: shadow.md
  - Transform: translateY(-4px)   ← card lifts, image does NOT zoom
  - Wishlist icon: visible (opacity 1 from 0)
  - Cursor: pointer

OUT OF STOCK:
  - Image: brightness(0.85) grayscale(20%)
  - Badge: "Out of Stock" (brand.subtle bg, text.muted text)
  - Card still clickable (goes to PDP where variants are shown)
  - "Add to Cart" does not appear

LOADING (skeleton):
  - All areas replaced with skeleton shimmer
  - Shimmer: base #E8E0D0, highlight #F5F0E8, animation: shimmer 1.5s infinite
  - aria-busy="true" on container
```

### 4.3 Wishlist Icon Behaviour

```
DESKTOP:
  - Hidden by default (opacity: 0)
  - Appears on card hover (opacity: 1, transition 150ms)
  - Position: top-left of image, 8px margin from edges
  - Size: 32px × 32px tap target, 20px icon
  - Background: rgba(255,255,255,0.9) circular pill
  - Not-wishlisted: Heart outline, text.secondary
  - Wishlisted: Heart filled, semantic.error (#C0392B)

MOBILE:
  - Always visible (no hover state exists)
  - Same position, same size
```

---

## Part 5 — Typography in Context

### 5.1 The IKEA Typography Principle

IKEA uses two typefaces maximum. Heavy weights for impact. Normal weights for readability.
We do the same: Playfair Display (headings only) + Inter (everything else).

### 5.2 When to Use Each Font

```
Playfair Display (serif) — ONLY for:
  ✅ H1: Hero headline ("Make it home.")
  ✅ H1: Page title on PDP (product name, but only at 36px+)
  ✅ H2: Major section headings on Homepage ("Best Sellers", "Living Room")
  ✅ Category page hero titles

Inter (sans-serif) — For EVERYTHING else:
  ✅ All body copy
  ✅ Navigation links
  ✅ Prices (Inter Bold)
  ✅ Buttons
  ✅ Labels, badges, tags
  ✅ Form fields
  ✅ Descriptions
  ✅ Breadcrumbs
  ✅ Meta information, timestamps
  ✅ Error messages
  ✅ H3 and below headings (section subheadings)
```

### 5.3 Type Scale Applied to Pages

**Hero Section (Homepage):**
```
H1 Headline:   Playfair Display Bold, 48px desktop / 32px mobile, line-height 1.15
Subheadline:   Inter Regular, 20px desktop / 16px mobile, text.secondary, line-height 1.5
CTA text:      Inter Semibold, 16px, text.inverse (on dark button)
```

**Product Name (PDP):**
```
H1 on PDP:     Playfair Display Semibold, 36px desktop / 28px mobile, line-height 1.2
Variant name:  Inter Regular, 16px, text.secondary ("Grey / L")
Price:         Inter Bold, 32px desktop / 28px mobile, text.primary
Original price: Inter Regular 20px, text.muted, text-decoration line-through
```

**Product Card:**
```
Category tag:  Inter Regular, 12px, text.muted, uppercase letter-spacing 0.5px
Product name:  Inter Semibold, 16px, text.primary, line-clamp: 2
Price:         Inter Bold, 18px, text.primary
Original:      Inter Regular, 14px, text.muted, strikethrough
Stock warning: Inter Medium, 12px, semantic.warning
```

**PLP Page:**
```
Page title:    Playfair Display Semibold, 36px desktop, 24px mobile
Result count:  Inter Regular, 14px, text.muted ("48 results")
Sort label:    Inter Medium, 14px, text.secondary
Filter heading: Inter Semibold, 14px, text.primary
Filter option: Inter Regular, 14px, text.secondary
```

**Section Headings (Homepage zones):**
```
Zone heading:  Playfair Display Semibold, 30px desktop / 24px mobile
"View All" link: Inter Medium, 14px, text.secondary, with underline on hover
```

---

## Part 6 — Whitespace & Breathing Room

### 6.1 The IKEA Whitespace Principle

IKEA leaves generous space around products. This communicates premium quality.
A crowded page feels like a bazaar. A spacious page feels like a showroom.

**Rule:** When in doubt, add 8px more. Furniture needs room to breathe.

### 6.2 Section Spacing (Homepage Zones)

```
Gap between homepage zones:    64px desktop / 48px mobile
Zone internal padding (top):   48px desktop / 32px mobile
Zone internal padding (bottom): 48px desktop / 32px mobile
Section heading to first card: 24px
```

### 6.3 Product Grid Gaps

```
Mobile (2 columns):  gap: 12px
Tablet (3 columns):  gap: 16px
Desktop (4 columns): gap: 24px
```

### 6.4 PDP Spacing

```
Gallery to product info (desktop): 48px gap (flex/grid)
Product name to price:             8px
Price to variants:                 24px
Variants to CTA:                   32px
CTA to delivery info:              24px
Each spec row:                     16px
```

### 6.5 Card Text Area

```
Image to text area:     0px (image is edge-to-edge to card top/sides)
Text area padding:      12px all sides
Product name to price:  8px
Price to stock warning: 4px
```

---

## Part 7 — Colour in Practice

### 7.1 Background Hierarchy

IKEA uses white backgrounds for products. We use our warm off-white.

```
Page background:       #F5F0E8 (brand.warm) — the overall page background
Card background:       #FFFFFF — cards on the warm background pop cleanly
Header background:     #FFFFFF — stays white even on warm pages
Section alternate bg:  #FFFFFF — use to separate homepage zones visually
Modal/drawer bg:       #FFFFFF
Input background:      #FFFFFF
```

### 7.2 How the Accent Colour (#C8A96E) is Used

The warm gold accent is used sparingly — it has power because it's rare.

```
USE IT FOR:
✅ Discount badges (text: brand.primary)
✅ Active navigation indicator (underline on active nav link)
✅ Hover state on text links ("View All →" link)
✅ Focus ring on all interactive elements (2px solid)
✅ "Sale" or "Hot" badge backgrounds
✅ Accent buttons (variant: accent — gold bg, dark text)
✅ Star rating fill colour (Phase 2)
✅ Decorative horizontal rule under hero headline

DO NOT USE IT FOR:
❌ Primary CTA buttons (those are brand.primary / black)
❌ Large background fills
❌ Body text
❌ Error states
❌ Borders (use brand.subtle for those)
```

### 7.3 How Black (#1A1A1A) is Used

```
USE IT FOR:
✅ Primary CTA button backgrounds
✅ All heading text
✅ Primary body text
✅ Navigation link text (hover state)
✅ Input focus border (2px solid)
✅ Price text

DO NOT USE IT FOR:
❌ Pure #000000 (too harsh on warm backgrounds — always use #1A1A1A)
❌ Large background fills on the storefront
❌ Decorative elements (use brand.subtle for dividers)
```

### 7.4 Colour Don'ts (Non-Negotiable)

```
NEVER:
❌ Purple, violet, indigo anywhere
❌ Neon or fluorescent colours
❌ Blue (except semantic.info for informational banners)
❌ Green for anything except "In Stock" and "Order Confirmed"
❌ Dark mode (not in scope for MVP)
❌ Gradients on backgrounds (flat colours only)
❌ Multiple accent colours — one accent, one primary, that's it
```

---

## Part 8 — Navigation Design

### 8.1 Category Navigation (Sub-header)

IKEA uses a persistent category nav below the main header on desktop.

```
HEIGHT: 44px
BACKGROUND: #FFFFFF
BORDER-BOTTOM: 1px solid #E8E0D0
OVERFLOW: hidden (hidden on mobile — hamburger takes over)

ITEMS (left to right, separated by 32px):
Living Room | Bedroom | Dining | Office | All Products | New Arrivals | Sale

STYLES:
Normal: Inter Medium 14px, color: #5A5A5A (text.secondary)
Hover: color: #1A1A1A (text.primary), text-decoration: underline, underline-color: #C8A96E
Active (current page): color: #1A1A1A, border-bottom: 2px solid #C8A96E on the nav item
```

### 8.2 Breadcrumb

```
POSITION: Below header/sub-nav, above page title
HEIGHT: 44px (with vertical centering)
SEPARATOR: ChevronRight icon (Lucide, 14px, text.muted)
FONT: Inter Regular 14px, text.muted
LAST ITEM: text.primary (current page, not a link)
FIRST ITEM: "Home" — always

Example: Home › Living Room › Sofas › Oslo 3-Seater Sofa

MOBILE: Show full breadcrumb (do not truncate on mobile — it's compact enough)
DESKTOP: Full breadcrumb always visible
```

### 8.3 SearchOverlay

IKEA's search is full-screen to give results maximum space.

```
TRIGGER: Click search icon in header
BEHAVIOUR: Full-screen overlay, fades in 150ms

LAYOUT:
┌──────────────────────────────────────────────────────────┐
│  [×]                                    [Shree Furniture]│
├──────────────────────────────────────────────────────────┤
│                                                          │
│  🔍 [_________________ Search furniture ________]        │
│                                                          │
│  Suggestions (shows after 1 character):                  │
│  • Oslo 3-Seater Sofa           Living Room             │
│  • Oslo Armchair                Living Room             │
│  • Oslo Coffee Table            Living Room             │
│                                                          │
│  POPULAR SEARCHES                                        │
│  [Sofa] [Bed Frame] [Dining Table] [Office Chair]       │
│                                                          │
└──────────────────────────────────────────────────────────┘

BEHAVIOUR:
- Debounce: 300ms before Algolia query fires
- Suggestions: product title + collection name
- No results: "No results for 'xyz'" + suggested categories
- Pressing Enter or selecting a result: navigates to search page or PDP
- Mobile: Full screen, keyboard opens automatically
```

---

## Part 9 — Cart & Checkout Design

### 9.1 Cart Drawer

```
DESKTOP (≥ 768px):
→ Slides in from RIGHT
→ Width: 420px
→ Full viewport height
→ Overlay: rgba(0,0,0,0.3) backdrop (clicking backdrop closes drawer)

MOBILE (< 768px):
→ Slides UP from bottom
→ Width: 100vw
→ Height: 85vh max
→ Top corners: border-radius 16px
→ Drag handle: 40px × 4px bar, brand.subtle, centred, 8px from top

DRAWER INTERNAL LAYOUT:
┌──────────────────────────────┐
│ Cart (3 items)           [×] │ ← Header with count and close
├──────────────────────────────┤
│ [img] Oslo 3-Seater Sofa     │ ← Line item
│       Grey / L               │
│       [- 2 +]   ₹49,999  [🗑]│
├──────────────────────────────┤
│ [img] Item 2                 │
│       ...                    │
├──────────────────────────────┤
│ ─────────────────────────── │
│ Subtotal:         ₹1,10,000  │
│ Shipping:              Free  │
│ (GST included)               │
│ ─────────────────────────── │
│ Total:            ₹1,10,000  │
├──────────────────────────────┤
│ [PROCEED TO CHECKOUT]        │ ← Primary, full-width, brand.primary
│ [Continue Shopping]          │ ← Ghost button, full-width
└──────────────────────────────┘

FREE SHIPPING INDICATOR:
If subtotal < ₹5,000:
"Add ₹X more for free shipping" ← progress bar style, brand.accent fill
If subtotal ≥ ₹5,000:
"✓ You have free shipping!" ← semantic.success colour
```

### 9.2 Checkout — Step Indicator

```
LAYOUT (3 steps, always visible at top of checkout pages):
┌─────────────────────────────────────────────────────┐
│   [1 Address] ──── [2 Shipping] ──── [3 Payment]   │
└─────────────────────────────────────────────────────┘

STEP STATES:
Completed: Circle filled brand.primary, white checkmark, line to next is brand.primary
Current: Circle filled brand.primary, number in white, no right line filled
Upcoming: Circle outline brand.subtle, number text.muted, dashed line

FONT: Inter Medium 12px
STEP WIDTH: Equal thirds of container width
MOBILE: Same layout, smaller font (11px), circles 24px
```

### 9.3 Checkout Forms — IKEA Clarity Standard

IKEA's checkout is famous for being distraction-free. Apply this strictly.

```
RULES:
- One column on mobile, two columns on desktop for address fields
- Full Name: single field (not split first/last on mobile)
- PIN Code first, then City and State auto-populate if possible (Phase 2)
- Phone: standard 10-digit input with "+91" prefix shown in the field (non-editable)
- Every field has a visible label (no floating labels — too clever, reduces clarity)
- Error messages appear immediately below the field, not in a toast
- "Continue" CTA is at the bottom, full-width, brand.primary
- No decorative elements, no sidebars — just the form and order summary sidebar (desktop)

FIELD ORDER (AddressForm):
1. Full Name (text)
2. Mobile Number (+91 prefix shown)
3. Address Line 1 (Flat/House No., Building, Street)
4. Address Line 2 (Area / Locality — optional but encouraged)
5. City
6. State (dropdown)
7. PIN Code
```

---

## Part 10 — Photography & Image Standards

### 10.1 IKEA's Image Philosophy

IKEA shows furniture in life. Not just product shots on white. This is their biggest visual differentiator.

**Two image types per product (aim for this):**

| Type | Description | Used Where |
|---|---|---|
| Studio shot | Product on neutral background — white or brand.warm | PDP primary, product card |
| Lifestyle shot | Product in a staged room setting | Hero section, collection covers, PDP gallery secondary images |

### 10.2 Image Composition Guidelines for Content Creation

These apply when briefing photographers or selecting/creating product imagery:

```
STUDIO SHOTS:
- Background: White (#FFFFFF) or warm off-white (#F5F0E8) only
- Lighting: Soft, even, no harsh shadows
- Angle: 3/4 perspective (slight angle) is preferred for 3D objects; dead-on for flat items
- Margins: 8% padding around the product within the frame
- Aspect ratio: 1:1 (square) — matches our grid

LIFESTYLE SHOTS:
- Room must look aspirational but achievable — Indian home context
- Warm lighting (no cold blue tones)
- Simple backgrounds — avoid clutter around the hero product
- The furniture must be clearly the focal point
- Aspect ratio: 16:9 or 4:3 for hero banners; 1:1 for PDP gallery

WHAT TO AVOID:
- Fake CGI renders that look obviously artificial
- Models posing awkwardly with furniture
- Outdoor furniture shots for indoor furniture
- Clashing colours in lifestyle staging
- Watermarks or logos from suppliers
```

### 10.3 Cloudinary Image Standards

Reference: `NewDocs/18-cloudinary-guide.md` for all technical implementation.

```
MINIMUM UPLOAD RESOLUTION: 1600×1600px for studio shots
ACCEPTED FORMATS: JPG, PNG, WebP (Cloudinary converts to WebP on delivery)
PDP GALLERY: Maximum 5 images (1 primary studio + up to 4 additional)
CARD THUMBNAIL: Always primary.webp — the first studio shot
HERO IMAGES: Minimum 1920×800px for full-width banners
```

---

## Part 11 — Micro-copy & Tone of Voice

### 11.1 The Shree Furniture Voice

IKEA has a warm, unpretentious, slightly playful voice. We match the warmth and honesty,
but with the professionalism appropriate for customers spending ₹10,000–₹2,00,000.

```
VOICE ATTRIBUTES:
✅ Warm — friendly, not corporate
✅ Honest — real information, no puffery
✅ Confident — we know our products are good; we don't need to over-sell
✅ Specific — exact dimensions, exact delivery times, exact prices

NOT:
❌ Jargony ("Elevate your space")
❌ Aggressive ("Buy Now! Limited Time!")
❌ Vague ("Premium quality" without specifics)
❌ Cold ("Your order has been processed.")
```

### 11.2 Specific Copy Standards by Element

**Page titles and H1:**
```
❌ "Oslo Sofa — Buy Online at Best Price in India"
✅ "Oslo 3-Seater Sofa"
(SEO keywords go in meta description, not visible H1)
```

**Product card CTA (if any):**
```
No CTA on the card itself — clicking the card goes to PDP
The only card-level action is the wishlist icon
```

**Add to Cart button:**
```
✅ "Add to Cart"   (standard state)
✅ "Adding..."     (loading, 0.5s optimistic)
✅ "Added ✓"      (success, shows 1.5s then reverts)
❌ "Buy Now"      (too aggressive for a considered purchase)
❌ "Shop Now"     (imprecise on a PDP — we know they want this item)
```

**Stock messaging:**
```
✅ "In Stock"                    (inventory > 3)
✅ "Only 3 left"                 (inventory 1–3)
✅ "Out of Stock"                (inventory 0)
❌ "Hurry! Limited stock!"      (manufactured urgency)
❌ "Only 2 left in stock — order soon!" (Amazon-style pressure)
```

**Delivery copy:**
```
✅ "Delivery in 7–10 business days"
✅ "Free delivery on orders above ₹5,000"
❌ "Ships fast!"
❌ "Quick delivery"
❌ "We'll get it to you soon"
```

**Error messages (forms):**
```
✅ "Please enter a valid 6-digit PIN code"
✅ "Mobile number must be 10 digits starting with 6, 7, 8 or 9"
✅ "Couldn't add to cart — please try again"
❌ "Error: validation failed"
❌ "POST /store/carts 422"
❌ "Invalid input"
```

**Empty states:**
```
Search — no results:
✅ "No results for 'xyz'. Try 'sofa', 'bed', or browse our collections."

Wishlist — empty:
✅ "Your wishlist is empty. Save items you love while browsing."

Cart — empty:
✅ "Your cart is empty. Ready to find your next favourite piece?"
```

---

## Part 12 — Animation & Motion

### 12.1 The Motion Hierarchy

Reference `NewDocs/13-design-system.md §7` for the full approved animation list.

**IKEA principle applied:** Motion communicates. It doesn't entertain.

```
TIER 1 — Structural (must be smooth, never janky):
→ Cart drawer open/close: 300ms cubic-bezier(0.32, 0.72, 0, 1)
→ Search overlay open/close: 150ms ease
→ Mobile menu open/close: 250ms ease-out
→ Sticky header shadow on scroll: instant (CSS transition none)

TIER 2 — Feedback (confirms user action):
→ Card hover lift: 200ms ease
→ Button hover colour change: 150ms ease
→ Button click scale: 100ms ease
→ Cart badge increment bounce: 200ms spring
→ Form field focus border: 100ms ease
→ Toast notification appear: 250ms ease-out

TIER 3 — Content reveal (progressive loading):
→ Skeleton to content fade: 200ms ease
→ Page section reveal on scroll: NOT used (keep it simple for Phase 1)
→ Image load blur-up: built into Next.js Image with blurDataURL

NEVER ANIMATE:
❌ Product images zooming on card hover
❌ Page transitions between routes
❌ Parallax effects
❌ Continuous/looping animations anywhere in the purchase path
❌ Animated backgrounds or gradient shifts
```

---

## Part 13 — Responsive Design Behaviour

### 13.1 Breakpoint Behaviour Reference

| Feature | Mobile (< 640px) | Tablet (640–1024px) | Desktop (> 1024px) |
|---|---|---|---|
| Product grid | 2 columns, 12px gap | 3 columns, 16px gap | 4 columns, 24px gap |
| Header | 56px, hamburger menu | 56px, hamburger | 64px, full nav |
| Cart | Bottom sheet, slides up | Bottom sheet | Right drawer, 420px |
| Filter panel | Bottom sheet, full screen | Bottom sheet | Left sidebar, 260px sticky |
| PDP layout | Single column, gallery full-width | Single column | Two column (50/50) |
| Add to Cart | Sticky bottom bar (always visible) | Sticky bottom bar | In-flow, right column |
| Search | Full-screen overlay | Full-screen overlay | Full-screen overlay |
| Breadcrumb | Visible (all) | Visible (all) | Visible (all) |
| Font sizes | Reduced (see §5.3) | Tablet | Desktop sizes |
| Page H-padding | 16px | 32px | 48px |

### 13.2 Touch Target Minimum

Minimum 44×44px touch target for ALL interactive elements on mobile.
For small icons (wishlist, close, etc.): the visible icon can be 20px but the
interactive area must be padded to at least 44×44px using padding.

```css
/* Example: 20px wishlist icon with 44px touch target */
.wishlist-button {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

---

## Part 14 — IKEA-Inspired Homepage Section Patterns

### 14.1 Collection Category Grid

IKEA's category cards use room photography — not just icons or coloured boxes.

```
4 CARDS DESKTOP / 2×2 MOBILE
Each card:
┌────────────────────────┐
│                        │  ← Room lifestyle photo
│    [Room image]        │     aspect-ratio: 4/3 desktop, 1/1 mobile
│                        │
├────────────────────────┤
│ Living Room            │  ← Playfair Display Semibold 20px
│ 48 items               │  ← Inter Regular 13px, text.muted (item count)
│ Explore →              │  ← Inter Medium 14px, brand.accent colour
└────────────────────────┘

HOVER: subtle shadow increase, "Explore →" underlines
CLICK: Navigates to /collections/{handle}
```

### 14.2 Trust Signal Bar

IKEA's equivalent of this is their delivery/service strip. Keep it clean.

```
LAYOUT: Horizontal flex, equal-width columns, centred content
BACKGROUND: #FFFFFF (contrasts with brand.warm page bg)
BORDER-TOP + BORDER-BOTTOM: 1px solid #E8E0D0
PADDING: 24px vertical
HEIGHT: ~90px desktop / auto (wraps to 2×2 grid) mobile

EACH SIGNAL:
┌──────────────────────┐
│ [Icon]               │  ← Lucide icon, 24px, text.secondary
│ Free Delivery        │  ← Inter Semibold 14px, text.primary
│ on orders ₹5,000+   │  ← Inter Regular 13px, text.muted
└──────────────────────┘
```

### 14.3 Product Carousel (Best Sellers, New Arrivals)

```
DESKTOP: 4 cards visible, no carousel controls (just a grid row)
TABLET: 3 cards visible
MOBILE: Horizontal scroll, 1.5 cards visible (shows there are more)

MOBILE HORIZONTAL SCROLL:
- overflow-x: auto, scroll-snap-type: x mandatory
- Each card: scroll-snap-align: start
- Scrollbar: hidden (scrollbar-width: none)
- Show "View All" as a final tappable card: [→ View all 48]

SECTION HEADER:
┌─────────────────────────────────────────────────────────┐
│ Best Sellers                              [View All →]  │
└─────────────────────────────────────────────────────────┘
Heading: Playfair Display Semibold 30px
"View All →": Inter Medium 14px, text.secondary, underline on hover
```

---

## Part 15 — Footer Design

### 15.1 Footer Layout

IKEA's footer is comprehensive and well-organised. It's the last trust signal.

```
BACKGROUND: #1A1A1A (brand.primary)
TEXT: #FFFFFF (text.inverse)

DESKTOP LAYOUT (4 columns):
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Shree        │ Shop         │ Help         │ Contact      │
│ Furniture    │              │              │              │
│              │ Living Room  │ Track Order  │ support@...  │
│ [Logo]       │ Bedroom      │ Returns      │ Mon-Sat      │
│              │ Dining       │ Shipping Info│ 10am-6pm IST │
│ India's home │ Office       │ FAQs         │              │
│ for quality  │ New Arrivals │ Care Guide   │ [Instagram]  │
│ furniture.   │ Sale         │ Assembly     │ [Facebook]   │
└──────────────┴──────────────┴──────────────┴──────────────┘
│         © 2026 Shree Furniture | Privacy | Terms          │

MOBILE: Single column stack, each section collapsible (accordion)
HEADING FONT: Inter Semibold 14px, #FFFFFF
LINK FONT: Inter Regular 13px, rgba(255,255,255,0.7), hover: #FFFFFF
DIVIDER: 1px solid rgba(255,255,255,0.1) between columns on desktop
PADDING: 48px top, 32px bottom, 48px horizontal desktop / 24px mobile
```

---

## Part 16 — Agent Checklist (Before Marking Any UI Task Done)

Every AI agent building frontend components should run through this checklist before
marking a task as complete. This encodes IKEA's quality standard into the workflow.

```
VISUAL QUALITY:
□ Does every element use a token from packages/ui/src/tokens.ts? (No hardcoded hex)
□ Is the product image the visual hero? (Is UI chrome minimal around it?)
□ Is the primary CTA obvious? (One action, prominent, unavoidable)
□ Is the price readable at a glance? (Inter Bold, large enough, near product name)
□ Is there enough whitespace? (Would a first-time ₹40,000 buyer trust this?)

IKEA PATTERN COMPLIANCE:
□ No image zoom on card hover (translateY lift only)
□ Playfair Display ONLY for H1/H2 headings (not labels, not buttons)
□ brand.accent (#C8A96E) used sparingly — only in the right places (§7.2)
□ Breadcrumb visible on all interior pages
□ Mobile cart is a bottom sheet (NOT a right drawer)

TYPOGRAPHY:
□ No heading uses Inter (except H3 and below)
□ No body text uses Playfair Display
□ Prices use Inter Bold (not Regular)
□ Line-clamp applied to product names where needed

ACCESSIBILITY:
□ All icon buttons have aria-label
□ Form inputs have visible labels (not just placeholder)
□ Focus ring: 2px solid #C8A96E, offset 2px
□ Touch targets minimum 44×44px on mobile
□ alt text on all product images (not empty)

INDIA-SPECIFIC:
□ Price shown as ₹XX,XXX (Indian format via formatPrice() from @shree/types)
□ Delivery copy uses exact strings from NewDocs/17-india-specific-guide.md §8
□ Phone field shows +91 prefix, accepts 10 digits only
□ PIN code field validates 6 digits
```

---

## Part 17 — Design Hierarchy for Conflict Resolution

When there is any conflict between design documents, use this hierarchy:

```
1. PREFERENCES.md  ← WINS ALWAYS. Owner's live overrides.
2. REJECTIONS.md   ← BLOCKS always. Nothing rejected appears again.
3. design-reference.md (this file) ← IKEA-inspired baseline
4. NewDocs/13-design-system.md    ← Component specs and tokens
5. Agent skill files              ← Generic design patterns (lowest priority)
```

**Practical example:**
If 13-design-system.md says "Primary button is black (#1A1A1A)" but PREFERENCES.md says
"I prefer the accent gold (#C8A96E) for primary buttons" — PREFERENCES.md wins.
If this file says "No image zoom on hover" and a skill file suggests it — this file wins.

---

*Shree Furniture | design-reference.md | v1.0 — March 2026*
*Place at: `NewDocs/design-reference.md`*
*After placing: Add to SHREE-FURNITURE.md session start protocol as step 6.*
*After placing: Update PREFERENCES.md header to reference this file as baseline.*
