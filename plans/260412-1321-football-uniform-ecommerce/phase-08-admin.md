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

## Overview
Admin panel gồm 2 phần:
1. **Order Management** — xem + filter + update status đơn hàng
2. **Product Import** — import sản phẩm từ CSV supplier; edit chi tiết sản phẩm sau import

Không có product CRUD form thủ công. Protected by Better Auth admin role (Phase 2).
<!-- Updated: Session 2 — replaced Prisma Studio approach with CSV import flow -->

## Architecture

```
/admin                          → Dashboard (stats)
/admin/orders                   → Order list (filter, search, paginate)
/admin/orders/[id]              → Order detail + status update
/admin/products                 → Product list (link to edit/import)
/admin/products/import          → CSV Import wizard
/admin/products/[id]/edit       → Product edit (images, colorsets, price, etc.)
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
      Xanh Navy | vốn:[__] | bán:[__] | stock:[__] | 🖼 [Upload mặt trước][Upload mặt sau][Upload tay áo][Upload quần]
      Đỏ Trắng  | vốn:[__] | bán:[250k]| stock:[8]  | 🖼 [thumb_front][thumb_back][+][+]
    ▼ Giày Nike  [category] [description]  [☐ Cho phép tùy chỉnh mockup]
      Đen/Trắng | vốn:[__] | bán:[__] | stock:[5]  | 🖼 [Upload ảnh...]
        ↓
[5] Confirm
    → Upload imageFiles[] per ColorSet per angle → Cloudinary (parallel, blob URLs locally until here)
    → POST /api/admin/products/import (với isCustomizable per product)
    → prisma.$transaction → Product[] (incl. customizable flag) + ColorSet[] + ProductImage[] (4 angles)
    → Redirect /admin/products
```

**Duplicate product names → create new records (no merge check)**
**isCustomizable**: admin toggle trong Step 4 preview (per product group). Default = false (unchecked). Sản phẩm đồng phục → check; giày/phụ kiện → không check.


### Create — Orders
- `src/app/admin/layout.tsx` — admin layout with sidebar + auth guard
- `src/app/admin/page.tsx` — dashboard (order count, revenue stats)
- `src/app/admin/orders/page.tsx` — order list
- `src/app/admin/orders/[id]/page.tsx` — order detail
- `src/components/admin/order-status-badge.tsx`
- `src/components/admin/order-status-updater.tsx`
- `src/components/admin/custom-order-preview.tsx` — preview component from Phase 5
- `src/components/admin/preview-canvas.tsx` — CSS overlay canvas from Phase 5
- `src/app/api/admin/orders/[id]/status/route.ts` — PATCH status

### Create — Product Import
- `src/app/admin/products/page.tsx` — product list
- `src/app/admin/products/import/page.tsx` — import wizard orchestrator
- `src/app/admin/products/[id]/edit/page.tsx` — full product edit
- `src/components/admin/csv-import/column-mapping-step.tsx` — mapping UI
- `src/components/admin/csv-import/preview-table.tsx` — grouped preview
- `src/components/admin/csv-import/product-group-row.tsx` — 1 product row (expandable); includes `[☑ Cho phép tùy chỉnh mockup]` checkbox (sets `isCustomizable`)
- `src/components/admin/csv-import/colorset-row.tsx` — 1 colorset row (inline edit + 4-angle image upload: front/back/sleeve/shorts)
- `src/components/admin/product-image-uploader.tsx` — reusable inline upload component
- `src/app/api/admin/products/import/route.ts` — POST bulk create
- `src/app/api/admin/products/[id]/route.ts` — GET + PUT for edit
- `src/lib/csv-column-mapper.ts` — fuzzy auto-suggest mapping logic
- `src/lib/product-grouper.ts` — group flat rows → ProductDraft[]

### Modify
- `src/lib/auth.ts` — admin role already configured in Phase 2

## Implementation Steps

### Part A: Orders (2d)
<!-- Session 7: Phase 6 đã tạo admin API routes (PATCH status, POST notes). Phase 8 chỉ build UI consume các API đó. -->
1. Admin role already configured in Phase 2 — skip auth setup
2. Create admin layout with auth guard middleware (check role=admin)
3. Dashboard: aggregate queries (order count, revenue by status, **doanh thu theo ngày/tuần**, **top 5 sản phẩm bán chạy**)
4. Order list: paginated table, **filter by status** (pending/processing/shipping/delivered/cancelled), search by order number or customer name
5. Order detail: customer info, items, player list, download Excel, logo link, status updater (calls Phase 6 PATCH API); show **cancelled** badge if applicable
6. **Admin preview**: Integrate Phase 5's `<CustomOrderDetail />` for custom orders (readonly MockupCanvas + player table)

