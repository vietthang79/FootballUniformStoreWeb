---
phase: 3
title: "Custom Builder UI + Logic"
status: pending
priority: P1
effort: 7d
---

# Phase 3: Custom Builder UI + Logic

## Context
- Depends on Phase 2 (product detail, cart store)
- [Custom Builder Research](../reports/researcher-260412-1321-custom-builder.md)
- [Brainstorm — Custom Builder Flow](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md)

## Overview
Multi-step builder: color selection → logo upload → player list → print config → preview → add to cart. Core differentiator of this e-commerce.

## Key Insights
- Builder pre-loaded with product being viewed (passed via URL param or state)
- React Hook Form + FormProvider across all steps — single form, multi-step navigation
- useFieldArray for player list (dynamic rows)
- Clipboard API for Excel paste into player table
- Logo upload to Cloudinary with client-side dimension validation
- Printing fee calculated real-time based on player count
- Builder state saved to Zustand when added to cart (as CustomCartItem)

## Architecture

```
/products/[slug]/custom  → Custom Builder page

UX: Step wizard — progress bar (step 1/6), Back/Next buttons per step.
    shadcn/ui Tabs or custom stepper component.

Steps (single React Hook Form + FormProvider):
  Step 1: ColorSetSelector — pick from product's predefined color sets
  Step 2: LogoUploader — upload club logo, sponsor, league, flag, enter team name
  Step 3: PlayerListEditor — inline table, paste from Excel, add/remove rows
  Step 4: PrintConfigEditor — checkboxes for front/back/sleeve/shorts
  Step 5: MockupPreview — (Phase 4, placeholder here)
  Step 6: AddToCart — summary + add button
```
<!-- Updated: Validation Session 1 - Step wizard UX confirmed; sleeve logo side: user-selectable (left/right/both) -->

## Form Data Type

```typescript
interface CustomBuilderData {
  productId: string
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

  // Step 4 — print config
  printConfig: {
    front: { clubLogo: boolean; sponsorLogo: boolean; smallNumber: boolean }
    sleeve: {
      leagueLogo: boolean; leagueLogoSide: 'left' | 'right' | 'both'
      sleeveSponsor: boolean; sleeveSponsorSide: 'left' | 'right' | 'both'
      flagPatch: boolean; flagPatchSide: 'left' | 'right' | 'both'
    }
    back: { teamName: boolean; backNumber: boolean; playerName: boolean }
    shorts: { printNumber: boolean; side: 'left' | 'right' }
  }
}
```

## Printing Fee Logic

```typescript
function calculatePrintingSurcharge(playerCount: number, subtotal: number): { rate: number; amount: number; nudge: string | null } {
  if (playerCount >= 10) return { rate: 0, amount: 0, nudge: null }
  if (playerCount >= 6) {
    const needed = 10 - playerCount
    return { rate: 0.1, amount: subtotal * 0.1, nudge: `Thêm ${needed} bộ để miễn phí in!` }
  }
  const neededFor10 = 6 - playerCount
  return { rate: 0.2, amount: subtotal * 0.2, nudge: `Thêm ${neededFor10} bộ để giảm phí in xuống 10%` }
}
```

## Related Code Files

### Create
- `src/app/products/[slug]/custom/page.tsx` — builder page
- `src/components/custom-builder/builder-stepper.tsx` — step navigation UI
- `src/components/custom-builder/step-color-select.tsx` — Step 1
- `src/components/custom-builder/step-logo-upload.tsx` — Step 2
- `src/components/custom-builder/step-player-list.tsx` — Step 3
- `src/components/custom-builder/step-print-config.tsx` — Step 4
- `src/components/custom-builder/step-preview.tsx` — Step 5 (placeholder)
- `src/components/custom-builder/step-add-to-cart.tsx` — Step 6
- `src/components/custom-builder/player-table.tsx` — editable table with paste
- `src/components/custom-builder/logo-uploader.tsx` — single logo upload widget
- `src/components/custom-builder/printing-fee-nudge.tsx` — fee display + nudge
- `src/lib/printing-fee.ts` — fee calculation logic
- `src/lib/clipboard-parser.ts` — Excel TSV paste parser
- `src/app/api/upload/route.ts` — Cloudinary upload endpoint

### Modify
- `src/stores/cart-store.ts` — add CustomCartItem support
- `src/components/cart/cart-item-row.tsx` — render custom bundles with [Sửa] button

## Implementation Steps

1. Create builder page route `/products/[slug]/custom`
2. Setup React Hook Form with FormProvider + defaultValues from product
3. Build step navigation (stepper UI with back/next, step validation)
4. **Step 1**: Color set selector — grid of color set images, click to select
5. **Step 2**: Logo upload widget - dropzone, Cloudinary upload, dimension validation (warn <500px), preview thumbnail, auto-generate quality note for low-res logos
6. Create `/api/upload` route — receives file, uploads to Cloudinary, returns URL
7. **Step 3**: Player list table
   - useFieldArray for dynamic rows
   - Columns: STT (auto), Tên cầu thủ, Số áo, Size áo (dropdown), Size quần (dropdown)
   - Add row button, remove row button per row
   - Paste handler: listen for paste event on table container, parse TSV, populate fields
   - Real-time fee calculation + nudge display
8. **Step 4**: Print config checkboxes grouped by area (front/sleeve/back/shorts)
   - Shorts number side radio (left/right) shown only when printNumber checked
   - Visual hint: small diagram showing where each element prints
9. **Step 5**: Placeholder — "Preview sẽ hiển thị ở đây" (implemented in Phase 4)
10. **Step 6**: Summary — product name, color, player count, printing fee, total price, [Thêm vào giỏ hàng]
11. Add to cart → creates CustomCartItem in Zustand store → redirect to /cart
12. Cart page: custom items show bundle summary + [Sửa] button → navigates back to builder with pre-filled data
13. Edit flow: builder loads from existing CustomCartItem, updates in place on re-submit

## Todo
- [ ] Builder page route + layout
- [ ] React Hook Form setup with multi-step
- [ ] Step navigation component (stepper)
- [ ] Step 1: Color set selector
- [ ] Step 2: Logo upload with Cloudinary integration
- [ ] API route for Cloudinary upload
- [ ] Client-side image dimension validation
- [ ] Step 3: Player list table with useFieldArray
- [ ] Excel paste handler (Clipboard API + TSV parser)
- [ ] Printing fee calculation + nudge component
- [ ] Step 4: Print config checkboxes
- [ ] Step 5: Preview placeholder
- [ ] Step 6: Summary + add to cart
- [ ] Custom item in cart with [Sửa] button
- [ ] Edit flow (re-open builder with existing data)
- [ ] Mobile responsive for all steps

## Success Criteria
- Complete builder flow from color → logos → players → config → cart
- Paste 10 rows from Excel → populates table correctly
- Fee nudge updates real-time as players added/removed
- [Sửa] in cart re-opens builder with all data preserved
- Form validation prevents empty required fields per step

## Risk
- Cloudinary upload rate limits on free tier (500 transforms/month) → batch uploads, validate client-side first
- Large player lists (50+) may slow useFieldArray → test performance, consider virtualization if needed
- Paste format varies between Excel/Google Sheets/Numbers → test all three, handle edge cases in parser

## Security
- Server-side upload validation (file type, size limit 5MB)
- Sanitize player names (XSS prevention)
- Rate limit upload endpoint
