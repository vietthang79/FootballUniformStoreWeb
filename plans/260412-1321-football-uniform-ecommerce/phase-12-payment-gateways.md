---
phase: 12
title: "Payment Gateways (VNPay / MoMo / Bank Transfer) — Post-MVP"
status: deferred
priority: P2
effort: TBD
---

# Phase 12: Payment Gateways — Post-MVP

## Context
- Created Session 10 (2026-04-17) as placeholder for deferred payment integrations
- MVP launches with **COD only** (Phase 7)
- Decision rationale: Vietnamese football uniform customers mostly comfortable with COD for team orders. Online payment added after first user feedback cycle.

## Scope (Planned, Not Yet Detailed)

### 1. VNPay Integration
- VNPay QR code checkout flow
- Webhook/IPN callback for payment verification
- Update `Order.paymentStatus` on verify success
- Handle failure/timeout scenarios
- Admin dashboard: VNPay transaction log

### 2. MoMo Integration
- MoMo app deeplink + QR code
- Webhook verification with signature
- Same `Order.paymentStatus` lifecycle as VNPay

### 3. Bank Transfer (manual confirm)
- Display bank account info + pre-filled transfer note (order number)
- Admin manually marks payment received in dashboard
- Optional: SePay VietQR automation (auto-match by order note)

## Related Files (Planned)

### Create
- `src/lib/payments/vnpay-client.ts`
- `src/lib/payments/momo-client.ts`
- `src/lib/payments/bank-transfer-verifier.ts`
- `src/app/api/payments/vnpay/callback/route.ts`
- `src/app/api/payments/momo/callback/route.ts`
- `src/components/checkout/payment-method-selector.tsx`
- `src/app/payment/[orderId]/page.tsx` — redirect/QR display page

### Modify
- `src/app/checkout/page.tsx` — add payment method selector (was COD-only in MVP)
- `src/app/api/orders/route.ts` — branch by paymentMethod, redirect to gateway if not COD
- `prisma/schema.prisma` — add `PaymentTransaction` model if needed (log attempts)

## Dependencies
- Phase 7 (COD infrastructure)
- Phase 8 (admin dashboard for transaction view)
- Production domain + HTTPS (required for payment webhooks)

## Unresolved / To Research Before Implementation
1. VNPay merchant account onboarding process + fees
2. MoMo business account KYC timeline
3. Whether to use SePay as bank transfer intermediary or self-manage
4. Idempotency strategy for webhook retries
5. PCI DSS considerations (should not handle card data directly — use redirect flow)

## Notes
- This phase is **post-MVP** — do not block Phase 1-10 implementation
- Revisit scope after 1-2 months of production feedback
- Consider phased rollout: Bank transfer first (simplest) → VNPay → MoMo
