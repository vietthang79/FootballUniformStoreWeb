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
Server-side order creation, Excel player list generation, email to shop with all order details + attachments. Customer sees success notification on screen. Status updates trigger customer emails from Phase 8 admin.

## Requirements
- POST `/api/orders` — **extends Phase 3 route** to handle both normal OrderItems + CustomOrders in 1 transaction. Cart can mix both types; result = 1 Order.
- Generate unique order number: `ORD-YYYYMMDD-NNN` — **NNN resets to 001 daily** (COUNT orders today + 1)
- For custom orders: generate **2-sheet Excel** (Sheet 1: order info; Sheet 2: player list)
- Server-side **print fee validation** — recalculate per CustomOrder from `players.length`, don't trust client. Each CustomOrder calculated independently.
- Email to shop: order info + Excel download link + **all logo URLs** (from `logos[]` array in printConfigJson)
- KHÔNG gửi email to customer — only shop email. Customer thấy thông báo trên màn hình
- Customer status emails: gửi từ Phase 8 admin PATCH status API khi status → "shipping" hoặc "delivered"
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
  │   └── Generate 2-sheet Excel → upload to Cloudinary (raw resource) → save URL in CustomOrder.excelFileUrl
  ├── Send email to shop (Resend):
  │   └── Body includes download link to excelFileUrl (Cloudinary raw URL) + logo URLs from logos[].url
  └── Return order confirmation (customer sees success notification on screen)

Customer Cancel → PATCH /api/orders/[id]/cancel
  ├── Verify userId + status=pending
  ├── Set status = 'cancelled'
  ├── Send cancel email to shop: "[Đơn huỷ] ORD-xxx — Tên khách"
  └── Return updated order

Admin Status Update → PATCH /api/admin/orders/[id]/status (Phase 8)
  ├── Verify admin auth
  ├── Update order status
  ├── If status = "shipping" or "delivered": trigger customer email (Resend)
  └── Return updated order
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
- Download link: Excel file + **tất cả logo URLs** (`logos[].url` từ printConfigJson per CustomOrder)
- Note: Admin xem mockup preview trong dashboard (Phase 5 component)

## Customer Status Emails (triggered from Phase 8 admin API)
- **Shipping**: Subject `[Đơn đang giao] ORD-xxx`, body: tracking info (if available)
- **Delivered**: Subject `[Đơn đã giao] ORD-xxx`, body: confirmation + thank you

## Cancel Email to Shop Content
- Subject: `[Đơn huỷ] ORD-20260412-001 — Nguyễn Văn A`
- Body: thông tin khách (tên, SĐT), tóm tắt đơn (sản phẩm, số bộ nếu custom), thời gian huỷ
- Trigger: customer gọi PATCH /api/orders/[id]/cancel (status pending → cancelled)

## Related Code Files

### Create
- `src/app/api/orders/route.ts` — POST: handle normal items + custom orders in 1 transaction
- `src/app/api/orders/[id]/cancel/route.ts` — PATCH cancel (pending only, verify ownership, send cancel email to shop)
- `src/app/api/admin/orders/[id]/status/route.ts` — PATCH order status (admin only, send customer email on shipping/delivered)
- `src/lib/order-number.ts` — generate sequential order number (unique constraint + retry on conflict)
- `src/lib/excel-generator.ts` — ExcelJS 2-sheet workbook generator
- `src/lib/email/send-shop-notification.ts` — shop email with Resend (+ logo URLs)
- `src/lib/email/send-shop-cancel-notification.ts` — shop cancel alert email
- `src/lib/email/send-customer-status-notification.ts` — customer email for shipping/delivered (called from Phase 8 API)
- `src/lib/email/templates/shop-order-email.tsx` — React Email template (shop, mixed order)
- `src/lib/email/templates/shop-cancel-email.tsx` — React Email template (cancel alert)
- `src/lib/email/templates/customer-shipping-email.tsx` — React Email template (shipping notification)
- `src/lib/email/templates/customer-delivered-email.tsx` — React Email template (delivered notification)
- `src/app/order-confirmation/[orderId]/page.tsx` — confirmation page

## Implementation Steps

