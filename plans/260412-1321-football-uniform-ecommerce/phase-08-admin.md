---
phase: 8
title: "Admin + Order Management + Product Import"
status: pending
priority: P2
effort: 5d
---

# Phase 8: Admin + Order Management + Product Import

## Context
- Depends on Phase 6+7 (orders exist in DB)
- [Brainstorm: CSV Import Flow](../reports/brainstorm-260413-1508-csv-product-import.md)
<!-- Session 8: Images free-form (sortOrder). Product edit FULL scope (info + images + ColorSet + stock per variant). Status PATCH sends customer email. -->

## Overview
Admin panel gồm 2 phần:
1. **Order Management** — xem + filter + update status đơn hàng + send customer emails
2. **Product Import + Edit** — import sản phẩm từ CSV supplier; full edit capability (info, images, ColorSet, stock)

Protected by Better Auth admin role (Phase 2).

## Architecture

```
/admin                          → Dashboard (stats)
/admin/orders                   → Order list (filter, search, paginate)
/admin/orders/[id]              → Order detail + status update
/admin/products                 → Product list (link to edit/import)
/admin/products/import          → CSV Import wizard
/admin/products/[id]/edit       → Product edit (FULL: info + images + ColorSet + stock)
```

### CSV Import Flow
```
[1] Upload CSV (PapaParse — client-side)
        ↓
[2] Column Mapping UI
    CSV Col       → DB Field (auto-suggest by name fuzzy match)
    "ten_sp"      → name
    "gia_von"     → basePrice
    "mau_sac"     → colorSet.name
    "so_luong"    → stock
    (unmapped)    → field visible with empty input
    Note: Không cần cột ảnh — ảnh upload thủ công trong Step 4
        ↓
[3] Client-side grouping by product name (case-insensitive trim)
    → ProductDraft[]: 1 Product = N ColorSets
        ↓
[4] Preview Table (grouped, all fields shown, inline editable)
    ▼ Áo CLB A  [category] [description]  [☑ Cho phép tùy chỉnh mockup]  ← isCustomizable toggle
      Xanh Navy | vốn:[__] | bán:[__] | stock:[__] | 🖼 [Upload ảnh...] (tự do sortOrder)
      Đỏ Trắng  | vốn:[__] | bán:[250k]| stock:[8]  | 🖼 [thumb][thumb][+]
    ▼ Giày Nike  [category] [description]  [☐ Cho phép tùy chỉnh mockup]
      Đen/Trắng | vốn:[__] | bán:[__] | stock:[5]  | 🖼 [Upload ảnh...]
        ↓
[5] Confirm
    → Upload imageFiles[] per ColorSet → Cloudinary (parallel, blob URLs locally until here)
    → POST /api/admin/products/import (với isCustomizable per product)
    → prisma.$transaction → Product[] + ColorSet[] + ProductImage[] (free-form sortOrder)
    → Redirect /admin/products
```

**Duplicate product names ARE allowed** — system creates new product regardless of name collision. Shop manually deletes duplicates from `/admin/products` if needed.
**isCustomizable**: admin toggle trong Step 4 preview (per product group). Default = false. Sản phẩm đồng phục → check; giày/phụ kiện → không check.
**Images**: tự do (sortOrder), không phải 4 angles cố định.

## Related Code Files

### Create — Orders
- `src/app/admin/layout.tsx` — admin layout with sidebar + auth guard
- `src/app/admin/page.tsx` — dashboard (stats)
- `src/app/admin/orders/page.tsx` — order list with filters
- `src/app/admin/orders/[id]/page.tsx` — order detail + status update
- `src/components/admin/order-status-badge.tsx`
- `src/components/admin/order-status-updater.tsx`
- `src/components/admin/custom-order-preview.tsx` — from Phase 5
- `src/components/admin/preview-canvas.tsx` — from Phase 5
- `src/app/api/admin/orders/[id]/status/route.ts` — PATCH status (trigger customer email on shipping/delivered)

