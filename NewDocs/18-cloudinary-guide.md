# 18 — Cloudinary Integration Guide
## Shree Furniture | Image Management & CDN

> **Version:** 1.0 | **Status:** Active
>
> **AI Agents:** Read this before building any component that renders product images,
> uploading images via the Medusa Admin, or configuring `next.config.ts` image domains.
> This document defines the naming conventions, transformation presets, and Next.js Image
> configuration for Cloudinary. Do not invent your own Cloudinary URL patterns.

---

## 1. Environment Variables

| Variable | Location | Public? | Purpose |
|---|---|---|---|
| `CLOUDINARY_CLOUD_NAME` | Backend `.env` | ❌ No | Your Cloudinary cloud name (e.g. `shree-furniture`) |
| `CLOUDINARY_API_KEY` | Backend `.env` | ❌ No | API key for server-side upload |
| `CLOUDINARY_API_SECRET` | Backend `.env` | ❌ No | API secret — never expose |
| `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` | Storefront `.env.local` | ✅ Yes | Read-only cloud name for constructing CDN URLs in components |

> The storefront **never** uploads to Cloudinary directly — all uploads go through Medusa Admin
> which uses the backend's Cloudinary plugin. The storefront only reads image URLs.

---

## 2. Folder Structure & Naming Convention

All product images follow a strict folder and naming structure. This ensures predictable URLs
and makes bulk operations (purging, migrating, auditing) straightforward.

```
shree-furniture/                          ← root folder (cloud name)
├── products/
│   └── {product_handle}/                 ← e.g. "oslo-3-seater-sofa"
│       ├── primary.webp                  ← Main product image (used as thumbnail)
│       ├── gallery-01.webp              ← Additional gallery images
│       ├── gallery-02.webp
│       ├── gallery-03.webp
│       └── gallery-04.webp              ← Max 5 gallery images per product for MVP
├── variants/
│   └── {variant_sku}/                   ← e.g. "SF-SOFA-OSLO-GRY-L"
│       └── swatch.webp                  ← Colour swatch image (optional per variant)
├── collections/
│   └── {collection_handle}/             ← e.g. "living-room"
│       └── cover.webp                   ← Collection cover image for Homepage grid
└── brand/
    ├── logo-dark.png                    ← Dark logo (for light backgrounds)
    ├── logo-light.png                   ← Light logo (for dark backgrounds)
    └── og-image.jpg                     ← Default Open Graph image (1200×630px)
```

### File naming rules

- Always use the `product_handle` (from Medusa) as the folder name — this keeps Cloudinary and Medusa in sync
- `primary.webp` is always the thumbnail — predictable URL without needing to store it
- Gallery images use `gallery-01` through `gallery-05` (not arbitrary names)
- Variant swatches use `swatch.webp` inside the variant SKU folder

---

## 3. Cloudinary Plugin Configuration (Backend)

```typescript
// backend/medusa-config.ts
import { defineConfig } from "@medusajs/framework/utils"

export default defineConfig({
  // ...
  plugins: [
    {
      resolve: "@medusajs/file-cloudinary",
      options: {
        cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
        api_key: process.env.CLOUDINARY_API_KEY,
        api_secret: process.env.CLOUDINARY_API_SECRET,
        folder: "shree-furniture",          // Root folder prefix
        use_filename: true,                 // Preserve uploaded filename
        unique_filename: false,             // Do not add random suffix — names are controlled
        overwrite: true,                    // Allow re-uploading same filename (useful for updates)
        resource_type: "auto",             // Auto-detect image format
      },
    },
  ],
})
```

---

## 4. Image Transformation Presets

Instead of calling Cloudinary's full transformation URL syntax ad-hoc in components,
always use these named presets. This ensures consistency and makes it easy to update
quality/format settings globally.

### Preset Reference

| Preset Name | Dimensions | Format | Quality | Use Case |
|---|---|---|---|---|
| `thumbnail` | 400×400 | WebP | auto | ProductCard grid images |
| `pdp_primary` | 800×800 | WebP | auto | PDP main large image |
| `pdp_zoom` | 1600×1600 | WebP | 90 | PDP pinch-to-zoom full resolution |
| `pdp_thumb_strip` | 120×120 | WebP | auto | PDP thumbnail strip below main image |
| `collection_cover` | 800×600 | WebP | auto | Homepage collection grid cards |
| `blur_placeholder` | 40×40 | WebP | 10 | Blur-up placeholder (base64 inline) |
| `og_image` | 1200×630 | JPEG | 85 | Open Graph / social sharing |

### Preset Implementation

