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
Phase này tạo **reusable component** để Admin Dashboard (Phase 8) hiển thị preview thiết kế của khách hàng. Preview dùng CSS overlay (client-side) từ `printConfigJson` đã lưu trong CustomOrder, giống như Step 5 của Custom Builder. Phase này chỉ tạo component, không tích hợp vào admin UI.

## Architecture

```
Admin Dashboard (/admin/orders/[id])
  └── CustomOrderPreview component
        ├── Load product image (từ ColorSet)
        ├── Load logo URLs (từ CustomOrder)
        ├── Load printConfigJson (từ CustomOrder)
        ├── Render design elements với CSS absolute positioning
        └── View toggle: [Mặt trước] [Mặt sau] [Quần]
            └── Player selector: chọn cầu thủ để xem tên/số
```

## What Gets Displayed

Preview hiển thị:
- Ảnh sản phẩm thật (từ Cloudinary, đúng bộ màu đã chọn)
- Overlay các elements theo `printConfigJson`:
  - Text: Team Name, Player Name, Player Number (với font + color)
  - Images: Club Logo, Sponsor Logo, League Logo, Flag/Patch
- Position/scale/rotation theo dữ liệu đã lưu từ Step 4 builder

View toggle:
- Mặt trước: Logo ngực trái, sponsor, số nhỏ
- Mặt sau: Tên đội, số, tên cầu thủ
- Quần: Số quần (nếu có)

Player selector: Dropdown chọn cầu thủ → xem tên/số của người đó trên mặt sau.

## Related Code Files

### Create
- `src/components/admin/custom-order-preview.tsx` — Admin preview component
- `src/components/admin/preview-canvas.tsx` — Canvas container với CSS overlay

### Dependency
- `src/lib/constants.ts` — font definitions (đã có từ Phase 1)
- Phase 4's print config types (ElementTransform interface)

## Implementation Steps

1. Tạo `custom-order-preview.tsx`:
   - Props: customOrder, product, colorSet
   - State: currentView ('front' | 'back' | 'shorts'), selectedPlayerIndex
   - Load printConfigJson từ CustomOrder
   - Render product image làm background
   - Loop qua elements: render với CSS absolute positioning
   - Apply position (%), scale (px), rotation từ ElementTransform

2. Tạo `preview-canvas.tsx`:
   - Container với aspect ratio cố định
   - Product image component
   - Overlay elements (text + images) với CSS transforms
   - View toggle buttons
   - Player selector dropdown

Note: Integration vào admin order detail page sẽ được thực hiện trong Phase 8.

## Todo
- [ ] `custom-order-preview.tsx`: load data, render canvas
- [ ] `preview-canvas.tsx`: CSS overlay implementation
- [ ] View toggle (front/back/shorts)
- [ ] Player selector dropdown
- [ ] Test preview với sample custom order data

## Success Criteria
- Preview hiển thị đúng như khách đã thấy trong builder
- Design elements đúng vị trí, đúng rotation, đúng scale
- View toggle hoạt động đúng
- Player selector hiển thị đúng tên/số từng cầu thủ
- Responsive trên mobile

## Risk
- CSS transforms cần sync với builder implementation → dùng cùng ElementTransform type
- Font loading: dùng cùng font file với builder
- Coordinate mapping: position lưu dạng % → CSS % mapping trực tiếp, không cần convert

## Note.
Không cần server-side canvas render vì admin chỉ cần xem, không cần export PNG
Client-side preview (Step 5 của Phase 4) và Admin preview (phase này) dùng cùng logic CSS overlay.
Cả hai render từ `printConfigJson` + logo URLs + product image.
Không cần server-side canvas render vì admin chỉ cần xem, không cần export PNG.
