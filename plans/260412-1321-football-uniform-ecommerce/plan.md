---
title: "Football Uniform E-Commerce + Custom Builder"
description: "Web bán đồ thể thao bóng đá với tính năng Custom Builder cho đồng phục đội"
status: pending
priority: P1
effort: 8w
issue: ~
branch: main
tags: [frontend, backend, database, feature, ecommerce]
created: 2026-04-12
---

# Football Uniform E-Commerce + Custom Builder

## Tech Stack
Next.js 15 App Router | PostgreSQL + Prisma | Better Auth | Zustand | React Hook Form | **shadcn/ui** | Cloudinary | Resend | ExcelJS | VNPay + MoMo + COD | Vercel + Railway

## Confirmed Decisions (Validation Session 1 — 2026-04-12)
- **UI**: shadcn/ui + Tailwind CSS
- **Builder UX**: Step wizard (progress bar + Back/Next)
- **Tồn kho**: Không tracking MVP — lưu cột stock nhưng không tự động trừ
- **Admin scope**: Chỉ xem + quản lý đơn hàng, không CRUD sản phẩm qua web
- **Tay áo logo**: Khách chọn tay trái / phải / cả hai
- **Ngôn ngữ**: Tiếng Việt thuần, không i18n
- **Dịch vụ in riêng**: Defer hoàn toàn sang Phase 2, không có form liên hệ
- **Logo quality**: Chấp nhận logo chất lượng kém, tự động thêm note "nếu đước thì shop làm nét lại (nếu có thể)"
- **Admin Dashboard**: Email chỉ để thông báo, Admin dashboard để quản lý toàn bộ đơn hàng, cập nhật trạng thái, xem chi tiết

## Phases

| # | Phase | Status | Effort | File |
|---|-------|--------|--------|------|
| 1 | Project Setup + DB Schema | pending | 3d | [phase-01](./phase-01-project-setup.md) |
| 2 | Product Catalog + Normal E-Commerce | pending | 5d | [phase-02](./phase-02-product-catalog.md) |
| 3 | Custom Builder UI + Logic | pending | 7d | [phase-03](./phase-03-custom-builder.md) |
| 4 | 2D Mockup Preview + Print Config | pending | 4d | [phase-04](./phase-04-mockup-preview.md) |
| 5 | Order Processing + Email + Excel | pending | 4d | [phase-05](./phase-05-order-processing.md) |
| 6 | Payment Integration | pending | 3d | [phase-06](./phase-06-payment.md) |
| 7 | Admin + Order Management | pending | 3d | [phase-07](./phase-07-admin.md) |
| 8 | Polish, SEO, Deploy | pending | 3d | [phase-08](./phase-08-polish-deploy.md) |

## Dependencies
- Phase 2 depends on Phase 1
- Phase 3 depends on Phase 2 (product data)
- Phase 4 depends on Phase 3 (builder state)
- Phase 5 depends on Phase 3+4 (custom order data)
- Phase 6 depends on Phase 5 (order flow)
- Phase 7 depends on Phase 5+6
- Phase 8 depends on all

## Key Architecture Decisions
1. **Cart = Zustand + localStorage** — no server-side cart, sync to DB only at checkout
2. **Custom order = bundle cart item** — 1 cart item = 1 team order with N players
3. **2D Mockup = CSS overlay** — no Canvas/Fabric for MVP, upgrade later if needed
4. **Excel paste = native Clipboard API** — no table library
5. **Logos uploaded to Cloudinary** — URLs stored in cart/order, originals preserved for print
6. **Guest checkout first** — Better Auth anonymous plugin, optional account creation

## Reports
- [Brainstorm](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md)
- [Tech Stack Research](../reports/researcher-260412-1321-tech-stack.md)
- [Custom Builder Research](../reports/researcher-260412-1321-custom-builder.md)

## Validation Log

### Session 1 — 2026-04-12
**Trigger:** Post-plan validation interview
**Questions asked:** 7

#### Confirmed Decisions
1. **[Architecture]** UI component library → **shadcn/ui + Tailwind** — copy-paste, no vendor lock-in
2. **[UX]** Custom Builder layout → **Step wizard** (progress bar + Back/Next) — clear flow, easy per-step validation
3. **[Scope]** Stock tracking MVP → **Skipped** — keep `stock` column in DB, no auto-decrement; shop manages manually
4. **[Scope]** Admin panel → **Order management only** — no product CRUD web UI; use seed + Prisma Studio
5. **[Architecture]** Sleeve logo side → **User-selectable** (left / right / both) — added `side` field per sleeve item
6. **[Scope]** UI language → **Vietnamese only** — no i18n needed
7. **[Scope]** Print service (bring own clothes) → **Full defer to Phase 2** — no contact form in MVP

#### Phase Impact
- phase-01: shadcn/ui added to install steps
- phase-03: sleeve `printConfig` updated with `side` field per item; step wizard UX confirmed
- phase-07: product CRUD removed, files list trimmed, success criteria updated
