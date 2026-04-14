---
phase: 6
title: "Order Processing + Email + Excel Generation"
status: pending
priority: P1
effort: 4d
---

# Phase 6: Order Processing + Email + Excel Generation

## Context
- Depends on Phase 4+5 (custom order data)
- [Tech Stack Research — Email + Excel](../reports/researcher-260412-1321-tech-stack.md)

## Overview
Server-side order creation, Excel player list generation, email to shop with all order details + attachments, and admin dashboard for order management.

## Requirements
- POST `/api/orders` — **extends Phase 3 route** to handle both normal OrderItems + CustomOrders in 1 transaction. Cart can mix both types; result = 1 Order.
- Generate unique order number: `ORD-YYYYMMDD-NNN` — **NNN resets to 001 daily** (COUNT orders today + 1)
- For custom orders: generate **2-sheet Excel** (Sheet 1: order info; Sheet 2: player list)
- Server-side **print fee validation** — recalculate per CustomOrder from `players.length`, don't trust client. Each CustomOrder calculated independently.
- Email to shop: order info + Excel attachment + **all logo files** (from `logos[]` array in printConfigJson)
- Email to customer: order confirmation with summary
- Order status: `pending → processing → shipping → delivered` (+ `cancelled`)
- Customer cancel: PATCH `/api/orders/[id]/cancel` — only when `status = pending`, verify ownership; **send cancel email to shop**

## Architecture

```
Checkout Submit → POST /api/orders  (Phase 3 route extended)
  ├── Validate cart data + session (userId from auth)
  ├── Create Order in DB
  ├── For each NormalCartItem:
  │   └── Create OrderItem in DB
  ├── For each CustomCartItem:
  │   ├── Create CustomOrder + Players in DB
  │   ├── Server-side recalculate print fee (per CustomOrder independently)
  │   ├── Save printConfigJson (incl. logos[]) into CustomOrder
  │   └── Generate 2-sheet Excel → upload to Cloudinary → save URL
  ├── Send email to shop (Resend):
  │   └── Attachments: Excel file + all logo files from logos[].url
  ├── Send email to customer (Resend)
  └── Return order confirmation

Customer Cancel → PATCH /api/orders/[id]/cancel
  ├── Verify userId + status=pending
  ├── Set status = 'cancelled'
  ├── Send cancel email to shop: "[Đơn huỷ] ORD-xxx — Tên khách"
  └── Return updated order
```

Admin Dashboard:
  ├── GET /admin/orders - List all orders with status
  ├── GET /admin/orders/[id] - Order details (same as email content)
  ├── PUT /admin/orders/[id]/status - Update order status
  └── POST /admin/orders/[id]/notes - Add admin notes
