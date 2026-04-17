---
phase: 10
title: "Product Catalog + Cart + Checkout"
status: pending
priority: P1
effort: 6d
---

# Phase 10: Product Catalog + Cart + Checkout

## Context
- Depends on Phase 2 (auth — user must be logged in for checkout), Phase 4 (builder sections)
- [Brainstorm](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md) — Luồng 1: Mua thường

<!-- Session 10 (2026-04-17):
  - Phase 3 scope trimmed → Phase 10 now OWNS: listing + detail + cart + checkout (Phase 3 = homepage only)
  - Cart sync to DB on login: new CartItem table, /api/cart/merge endpoint
  - POST /api/orders owned here (Phase 6 provides email/Excel handlers)
  - Stock: display only, no validation at order time
  - Effort 5d → 6d due to cart sync work
-->

## Overview
Product listing with search + filters, detail page with stock display, cart (Zustand), checkout with auto-fill capability. Phase 10 = Product Catalog + Cart + Checkout (5d). Phase 3 = Homepage only (2d). Order confirmation displays order summary without mockup canvas (read-only Phase 5 component).

<!-- Session 8 updates:
- Stock display per (ColorSet + size), not just per Product
- Search + Filter: AND logic, URL params ?q=...&category=...
- Order confirmation: mã đơn + tổng tiền + player size summary (no mockup canvas)
- Checkout: auto-fill from User.shippingName/Phone/Address; save after successful checkout
-->

## Key Insights
- Products with `customizable: true`: trang product detail hiển thị **builder sections** (gallery → color select → logo upload → mockup preview → player list → add to cart) — KHÔNG có nút [Custom] riêng, KHÔNG redirect sang route khác
- Products with `customizable: false` (giày, phụ kiện): trang detail thông thường (gallery, size, add to cart)
- Cart = client-side Zustand + localStorage (guest) + **DB sync on login** (Session 10): `CartItem` table, merge localStorage with server cart on login via `/api/cart/merge`
- **Print fee nudge** hiển thị per-CustomOrder riêng lẻ trong cart (không pool tổng). Mỗi custom item có nudge riêng: `<6 bộ +20% | 6-9 bộ +10% | 10+ miễn phí in`. Print fee scope: chỉ tính trên giá CustomOrder, không tính sản phẩm thường.
- **Phase 3 checkout scope**: POST `/api/orders` tạo Order + OrderItems cho **normal items chỉ**. Phase 6 extends API này thêm CustomOrder handling. Cart có thể chứa mixed items (normal + custom) — checkout 1 lần, 1 đơn hàng.
- **Order cancel**: khách có thể hủy đơn khi status = `pending` từ trang order history; Phase 6 thêm cancel email notify shop
- **Stock display**: Show stock per (ColorSet + size) trên product detail — NOT per Product. Selected ColorSet's variants show stock. Khi `stock = 0` cho 1 size: hiện "Hết hàng" + disable size chip đó (không ẩn). Sản phẩm vẫn visible trong catalog. **Session 10**: display-only — checkout KHÔNG validate stock, admin quản lý thủ công.
- **Mobile responsive**: All pages mobile-friendly; **builder sections desktop-only** (show notice < 768px)
- **Multiple CustomOrders in cart**: Khách có thể add nhiều đơn đội khác nhau vào cart; mỗi đơn tính print fee độc lập

## Architecture

```
/                       → Homepage (featured products + categories)
/products               → Product listing with filters (category, price)
/products/[slug]        → Product detail
                           - customizable=false: gallery, size, add to cart
                           - customizable=true:  gallery + builder sections (Phase 4)
/cart                   → Cart page (normal items + custom bundles + print fee nudge)
/checkout               → Checkout form (shipping + COD) — requires auth
/orders/[id]            → Order detail + cancel button (if status=pending)
/orders                 → Order history
```

## Related Code Files

### Create
- `src/app/page.tsx` — homepage with hero + featured products
- `src/app/products/page.tsx` — listing with filters
- `src/app/products/[slug]/page.tsx` — product detail (RSC + builder sections in Phase 4)
- `src/app/cart/page.tsx` — cart page with print fee nudge
- `src/app/checkout/page.tsx` — checkout form (name, phone, address, note)
- `src/app/orders/page.tsx` — order history
- `src/app/orders/[id]/page.tsx` — order detail + cancel button
- `src/app/api/orders/[id]/cancel/route.ts` — PATCH cancel order (pending only)
- `src/components/product/product-card.tsx` — card (customizable = badge "Đồng phục tùy chỉnh")
- `src/components/product/product-gallery.tsx` — image gallery with color set switching
- `src/components/product/size-selector.tsx` — size picker
- `src/components/product/color-set-selector.tsx` — color set picker (image swatches)
- `src/components/cart/cart-icon.tsx` — header cart icon with count
- `src/components/cart/cart-item-row.tsx` — single cart item
- `src/components/cart/print-fee-nudge.tsx` — print fee tier display + nudge message
- `src/components/checkout/checkout-form.tsx` — shipping + COD form
- `src/stores/cart-store.ts` — Zustand cart store with localStorage persistence + login-triggered merge hook
- `src/app/api/products/route.ts` — product listing API (if needed for client filtering)
- `src/app/api/cart/merge/route.ts` — **Session 10**: POST endpoint merges localStorage cart (from client body) with server `CartItem` records for userId; union strategy (de-dup by productId+colorSetId+size); returns merged cart
- `src/app/api/cart/route.ts` — GET/POST/DELETE server cart items (authenticated users only)
- `src/app/api/orders/route.ts` — **Session 10**: POST order creation (owned by Phase 10). Uses Phase 6's email/Excel handlers under the hood.

