---
phase: 4
title: "Custom Builder — Sections on Product Detail"
status: pending
priority: P1
effort: 5d
---

# Phase 4: Custom Builder UI + Logic

## Context
- Depends on Phase 3 (product detail page shell, cart store)
- [Custom Builder Research](../reports/researcher-260412-1321-custom-builder.md)
- [Brainstorm — Project Understanding](../reports/brainstorm-260413-1435-project-understanding.md)
<!-- Updated: Session 5 (2026-04-14) — Complete rewrite. Wizard removed. Builder is sections embedded in /products/[slug] -->
<!-- Session 8: Player size field required, overlay structure uses overlays: OverlayElement[][] (indexed by image slot) -->

## Overview
Builder là các **sections cuộn dọc** trực tiếp trên trang `/products/[slug]` khi sản phẩm có `customizable = true`. Không có wizard, không có route riêng. Khách tương tác theo thứ tự bất kỳ, add to cart từ nút cuối trang.

**Desktop-only**: Trên mobile < 768px, thay toàn bộ builder bằng thông báo "Vui lòng dùng máy tính để thiết kế đồng phục". Catalog, cart, checkout vẫn responsive bình thường.

**Builder tech**: `react-moveable` library — logo/text là `absolute` div đè lên ảnh sản phẩm thật. Drag/resize/rotate handled by `<Moveable />` component (replaces raw mouse events).

## Page Layout (scroll-through sections)

```
/products/[slug]   (customizable=true)
│
├── [Section 1] Gallery — free-form images with thumbnail grid (normal view) vs tabs (builder view)
│               Dual-mode: thumbnail grid when browsing, tabs when in builder
│               Click thumbnail → switch main image
│
├── [Section 2] Chọn màu sắc — ColorSet selector
│               Grid ảnh thật của từng bộ màu admin định sẵn
│               Click → cập nhật gallery + ảnh nền cho mockup
│               Giữ overlay positions, đổi ảnh nền. Nếu ColorSet mới ít ảnh hơn → tab thừa ẩn, data preserved
│
├── [Section 3] Logo Library + Tên đội
│               [+ Upload logo] → Dropzone/button, chấp nhận PNG/JPG/SVG, max 5MB
│               SVG sanitized trước khi upload (strip scripts/events)
│               Dimension warning nếu logo < 300px → badge "⚠ Chất lượng thấp" + auto-note; không block upload
│               Hiển thị danh sách logos đã upload (thumbnail grid + tên file + [×] xóa)
│               Input: tên đội (optional)
│
├── [Section 4] Thiết kế mockup (CORE)
│               Layout: [Logo Panel bên trái | Canvas + Tabs]
│               Logo Panel: danh sách logos từ library — drag logo vào canvas
│               Tab switcher: tabs for each image slot (0, 1, 2...)
│               Mỗi tab:
│                 - Ảnh sản phẩm thật làm nền
│                 - Drag logo từ panel thả vào canvas → xuất hiện element
│                 - Drag text overlay (tên đội): giống logo
│                 - Mỗi element: corner handles resize, top handle rotate (snap 0/90/180/270°)
│                 - Nhiều logos có thể trên cùng 1 tab (club logo + sponsor logo + số nhỏ...)
│               Positions lưu per-image-slot + per-element (x%, y%, width%, rotation)
│               **Switching ColorSet: giữ nguyên element positions**
│               Shared <MockupCanvas /> component (interactive mode)
│
├── [Section 5] Danh sách cầu thủ
│               Luôn hiển thị, mặc định 1 dòng
│               Columns: STT | Tên cầu thủ* | Số áo* | Size áo (required) | Size quần*
│               (* optional — nếu trống → áo trơn không in)
│               [+ Thêm cầu thủ] | [Paste từ Excel]
│               Real-time print fee calculation theo số dòng
│
├── [Section 6] Phí in (nudge)
│               < 6 bộ: +20% | 6-9 bộ: +10% | 10+ bộ: Miễn phí in
│               Nudge: "Thêm N bộ để giảm phí in xuống X%"
│
└── [Add to Cart button] — tổng giá = (đơn giá × số bộ) + phí in
```

