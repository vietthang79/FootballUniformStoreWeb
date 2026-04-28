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
<!-- Session 10 (2026-04-17): OVERRIDE Session 9 — use `react-moveable` (drag+resize+rotate+snap, not react-draggable). Dynamic tabs by ProductImage count (not hardcoded 4). Logo quality check < 300px. Per-CustomCartItem logo scope. ColorSet switch preserves % coords (auto-scales). -->

## Overview
Builder là các **sections cuộn dọc** trực tiếp trên trang `/products/[slug]` khi sản phẩm có `customizable = true`. Không có wizard, không có route riêng. Khách tương tác theo thứ tự bất kỳ, add to cart từ nút cuối trang.

**Desktop-only**: Trên mobile < 768px, thay toàn bộ builder bằng thông báo "Vui lòng dùng máy tính để thiết kế đồng phục". Catalog, cart, checkout vẫn responsive bình thường.

**Builder tech**: `react-moveable` library (Session 10 override) — logo/text là `absolute` div đè lên ảnh sản phẩm thật. `<Moveable draggable resizable rotatable />` cung cấp drag + resize (corner handles) + rotate (top handle, snap 0/90/180/270°). All positions stored as percentage (0-1) of canvas dimensions → auto-scale across ColorSet switches and screen sizes.

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
│               Tab switcher: **dynamic tabs** (Session 10) — one tab per ProductImage,
│                 ordered by sortOrder, labeled by ProductImage.label
│                 (2-10 tabs depending on admin upload count, not fixed 4 angles)
│               Mỗi tab:
│                 - Ảnh sản phẩm thật làm nền
│                 - Drag logo từ panel thả vào canvas → xuất hiện element (react-moveable)
│                 - Drag text overlay (tên đội): giống logo
│                 - Mỗi element: corner handles resize, top handle rotate (snap 0/90/180/270°)
│                 - Nhiều logos có thể trên cùng 1 tab (club logo + sponsor logo + số nhỏ...)
│               Positions: **percentage coords** (x/y/w/h ∈ [0,1])
│                 lưu per-image-slot + per-element (rotation in degrees)
│               **Switching ColorSet** (Session 10): giữ nguyên overlays[][].
│                 Tabs re-rendered for new ColorSet's images. Tab thừa ẩn (data preserved).
│                 % coords auto-scale to new image dimensions.
│               Shared <MockupCanvas /> component (interactive mode)
│
├── [Section 5] Danh sách cầu thủ
│               Luôn hiển thị, mặc định 1 dòng
│               Columns: STT | Tên cầu thủ* | Số áo* | Size (required — áo + quần cùng size, Session 8)
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
/products/[slug]/page.tsx (RSC)  — owned by Phase 10 (product catalog)
  ├── ProductGallery           (Phase 10 — dual-mode: grid + tabs)
  ├── ColorSetSelector         (Phase 10 — swap images)
  ├── [if !customizable]
  │   ├── SizeSelector
  │   └── AddToCartButton (normal)
  └── [if customizable]
      ├── MobileBuilderNotice  (< 768px only)
      ├── [hidden on mobile]
      │   ├── LogoUploader     — Phase 4
      │   ├── MockupCanvas     — Phase 4 (interactive, react-moveable, % coords)
      │   │   └── Dynamic tabs: one per ProductImage (Session 10)
      │   ├── PlayerListEditor — Phase 4
      │   ├── PrintFeeNudge    — Phase 4 (owned here, reused in cart Phase 10)
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

