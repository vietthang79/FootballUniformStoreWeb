---
phase: 2
title: "Product Catalog + Normal E-Commerce"
status: pending
priority: P1
effort: 5d
---

# Phase 2: Product Catalog + Normal E-Commerce Flow

## Context
- Depends on Phase 1 (DB + layout ready)
- [Brainstorm](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md) — Luồng 1: Mua thường

## Overview
Product listing, detail page, size/color selection, cart (Zustand), checkout page (without payment).

## Key Insights
- Products with `customizable: true` show [Custom] button alongside [Add to Cart]
- Cart = client-side Zustand + localStorage, no server cart
- Checkout form collects shipping info, payment method selection (UI only, integration in Phase 6)

## Architecture

```
/                       → Homepage (featured products + categories)
/products               → Product listing with filters (category, price)
/products/[slug]        → Product detail (images, sizes, color sets, add to cart / custom)
/cart                   → Cart page (normal items + custom bundles)
/checkout               → Checkout form (shipping + payment method)
```

## Related Code Files

### Create
- `src/app/page.tsx` — homepage with hero + featured products
- `src/app/products/page.tsx` — listing with filters
- `src/app/products/[slug]/page.tsx` — product detail (RSC)
- `src/app/cart/page.tsx` — cart page
- `src/app/checkout/page.tsx` — checkout form
- `src/components/product/product-card.tsx` — card with [Custom] badge
- `src/components/product/product-gallery.tsx` — image gallery with color set switching
- `src/components/product/size-selector.tsx` — size picker
- `src/components/product/color-set-selector.tsx` — color set picker (image swatches)
- `src/components/cart/cart-icon.tsx` — header cart icon with count
- `src/components/cart/cart-item-row.tsx` — single cart item
- `src/components/checkout/checkout-form.tsx` — shipping + payment form
- `src/stores/cart-store.ts` — Zustand cart store
- `src/app/api/products/route.ts` — product listing API (if needed for client filtering)

## Cart Store Design (Zustand)

```typescript
// Two item types in cart:
interface NormalCartItem {
  type: 'normal'
  id: string
  productId: string
  productName: string
  productImage: string
  colorSetName: string
  size: string
  quantity: number
  unitPrice: number
}

interface CustomCartItem {
  type: 'custom'
  id: string
  // Full custom builder state — defined in Phase 3
  builderData: CustomBuilderData
  summary: string // "Đội ABC FC — 12 bộ"
  totalPrice: number
}

type CartItem = NormalCartItem | CustomCartItem
```

## Implementation Steps

1. Create `src/stores/cart-store.ts` — Zustand with persist middleware
2. Build product listing page — RSC fetching from Prisma, category filter
3. Build product card component — image, name, price, [Add to Cart], [Custom] badge
4. Build product detail page — gallery, color set switch, size selector, add to cart
5. Build cart page — list items, quantity +/-, remove, subtotal
6. Build cart icon in header — item count badge
7. Build checkout page — React Hook Form, shipping fields, payment method radio
8. Add category navigation in header/sidebar
9. Responsive design for all pages

## Todo
- [ ] Zustand cart store with persist
- [ ] Homepage with featured products grid
- [ ] Product listing page with category filter
- [ ] Product card component
- [ ] Product detail page with gallery
- [ ] Color set selector (swap images on pick)
- [ ] Size selector
- [ ] Add to cart functionality
- [ ] Cart page with quantity controls
- [ ] Cart icon with badge in header
- [ ] Checkout form (shipping info)
- [ ] Responsive layout for mobile

## Success Criteria
- Browse products → view detail → select size/color → add to cart → see in cart
- Cart persists across page reload (localStorage)
- Checkout form validates and collects all required fields
- [Custom] button visible only on customizable products
- Mobile responsive

## Risk
- Image loading performance → use Next.js Image component with Cloudinary loader
- Cart state migration if schema changes → use Zustand version + migrate
