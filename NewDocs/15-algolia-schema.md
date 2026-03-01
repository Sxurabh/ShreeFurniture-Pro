# 15 — Algolia Index Schema & Search Configuration
## Shree Furniture | Search-as-a-Service Specification

> **Version:** 1.0 | **Status:** Active | **Phase:** 1 (MVP)
>
> **AI Agents:** Read this before building `backend/src/subscribers/algolia-sync.ts`,
> `apps/storefront/lib/algolia/client.ts`, or any component in `components/search/`.
> This document defines exactly what gets indexed, how it's structured, and how the search UI
> should be configured. Do not invent your own schema.
>
> **Phase note:** This project uses Algolia for MVP. Phase 2 may migrate to self-hosted MeiliSearch.
> See `STACK-CHANGES.md` before starting — if an entry there replaces Algolia, follow that instead.

---

## 1. Environment Variables

| Variable | Location | Public? | Purpose |
|---|---|---|---|
| `NEXT_PUBLIC_ALGOLIA_APP_ID` | Storefront `.env.local` | ✅ Yes | Identifies your Algolia application |
| `NEXT_PUBLIC_ALGOLIA_SEARCH_KEY` | Storefront `.env.local` | ✅ Yes | Read-only search key (never the admin key) |
| `ALGOLIA_WRITE_API_KEY` | Backend `.env` | ❌ No | Write-access key for sync subscriber only |

> The search key (`NEXT_PUBLIC_ALGOLIA_SEARCH_KEY`) is a **search-only** API key with no write permissions.
> The admin/write key must stay server-side only. Never use the admin key on the frontend.

---

## 2. Index Name

| Environment | Index Name |
|---|---|
| Production | `shree_furniture_products` |
| Staging | `shree_furniture_products_staging` |
| Development | `shree_furniture_products_dev` |

> Use an environment variable to select the index name:
> ```typescript
> const INDEX_NAME = process.env.ALGOLIA_INDEX_NAME ?? "shree_furniture_products_dev"
> ```

---

## 3. Index Record Schema

Each Algolia record corresponds to one **product variant** (not the product parent).
This approach allows faceting by colour, size, and price at the variant level without
requiring frontend logic to explode parent products.

```typescript
// The shape of each document pushed to Algolia
interface AlgoliaProductRecord {
  // === Primary ID ===
  objectID: string              // REQUIRED by Algolia — use variant.id (e.g. "variant_01ABCDEF")

  // === Product-level fields ===
  product_id: string            // Medusa product ID
  product_title: string         // e.g. "Oslo 3-Seater Sofa"
  product_handle: string        // e.g. "oslo-3-seater-sofa" — used to build PDP URL
  product_description: string   // Plain text, truncated to 500 chars
  collection_id: string | null
  collection_title: string | null   // e.g. "Living Room" — used as category facet
  collection_handle: string | null  // e.g. "living-room"
  thumbnail: string | null          // Cloudinary URL of primary product image
  tags: string[]                    // e.g. ["scandinavian", "fabric", "grey"]

  // === Variant-level fields ===
  variant_id: string            // same as objectID
  variant_title: string         // e.g. "Grey / L"
  sku: string | null            // e.g. "SF-SOFA-OSLO-GRY-L"
  inventory_quantity: number    // current stock at time of sync
  in_stock: boolean             // inventory_quantity > 0

  // === Price (paise integers) ===
  price_amount: number          // paise — e.g. 4999900 for ₹49,999
  price_inr: number             // divide by 100 for display — kept for Algolia numeric filters

  // === Facet fields (must be strings for Algolia faceting) ===
  room_type: string | null      // e.g. "Living Room", "Bedroom", "Dining", "Office"
  material: string | null       // e.g. "Fabric", "Solid Wood", "Metal", "MDF"
  colour: string | null         // e.g. "Grey", "Walnut", "White", "Natural"
  style: string | null          // e.g. "Scandinavian", "Industrial", "Classic", "Minimalist"
  assembly_required: boolean    // true/false — used as filter on PLP

  // === Timestamps (for ranking) ===
  created_at: number            // Unix timestamp (seconds) — used for "Newest" sort
  updated_at: number            // Unix timestamp (seconds)
}
```

