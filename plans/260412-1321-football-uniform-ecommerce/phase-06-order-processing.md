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
- POST `/api/orders` — create order from cart data (user must be authenticated — Phase 2)
- Generate unique order number (ORD-YYYYMMDD-NNN)
- For custom orders: generate Excel file with player list
- Email to shop: order info + Excel + logo files + print config (admin views mockup preview via Phase 5 CSS overlay component)
- Email to customer: order confirmation with summary
- Order status tracking (basic: new → confirmed → processing → shipped → delivered)

## Architecture

```
Checkout Submit → POST /api/orders
  ├── Validate cart data + session (userId from auth)
  ├── Create Order + OrderItems in DB
  ├── For each CustomCartItem:
  │   ├── Create CustomOrder + Players in DB
  │   ├── Generate Excel (ExcelJS) → upload to Cloudinary
  │   └── Save printConfigJson from builder state
  ├── Calculate printing surcharge
  ├── Send email to shop (Resend) with attachments
  ├── Send email to customer (Resend)
  └── Return order confirmation

Admin Dashboard:
  ├── GET /admin/orders - List all orders with status
  ├── GET /admin/orders/[id] - Order details (same as email content)
  ├── PUT /admin/orders/[id]/status - Update order status
  └── POST /admin/orders/[id]/notes - Add admin notes
```

## Excel Template Design

| STT | Tên cầu thủ | Số áo | Size áo | Size quần |
|-----|-------------|-------|---------|-----------|
| 1   | Nguyễn Văn A | 10   | L       | M         |

Headers styled: bold white text, blue background. Auto-width columns.

## Email to Shop Content
- Subject: `[Đơn mới] ORD-20260412-001 — Đội ABC FC — 12 bộ`
- Body: customer info, product name, color set, print config summary, total + surcharge
- Attachments: Excel file, logo files (from Cloudinary URLs)
- Note: Admin xem mockup preview trong dashboard (Phase 5 component)

## Related Code Files

### Create
- `src/app/api/orders/route.ts` - POST handler
- `src/app/admin/orders/page.tsx` - admin order list
- `src/app/admin/orders/[id]/page.tsx` - admin order details
- `src/app/api/admin/orders/[id]/status/route.ts` - update order status
- `src/app/api/admin/orders/[id]/notes/route.ts` - add admin notes
- `src/components/admin/order-list-table.tsx` - order list with status badges
- `src/components/admin/order-detail-view.tsx` - detailed order view
- `src/components/admin/status-update-form.tsx` - status update form
- `src/lib/order-number.ts` - generate sequential order number
- `src/lib/excel-generator.ts` - ExcelJS player list generator
- `src/lib/email/send-shop-notification.ts` - shop email with Resend
- `src/lib/email/send-customer-confirmation.ts` - customer email
- `src/lib/email/templates/shop-order-email.tsx` - React Email template (shop)
- `src/lib/email/templates/customer-order-email.tsx` - React Email template (customer)
- `src/app/order-confirmation/[orderId]/page.tsx` - confirmation page

## Implementation Steps

1. Create `order-number.ts` — query DB for today's max order number, increment
2. Create `excel-generator.ts` — ExcelJS workbook with styled player list, returns Buffer
3. Create `POST /api/orders`:
   - Validate request body (zod schema)
   - Get userId from session (required — Phase 2 ensures auth)
   - Transaction: create Order → OrderItems → CustomOrder → Players
   - Save printConfigJson from builder state into CustomOrder
   - Calculate printing surcharge per custom order
   - Generate Excel per custom order → upload to Cloudinary → store URL
   - Return orderId
4. Create shop email template — React Email, includes all order info
5. Create customer email template — simpler confirmation
6. Create `send-shop-notification.ts` — Resend API call with Excel + logo attachments
7. Create `send-customer-confirmation.ts` — Resend API call
8. Trigger emails after order creation (in API route, non-blocking)
9. Create order confirmation page — shows order number, summary, "email đã được gửi"
10. Update checkout page to call `/api/orders` and redirect to confirmation
11. **Admin Dashboard**: Create order list page with filtering and status badges
12. **Admin Dashboard**: Create order detail page showing same content as email
13. **Admin Dashboard**: Create API routes for status updates and notes
14. **Admin Dashboard**: Add authentication middleware for admin routes

## Todo
- [ ] Order number generator
- [ ] ExcelJS player list generator
- [ ] POST /api/orders endpoint with validation
- [ ] Order creation in DB (transaction) with userId from session
- [ ] Save printConfigJson from builder state into CustomOrder
- [ ] Printing surcharge calculation in order
- [ ] Excel upload to Cloudinary
- [ ] Shop notification email (Resend + React Email)
- [ ] Customer confirmation email
- [ ] Order confirmation page
- [ ] Wire checkout form → API → confirmation redirect
- [ ] Error handling (payment failed, email failed)
- [ ] **Admin Dashboard**: Order list page with status filtering
- [ ] **Admin Dashboard**: Order detail page with full information
- [ ] **Admin Dashboard**: API routes for status/notes updates
- [ ] **Admin Dashboard**: Authentication middleware for admin access

## Success Criteria
- Checkout → order created in DB with correct totals + userId
- Excel file generated with all players, proper formatting
- Shop receives email with Excel + logo attachments within 30s
- Customer receives confirmation email
- Order confirmation page shows order number
- **Admin Dashboard**: Shop can view all orders and update status
- **Admin Dashboard**: Order details show same content as email attachments

## Risk
- Resend rate limits (free: 100 emails/day) → sufficient for MVP
- Large Excel files with 50+ players → test performance, should be fine with ExcelJS streaming
- Email attachment size limits (Resend: 40MB total) → logos should be well under

## Security
- Validate all input server-side (zod)
- Sanitize player names in Excel output
- Don't expose internal order IDs in URLs (use order number)
- Verify userId from session server-side (not from request body)