## Architecture

```
/products/[slug]/page.tsx (RSC)
  ├── ProductGallery           (Phase 3 — images by slot)
  ├── ColorSetSelector         (Phase 3 — swap images)
  ├── [if !customizable]
  │   ├── SizeSelector
  │   └── AddToCartButton (normal)
  └── [if customizable]
      ├── MobileBuilderNotice  (< 768px only)
      ├── [hidden on mobile]
      │   ├── LogoUploader     — Phase 4
      │   ├── MockupCanvas     — Phase 4 (interactive)
      │   │   └── Tabs: one per image slot
      │   ├── PlayerListEditor — Phase 4
      │   ├── PrintFeeNudge    — Phase 3 (reused)
      │   └── AddToCartButton  (custom)
```

## Data Types

```typescript
// Logo item trong library
interface LogoItem {
  id: string            // local ID: 'logo_0', 'logo_1'...
  url: string           // Cloudinary URL sau khi upload
  filename: string      // tên file gốc (để hiện trong panel)
  qualityNote?: string  // auto-set nếu width < 300px
}

// Per-image-slot overlay element
interface OverlayElement {
  id: string
  type: 'logo' | 'text'
  logoId?: string        // reference logos[].id (khi type='logo')
  text?: string          // giá trị text (khi type='text')
  x: number             // % of canvas width (left edge)
  y: number             // % of canvas height (top edge)
  width: number         // % of canvas width
  height: number        // % of canvas height (maintain aspect ratio)
  rotation: number      // degrees
  zIndex: number
}

// Full custom config saved to cart + DB (printConfigJson)
interface CustomConfig {
  teamName?: string
  logos: LogoItem[]           // multi-logo support (club, sponsor, flag patch...)
  overlays: OverlayElement[][]  // indexed by image slot [0], [1], [2]...
  players: {
    sortOrder: number
    playerName?: string
    playerNumber?: string
    size: string          // required — from ProductVariant sizes (disabled if stock=0)
    shortsSize?: string
  }[]
}

// Custom cart item
interface CustomCartItem {
  type: 'custom'
  id: string
  productId: number
  productName: string
  colorSetId: number
  colorSetName: string
  productImages: string[]       // images per slot
  config: CustomConfig
  unitPrice: number
  printingSurcharge: number
  totalPrice: number
}
```

## Print Fee Logic

```typescript
// src/lib/printing-fee.ts
export function calculatePrintFee(playerCount: number, subtotal: number) {
  if (playerCount >= 10) return { rate: 0, amount: 0, nudge: null }
  if (playerCount >= 6) {
    return { rate: 0.1, amount: subtotal * 0.1,
      nudge: `Them ${10 - playerCount} bo de mien phi in!` }
  }
  return { rate: 0.2, amount: subtotal * 0.2,
    nudge: `Them ${6 - playerCount} bo de giam phi in xuong 10%` }
}
```

## MockupCanvas Component (shared)

```
<MockupCanvas
  imageUrl={string}          // product image for current slot
  elements={OverlayElement[]}
  mode="interactive" | "readonly"
  onElementsChange={(els) => void}  // only in interactive mode
/>
```

- Interactive mode (builder): drag-drop, resize, rotate
- Read-only mode (admin Phase 5, order confirmation): display only

**Drag interaction** (`react-moveable`, desktop only):
- Wrap each element with `<Moveable draggable resizable rotatable />`
- `onDrag` → update x, y (clamped to canvas bounds)
- `onResize` → update width/height (maintain aspect ratio via keepRatio)
- `onRotate` → update rotation (snap at 0°/90°/180°/270° via throttleRotate)
- Click outside → deselect (clear `target` ref)

## Related Code Files