### Field Notes

**`objectID` = `variant.id`:** Algolia requires a unique `objectID` per record. Using the variant ID makes
upserts in the sync subscriber straightforward — `saveObject()` with the same `objectID` overwrites.

**`product_handle` for PDP links:** Build the storefront URL as `/products/${product_handle}` from this field.
Do not store the full URL — it would need updating if the domain changes.

**Price stored twice:** `price_amount` (paise) is used for exact arithmetic; `price_inr` (rupees, float) enables
Algolia's numeric range filter UI without requiring the frontend to divide by 100 in filter expressions.

**Inventory at sync time:** `inventory_quantity` and `in_stock` reflect stock at the moment of indexing,
not real-time. This is acceptable for search — users see real stock on the PDP.

---

## 4. Sync Subscriber

The subscriber runs on Medusa events and pushes records to Algolia.

```typescript
// backend/src/subscribers/algolia-sync.ts
import algoliasearch from "algoliasearch"
import { SubscriberConfig, SubscriberArgs } from "@medusajs/framework"

const algoliaClient = algoliasearch(
  process.env.ALGOLIA_APP_ID!,
  process.env.ALGOLIA_WRITE_API_KEY!
)
const index = algoliaClient.initIndex(process.env.ALGOLIA_INDEX_NAME!)

// Events that trigger a re-index
export const config: SubscriberConfig = {
  event: [
    "product.created",
    "product.updated",
    "product.deleted",
    "product-variant.created",
    "product-variant.updated",
    "product-variant.deleted",
  ],
}

export default async function algoliaSync({
  event,
  container,
}: SubscriberArgs<{ id: string }>) {
  const productService = container.resolve("productService")

  if (event.name === "product.deleted") {
    // Remove all variants for this product from the index
    // Algolia doesn't support deleteByFilter on free tier — fetch variants first
    const { hits } = await index.search("", {
      filters: `product_id:${event.data.id}`,
      attributesToRetrieve: ["objectID"],
    })
    const objectIDs = hits.map((h) => h.objectID)
    if (objectIDs.length > 0) await index.deleteObjects(objectIDs)
    return
  }

  // Fetch full product from Medusa
  const product = await productService.retrieveProduct(event.data.id, {
    relations: ["variants", "variants.prices", "images", "collection", "tags"],
  })

  if (product.status !== "published") {
    // Draft/archived products: remove from index if they were previously indexed
    const { hits } = await index.search("", {
      filters: `product_id:${product.id}`,
      attributesToRetrieve: ["objectID"],
    })
    if (hits.length > 0) await index.deleteObjects(hits.map((h) => h.objectID))
    return
  }

  // Build one Algolia record per variant
  const records: AlgoliaProductRecord[] = product.variants.map((variant) => {
    const inrPrice = variant.prices.find((p) => p.currency_code === "inr")
    return {
      objectID: variant.id,
      product_id: product.id,
      product_title: product.title,
      product_handle: product.handle,
      product_description: product.description?.slice(0, 500) ?? "",
      collection_id: product.collection?.id ?? null,
      collection_title: product.collection?.title ?? null,
      collection_handle: product.collection?.handle ?? null,
      thumbnail: product.thumbnail ?? null,
      tags: product.tags.map((t) => t.value),
      variant_id: variant.id,
      variant_title: variant.title,
      sku: variant.sku ?? null,
      inventory_quantity: variant.inventory_quantity,
      in_stock: variant.inventory_quantity > 0,
      price_amount: inrPrice?.amount ?? 0,
      price_inr: (inrPrice?.amount ?? 0) / 100,
      room_type: extractMetadata(product, "room_type"),
      material: extractMetadata(product, "material"),
      colour: extractOptionValue(variant, "Colour"),
      style: extractMetadata(product, "style"),
      assembly_required: product.metadata?.assembly_required === true,
      created_at: Math.floor(new Date(product.created_at).getTime() / 1000),
      updated_at: Math.floor(new Date(product.updated_at).getTime() / 1000),
    }
  })

  // Upsert all records (saveObjects handles both create and update)
  await index.saveObjects(records)

  // Trigger ISR revalidation on the storefront
  // See NewDocs/MEDUSA-V2-PATTERNS.md § ISR + Revalidation Pattern
  await fetch(`${process.env.STOREFRONT_URL}/api/revalidate`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      secret: process.env.REVALIDATION_SECRET,
      path: `/products/${product.handle}`,
    }),
  })
}

// Helpers for Medusa product metadata
function extractMetadata(product: any, key: string): string | null {
  return product.metadata?.[key] ?? null
}

function extractOptionValue(variant: any, optionName: string): string | null {
  return variant.options?.find((o: any) => o.option?.title === optionName)?.value ?? null
}
```

