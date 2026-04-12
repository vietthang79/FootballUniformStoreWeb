---
phase: 6
title: "Payment Integration"
status: pending
priority: P1
effort: 3d
---

# Phase 6: Payment Integration (VNPay + MoMo + COD)

## Context
- Depends on Phase 5 (order creation flow)
- [Tech Stack Research — Payment](../reports/researcher-260412-1321-tech-stack.md)

## Overview
Integrate VNPay, MoMo redirect payment, and COD. Order created first as pending → redirect to payment → webhook confirms → update status → send emails.

## Flow

```
Checkout → Create Order (status: pending) → Payment Gateway
  ├── VNPay: redirect to VNPay portal → user pays → VNPay IPN callback → update order
  ├── MoMo: redirect to MoMo → user confirms → MoMo IPN callback → update order
  └── COD: mark order as confirmed immediately → send emails
```

## Related Code Files

### Create
- `src/lib/payment/vnpay.ts` — create payment URL, verify return/IPN
- `src/lib/payment/momo.ts` — create payment request, verify callback
- `src/lib/payment/process-payment.ts` — unified payment handler
- `src/app/api/payment/vnpay/create/route.ts` — create VNPay URL
- `src/app/api/payment/vnpay/ipn/route.ts` — VNPay IPN webhook
- `src/app/api/payment/vnpay/return/route.ts` — VNPay return URL (user redirect back)
- `src/app/api/payment/momo/create/route.ts` — create MoMo payment
- `src/app/api/payment/momo/ipn/route.ts` — MoMo IPN webhook
- `src/app/payment/result/page.tsx` — payment result page (success/fail)

### Modify
- `src/app/api/orders/route.ts` — integrate payment flow after order creation
- `src/app/checkout/page.tsx` — payment method selection triggers correct flow

## Implementation Steps

1. Install `vnpay` npm package, configure with merchant credentials
2. Create VNPay payment URL generator — amount, order ID, return URL, IPN URL
3. Create VNPay IPN handler — verify signature, update order payment status
4. Create VNPay return handler — redirect user to result page
5. Create MoMo payment request — API call to MoMo, get payment URL
6. Create MoMo IPN handler — verify signature, update order
7. Update checkout flow:
   - COD → create order (confirmed) → send emails → redirect to confirmation
   - VNPay/MoMo → create order (pending) → redirect to payment gateway
   - On IPN success → update to paid → send emails
   - On return → show result page
8. Payment result page — success: show order details, fail: retry option
9. Handle edge cases: timeout, double payment, IPN retry

## Todo
- [ ] VNPay integration (create URL + IPN + return)
- [ ] MoMo integration (create + IPN)
- [ ] COD flow (immediate confirmation)
- [ ] Payment result page
- [ ] Update checkout to handle payment methods
- [ ] IPN signature verification
- [ ] Edge case handling (timeout, retry)
- [ ] Test with VNPay/MoMo sandbox

## Success Criteria
- VNPay sandbox payment completes end-to-end
- MoMo sandbox payment completes end-to-end
- COD order created and confirmed immediately
- IPN correctly updates payment status
- Emails sent only after payment confirmed (not for pending COD)

## Risk
- VNPay/MoMo merchant account registration takes time → start registration early
- IPN may arrive before user return → handle both orderings
- Sandbox vs production URL differences → env-based config

## Security
- Verify IPN signatures (HMAC) — never trust client-side payment confirmation
- Idempotent IPN handling (same IPN may arrive multiple times)
- Log all payment events for audit trail