```typescript
// apps/storefront/lib/cloudinary/presets.ts

const CLOUD = process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME

type Preset = "thumbnail" | "pdp_primary" | "pdp_zoom" | "pdp_thumb_strip" | "collection_cover" | "blur_placeholder" | "og_image"

const PRESET_PARAMS: Record<Preset, string> = {
  thumbnail:         "c_fill,w_400,h_400,f_webp,q_auto",
  pdp_primary:       "c_fill,w_800,h_800,f_webp,q_auto",
  pdp_zoom:          "c_fill,w_1600,h_1600,f_webp,q_90",
  pdp_thumb_strip:   "c_fill,w_120,h_120,f_webp,q_auto",
  collection_cover:  "c_fill,w_800,h_600,f_webp,q_auto",
  blur_placeholder:  "c_fill,w_40,h_40,f_webp,q_10",
  og_image:          "c_fill,w_1200,h_630,f_jpg,q_85",
}

/**
 * Transforms a raw Cloudinary URL or public_id into a preset URL.
 *
 * Accepts either:
 *   - A full Cloudinary URL: "https://res.cloudinary.com/shree-furniture/image/upload/..."
 *   - A public_id: "shree-furniture/products/oslo-3-seater-sofa/primary"
 */
export function cloudinaryUrl(
  sourceUrl: string,
  preset: Preset
): string {
  const params = PRESET_PARAMS[preset]

  // If already a full URL, inject transformation params after "/upload/"
  if (sourceUrl.startsWith("https://res.cloudinary.com")) {
    return sourceUrl.replace("/upload/", `/upload/${params}/`)
  }

  // Otherwise treat as public_id and build URL from scratch
  return `https://res.cloudinary.com/${CLOUD}/image/upload/${params}/${sourceUrl}`
}

/**
 * Returns a base64-encoded blur placeholder for use with Next.js Image blurDataURL.
 * Call this server-side during page render.
 */
export async function getBlurPlaceholder(sourceUrl: string): Promise<string> {
  const blurUrl = cloudinaryUrl(sourceUrl, "blur_placeholder")
  const response = await fetch(blurUrl)
  const buffer = await response.arrayBuffer()
  const base64 = Buffer.from(buffer).toString("base64")
  const mimeType = "image/webp"
  return `data:${mimeType};base64,${base64}`
}
```

---

## 5. Next.js Image Configuration

```typescript
// apps/storefront/next.config.ts
import type { NextConfig } from "next"

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "res.cloudinary.com",
        port: "",
        pathname: `/${process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME}/**`,
      },
    ],
    // Use Cloudinary's own CDN for image optimisation — disable Next.js image optimiser
    // to avoid double-optimising (Cloudinary does it better and charges less)
    loader: "custom",
    loaderFile: "./lib/cloudinary/loader.ts",
  },
}

export default nextConfig
```

```typescript
// apps/storefront/lib/cloudinary/loader.ts
// Custom Next.js image loader — uses Cloudinary transformations instead of Next.js built-in
import type { ImageLoaderProps } from "next/image"

export default function cloudinaryLoader({ src, width, quality }: ImageLoaderProps): string {
  // Map Next.js width requests to Cloudinary width param
  const q = quality ?? "auto"
  return src.replace("/upload/", `/upload/w_${width},q_${q},f_webp/`)
}
```

---

## 6. Component Usage Patterns

### ProductCard (thumbnail preset)

```tsx
// components/product/ProductCard.tsx
import Image from "next/image"
import { cloudinaryUrl } from "@/lib/cloudinary/presets"

interface ProductCardProps {
  product: Product
}

export function ProductCard({ product }: ProductCardProps) {
  const thumbnailUrl = product.thumbnail
    ? cloudinaryUrl(product.thumbnail, "thumbnail")
    : "/images/placeholder-product.webp"  // Local fallback in /public

  return (
    <div className="...">
      <div className="aspect-square relative">
        <Image
          src={thumbnailUrl}
          alt={product.title}
          fill
          sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"
          className="object-cover rounded-t-xl"
          // No priority here — cards are below the fold
        />
      </div>
      {/* ... card content */}
    </div>
  )
}
```

### ProductGallery (pdp_primary + pdp_zoom + pdp_thumb_strip)

```tsx
// components/product/ProductGallery.tsx
import Image from "next/image"
import { cloudinaryUrl, getBlurPlaceholder } from "@/lib/cloudinary/presets"

interface ProductGalleryProps {
  images: { url: string }[]
  productTitle: string
  blurDataURLs: string[]    // Pre-fetched server-side with getBlurPlaceholder()
}

