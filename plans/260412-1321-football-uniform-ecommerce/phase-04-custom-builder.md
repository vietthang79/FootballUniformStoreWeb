---
phase: 4
title: "Custom Mockup Builder UI + Logic"
status: pending
priority: P1
effort: 5d
---

# Phase 4: Custom Builder UI + Logic

## Context
- Depends on Phase 3 (product detail, cart store)
- [Custom Builder Research](../reports/researcher-260412-1321-custom-builder.md)
- [Brainstorm — Custom Builder Flow](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md)

## Overview
Multi-step custom mockup builder: color → logo → players → print config (2D direct editor) → mockup preview (overlay trên ảnh thật) → add to cart. Step 4 dùng direct manipulation để đặt vị trí các thành phần. Step 5 chỉ hiển thị kết quả overlay trên ảnh sản phẩm thật — không có 3D.

**Implementation approach**: Desktop-first với mouse events. Touch event equivalents (touchstart/touchmove/touchend) sẽ được implement sau khi desktop hoàn thiện.

## Key Insights
- Builder pre-loaded với product đang xem (URL param hoặc state)
- React Hook Form + FormProvider xuyên suốt tất cả steps — 1 form duy nhất, multi-step navigation
- useFieldArray cho player list (dynamic rows)
- Clipboard API để paste từ Excel vào bảng cầu thủ
- Logo upload lên Cloudinary với client-side dimension validation
- Phí in tính real-time theo số cầu thủ
- Builder state lưu vào Zustand khi add to cart (as CustomCartItem)
- **2D Direct Editor (Step 4)**: Click-thả trực tiếp, KHÔNG có button zoom/rotate/resize
- **2D Mockup Preview (Step 5)**: Design overlay lên ảnh thật sản phẩm — front/back/s
- **Independent Components**: Name, Number, Logo là 3 đối tượng riêng biệt trong Step 4

## Architecture

```
/products/[slug]/custom  → Custom Builder page

UX: Step wizard — progress bar (step 1/6), Back/Next buttons per step.
    shadcn/ui Tabs hoặc custom stepper component.

Steps (single React Hook Form + FormProvider):
  Step 1: ColorSetSelector — chọn bộ màu định sẵn của sản phẩm
  Step 2: LogoUploader — upload club logo, sponsor, giải đấu, flag, nhập tên đội
  Step 3: PlayerListEditor — bảng inline, paste từ Excel, thêm/xoá dòng
  Step 4: PrintConfigEditor — 2D direct manipulation: kéo thả logo/tên/số lên ảnh sản phẩm
  Step 5: MockupPreview — overlay design lên ảnh thật, chọn góc nhìn (front/back/sleeve/shorts), chọn cầu thủ
  Step 6: AddToCart — thêm vào giỏ, lưu thiết kế
```
<!-- Updated: Validation Session 1 - Step wizard UX confirmed; sleeve logo side: user-selectable (left/right/both) -->
<!-- Updated: 2026-04-13 - Removed 3D/Three.js. Step 5 is now 2D overlay on real product photos. -->
<!-- Updated: 2026-04-13 - 4 image angles: front/back/sleeve/shorts -->

## Form Data Type

```typescript
interface CustomBuilderData {
  productId: number
  productName: string
  colorSetSlug: string
  colorSetName: string

  // Step 2 — logos
  teamName: string
  clubLogoUrl: string | null
  sponsorLogoUrl: string | null
  leagueLogoUrl: string | null
  flagPatchUrl: string | null
  logoQualityNote: string | null // Auto-generated for low quality logos

  // Step 3 — players
  players: {
    sortOrder: number
    playerName: string
    playerNumber: number
    jerseySize: string
    shortsSize: string
  }[]

  // Step 4 — print config: position/scale/rotation per element per view
  printConfig: {
    front: {
      clubLogo: ElementTransform | null    // ngực trái
      sponsorLogo: ElementTransform | null // giữa ngực
      smallNumber: ElementTransform | null // ngực phải
    }
    back: {
      teamName: ElementTransform | null    // trên cùng
      backNumber: ElementTransform | null  // giữa
      playerName: ElementTransform | null  // dưới cùng
    }
    sleeve: {
      leagueLogo: ElementTransform | null
      sleeveSponsor: ElementTransform | null
      flagPatch: ElementTransform | null
      side: 'left' | 'right' | 'both'
    }
    shorts: {
      shortsNumber: ElementTransform | null
      shortsNumberSide: 'left' | 'right' | null
    }
  }

  // Step 5 — preview state
  previewView: 'front' | 'back' | 'shorts'
  previewPlayerIndex: number

  // Canvas
  canvasSize: { width: number; height: number }
}

interface ElementTransform {
  enabled: boolean
  position: { x: number; y: number }       // % của canvas width/height
  scale: { width: number; height: number }  // px
  rotation: number                          // 0-360°
  zIndex: number
}
```