### Create — Product Import + Edit
- `src/app/admin/products/page.tsx` — product list
- `src/app/admin/products/import/page.tsx` — import wizard
- `src/app/admin/products/[id]/edit/page.tsx` — FULL product edit
- `src/components/admin/csv-import/column-mapping-step.tsx`
- `src/components/admin/csv-import/preview-table.tsx`
- `src/components/admin/csv-import/product-group-row.tsx`
- `src/components/admin/csv-import/colorset-row.tsx`
- `src/components/admin/product-image-uploader.tsx`
- `src/app/api/admin/products/import/route.ts`
- `src/app/api/admin/products/[id]/route.ts`
- `src/lib/csv-column-mapper.ts`
- `src/lib/product-grouper.ts`

### Modify
- `src/lib/auth.ts` — already configured

## Implementation Steps

### Part A: Orders (2d)
1. Admin auth guard middleware (check role=admin)
2. Dashboard: order count, revenue by status, doanh thu theo ngày/tuần, top 5 sản phẩm
3. Order list: paginated table, filter by status, search by order number/customer name
4. Order detail: customer info, items, player list, download Excel link, logo links, status updater
5. Admin preview: readonly MockupCanvas + player table for custom orders
6. Status update API: PATCH /api/admin/orders/[id]/status → send customer email on shipping/delivered

### Part B: Product Import + Edit (3d)
7. `csv-column-mapper.ts` — fuzzy match CSV header → DB field
8. `product-grouper.ts` — group rows by product name → ProductDraft[]
9. `column-mapping-step.tsx` — CSV col → DB field mapping UI
10. `colorset-row.tsx` — inline edit + free-form image upload (sortOrder-based)
11. `product-group-row.tsx` — expandable product group + `isCustomizable` toggle
12. `preview-table.tsx` — grouped product table with state mutations
13. `import/page.tsx` — wizard: upload → mapping → preview → confirm
14. `api/admin/products/import/route.ts` — bulk create Products + ColorSets + ProductImages + ProductVariants
15. **`products/[id]/edit/page.tsx`** — FULL scope:
    - Product info (name, category, price, description)
    - Image management: reorder/add/delete per ColorSet (free-form sortOrder)
    - ColorSet management: add/remove/rename
    - Stock per (ColorSet + size) variant
16. **`api/admin/products/[id]/route.ts`** — GET + PUT for all fields above

## Todo

### Orders
- [ ] Admin auth guard
- [ ] Dashboard with stats
- [ ] Order list with filters + search + pagination
- [ ] Order detail page
- [ ] Status update PATCH API (with customer email trigger)

### Product Management
- [ ] CSV column mapper
- [ ] Product grouper
- [ ] Column mapping step UI
- [ ] Colorset row UI (free-form image upload, sortOrder)
- [ ] Product group row UI
- [ ] Preview table UI
- [ ] Import wizard page
- [ ] Bulk import API
- [ ] Product list page with stock display
- [ ] **Product edit page (FULL SCOPE)**:
  - [ ] Product info form
  - [ ] Image reorder/add/delete per ColorSet
  - [ ] ColorSet management (add/remove/rename)
  - [ ] Stock per variant management
  - [ ] PUT update API

## Success Criteria
- Admin can view/filter/search orders, update status → customer email sent on shipping/delivered
- Admin can download Excel + view logos from order detail
- Admin can upload CSV → map columns → preview + confirm import
- Admin can edit product (info, images, ColorSet, stock) via `/admin/products/[id]/edit` **FULL SCOPE**
- Images upload free-form with sortOrder (not 4 angles)
- Stock display visible in product list + edit page
- Non-admin users cannot access `/admin`

## Risk
| Risk | Level | Mitigation |
|------|-------|------------|
| Supplier CSV format varies | Medium | Column mapping UI, re-map each import |
| Product name inconsistency | Low | Show grouped preview clearly |
| Cloudinary upload failure | Low | Try/catch, report failed, allow retry |
| Large CSV performance | Unlikely | PapaParse + virtual scroll if needed |
| Stock variant UI complexity | Medium | Table/form per ColorSet+size, keep simple |

## Security
- All `/admin/*` routes protected by Better Auth + admin role middleware
- File upload validation: CSV only, image MIME check before Cloudinary
- API routes validate admin session server-side