### Create
- `src/components/custom-builder/logo-library-panel.tsx` — sidebar panel: upload logos + display thumbnail grid + drag source
- `src/components/custom-builder/logo-uploader.tsx` — dropzone + SVG sanitize + Cloudinary upload + quality warning badge
- `src/components/custom-builder/mockup-canvas.tsx` — **shared CSS overlay component** (interactive + readonly modes)
- `src/components/custom-builder/overlay-element.tsx` — single draggable/resizable/rotatable element (references LogoItem)
- `src/components/custom-builder/image-tab-switcher.tsx` — tabs for each image slot
- `src/components/custom-builder/player-list-editor.tsx` — table với useFieldArray + paste
- `src/components/custom-builder/player-row.tsx` — 1 row: tên, số, size (required dropdown from ProductVariant), size quần
- `src/components/custom-builder/mobile-builder-notice.tsx` — notice for < 768px
- `src/lib/printing-fee.ts` — fee calculation per CustomOrder (shared with Phase 6 server-side)
- `src/lib/clipboard-parser.ts` — Excel TSV paste: auto-detect columns (pattern matching), fallback mapping dialog
- `src/components/custom-builder/column-mapping-dialog.tsx` — simple dialog with one `<select>` dropdown per detected column
- `src/lib/svg-sanitizer.ts` — strip scripts/event handlers from SVG before upload
- `src/app/api/upload/logo/route.ts` — **server-side** Cloudinary upload endpoint (validate MIME + size, SVG sanitize)

### Modify
- `src/app/products/[slug]/page.tsx` — add builder sections for customizable products
- `src/stores/cart-store.ts` — add CustomCartItem type + actions
- `src/components/cart/cart-item-row.tsx` — render custom bundles với summary

## Implementation Steps

1. Create `src/lib/svg-sanitizer.ts` — strip `<script>`, event attrs (on*), `href=javascript:` from SVG string; return sanitized SVG
2. Create `src/app/api/upload/logo/route.ts` — validate MIME (image/jpeg, image/png, image/svg+xml), max 5MB; sanitize SVG before upload; return Cloudinary URL
3. Create `clipboard-parser.ts`:
   - Parse TSV from `navigator.clipboard.readText()`
   - Auto-detect columns by pattern: integers 1-99 → Số áo; S/M/L/XL/XXL tokens → Size áo/quần; long strings → Tên
   - Confidence ≥ 80% → map directly; else → emit `needsMapping: true` with sample data
   - Consumer shows `<ColumnMappingDialog />` when `needsMapping = true`
4. Create `printing-fee.ts` — `calculatePrintFee(playerCount, subtotal)`: per-CustomOrder, returns `{ rate, amount, nudge }`
5. Create `logo-uploader.tsx` — dropzone; PNG/JPG/SVG max 5MB; SVG → sanitize via `svg-sanitizer.ts`; dimension check → qualityNote if < 300px; quality warning badge + auto-note; returns `LogoItem`
6. Create `logo-library-panel.tsx` — sidebar showing uploaded `LogoItem[]`; thumbnail grid; [+ Upload] button; [×] remove; drag source (`draggable` attr) for canvas drop
7. Create `overlay-element.tsx` — absolute-positioned div; interactive: mousedown drag, corner handles resize, top handle rotate (snap 0/90°/180°/270°); readonly: static; references `logoId` or renders `text`
8. Create `mockup-canvas.tsx` — container (position:relative) with product image + overlay elements; `onDrop` receives logoId from panel drag; mode prop switches interactivity; positions stay on ColorSet switch
9. Create `image-tab-switcher.tsx` — tabs for each image slot; switching saves/restores elements per slot (state keyed by slot index)
10. Create `player-list-editor.tsx`:
    - useFieldArray for dynamic rows
    - Default 1 row on mount
    - Columns: STT (auto), Tên (optional), Số áo (optional), Size áo (required dropdown from ProductVariant.sizes — disabled if stock=0), Size quần (optional)
    - Add row button; remove per row
    - Paste handler: call `clipboard-parser`; if confident → append rows directly; else → show `<ColumnMappingDialog />`
    - Emit player count → parent for print fee recalculation