// Per-image-slot overlay element (Session 10: all coords are percentages in [0,1])
interface OverlayElement {
  id: string
  type: 'logo' | 'text'
  logoId?: string        // reference logos[].id (khi type='logo')
  text?: string          // giá trị text (khi type='text')
  x: number             // 0-1 of canvas width (left edge)
  y: number             // 0-1 of canvas height (top edge)
  width: number         // 0-1 of canvas width
  height: number        // 0-1 of canvas height (maintain aspect ratio)
  rotation: number      // degrees [0-360], snap at 0/90/180/270
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
    // Session 8: shortsSize removed — 1 field `size` for both jersey + shorts
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

**Rule (Session 12):** Phí in CHỈ tính khi thực sự có thứ để in. Đặt áo trơn (không logo, không tên/số, không tên đội) → đơn custom hợp lệ nhưng phí in = 0.

**Trigger điều kiện in (≥1 đủ):**
- Có overlay (logo hoặc text) thả vào canvas (`overlays.some(slot => slot.length > 0)`)
- Có ít nhất 1 cầu thủ nhập tên HOẶC số áo
- Có nhập tên đội (teamName)

```typescript
// src/lib/printing-fee.ts
import type { CustomConfig } from '~types/custom-config'

export function hasPrintingWork(config: CustomConfig): boolean {
  const hasOverlays   = config.overlays.some(slot => slot.length > 0)
  const hasPlayerText = config.players.some(p => p.playerName?.trim() || p.playerNumber?.trim())
  const hasTeamName   = !!config.teamName?.trim()
  return hasOverlays || hasPlayerText || hasTeamName
}

export function calculatePrintFee(config: CustomConfig, subtotal: number) {
  // KHÔNG IN GÌ → KHÔNG PHÍ IN (Session 12)
  if (!hasPrintingWork(config)) {
    return { rate: 0, amount: 0, nudge: null, label: 'Không có in' }
  }
  const count = config.players.length
  if (count >= 10) return { rate: 0,    amount: 0,                nudge: null,                                                  label: 'Miễn phí in' }
  if (count >= 6)  return { rate: 0.10, amount: subtotal * 0.10,  nudge: `Thêm ${10 - count} bộ để miễn phí in`,                label: 'Phí in +10%' }
  return                  { rate: 0.20, amount: subtotal * 0.20,  nudge: `Thêm ${6 - count} bộ để giảm phí in xuống 10%`,       label: 'Phí in +20%' }
}
```

**UX behavior trong Section 6 (PrintFeeNudge):**
- `!hasPrintingWork` → card neutral (xám nhạt) "Không có in — phí in: 0₫. Bạn đang đặt áo trơn."
- `hasPrintingWork && count < 6` → card amber "Phí in +20% (XX,XXX₫). Thêm N bộ để giảm còn 10%"
- `count 6-9` → card amber nhẹ "Phí in +10%. Thêm N bộ để miễn phí in"
- `count ≥ 10` → card green "Miễn phí in 🎉"
- Real-time recompute khi: thêm/xóa overlay, edit player name/number, edit teamName, thêm/xóa player row

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
- Wrap each element with `<Moveable draggable resizable rotatable container={canvasRef.current} keepRatio={true} />`
- `onDrag` → mutate DOM directly (`e.target.style.transform = translate3d(...)`), commit state ở `onDragEnd`
- `onResize` → update width/height (maintain aspect ratio via `keepRatio`), mutate DOM during, commit on end
- `onRotate` → update rotation, snap 0/90/180/270/360° (custom: nếu `|rotation % 90| < 5` → lock exact)
- Click outside → deselect (clear `target` ref)

### CRITICAL — Coordinate accuracy (logo MUST follow cursor exactly)

Sai 1 px trong conversion math = logo drift, lệch tỉ lệ. Setup đúng từ đầu để tránh debug khó.

**Canvas DOM structure (mandatory):**
```tsx
// mockup-canvas.tsx
<div ref={canvasRef}
  className="relative w-full select-none"
  style={{ aspectRatio: `${imgNaturalW}/${imgNaturalH}` }}>  {/* aspect-ratio = ảnh natural */}
  <img src={activeImage.url}
       className="absolute inset-0 w-full h-full object-contain pointer-events-none"
       alt="" />
  {overlays[activeSlot].map(el => (
    <OverlayElement key={el.id} element={el} canvasRef={canvasRef} />
  ))}
</div>
```

**Overlay positioning (mandatory — ALL coords là %, KHÔNG mix px):**
```tsx
// overlay-element.tsx
<div ref={ref}
  className="absolute will-change-transform"
  style={{
    left:   `${el.x * 100}%`,
    top:    `${el.y * 100}%`,
    width:  `${el.width * 100}%`,
    height: `${el.height * 100}%`,
    transform: `rotate(${el.rotation}deg) translate3d(0,0,0)`,  // GPU layer
    transformOrigin: 'center center',                           // rotate around center
  }}>
  <img src={logo.url} className="w-full h-full object-contain pointer-events-none" />
</div>
```

**Coord conversion (CRITICAL — sai sẽ drift):**
```typescript
// pixel ↔ percent
function pixelToPercent(clientX: number, clientY: number, canvas: HTMLElement) {
  const rect = canvas.getBoundingClientRect()  // viewport-aware, scroll-aware
  return {
    x: (clientX - rect.left) / rect.width,    // 0-1 of canvas width
    y: (clientY - rect.top)  / rect.height,   // 0-1 of canvas height
  }
}

// react-moveable onDrag — mutate DOM directly (perf), commit on dragEnd
const handleDrag = useCallback((e) => {
  const rect = canvasRef.current!.getBoundingClientRect()
  const xPct = e.left / rect.width
  const yPct = e.top  / rect.height
  e.target.style.left = `${xPct * 100}%`
  e.target.style.top  = `${yPct * 100}%`
}, [])
const handleDragEnd = useCallback((e) => {
  const rect = canvasRef.current!.getBoundingClientRect()
  setOverlays(prev => updateOverlay(prev, activeSlot, el.id, {
    x: e.lastEvent.left / rect.width,
    y: e.lastEvent.top  / rect.height,
  }))
}, [activeSlot, el.id])
```

**TUYỆT ĐỐI tránh:**
- `window.innerWidth` thay `canvasRect.width`
- Quên trừ `canvasRect.left/top` khi convert clientX/Y
- Mix `px + %` trong cùng element
- Bỏ qua scroll offset (dùng `clientX` chứ KHÔNG `pageX`)
- `position: fixed` cho overlay (đứng yên khi scroll)
- `transform-origin: top left` (rotate "dance" quanh corner thay center)

### Drag performance (must implement)

react-moveable mặc định setState mỗi mousemove → re-render React tree → lag. Optimize:

1. **GPU**: `transform: translate3d(x,y,0)` (NOT `top/left` for animation), `will-change: transform` khi active drag
2. **Ref during, state on end**: `onDrag` mutate `e.target.style.transform` trực tiếp; `setState` CHỈ ở `onDragEnd`
3. **Memoize**: `React.memo` cho `<Moveable>` + targets, `useCallback` cho callbacks
4. **rAF throttle** nếu vẫn lag: bọc handler trong `requestAnimationFrame`
5. **Validate**: Chrome DevTools Performance → record drag → scripting frames < 16ms (60fps)

### Rotate snap (mandatory)

```typescript
// react-moveable config
<Moveable
  throttleRotate={15}                    // smooth 15° increments
  onRotate={(e) => {
    let r = e.rotation
    if (Math.abs(r % 90) < 5) r = Math.round(r / 90) * 90  // snap to 0/90/180/270
    if (r === 360) r = 0                                    // modulo cleanup
    e.target.style.transform = `rotate(${r}deg)`
  }}
/>
```

- Snap angles: 0/90/180/270/360° (360 = 0)
- Visual feedback khi snap: border flash #FDD017 (200ms) + angle badge "90°"
- Default rotation = 0°

### ColorSet → Image driver (single source of truth)

```typescript
// product-detail-store.ts (Zustand or React state)
interface ProductDetailState {
  activeColorSetId: number          // single source of truth
  activeImageSlot: number           // tab index within active ColorSet
}

// Initial state on page load:
// activeColorSetId = colorSets[0].id (first ColorSet by sortOrder/createdAt)
// activeImageSlot = 0 (first image of that ColorSet)

// Khi user click ColorSet:
setActiveColorSet(id) → {
  // Gallery (normal mode): main image = newColorSet.images[0]
  // Builder (custom mode): tabs re-render = newColorSet.images, activeImageSlot reset to 0 NẾU ngoài range
  // overlays[][] giữ nguyên — % coords auto-scale to new image dimensions
  // Tab thừa nếu newColorSet ít ảnh hơn → ẩn (data preserved trong overlays array)
}
```

**Cả 2 chế độ (normal + custom)** dùng chung `<ColorSetSelector />` + `<ProductGallery />` — chỉ khác phần dưới (size picker vs builder sections).

## Related Code Files

### Create
- `src/components/custom-builder/logo-library-panel.tsx` — sidebar panel: upload logos + display thumbnail grid + drag source
- `src/components/custom-builder/logo-uploader.tsx` — dropzone + SVG sanitize + Cloudinary upload + quality warning badge
- `src/components/custom-builder/mockup-canvas.tsx` — **shared CSS overlay component** (interactive + readonly modes)
- `src/components/custom-builder/overlay-element.tsx` — single draggable/resizable/rotatable element (references LogoItem)
- `src/components/custom-builder/image-tab-switcher.tsx` — tabs for each image slot
- `src/components/custom-builder/player-list-editor.tsx` — table với useFieldArray + paste
- `src/components/custom-builder/player-row.tsx` — 1 row: tên, số, size (required dropdown from ProductVariant) — single size field (Session 8)
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

0. Install builder deps (apps/web): `pnpm add react-moveable isomorphic-dompurify uuid`
1. Create `src/lib/svg-sanitizer.ts` — strip `<script>`, event attrs (on*), `href=javascript:` from SVG string; return sanitized SVG (use `isomorphic-dompurify` under the hood)
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
8. Create `mockup-canvas.tsx` — container (position:relative) with product image + overlay elements (absolute positioned via `left: ${x*100}%; top: ${y*100}%; width: ${w*100}%` etc); `onDrop` receives logoId from panel drag; mode prop switches interactivity; positions as % (0-1) stay on ColorSet switch (auto-scale)
9. Create `image-tab-switcher.tsx` — **dynamic tabs** generated from `productImages.filter(img => img.colorSetId === currentColorSetId).sort(sortOrder)` (Session 10); labels from `ProductImage.label` or fallback "Ảnh N"; state keyed by slot index (overlays[slotIndex]); on ColorSet switch tabs re-render but overlays data preserved
10. Create `player-list-editor.tsx`:
    - useFieldArray for dynamic rows
    - Default 1 row on mount
    - Columns: STT (auto), Tên (optional), Số áo (optional), Size (required dropdown from ProductVariant.sizes — disabled if stock=0, Session 10: display-only check, no order-time validation)
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
