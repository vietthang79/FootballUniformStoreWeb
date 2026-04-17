---
phase: 5
title: "Admin Preview Component"
status: pending
priority: P1
effort: 1d
---

# Phase 5: Admin Preview Component

## Context
- Depends on Phase 4 (builder state: printConfig + player list + logo URLs)
- Used by Phase 8 (Admin Dashboard) to display custom order design
<!-- Updated: 2026-04-13 - Removed server-side canvas render. Admin preview uses client-side CSS overlay. -->
<!-- Updated Session 10 (2026-04-17): dynamic tabs (one per ProductImage, not fixed 4 angles). Readonly mode of `react-moveable` canvas (no drag handles rendered). -->

## Overview
**`<MockupCanvas mode="readonly" />`** đã được xây dựng ở Phase 4. Phase này tích hợp component đó vào trang admin order detail (`/admin/orders/[id]`), thêm player table + logo download link.

Phase 5 KHÔNG tạo thêm component mới — chỉ tích hợp Phase 4 deliverable vào admin UI.

## Architecture

```
/admin/orders/[id]/page.tsx
  ├── Order info (customer, address, totals, status updater)
  └── [if custom order]
      ├── <MockupCanvas                   ← built in Phase 4
      │     mode="readonly"
      │     images={ProductImage[]}       ← from ColorSet, ordered by sortOrder (Session 10: dynamic)
      │     config={printConfigJson}      ← overlays: OverlayElement[][]
      │   />
      │   ImageTabSwitcher inside: dynamic tabs per ProductImage (Session 10 — not fixed 4 angles)
      │   Labels from ProductImage.label or fallback "Ảnh N"
      │   Player selector: chọn cầu thủ → hiện tên/số trên màn hình
      ├── Player table (tên, số áo, size) — single size field (Session 8)
      └── Logos download: multi-logo support via printConfigJson.logos[] array
```

## Related Code Files

### Reuse from Phase 4
- `src/components/custom-builder/mockup-canvas.tsx` — **already built**, use `mode="readonly"` (disable react-moveable handles)
- `src/components/custom-builder/image-tab-switcher.tsx` — **already built**, dynamic tabs (Session 10), reuse as-is

### Create
- `src/components/admin/custom-order-detail.tsx` — wraps MockupCanvas readonly + player table + logo link

### Modify
- `src/app/admin/orders/[id]/page.tsx` — integrate CustomOrderDetail component

## Implementation Steps

1. Build `custom-order-detail.tsx`:
   - Props: `customOrder` (with `printConfigJson`, players), `colorSet` (with ordered `ProductImage[]`)
   - Render `<MockupCanvas mode="readonly" />` with `images` (sorted by sortOrder) + `config` from `printConfigJson`
   - Render player table (sortOrder, playerName, playerNumber, size — single field Session 8)
   - Render logos download list from `printConfigJson.logos[]` (multi-logo support Session 6)
2. Integrate into `/admin/orders/[id]/page.tsx` — query CustomOrder + Players + ColorSet + ProductImages (order by sortOrder)

## Todo
- [ ] `custom-order-detail.tsx`: readonly preview + player table + logo link
- [ ] Integrate into admin order detail page
- [ ] Test with sample custom order data (verify positions match builder output)

## Success Criteria
- Admin order detail shows mockup preview identical to what customer saw in builder
- Dynamic tabs: one per ProductImage (2-10 depending on product), labels preserved (Session 10)
- Player table displays all players with single size field
- All logos from `printConfigJson.logos[]` downloadable
- No new CSS overlay logic needed — fully reuses Phase 4 MockupCanvas in readonly mode

## Risk
- `printConfigJson` structure must match `CustomConfig` type from Phase 4 exactly → share type definitions
- Coordinate mapping: positions stored as % → renders correctly at different admin screen sizes
