# 13 — Design System
## Shree Furniture | Component Specs, Interaction States, Layout Rules

> **Version:** 1.0 | **Status:** Active
>
> **AI Agents:** Read this file before building any UI component or page layout.
> This file defines the fixed design decisions — what every component looks like by default.
> For the owner's personal taste overrides, read PREFERENCES.md (project root) afterward.
> PREFERENCES.md wins over this file for any entry it contains.
>
> **Source of truth hierarchy for design:**
> PREFERENCES.md (owner taste, changes over time) > this file (project defaults) > agent skill files (generic)

---

## 1. Design Principles

### The One Question
> "Would a first-time buyer trust this enough to spend ₹40,000 here?"

Every visual decision runs through this filter. When in doubt, choose the more conventional option. Trust beats cleverness.

### Four Principles
1. **Warmth** — furniture is emotional. Colours and typography should feel like a well-lit showroom, not a tech product.
2. **Clarity** — the purchase path must be obvious. Product → Price → Add to Cart should dominate every screen.
3. **Honesty** — show real dimensions, real prices, real delivery times. No dark patterns.
4. **Speed** — every animation and transition must earn its right to exist. If it doesn't help the user, remove it.

---

## 2. Colour System

All values from `packages/ui/src/tokens.ts`. **Never hardcode hex in component files.**

### Brand Palette

| Token | Hex | Use |
|---|---|---|
| `brand.primary` | `#1A1A1A` | Headings, primary text, nav links, borders on active states |
| `brand.accent` | `#C8A96E` | Warm gold — sale badges, discount %, hover highlights, active nav indicator |
| `brand.warm` | `#F5F0E8` | Page backgrounds, card backgrounds, section fills |
| `brand.subtle` | `#E8E0D0` | Card borders, dividers, input borders, skeleton base |

### Semantic Colours

| Token | Hex | Use |
|---|---|---|
| `semantic.success` | `#2D7A4F` | "In Stock", order confirmed, successful form submission |
| `semantic.error` | `#C0392B` | "Out of Stock", form validation errors, payment failure |
| `semantic.warning` | `#E67E22` | "Only 3 left", low stock alert |
| `semantic.info` | `#2980B9` | Informational banners, tooltips |

### Text Colours

| Token | Hex | Use |
|---|---|---|
| `text.primary` | `#1A1A1A` | Body text, labels |
| `text.secondary` | `#5A5A5A` | Supporting text, meta info, descriptions |
| `text.muted` | `#9A9A9A` | Placeholders, disabled labels, timestamps |
| `text.inverse` | `#FFFFFF` | Text on dark backgrounds |

### What to Never Use
- Purple, violet, indigo, magenta — generic AI aesthetic
- Pure `#000000` black — too harsh against warm backgrounds
- Neon or high-contrast palettes
- Dark mode colours — not in scope for MVP

---

## 3. Typography

### Font Families
| Use | Font | Fallback |
|---|---|---|
| All headings (H1–H4) | Playfair Display | Georgia, serif |
| All body text | Inter | -apple-system, sans-serif |
| Prices | Inter (bold) | same |
| Monospace (order IDs, SKUs) | JetBrains Mono | monospace |

### Type Scale
| Size token | px | Use |
|---|---|---|
| `text.xs` | 12px | Badge labels, legal footnotes |
| `text.sm` | 14px | Supporting text, metadata, nav links |
| `text.base` | 16px | Body text default |
| `text.lg` | 18px | Card product name, form labels |
| `text.xl` | 20px | Section subheadings |
| `text.2xl` | 24px | Card/section headings (H3) |
| `text.3xl` | 30px | Page headings (H2) |
| `text.4xl` | 36px | Hero headings (H1, desktop) |

### Font Weights
- Regular (400): body text, descriptions
- Medium (500): nav links, labels, card subtitles
- Semibold (600): product names, section headings
- Bold (700): hero headings, prices, primary CTAs

### Line Heights
- Headings: 1.2 (tight)
- Body: 1.6 (comfortable reading)
- UI labels/buttons: 1.0 (single line)

---

## 4. Spacing System

Base unit: **8px**. All spacing should be multiples of 8 (or 4 for fine adjustments).

| Name | Value | Use |
|---|---|---|
| `space.1` | 4px | Icon gaps, tight internal padding |
| `space.2` | 8px | Default gap between small elements |
| `space.3` | 12px | |
| `space.4` | 16px | Default component padding |
| `space.5` | 20px | |
| `space.6` | 24px | Section internal padding, card padding |
| `space.8` | 32px | Between components |
| `space.10` | 40px | |
| `space.12` | 48px | Between page sections |
| `space.16` | 64px | Large section gaps |

Page max-width: `1280px`. Page horizontal padding: `24px` mobile, `48px` desktop.

---

## 5. Border & Shadow System