```

## Excel Template Design (2 sheets)

**Sheet 1 — Thông tin đơn hàng:**
| Mã đơn | Ngày đặt | Khách hàng | SĐT | Địa chỉ | Tổng tiền | Trạng thái |

**Sheet 2 — Chi tiết cầu thủ:**
| Mã đơn | Sản phẩm | Màu | Tên cầu thủ | Số áo | Size áo | Size quần |

Headers styled: bold white text, blue background (#00AEEF). Auto-width columns.

## Email to Shop Content
- Subject: `[Đơn mới] ORD-20260412-001 — Đội ABC FC — 12 bộ` (teamName fallback = product.name nếu khách không nhập)
- Body: customer info, sản phẩm thường (nếu có), custom orders (tên đội, số bộ, surcharge), total
- Attachments: Excel file + **tất cả logo files** (`logos[].url` từ printConfigJson per CustomOrder)
- Note: Admin xem mockup preview trong dashboard (Phase 5 component)

## Cancel Email to Shop Content
- Subject: `[Đơn huỷ] ORD-20260412-001 — Nguyễn Văn A`
- Body: thông tin khách (tên, SĐT), tóm tắt đơn (sản phẩm, số bộ nếu custom), thời gian huỷ
- Trigger: customer gọi PATCH /api/orders/[id]/cancel (status pending → cancelled)

## Related Code Files

<!-- Session 7: Phase 6 = API + email + Excel only. Admin UI (order list, order detail, status UI) moved to Phase 8 -->

### Create
- `src/app/api/orders/route.ts` — POST: handle normal items + custom orders in 1 transaction
- `src/app/api/orders/[id]/cancel/route.ts` — PATCH cancel (pending only, verify ownership, send cancel email to shop)
- `src/app/api/admin/orders/[id]/status/route.ts` — PATCH order status (admin only)
- `src/app/api/admin/orders/[id]/notes/route.ts` — POST admin notes
- `src/lib/order-number.ts` — generate sequential order number (unique constraint + retry on conflict)
- `src/lib/excel-generator.ts` — ExcelJS 2-sheet workbook generator
- `src/lib/email/send-shop-notification.ts` — shop email with Resend (+ all logo URLs as attachments)
- `src/lib/email/send-shop-cancel-notification.ts` — shop cancel alert email
- `src/lib/email/send-customer-confirmation.ts` — customer email
- `src/lib/email/templates/shop-order-email.tsx` — React Email template (shop, mixed order)
- `src/lib/email/templates/shop-cancel-email.tsx` — React Email template (cancel alert)
- `src/lib/email/templates/customer-order-email.tsx` — React Email template (customer)
- `src/app/order-confirmation/[orderId]/page.tsx` — confirmation page

## Implementation Steps

1. Create `order-number.ts` — COUNT today's orders in DB, return `ORD-YYYYMMDD-{NNN padded to 3 digits}` (resets daily). Wrap order creation in try/catch: on unique constraint violation for orderNumber → retry up to 3 times with incremented count.
2. Create `excel-generator.ts` — ExcelJS 2-sheet workbook (Sheet 1: order info, Sheet 2: players) with styled headers, returns Buffer
3. Extend `POST /api/orders` (Phase 3 route):
   - Validate request body (zod schema — both normal items + custom items)
   - Get userId from session
   - Transaction: create Order → OrderItems (normal) → CustomOrder + Players (custom)
   - For each CustomOrder: **server-side recalculate print fee independently** from `players.length` using `printing-fee.ts`
   - Save `printConfigJson` (incl. `logos[]`) from builder state into CustomOrder
   - Generate 2-sheet Excel per custom order → upload Cloudinary → store URL
   - Return orderId
4. Create `PATCH /api/orders/[id]/cancel`:
   - Verify `userId` from session matches `order.userId`
   - Verify `order.status === 'pending'`
   - Set `status = 'cancelled'`
   - **Trigger `send-shop-cancel-notification.ts`** (non-blocking)
   - Return updated order
5. Create shop order email template — React Email, mixed order (normal items + custom orders), logo attachments from `logos[].url`
6. Create shop cancel email template — React Email, order summary + customer contact
7. Create customer confirmation email template — simpler summary
8. Create `send-shop-notification.ts` — Resend API call with Excel + all logo URLs as attachments
9. Create `send-shop-cancel-notification.ts` — Resend API call for cancel alert
10. Create `send-customer-confirmation.ts` — Resend API call
11. Trigger emails after order creation/cancel (non-blocking, log errors but don't block)
12. Create order confirmation page — shows order number, summary, "email xác nhận đã được gửi"
13. Update checkout page to call `/api/orders` → redirect to confirmation
14. Create admin API routes: PATCH status + POST notes (auth guard — admin only). Send customer email on shipping/delivered.
<!-- Admin UI (order list page, order detail page) → Phase 8 -->

## Todo
- [ ] Order number generator (daily reset, ORD-YYYYMMDD-NNN)
- [ ] ExcelJS 2-sheet generator (Sheet 1: order info, Sheet 2: players)
- [ ] Extend POST /api/orders — mixed normal + custom items, per-CustomOrder print fee recalculation
- [ ] Save printConfigJson (incl. logos[]) from builder state into CustomOrder
- [ ] Excel upload to Cloudinary per CustomOrder
- [ ] PATCH /api/orders/[id]/cancel — ownership check + pending status + cancel email to shop
- [ ] Shop order email (Resend + React Email, all logo attachments from logos[])
- [ ] Shop cancel alert email
- [ ] Customer confirmation email
- [ ] Order confirmation page
- [ ] Wire checkout form → API → confirmation redirect
- [ ] Error handling (email failed → log, don't block order)
- [ ] Admin API routes for status/notes updates (auth guard — Phase 8 UI will consume these)

## Success Criteria
- Checkout → order created in DB with correct totals (server-side print fee) + userId
- 2-sheet Excel file generated: Sheet 1 order info, Sheet 2 players, proper formatting
- Shop receives email with Excel attachment within 30s
- Customer receives confirmation email
- Order confirmation page shows order number
- Customer can cancel pending order from `/orders/[id]`; cannot cancel after processing
- Admin API endpoints functional (status update, notes) — UI in Phase 8

## Risk
- Resend rate limits (free: 100 emails/day) → sufficient for MVP
- Large Excel files with 50+ players → test performance, should be fine with ExcelJS streaming
- Email attachment size limits (Resend: 40MB total) → logos should be well under

## Security
- Validate all input server-side (zod)
- Sanitize player names in Excel output
- Don't expose internal order IDs in URLs (use order number)
- Verify userId from session server-side (not from request body)
