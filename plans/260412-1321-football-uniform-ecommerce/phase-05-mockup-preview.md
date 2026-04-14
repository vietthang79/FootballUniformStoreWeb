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
      │     imagesByAngle={...}           ← from CustomOrder + ColorSet
      │     config={printConfigJson}      ← from CustomOrder
      │   />
      │   AngleTabSwitcher inside: [Mặt trước][Mặt sau][Tay áo][Quần]
      │   Player selector: chọn cầu thủ → hiện tên/số trên màn hình
      ├── Player table (tên, số áo, size)
      └── Logo download link (Cloudinary URL)
```

## Related Code Files

### Reuse from Phase 4
- `src/components/custom-builder/mockup-canvas.tsx` — **already built**, use `mode="readonly"`
- `src/components/custom-builder/angle-tab-switcher.tsx` — **already built**, reuse as-is

### Create
- `src/components/admin/custom-order-detail.tsx` — wraps MockupCanvas readonly + player table + logo link

### Modify
- `src/app/admin/orders/[id]/page.tsx` — integrate CustomOrderDetail component

## Implementation Steps

1. Build `custom-order-detail.tsx`:
   - Props: `customOrder` (with `printConfigJson`, logo URLs, players), `colorSet` (with images by angle)
   - Render `<MockupCanvas mode="readonly" />` with `imagesByAngle` + `config` from `printConfigJson`
   - Render player table (sortOrder, playerName, playerNumber, jerseySize, shortsSize)
   - Render logo download link if `logoUrl` exists
2. Integrate into `/admin/orders/[id]/page.tsx` — query CustomOrder + Players + ColorSet + ProductImages

## Todo
- [ ] `custom-order-detail.tsx`: readonly preview + player table + logo link
- [ ] Integrate into admin order detail page
- [ ] Test with sample custom order data (verify positions match builder output)

## Success Criteria
- Admin order detail shows mockup preview identical to what customer saw in builder
- All 4 angle tabs work (front/back/sleeve/shorts)
- Player table displays all players with sizes
- Logo URL is downloadable
- No new CSS overlay logic needed — fully reuses Phase 4 MockupCanvas

## Risk
- `printConfigJson` structure must match `CustomConfig` type from Phase 4 exactly → share type definitions
- Coordinate mapping: positions stored as % → renders correctly at different admin screen sizes