## Printing Fee Logic

```typescript
function calculatePrintingSurcharge(playerCount: number, subtotal: number) {
  if (playerCount >= 10) return { rate: 0, amount: 0, nudge: null }
  if (playerCount >= 6) {
    const needed = 10 - playerCount
    return { rate: 0.1, amount: subtotal * 0.1, nudge: `Thêm ${needed} bộ để miễn phí in!` }
  }
  const needed = 6 - playerCount
  return { rate: 0.2, amount: subtotal * 0.2, nudge: `Thêm ${needed} bộ để giảm phí in xuống 10%` }
}
```

## Related Code Files

### Create
- `src/app/products/[slug]/custom/page.tsx` — builder page
- `src/components/custom-builder/builder-stepper.tsx` — step navigation UI
- `src/components/custom-builder/step-color-select.tsx` — Step 1
- `src/components/custom-builder/step-logo-upload.tsx` — Step 2
- `src/components/custom-builder/step-player-list.tsx` — Step 3
- `src/components/custom-builder/step-print-config.tsx` — Step 4: 2D direct editor
- `src/components/custom-builder/step-mockup-preview.tsx` — Step 5: overlay preview
- `src/components/custom-builder/step-add-to-cart.tsx` — Step 6
- `src/components/custom-builder/player-table.tsx` — editable table với paste support
- `src/components/custom-builder/logo-uploader.tsx` — single logo upload widget
- `src/components/custom-builder/printing-fee-nudge.tsx` — fee display + nudge
- `src/components/custom-builder/editable-element.tsx` — draggable/resizable/rotatable element
- `src/components/custom-builder/selection-handles.tsx` — bounding box + resize/rotate handles
- `src/lib/printing-fee.ts` — fee calculation logic
- `src/lib/clipboard-parser.ts` — Excel TSV paste parser
- `src/app/api/upload/route.ts` — Cloudinary upload endpoint

### Modify
- `src/stores/cart-store.ts` — add CustomCartItem support
- `src/components/cart/cart-item-row.tsx` — render custom bundles với [Sửa] button

## Implementation Steps

1. Tạo builder page route `/products/[slug]/custom`
2. Setup React Hook Form với FormProvider + defaultValues từ product
3. Build step navigation (stepper UI với back/next, per-step validation)
4. **Step 1**: Color set selector — grid ảnh thật của từng bộ màu admin đã định sẵn cho sản phẩm này, click để chọn. Không có color picker tự do. Bộ màu được chọn xác định ảnh nền dùng cho Step 4 và Step 5.
5. **Step 2**: Logo upload widget — dropzone, Cloudinary upload, dimension validation (warn <300px), preview thumbnail, auto-generate quality note cho logo nhỏ
6. Tạo `/api/upload` route — nhận file, upload lên Cloudinary, trả URL
7. **Step 3**: Player list table
   - useFieldArray cho dynamic rows
   - Columns: STT (auto), Tên cầu thủ, Số áo, Size áo (dropdown), Size quần (dropdown)
   - Add row / remove row per row
   - Paste handler: lắng nghe paste event trên table container, parse TSV, populate fields
   - Real-time fee calculation + nudge display
8. **Step 4**: 2D Print Config Editor (Direct Manipulation)
   - Ảnh sản phẩm làm canvas background (front view mặc định)
   - View toggle: [Mặt trước] [Mặt sau] [Tay áo] [Quần]
   - Independent editable elements: Club Logo, Sponsor Logo, Team Name, Player Number, Player Name
   - Click element → hiện bounding box + handles
   - Drag để move, corner handles để resize (giữ aspect ratio), edge handles để scale 1 chiều
   - Rotate handle: kéo tự do, snap tại 0°/90°/180°/270° (dừng nhẹ, vẫn có thể tiếp tục)
   - Không có UI buttons (không zoom/rotate/resize buttons)