### Border Radius
| Use | Value |
|---|---|
| Small elements (badges, tags) | `4px` |
| Buttons | `8px` (default), `9999px` for pill-style |
| Cards, inputs, drawers | `12px` |
| Modals, bottom sheets | `16px` top corners only |
| Circular (avatar, icon buttons) | `50%` |

### Borders
- Default card border: `1px solid #E8E0D0` (`brand.subtle`)
- Input border (default): `1px solid #E8E0D0`
- Input border (focus): `2px solid #1A1A1A`
- Input border (error): `1px solid #C0392B`

### Shadows (elevation system)
```
shadow.sm:  0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04)   → cards default
shadow.md:  0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.05)   → cards hover, dropdown menus
shadow.lg:  0 10px 15px rgba(0,0,0,0.08), 0 4px 6px rgba(0,0,0,0.05) → modals, drawers
shadow.xl:  0 20px 25px rgba(0,0,0,0.09), 0 8px 10px rgba(0,0,0,0.04) → sticky header on scroll
```

---

## 6. Component Specifications

### 6.1 Button

| Variant | Background | Text | Border | Hover | Min height |
|---|---|---|---|---|---|
| Primary | `brand.primary` (#1A1A1A) | `text.inverse` | none | `#333333` bg | 48px mobile / 44px desktop |
| Secondary | transparent | `brand.primary` | `1px solid #1A1A1A` | `brand.warm` bg | same |
| Accent | `brand.accent` (#C8A96E) | `brand.primary` | none | `#B8980E` bg | same |
| Ghost | transparent | `text.secondary` | none | `brand.warm` bg | same |
| Destructive | `semantic.error` | `text.inverse` | none | `#A93226` bg | same |
| Disabled (all) | `#E8E0D0` | `text.muted` | none | no change, cursor-not-allowed | same |

Padding: `16px 24px` (default), `12px 20px` (sm), `20px 32px` (lg)
Font: Inter Semibold, 14px (sm) / 16px (default) / 18px (lg)
Transition: `background-color 150ms ease, transform 100ms ease`

**Loading state:** Spinner icon replaces label text. Button stays same width (no layout shift).
**Focus ring:** `2px solid #C8A96E`, offset `2px` (brand accent, not browser default blue)

---

### 6.2 ProductCard

Dimensions: width 100% of grid column, image aspect-ratio `1/1` (square), card height auto.

```
┌─────────────────────┐
│                     │  ← Image (aspect-ratio: 1/1, object-fit: cover)
│      [image]    [♡] │  ← Wishlist icon: top-right, 32px tap target
│                     │
│ [-17%]              │  ← Discount badge: top-left, only if sale
├─────────────────────┤
│ Product Name        │  ← Inter Semibold 16px, line-clamp-2
│ ₹49,999  ~~₹59,999~~│  ← PriceDisplay: accent for sale, muted strikethrough
│ ⚠ Only 3 left       │  ← Warning colour, only if qty <= 3
└─────────────────────┘
```

Card padding: `0` (image edge-to-edge), text area `12px`
Card border-radius: `12px`
Card border: `1px solid #E8E0D0`
Card shadow (default): `shadow.sm`
Card shadow (hover): `shadow.md` + `translateY(-4px)` (NO image zoom)
Transition: `transform 200ms ease, box-shadow 200ms ease`

---

### 6.3 Input / Form Fields

Height: `48px` (mobile), `44px` (desktop)
Border: `1px solid #E8E0D0` → focus: `2px solid #1A1A1A`
Border-radius: `8px`
Background: `#FFFFFF`
Padding: `12px 16px`
Label: Inter Medium 14px, `text.primary`, margin-bottom `6px`
Error message: Inter 13px, `semantic.error`, margin-top `4px`
Placeholder: `text.muted`

**States:**
- Default: subtle border
- Focus: dark border, no shadow
- Filled (has value): same as default
- Error: red border + error message below
- Disabled: `brand.warm` background, `text.muted` text, `cursor-not-allowed`

---

### 6.4 CartDrawer / Bottom Sheet

**Desktop (≥ md breakpoint):**
- Slides in from RIGHT
- Width: `420px` fixed
- Full viewport height
- Overlay: `rgba(0,0,0,0.3)` backdrop

**Mobile (< md breakpoint):**
- Slides UP from bottom
- Width: `100vw`
- Height: `85vh` max, auto min
- Top corners: `border-radius: 16px`
- Drag handle: `40px × 4px`, `brand.subtle` colour, centered, `8px` from top

Transition: `transform 300ms cubic-bezier(0.32, 0.72, 0, 1)`
Z-index: `50`

---

### 6.5 Header

Height: `64px` desktop, `56px` mobile
Background: `#FFFFFF`
Border-bottom: `1px solid #E8E0D0`
Box-shadow on scroll: `shadow.xl`
Position: `sticky`, `top: 0`
Z-index: `40`

---

### 6.6 Badge

| Type | Background | Text | Use |
|---|---|---|---|
| Discount | `brand.accent` | `brand.primary` | "-17%" on ProductCard |
| New | `brand.primary` | `text.inverse` | "New" on recent products |
| Low stock | `semantic.warning` bg at 15% opacity | `semantic.warning` | "Only 3 left" |
| Out of stock | `brand.subtle` | `text.muted` | "Out of Stock" |
| Sale | `semantic.error` bg at 10% opacity | `semantic.error` | "Sale" tag |

Size: `border-radius: 4px`, padding `4px 8px`, font Inter Medium 12px

---

## 7. Interaction & Animation Rules

### The Minimal Animation Principle
Every animation must serve a functional purpose. If removing it doesn't confuse the user, remove it.

### Approved Animations

| Interaction | Animation | Duration | Easing |
|---|---|---|---|
| Card hover | `translateY(-4px)` + shadow increase | `200ms` | `ease` |
| Button hover | background-color change | `150ms` | `ease` |
| Button click | `scale(0.98)` | `100ms` | `ease` |
| Cart drawer open | slide in | `300ms` | `cubic-bezier(0.32, 0.72, 0, 1)` |
| Cart badge increment | `scale(1.3)` → `scale(1.0)` | `200ms` | `spring` |
| Skeleton → content | `opacity 0 → 1` | `200ms` | `ease` |
| Toast notification | slide in from top-right | `250ms` | `ease-out` |
| Search overlay | `opacity 0 → 1` | `150ms` | `ease` |

### Explicitly Forbidden Animations
- Image zoom on card hover (scale-110) — feels cheap
- Page transition animations — unnecessary, adds latency perception
- Parallax scrolling effects — distract from products
- Bouncing or elastic animations outside of micro-interactions
- Continuous/looping animations anywhere in the purchase path

---

## 8. Layout Grid

### Page Grid
```
Mobile (< 768px):  1 column, 24px gutter, 16px page margins
Tablet (768-1024px): 12-column, 24px gutter, 32px page margins
Desktop (> 1024px): 12-column, 32px gutter, 48px page margins, max-width 1280px
```

### Product Grids
| Breakpoint | Columns | Gap |
|---|---|---|
| Mobile (< 640px) | 2 | 12px |
| Tablet (640–1024px) | 3 | 16px |
| Desktop (> 1024px) | 4 (PLP) / 3 (related products) | 24px |

### Breakpoints (Tailwind defaults, mobile-first)
```
sm:  640px   → tablet starts
md:  768px   → desktop nav shows, cart slide-in (vs bottom sheet)
lg:  1024px  → 4-col product grid
xl:  1280px  → max-width container
```

---

## 9. Icon System

Library: **Lucide React** (`lucide-react`) — already in the React skill set.
Size: `20px` default, `16px` small (within text), `24px` large (primary actions).
Stroke width: `1.5px` (matches Playfair Display's elegance).
Colour: inherit from parent text colour.

Key icons used:
```
ShoppingCart  → cart icon in header
Heart         → wishlist toggle
Search        → search icon
User          → account icon
Menu          → hamburger (mobile)
X             → close buttons
ChevronRight  → breadcrumb separator, accordion expand
Star          → (Phase 2: reviews)
Truck         → delivery estimate
Shield        → trust signals
```

---

## 10. Z-Index Scale

```
z-10:   Sticky header scroll shadow effect (not position)
z-20:   Dropdown menus, tooltips
z-30:   Sticky "Add to Cart" bar on PDP mobile
z-40:   Header (sticky)
z-50:   CartDrawer, SearchOverlay, MobileMenu
z-60:   Modal dialogs (checkout steps if modal)
z-70:   Toast notifications
z-80:   (reserved)
z-90:   (reserved)
z-100:  Loading overlays
```

---

## 11. Accessibility Baseline

Required on all interactive components:
- Focus visible: `outline: 2px solid #C8A96E; outline-offset: 2px`
- Minimum touch target: `44px × 44px` on mobile
- ARIA labels on icon-only buttons (cart, wishlist, search icons)
- Form fields: `id` + `htmlFor` pairing for all label-input pairs
- Images: meaningful `alt` text (never empty for product images)
- Colour contrast: minimum 4.5:1 for text on backgrounds (WCAG AA)
- Skeleton screens must have `aria-busy="true"` on the container

---

## 12. Relationship to Other Files

| This file defines | PREFERENCES.md holds | REJECTIONS.md blocks |
|---|---|---|
| Default specs (fixed at project start) | Owner overrides (added as built) | Approaches already tried and rejected |
| What components look like by default | Personal taste that diverges from defaults | Never-again patterns |
| Token values for each component | Changes to token values | Specific visual choices to never repeat |

**If there is a conflict between this file and PREFERENCES.md, PREFERENCES.md wins.**
That is the entire purpose of PREFERENCES.md — to let the owner personalise without editing specs.

---

*Owner: Product + Engineering | Shree Furniture | v1.0 — Q1 2026*
*Place at: `NewDocs/13-design-system.md`*