1. Create `order-number.ts` — COUNT today's orders in DB, return `ORD-YYYYMMDD-{NNN padded to 3 digits}` (resets daily). Wrap order creation in try/catch: on unique constraint violation for orderNumber → retry up to 3 times with incremented count.
2. Create `excel-generator.ts` — ExcelJS 2-sheet workbook (Sheet 1: order info, Sheet 2: players) with styled headers, returns Buffer. Update structure reference to use `overlays: OverlayElement[][]` if applicable.
3. Extend `POST /api/orders` (Phase 3 route):
   - Validate request body (zod schema — both normal items + custom items)
   - Get userId from session
   - Transaction: create Order → OrderItems (normal) → CustomOrder + Players (custom)
   - For each CustomOrder: **server-side recalculate print fee independently** from `players.length` using `printing-fee.ts`
   - Save `printConfigJson` (incl. `logos[]`) from builder state into CustomOrder
   - Generate 2-sheet Excel per custom order → upload to Cloudinary as **raw resource** → store URL in `CustomOrder.excelFileUrl`
   - Return orderId
4. Create `PATCH /api/orders/[id]/cancel`:
   - Verify `userId` from session matches `order.userId`
   - Verify `order.status === 'pending'`
   - Set `status = 'cancelled'`
   - **Trigger `send-shop-cancel-notification.ts`** (non-blocking)
   - Return updated order
5. Create `PATCH /api/admin/orders/[id]/status` (Phase 8 API):
   - Verify admin auth
   - Update status in DB
   - If status = "shipping" or "delivered": call `send-customer-status-notification.ts` (non-blocking)
   - Return updated order
6. Create shop order email template — React Email, mixed order (normal items + custom orders), logo URLs from `logos[].url`
7. Create shop cancel email template — React Email, order summary + customer contact
8. Create customer shipping email template — React Email, "Đơn đang giao"
9. Create customer delivered email template — React Email, "Đơn đã giao" + thank you
10. Create `send-shop-notification.ts` — Resend API call; email body includes **download link** to `excelFileUrl` (Cloudinary raw URL) + logo URLs. No Excel attachment.
11. Create `send-shop-cancel-notification.ts` — Resend API call for cancel alert
12. Create `send-customer-status-notification.ts` — Resend API call for shipping/delivered (template selection based on status)
13. Trigger emails after order creation/cancel (non-blocking, log errors but don't block)
14. Create order confirmation page — shows order number, summary, success notification on screen
15. Update checkout page to call `/api/orders` → redirect to confirmation

## Todo
- [ ] Order number generator (daily reset, ORD-YYYYMMDD-NNN)
- [ ] ExcelJS 2-sheet generator (Sheet 1: order info, Sheet 2: players)
- [ ] Extend POST /api/orders — mixed normal + custom items, per-CustomOrder print fee recalculation
- [ ] Save printConfigJson (incl. logos[]) from builder state into CustomOrder
- [ ] Excel upload to Cloudinary per CustomOrder
- [ ] PATCH /api/orders/[id]/cancel — ownership check + pending status + cancel email to shop
- [ ] Shop order email (Resend + React Email, logo URLs)
- [ ] Shop cancel alert email
- [ ] Order confirmation page (success notification on screen)
- [ ] Wire checkout form → API → confirmation redirect
- [ ] Error handling (email failed → log, don't block order)
- [ ] PATCH /api/admin/orders/[id]/status API (auth guard — Phase 8 UI will consume this)
- [ ] Customer status email templates (shipping/delivered) + send function

## Success Criteria
- Checkout → order created in DB with correct totals (server-side print fee) + userId
- 2-sheet Excel file generated: Sheet 1 order info, Sheet 2 players, proper formatting
- Shop receives email with download link + logo URLs within 30s
- Customer sees success notification on screen (NO email sent to customer at checkout)
- Order confirmation page shows order number
- Customer can cancel pending order from `/orders/[id]`; cannot cancel after processing
- Admin API endpoint functional for status update (auth guard). Sending customer emails on status change in Phase 8

## Risk
- Resend rate limits (free: 100 emails/day) → sufficient for MVP
- Large Excel files with 50+ players → test performance, should be fine with ExcelJS streaming
- Email attachment size limits (Resend: 40MB total) → logos should be well under

## Security
- Validate all input server-side (zod)
- Sanitize player names in Excel output
- Don't expose internal order IDs in URLs (use order number)
- Verify userId from session server-side (not from request body)