9. **Step 5**: 2D Mockup Preview
   - Hiện ảnh sản phẩm thật + overlay design elements (CSS absolute positioning)
   - View switcher: [Mặt trước] [Mặt sau] [Quần]
   - Player selector dropdown: chọn cầu thủ → xem tên/số của cầu thủ đó
   - Read-only — không edit trong preview (Back để quay về Step 4)
   - Disclaimer nhỏ: "Màu sắc thực tế có thể khác nhẹ so với màn hình"
10. **Step 6**: Add to cart
    - Tạo CustomCartItem trong Zustand store
    - Redirect đến /cart
11. Cart page: custom items hiện bundle summary + [Sửa] button → navigate lại builder với pre-filled data
12. Edit flow: builder load từ CustomCartItem có sẵn, update in-place khi re-submit

## Todo
- [ ] Builder page route + layout
- [ ] React Hook Form setup với multi-step
- [ ] Step navigation component (stepper)
- [ ] Step 1: Color set selector
- [ ] Step 2: Logo upload với Cloudinary integration
- [ ] API route for Cloudinary upload
- [ ] Client-side image dimension validation
- [ ] Step 3: Player list table với useFieldArray
- [ ] Excel paste handler (Clipboard API + TSV parser)
- [ ] Printing fee calculation + nudge component
- [ ] Step 4: 2D Print Config Editor
  - [ ] EditableElement component (text + image types)
  - [ ] SelectionHandles (resize handles + rotate handle)
  - [ ] Snap system (0°/90°/180°/270°)
  - [ ] Direct manipulation: drag move, corner/edge resize, rotate
  - [ ] View toggle (front/back/sleeve/shorts)
- [ ] Step 5: 2D Mockup Preview
  - [ ] Overlay design trên ảnh sản phẩm thật
  - [ ] View switcher (front/back/shorts)
  - [ ] Player selector dropdown
- [ ] Step 6: Add to cart với CustomCartItem
- [ ] [Sửa] button trong cart → re-open builder với existing data
- [ ] Mobile responsive cho tất cả steps
- [ ] No vertical scroll — fit viewport

## Success Criteria
- Complete builder flow: color → logos → players → print config → mockup preview → add to cart
- Step 4: direct manipulation KHÔNG có UI buttons (click/select/drag/resize/rotate only)
- Step 4: Name, Number, Logo là 3 objects độc lập, chỉnh sửa riêng biệt
- Step 4: snap rotation tại góc chuẩn với smooth transitions
- Step 5: design hiển thị overlay chính xác lên ảnh sản phẩm thật
- Step 5: view switcher + player selector hoạt động đúng
- Paste 10 dòng từ Excel → bảng populate đúng
- Printing fee nudge hiển thị đúng tại từng tier
- Cart [Sửa] → mở lại builder với đầy đủ data cũ

## Risk
- Cloudinary free tier: 500 transforms/month → validate client-side trước, batch upload
- Player list lớn (50+) có thể làm chậm useFieldArray → test performance, consider virtualization
- Paste format khác nhau giữa Excel/Google Sheets/Numbers → test cả 3, handle edge cases
- **Mobile touch support**: Desktop-first implementation, touch event equivalents (touchstart/touchmove/touchend) sẽ được implement sau khi desktop hoàn thiện
- Viewport nhỏ → responsive scaling phải smooth, không break interaction

## Security
- Server-side validation khi upload (file type, size limit 5MB)
- Sanitize player names (XSS prevention)
- Rate limit upload endpoint

## Interaction Detail — Step 4 (Print Config Editor)

**Selection:**
- Click element → hiện bounding box
- Bounding box: 4 corner resize handles, 4 edge resize handles, 1 rotate handle (trên element)
- Click outside → deselect

**Move:** Drag element bất kỳ trên canvas, giới hạn trong canvas boundaries

**Resize:**
- Corner handles: scale width + height đồng thời (giữ aspect ratio)
- Edge handles: scale 1 chiều duy nhất
- Minimum size: 20×20px

**Rotate:**
- Drag rotate handle tự do (0–360°)
- Snap nhẹ tại 0°, 90°, 180°, 270° — vẫn tiếp tục nếu user kéo qua
- Hiện góc hiện tại (ví dụ "45°") khi đang drag

**UI Rules:**
- ❌ Không có button zoom/rotate/resize
- ✅ Chỉ tương tác trực tiếp trên element
- ✅ No vertical scroll — fit viewport hoàn toàn
- ✅ Hover cursor thay đổi theo context (move, resize, rotate)
