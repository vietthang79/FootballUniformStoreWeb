---
phase: 7
title: "Payment Integration (COD Only)"
status: pending
priority: P1
effort: 1d
---

# Phase 7: Payment Integration (COD Only - MVP)

## Context
- Depends on Phase 6 (order creation flow)
- VNPay + MoMo deferred to Phase 2 (post-launch)
- [Tech Stack Research — Payment](../reports/researcher-260412-1321-tech-stack.md)

## Overview
COD (Cash on Delivery) payment only for MVP. Order created as confirmed immediately → send emails → redirect to confirmation page. No payment gateway integration needed.

## Flow

```
Checkout → Create Order (status: confirmed, paymentMethod: "cod", paymentStatus: "pending_payment")
  → Send emails (shop notification + customer confirmation)
  → Redirect to order confirmation page
```

## Related Code Files

### Modify
- `src/app/api/orders/route.ts` — set paymentMethod="cod", paymentStatus="pending_payment", orderStatus="confirmed"
- `src/app/checkout/page.tsx` — remove payment method selection (COD only)
- `src/lib/order-number.ts` — ensure order number generated before email

## Implementation Steps

1. Update `POST /api/orders`:
   - Set `paymentMethod = "cod"` (hardcoded)
   - Set `paymentStatus = "pending_payment"`
   - Set `orderStatus = "confirmed"`
   - No payment gateway redirect
2. Update checkout page:
   - Remove payment method selection UI
   - Show "Thanh toán khi nhận hàng (COD)" note
   - Submit directly to `/api/orders`
3. Order confirmation page:
   - Display order number
   - Show "Thanh toán khi nhận hàng" note
   - Link to view order in profile
4. Email templates:
   - Shop notification: include "COD - Thanh toán khi nhận hàng"
   - Customer confirmation: include payment method note

## Todo
- [ ] Update `/api/orders` to set COD payment fields
- [ ] Remove payment method selection from checkout page
- [ ] Update order confirmation page with COD note
- [ ] Update email templates with COD payment method
- [ ] Test COD end-to-end flow

## Success Criteria
- COD order created and confirmed immediately
- Emails sent with COD payment method noted
- Order confirmation page shows correct information
- No payment gateway errors

## Risk
- Minimal risk — COD is simplest payment flow
- Customer may expect online payment → add note in checkout

## Security
- No additional security needed for COD
- Order validation still required (server-side)