---

## 5. Algolia Index Settings

Run this **once** during initial setup (or when recreating the index). This is the canonical
configuration — any dashboard changes should also be reflected here.

```typescript
// scripts/algolia-setup.ts — run with: npx ts-node scripts/algolia-setup.ts
import algoliasearch from "algoliasearch"

const client = algoliasearch(process.env.ALGOLIA_APP_ID!, process.env.ALGOLIA_WRITE_API_KEY!)
const index = client.initIndex(process.env.ALGOLIA_INDEX_NAME!)

await index.setSettings({
  // === Searchable fields (order = ranking priority) ===
  searchableAttributes: [
    "product_title",                      // Rank 1: exact product name matches
    "tags",                               // Rank 2: tag matches (e.g. "scandinavian")
    "collection_title",                   // Rank 3: collection/category name
    "variant_title",                      // Rank 4: colour/size variant name
    "product_description",                // Rank 5: description keyword matches
    "sku",                                // Rank 6: exact SKU lookup
  ],

  // === Faceting (available for filter panel on PLP + search page) ===
  attributesForFaceting: [
    "filterOnly(product_id)",             // Group variants by product (UI dedup)
    "searchable(collection_title)",       // "Living Room", "Bedroom", etc.
    "searchable(room_type)",              // Same as collection but from metadata
    "searchable(material)",              // "Fabric", "Solid Wood", "Metal", "MDF"
    "searchable(colour)",               // "Grey", "Walnut", "White", "Natural"
    "searchable(style)",                // "Scandinavian", "Industrial", etc.
    "filterOnly(in_stock)",              // Boolean: show only in-stock items
    "filterOnly(assembly_required)",     // Boolean: filter by assembly requirement
  ],

  // === Numeric attributes for range filters (price slider) ===
  numericAttributesForFiltering: ["price_inr", "price_amount", "inventory_quantity"],

  // === Attributes to retrieve (return to the frontend) ===
  // Only fetch what SearchResults.tsx needs — reduces response size
  attributesToRetrieve: [
    "objectID",
    "product_id",
    "product_title",
    "product_handle",
    "thumbnail",
    "collection_title",
    "collection_handle",
    "variant_id",
    "variant_title",
    "price_amount",
    "price_inr",
    "in_stock",
    "inventory_quantity",
    "colour",
    "material",
    "room_type",
  ],

  // === Attributes for snippet highlighting in search results ===
  attributesToSnippet: ["product_description:20"],
  attributesToHighlight: ["product_title", "tags", "collection_title"],

  // === Ranking ===
  ranking: [
    "typo",
    "geo",
    "words",
    "filters",
    "proximity",
    "attribute",
    "exact",
    "custom",
  ],
  customRanking: [
    "desc(in_stock)",       // In-stock items before out-of-stock
    "desc(updated_at)",     // More recently updated products first
  ],

  // === Typo tolerance ===
  minWordSizefor1Typo: 4,   // Allow 1 typo for words >= 4 chars (e.g. "soaf" → "sofa")
  minWordSizefor2Typos: 8,  // Allow 2 typos for words >= 8 chars
  typoTolerance: true,

  // === Query behaviour ===
  removeStopWords: true,
  ignorePlurals: true,        // "sofas" matches "sofa"
  queryLanguages: ["en"],

  // === Pagination ===
  hitsPerPage: 12,            // Matches PLP page size (see NewDocs/01-product-requirements.md)
  paginationLimitedTo: 1000,  // Algolia cap for offset pagination
})
```

---

## 6. Virtual Replicas (Sorting)

Algolia replicas power the sort dropdown on PLP. Create these in the dashboard
and configure them with `setSettings()` on each replica:

| Replica Index Name | Sort By | Direction | Label in UI |
|---|---|---|---|
| `shree_furniture_products_price_asc` | `price_inr` | Ascending | "Price: Low to High" |
| `shree_furniture_products_price_desc` | `price_inr` | Descending | "Price: High to Low" |
| `shree_furniture_products_newest` | `created_at` | Descending | "Newest" |

> The primary index (`shree_furniture_products`) serves as the "Most Relevant" (default) sort.

---

## 7. Storefront: Algolia Client Initialisation

```typescript
// apps/storefront/lib/algolia/client.ts
import { liteClient as algoliasearch } from "algoliasearch/lite"

// Use liteClient for the storefront — smaller bundle size than full client
export const searchClient = algoliasearch(
  process.env.NEXT_PUBLIC_ALGOLIA_APP_ID!,
  process.env.NEXT_PUBLIC_ALGOLIA_SEARCH_KEY!
)

export const ALGOLIA_INDEX = process.env.NEXT_PUBLIC_ALGOLIA_INDEX_NAME ?? "shree_furniture_products"
```

---

## 8. Storefront: InstantSearch Setup

Use `react-instantsearch` (not the deprecated `react-instantsearch-hooks-web`).

```tsx
// apps/storefront/app/(store)/search/page.tsx
import { InstantSearchNext } from "react-instantsearch-nextjs"
import { SearchBox, Hits, RefinementList, RangeInput, SortBy } from "react-instantsearch"
import { searchClient, ALGOLIA_INDEX } from "@/lib/algolia/client"

export default function SearchPage() {
  return (
    <InstantSearchNext indexName={ALGOLIA_INDEX} searchClient={searchClient}>
      {/* Search input (debounced 300ms — see product requirements) */}
      <SearchBox queryHook={(query, refine) => {
        clearTimeout(searchTimeout)
        searchTimeout = setTimeout(() => refine(query), 300)
      }} />

      {/* Facet filters — matches filter panel spec in NewDocs/01 */}
      <RefinementList attribute="collection_title" />
      <RefinementList attribute="material" />
      <RefinementList attribute="colour" />
      <RefinementList attribute="style" />
      <RangeInput attribute="price_inr" />  {/* ₹ range slider */}

      {/* Sort dropdown — uses replicas defined in §6 */}
      <SortBy items={[
        { value: ALGOLIA_INDEX,                              label: "Most Relevant" },
        { value: `${ALGOLIA_INDEX}_price_asc`,               label: "Price: Low to High" },
        { value: `${ALGOLIA_INDEX}_price_desc`,              label: "Price: High to Low" },
        { value: `${ALGOLIA_INDEX}_newest`,                  label: "Newest" },
      ]} />

      {/* Results */}
      <Hits hitComponent={SearchHitCard} />
    </InstantSearchNext>
  )
}
```

---

## 9. Deduplication: One ProductCard per Product

Since records are indexed per-variant, search results may return multiple records
for the same product (e.g., Oslo Sofa in Grey and White).

**Strategy:** Use Algolia's distinct feature to deduplicate by `product_id`:

```typescript
// Add to index settings (§5 setSettings call):
distinct: true,
attributeForDistinct: "product_id",
```

With `distinct: true`, Algolia returns at most one variant per `product_id`. The `customRanking`
(`desc(in_stock)`, `desc(updated_at)`) determines which variant is the representative record.

---

## 10. Common Mistakes — Do Not Repeat

| ❌ Mistake | ✅ Correct Approach |
|---|---|
| Using Algolia admin key on the frontend | Always `NEXT_PUBLIC_ALGOLIA_SEARCH_KEY` (read-only) |
| Indexing draft/archived products | Check `product.status === "published"` before indexing |
| Storing price in rupees only | Store both `price_amount` (paise) and `price_inr` (rupees) |
| Using product ID as `objectID` | Use variant ID — one record per variant, not per product |
| Forgetting to trigger ISR revalidation | Always call `/api/revalidate` after algolia sync |
| Hardcoding index name | Use env var `ALGOLIA_INDEX_NAME` |

---

*Shree Furniture | v1.0 — Q1 2026 | Place at: `NewDocs/15-algolia-schema.md`*
