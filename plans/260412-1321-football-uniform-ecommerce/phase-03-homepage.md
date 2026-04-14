---
phase: 3
title: "Homepage (Marketing Landing Page)"
status: pending
priority: P1
effort: 2d
---

# Phase 3: Homepage (Marketing Landing Page)

## Context
- Depends on Phase 1 (DB for featured products query) + Phase 2 (auth for nav header user state)
- Brand colors: Vàng #FDD017 (primary), Đỏ #E31E26 (secondary), Xanh dương #00AEEF (accent)
- Created: Session 8 brainstorm — "làm mới hoàn toàn, không dùng index.html làm reference"

## Overview
Marketing landing page tại `/`. Server-fetched featured products from DB. Static sections (features, testimonials) hardcoded. Nav header reads auth session for user state (login/logout).

## Page Sections

```
/  (Homepage)
│
├── [Nav Header]      — from Phase 1 base layout; shows user name or login button
│
├── [Hero]            — image carousel/slider (2-3 images)
│                       shadcn Carousel (wraps embla-carousel)
│                       Headline + CTA button → /products
│
├── [Features]        — 3-4 key service features (icon + title + description)
│                       e.g.: "Tự thiết kế mockup", "In chất lượng cao",
│                             "Giao hàng toàn quốc", "Hỗ trợ đội nhỏ từ 6 bộ"
│
├── [Featured Products] — 6-8 products fetched from DB (server component)
│                         Filter: active=true, order by createdAt desc (or featured flag later)
│                         Uses <ProductCard /> from Phase 4
│
├── [Testimonials]    — 3-5 hardcoded Vietnamese reviews (shop edits later via code)
│                       Star rating + customer name (first name only) + short quote
│
└── [Footer]          — from Phase 1 base layout
```

## Architecture

```
src/app/page.tsx  (RSC — server component)
  ├── HeroCarousel          — client component (shadcn Carousel / embla-carousel)
  ├── FeaturesSection       — static, server component
  ├── FeaturedProductsSection — RSC, Prisma query: take 8 active products
  │   └── <ProductCard />   — reused from Phase 4
  └── TestimonialsSection   — static, hardcoded data, server component
```

## Related Code Files

### Create
- `src/app/page.tsx` — homepage RSC (update Phase 1 placeholder)
- `src/components/home/hero-carousel.tsx` — shadcn Carousel with 2-3 slides (image + headline + CTA)
- `src/components/home/features-section.tsx` — 3-4 feature cards (icon + title + desc)
- `src/components/home/featured-products-section.tsx` — server component, Prisma fetch + grid
- `src/components/home/testimonials-section.tsx` — hardcoded reviews grid

### Depends on (Phase 4 will create)
- `src/components/product/product-card.tsx` — if Phase 3 (catalog) not yet done, create a minimal version here and update in Phase 4

## Implementation Steps

1. Install embla-carousel if not already present: `npx shadcn@latest add carousel`
2. Create `hero-carousel.tsx` — `"use client"`, shadcn `<Carousel autoplay>` with 2-3 slides; each slide: full-width image (Next.js `<Image>`), overlay text (headline in Vietnamese), CTA button styled with brand Vàng #FDD017
3. Create `features-section.tsx` — grid of 3-4 cards; use lucide-react icons; brand accent colors; Vietnamese copy
4. Create `featured-products-section.tsx` — RSC; `prisma.product.findMany({ where: { active: true }, take: 8, orderBy: { createdAt: 'desc' } })`; render as responsive grid using `<ProductCard />`
5. Create `testimonials-section.tsx` — hardcoded array of 3-5 `{ name, quote, rating }` objects; render star rating + quote card; background: brand Đỏ #E31E26 or Vàng #FDD017 subtle tint
6. Update `src/app/page.tsx` — compose all sections in order; keep as RSC (hero is the only client island)
7. Verify responsive layout (mobile + desktop); check Montserrat font applied; brand colors consistent

## Todo
- [ ] Add shadcn Carousel component
- [ ] HeroCarousel (2-3 slides, CTA, brand colors)
- [ ] FeaturesSection (3-4 feature cards)
- [ ] FeaturedProductsSection (server fetch, 6-8 products)
- [ ] TestimonialsSection (hardcoded 3-5 Vietnamese reviews)
- [ ] Wire all sections into page.tsx
- [ ] Verify mobile responsive + brand colors

## Success Criteria
- Homepage renders at `/` with all 4 sections
- Hero carousel auto-plays and is responsive
- Featured products fetched from DB (6-8 items visible)
- Mobile responsive (all sections stack correctly on small screens)
- Brand colors (Vàng, Đỏ) visible and consistent with design guidelines
- Montserrat font applied

## Risk
- ProductCard not yet created (Phase 4 owns it) → create minimal version in this phase; Phase 4 will enrich it
- Placeholder images for hero until shop provides real photos → use Next.js placeholder or stock image URLs

## Security
- No user input on this page — no security concerns
- Featured products query uses server-side Prisma (no SQL injection risk)