## Cart Store Design (Zustand)

```typescript
// Two item types in cart:
interface NormalCartItem {
  type: 'normal'
  id: string
  productId: number
  productName: string
  productImage: string
  colorSetName: string
  size: string
  quantity: number
  unitPrice: number
}

interface CustomCartItem {
  type: 'custom'
  id: string
  // Full custom builder state — defined in Phase 4
  builderData: CustomBuilderData
  summary: string // "Đội ABC FC — 12 bộ"
  totalPrice: number
}

type CartItem = NormalCartItem | CustomCartItem
```

## Implementation Steps

1. Create `src/stores/cart-store.ts` — Zustand with persist middleware (localStorage). **Session 10**: add `mergeFromServer(serverItems)` action — called after login; union localStorage + server items, de-dup by `(productId, colorSetId, size, type)`; POST result back via `/api/cart/merge` → commits to DB.
2. Build product listing page — RSC fetching from Prisma, category filter. Add **search + filter**: URL params `?q=...&category=...` (AND logic). Search: `WHERE name ILIKE '%query%'` (server-side Postgres, no Elasticsearch). Stock display per (ColorSet + size) in product cards.
3. Build product card — image, name, price; customizable badge
4. Build product detail page — gallery (images per ColorSet+slot), color set switch, size selector with stock per (ColorSet+size); for `customizable=true`: render builder sections (Phase 4); mobile < 768px shows notice. Size selector: show all sizes from ProductVariant; disabled + tooltip "Hết hàng" nếu `stock = 0`. **Auth gate**: `/products/[slug]` is public (no auth required to view); only the add-to-cart action checks session → redirect to `/login?redirect=/products/[slug]` if not authenticated.
5. Build `print-fee-nudge.tsx` — displays fee tier based on total player count in cart
6. Build cart page — list items (normal + custom bundles), quantity +/- (normal only), [Xóa] per item (custom bundles: xóa rồi vào lại /products/[slug] custom lại — không có nút Sửa), subtotal, print fee nudge per-CustomOrder. Custom bundle cart items show **"Đồng phục tùy chỉnh" badge** (`isCustom`) to distinguish from normal items.
7. Build cart icon in header — item count badge
8. Build checkout page — React Hook Form, name/phone/address/note fields, COD confirmation. **Auto-fill from User.shippingName/Phone/Address if exists**. Save shipping info to User after successful order. POST `/api/orders` (owned here — Session 10) tạo normal OrderItems + CustomOrders; calls Phase 6 email/Excel handlers.
8a. Build `/api/cart/merge` — **Session 10**: called once after login. Accepts localStorage cart array, reads server `CartItem` records for userId, unions + de-dups, upserts to DB, clears client localStorage, returns merged cart state.
9. Build order confirmation page — shows mã đơn, tổng tiền, player size summary per custom bundle (no mockup canvas — that's readonly in Phase 5 admin)
10. Build order history page — list orders with status badges
11. Build order detail page — full info + cancel button (if status=pending)
12. Create cancel order API route — verify ownership + pending status, set `cancelled`

## Todo
- [ ] Zustand cart store with persist + server sync merge action (Session 10)
- [ ] POST /api/cart/merge endpoint (login-triggered union)
- [ ] CartItem CRUD endpoints (authenticated users)
- [ ] POST /api/orders endpoint (owns order creation, delegates Phase 6 email/Excel)
- [ ] Product listing page with search + filter (AND logic, URL params)
- [ ] Product card component with stock per (ColorSet+size)
- [ ] Product detail page with gallery, stock display per (ColorSet+size)
- [ ] Color set selector (swap images + stock on pick)
- [ ] Size selector with stock per variant
- [ ] Add to cart functionality (normal products)
- [ ] Print fee nudge component
- [ ] Cart page with quantity controls + print fee nudge
- [ ] Cart icon with badge in header
- [ ] Checkout form with auto-fill + save shipping info
- [ ] Order confirmation page (mã đơn, tổng tiền, player summary)
- [ ] Order history page
- [ ] Order detail page
- [ ] Cancel order API + UI (pending only)
- [ ] Mobile notice for builder (< 768px)
- [ ] Responsive layout for all non-builder pages

## Success Criteria
- Browse products → view detail → select size/color → add to cart → see in cart
- **Stock display visible on product detail page** (stock quantity per size)
- Cart persists across page reload (localStorage)
- Print fee nudge shows correct tier based on player count
- Checkout form validates and collects name/phone/address/note
- Order history shows all user orders with status
- Customer can cancel pending order; cannot cancel after processing
- Mobile responsive (except builder sections — show notice instead)

## Risk
- Image loading performance → use Next.js Image component with Cloudinary loader
- Cart state migration if schema changes → use Zustand version + migrate
