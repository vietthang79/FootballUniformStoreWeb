---
phase: 3
title: "Product Catalog + Normal E-Commerce"
status: pending
priority: P1
effort: 5d
---

# Phase 3: Product Catalog + Normal E-Commerce Flow

## Context
- Depends on Phase 2 (auth — user must be logged in for checkout)
- [Brainstorm](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md) — Luồng 1: Mua thường

## Overview
Product listing, detail page, size/color selection, cart (Zustand), checkout page (without payment).

## Key Insights
- Products with `customizable: true`: trang product detail hiển thị **builder sections** (gallery → color select → logo upload → mockup preview → player list → add to cart) — KHÔNG có nút [Custom] riêng, KHÔNG redirect sang route khác
- Products with `customizable: false` (giày, phụ kiện): trang detail thông thường (gallery, size, add to cart)
- Cart = client-side Zustand + localStorage, no server cart
- **Print fee nudge** hiển thị per-CustomOrder riêng lẻ trong cart (không pool tổng). Mỗi custom item có nudge riêng: `<6 bộ +20% | 6-9 bộ +10% | 10+ miễn phí in`. Print fee scope: chỉ tính trên giá CustomOrder, không tính sản phẩm thường.
- **Phase 3 checkout scope**: POST `/api/orders` tạo Order + OrderItems cho **normal items chỉ**. Phase 6 extends API này thêm CustomOrder handling. Cart có thể chứa mixed items (normal + custom) — checkout 1 lần, 1 đơn hàng.
- **Order cancel**: khách có thể hủy đơn khi status = `pending` từ trang order history; Phase 6 thêm cancel email notify shop
- **Stock display**: Show stock per size trên product detail. Khi `stock = 0` cho 1 size: hiện "Hết hàng" + disable size chip đó (không ẩn). Sản phẩm vẫn visible trong catalog.
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
- `src/stores/cart-store.ts` — Zustand cart store
- `src/app/api/products/route.ts` — product listing API (if needed for client filtering)

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

1. Create `src/stores/cart-store.ts` — Zustand with persist middleware
2. Build product listing page — RSC fetching from Prisma, category filter, stock display per size
3. Build product card — image, name, price; customizable badge
4. Build product detail page — gallery, color set switch, size selector, stock display; for `customizable=true`: render builder section slots (implemented in Phase 4); mobile < 768px shows "Vui lòng dùng máy tính để thiết kế". Size selector: show all sizes from ProductVariant; disabled + tooltip "Hết hàng" nếu `stock = 0`
5. Build `print-fee-nudge.tsx` — displays fee tier based on total player count in cart
6. Build cart page — list items (normal + custom bundles), quantity +/- (normal only), [Xóa] per item (custom bundles: xóa rồi vào lại /products/[slug] custom lại — không có nút Sửa), subtotal, print fee nudge per-CustomOrder
7. Build cart icon in header — item count badge
8. Build checkout page — React Hook Form, name/phone/address/note fields, COD confirmation. POST `/api/orders` tạo normal OrderItems; Phase 6 sẽ extend route này cho CustomOrder.
9. Build order history page — list orders with status badges
10. Build order detail page — full info + cancel button (if status=pending)
11. Create cancel order API route — verify ownership + pending status, set `cancelled`

## Todo
- [ ] Zustand cart store with persist
- [ ] Homepage with featured products grid
- [ ] Product listing page with category filter
- [ ] Product card component
- [ ] Product detail page with gallery, stock display per size
- [ ] Stock indicator (in stock / low stock / out of stock)
- [ ] Color set selector (swap images on pick)
- [ ] Size selector
- [ ] Add to cart functionality (normal products)
- [ ] Print fee nudge component
- [ ] Cart page with quantity controls + print fee nudge
- [ ] Cart icon with badge in header
- [ ] Checkout form (name, phone, address, note)
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