11. Create `mobile-builder-notice.tsx` — `md:hidden` block with message "Vui lòng dùng máy tính để thiết kế"
12. Update `src/app/products/[slug]/page.tsx`:
    - If `product.customizable = false`: render Phase 3 normal flow
    - If `product.customizable = true`: render builder sections + `<MobileBuilderNotice className="md:hidden" />`; wrap builder in `<div className="hidden md:block">`
    - Fetch ProductVariant sizes for size dropdown in player list
13. Update `cart-store.ts` — add `CustomCartItem` interface (with `logos: LogoItem[]`) + `addCustomItem`, `removeItem` actions; persist to localStorage
14. Wire Add to Cart: collect `CustomConfig` (incl. `logos[]`) from state → create `CustomCartItem` → add to store → redirect to `/cart`
15. Cart page: custom items show bundle summary (team name fallback = product.name if empty, count, total) + [Xóa] only — no Sửa button

## Todo
- [ ] SVG sanitizer utility (`svg-sanitizer.ts`)
- [ ] Cloudinary upload API route (PNG/JPG/SVG, max 5MB, SVG sanitize)
- [ ] Clipboard TSV parser with auto-detect + mapping dialog fallback
- [ ] Print fee calculation utility (per-CustomOrder)
- [ ] LogoUploader (dropzone + SVG sanitize + Cloudinary + quality warning badge)
- [ ] LogoLibraryPanel (thumbnail grid + drag source + remove)
- [ ] OverlayElement component (drag/resize/rotate, references logoId)
- [ ] MockupCanvas (interactive + readonly modes, onDrop from logo panel)
- [ ] ImageTabSwitcher (tabs per slot, per-slot state, positions preserved on ColorSet switch)
- [ ] PlayerListEditor (useFieldArray + paste + mapping dialog + fee emit, sizes from ProductVariant)
- [ ] MobileBuilderNotice
- [ ] Wire builder sections into product detail page
- [ ] Cart store CustomCartItem support (logos[] array)
- [ ] Add to cart flow → redirect to /cart
- [ ] Cart page renders custom bundles (summary + [Xóa] only, no Sửa)

## Success Criteria
- Customizable product detail shows all builder sections on desktop
- Mobile (< 768px) shows notice instead of builder
- Multiple logos uploadable; all appear in logo panel sidebar
- Drag logo from panel → drops on canvas; drag/resize/rotate works
- Switch between image tabs: each tab has independent elements
- Switch ColorSet: element positions preserved
- Resize (corner handles) + rotate (top handle, snap 0°/90°/180°/270°) work correctly
- Player list: paste 10 rows from Excel → auto-detect columns → populated; unclear format → mapping dialog
- Size dropdown shows ProductVariant sizes; stock=0 size is disabled with tooltip
- Size field required; name/number/shorts-size optional
- Print fee nudge shows correct tier per CustomOrder in real-time
- Add to cart → CustomCartItem (with `logos[]`) in Zustand store → visible in cart
- Cart shows bundle summary: teamName (fallback = product.name) + player count + total + [Xóa]

## Risk
- Coordinate system: store positions as % to be responsive across screen sizes
- Cloudinary free tier: 25 credits/month → client-side validation first, minimize transforms
- Large player list (50+ rows) → test useFieldArray performance
- Paste format varies between Excel/Google Sheets → test both, handle tab vs comma
- Image slot tab switch must preserve element state for each slot independently (use array keyed by slot index)

## Security
- Upload endpoint: validate MIME type (image/jpeg, image/png, image/svg+xml only), max 5MB
- **SVG sanitization (mandatory)**: strip `<script>` tags, `on*` event attributes, `href="javascript:..."` before upload — XSS vector via SVG
- Sanitize player names + team name (XSS prevention) before storing
- Rate limit upload endpoint (10 req/min per user)