export function ProductGallery({ images, productTitle, blurDataURLs }: ProductGalleryProps) {
  const [activeIndex, setActiveIndex] = useState(0)

  return (
    <div>
      {/* Primary large image */}
      <div className="aspect-square relative">
        <Image
          src={cloudinaryUrl(images[activeIndex].url, "pdp_primary")}
          alt={`${productTitle} — image ${activeIndex + 1}`}
          fill
          sizes="(max-width: 768px) 100vw, 50vw"
          priority           // IMPORTANT: above-fold PDP image must use priority
          placeholder="blur"
          blurDataURL={blurDataURLs[activeIndex]}
          className="object-cover rounded-xl"
        />
      </div>

      {/* Thumbnail strip */}
      <div className="flex gap-2 mt-3">
        {images.map((img, i) => (
          <button key={i} onClick={() => setActiveIndex(i)}>
            <Image
              src={cloudinaryUrl(img.url, "pdp_thumb_strip")}
              alt={`${productTitle} thumbnail ${i + 1}`}
              width={80}
              height={80}
              className={`rounded-lg object-cover border-2 ${i === activeIndex ? "border-brand-primary" : "border-brand-subtle"}`}
            />
          </button>
        ))}
      </div>
    </div>
  )
}
```

### Server-side blur placeholder (in page.tsx)

```typescript
// apps/storefront/app/(store)/products/[handle]/page.tsx
import { getBlurPlaceholder } from "@/lib/cloudinary/presets"

export default async function ProductPage({ params }) {
  const product = await getProduct(params.handle)
  if (!product) notFound()

  // Pre-fetch blur placeholders for all gallery images server-side
  // This happens at render time — no client-side delay
  const blurDataURLs = await Promise.all(
    product.images.map((img) => getBlurPlaceholder(img.url))
  )

  return (
    <ProductGallery
      images={product.images}
      productTitle={product.title}
      blurDataURLs={blurDataURLs}
    />
  )
}
```

---

## 7. Upload Restrictions

These limits are enforced by Cloudinary plugin configuration and should be communicated
to anyone uploading via Medusa Admin:

| Setting | Limit | Notes |
|---|---|---|
| Accepted formats | JPG, PNG, WebP | Cloudinary converts to WebP on delivery |
| Max file size | 10MB | Per `NewDocs/01-product-requirements.md §4.8` |
| Max gallery images per product | 5 | MVP limit — enforced in Admin documentation |
| Recommended resolution | Minimum 1600×1600px | Ensures zoom quality |
| Recommended aspect ratio | 1:1 (square) | Consistent grid display |

> Images uploaded at lower resolution will work but will appear soft at zoom level.
> Recommend using `pdp_zoom` URL in a test before publishing to confirm quality.

---

## 8. SEO & Open Graph Images

For Open Graph images (shared on social media), use the `og_image` preset:

```typescript
// apps/storefront/app/(store)/products/[handle]/page.tsx
import { cloudinaryUrl } from "@/lib/cloudinary/presets"

export async function generateMetadata({ params }) {
  const product = await getProduct(params.handle)

  return {
    openGraph: {
      title: product.title,
      images: [
        {
          url: cloudinaryUrl(product.thumbnail ?? "", "og_image"),
          width: 1200,
          height: 630,
          alt: product.title,
        },
      ],
    },
  }
}
```

---

## 9. Fallback & Error Handling

```typescript
// Always have a local fallback for missing images
// Place at: apps/storefront/public/images/placeholder-product.webp
// This is a neutral tan-coloured square matching brand.warm (#F5F0E8)

// In any Image component:
const src = product.thumbnail
  ? cloudinaryUrl(product.thumbnail, "thumbnail")
  : "/images/placeholder-product.webp"

// In Next.js <Image>:
<Image
  src={src}
  alt={product.title}
  onError={(e) => {
    // If Cloudinary URL fails, fall back to local placeholder
    e.currentTarget.src = "/images/placeholder-product.webp"
  }}
/>
```

---

## 10. Common Mistakes — Do Not Repeat

| ❌ Mistake | ✅ Correct Approach |
|---|---|
| Using `<img>` instead of `<Image>` for products | Always `next/image` `<Image>` — never raw `<img>` |
| Missing `width` and `height` on `<Image>` | Always explicit — prevents CLS (see CLAUDE.md) |
| Hardcoding Cloudinary cloud name in URLs | Use `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` env var |
| Using JPEG format directly | Use WebP via presets — significantly smaller file sizes |
| `priority` on below-fold images | `priority` only on above-fold (hero, PDP primary image) |
| Not using `sizes` prop on responsive images | Always set `sizes` to help browser choose correct srcset |
| Building raw Cloudinary transformation strings in components | Use `cloudinaryUrl(src, preset)` helper — never raw strings |
| Skipping blur placeholder on PDP | The blur-up pattern is required — images are large, load is slow |

---

*Shree Furniture | v1.0 — Q1 2026 | Place at: `NewDocs/18-cloudinary-guide.md`*