### Part B: Product Import (3d)
7. `csv-column-mapper.ts` — fuzzy match CSV header → DB field name; returns `{ csvCol, dbField, confidence }[]`
8. `product-grouper.ts` — group MappedRow[] by `name` (trim + lowercase) → `ProductDraft[]`
9. `column-mapping-step.tsx` — show CSV cols on left, `<select>` DB field on right, auto-fill defaults
10. `colorset-row.tsx` — inline inputs for all ColorSet fields; `product-image-uploader.tsx` for 4 angles (front/back/sleeve/shorts) File[] + blob preview
11. `product-group-row.tsx` — expandable row, product-level fields (name, category, description) + **`isCustomizable` checkbox toggle** (default false), list of colorset rows
12. `preview-table.tsx` — render ProductDraft[] as grouped table; wire up state mutations (edit field, add/remove image)
13. `import/page.tsx` — wizard: step 1 = upload + mapping, step 2 = preview (with `isCustomizable` toggle + 4-angle image upload per ColorSet), step 3 = confirm
    - On Confirm: upload all File[] to Cloudinary in parallel → get URLs → POST to API
14. `api/admin/products/import/route.ts` — validate body, `prisma.$transaction` bulk create Products (with `customizable` flag) + ColorSets + ProductImages (4 angles: front/back/sleeve/shorts) + ProductVariants (stock per size)
15. `products/[id]/edit/page.tsx` — full edit form: name, category, price, description; manage ColorSets (add/remove); upload images per ColorSet
16. `api/admin/products/[id]/route.ts` — GET product with relations; PUT update

## Todo

### Orders
- [ ] Admin auth guard (check role=admin from Phase 2 Better Auth config)
- [ ] Dashboard with stats
- [ ] Order list with filters + search + pagination
- [ ] Order detail page (player list, logos, status)
- [ ] Order status update API + customer email notification

### Product Import
- [ ] `csv-column-mapper.ts` — fuzzy match logic
- [ ] `product-grouper.ts` — grouping logic
- [ ] Column mapping step UI
- [ ] `colorset-row.tsx` — inline edit + 4-angle image upload (front/back/sleeve/shorts)
- [ ] `product-group-row.tsx` — expandable product group
- [ ] Preview table with full state management
- [ ] Import wizard page (upload → mapping → preview with isCustomizable toggle + image upload → confirm)
- [ ] Confirm flow: Cloudinary upload + bulk import API
- [ ] Product list page (`/admin/products`) with **stock display**
- [ ] Product edit page (`/admin/products/[id]/edit`) with **stock field** and **4 image angles per ColorSet**
## Success Criteria
- Admin can view all orders, filter/search, update status → customer email sent
- Admin can download Excel + view logos from order detail
- Admin can upload CSV → map columns → preview grouped products with inline edit
- Admin can upload images per ColorSet in preview before confirming
- Confirm import → products visible in `/admin/products` and storefront
- Admin can edit product details post-import via `/admin/products/[id]/edit`
- Non-admin users cannot access `/admin`
- **Stock display visible** in product list and product edit page

## Risk
| Risk | Level | Mitigation |
|------|-------|------------|
| Supplier CSV format varies each time | Medium | Column mapping UI handles this — user re-maps each import |
| Product name inconsistency in CSV causes wrong grouping | Low | Show grouped preview clearly; user can spot and edit |
| Cloudinary upload failure mid-confirm | Low | Wrap in try/catch; report failed images, allow retry |
| Large CSV (>500 rows) → slow preview render | Unlikely | PapaParse streaming + virtual scroll if needed |
| Scope creep on product edit page | Medium | Keep edit minimal: fields + images only, no variant management |

## Security
- All `/admin/*` routes protected by Better Auth session + admin role middleware (Phase 2)
- File upload validation: CSV only for import, image MIME check before Cloudinary upload
- API routes validate admin session server-side (not just client route guard)