---
title: "Custom Mockup Platform - Football Uniform"
description: "Website custom mockup đồng phục bóng đá - khách tự thêm mockup vào vị trí tùy chọn trên ảnh sản phẩm thật"
status: pending
priority: P1
effort: ~6w
issue: ~
branch: main
tags: [frontend, backend, database, feature, ecommerce]
created: 2026-04-12
---

# Football Uniform E-Commerce + Mockup Builder

## Tech Stack
Next.js 15 App Router | PostgreSQL + Prisma | Better Auth | Zustand | React Hook Form | **shadcn/ui** | Cloudinary | Resend | ExcelJS | COD (MVP) | Vercel + Railway

## Confirmed Decisions (Validation Session 1 — 2026-04-12)
- **UI**: shadcn/ui + Tailwind CSS
- **Brand colors**: Đỏ #FDD017(primary), Vàng #E31E26 (secondary), Xanh dương #00AEEF (accent), Xám #A7A9AC (accent)
- **Builder UX**: Step wizard (progress bar + Back/Next)
- **Tồn kho**: Không tracking MVP — lưu cột stock nhưng không tự động trừ
- **Admin scope**: Quản lý đơn hàng + **CSV product import**. Không CRUD thủ công — import sản phẩm từ file CSV supplier xuất. Product Edit page riêng cho sửa chi tiết sau import.
- **Tay áo logo**: Khách chọn tay trái / phải / cả hai
- **Ngôn ngữ**: Tiếng Việt thuần, không i18n
- **Dịch vụ in riêng**: Defer hoàn toàn sang Phase 2, không có form liên hệ
- **Logo quality**: Chấp nhận logo chất lượng kém, tự động thêm note "nếu đước thì shop làm nét lại (nếu có thể)"
- **Admin Dashboard**: Email chỉ để thông báo, Admin dashboard để quản lý toàn bộ đơn hàng, cập nhật trạng thái, xem chi tiết

## Phases

| # | Phase | Status | Effort | File |
|---|-------|--------|--------|------|
| 1 | Project Setup + DB Schema | pending | 3d | [phase-01](./phase-01-project-setup.md) |
| 2 | Auth — Login / Register / Forgot Password | pending | 2d | [phase-02](./phase-02-auth.md) |
| 3 | Product Catalog + Normal E-Commerce | pending | 5d | [phase-03](./phase-03-product-catalog.md) |
| 4 | Custom Mockup Builder UI + Logic | pending | 5d | [phase-04](./phase-04-custom-builder.md) |
| 5 | Admin Preview Component | pending | 1d | [phase-05](./phase-05-mockup-preview.md) |
| 6 | Order Processing + Email + Excel | pending | 4d | [phase-06](./phase-06-order-processing.md) |
| 7 | Payment Integration (COD only) | pending | 1d | [phase-07](./phase-07-payment.md) |
| 8 | Admin + Order Management + Product Import | pending | 5d | [phase-08](./phase-08-admin.md) |
| 9 | Polish, SEO, Deploy | pending | 3d | [phase-09](./phase-09-polish-deploy.md) |

## Dependencies
- Phase 2 depends on Phase 1
- Phase 3 depends on Phase 2 (auth required for checkout)
- Phase 4 depends on Phase 3 (product data)
- Phase 5 depends on Phase 4 (builder state)
- Phase 6 depends on Phase 4+5 (custom order data)
- Phase 7 depends on Phase 6 (order flow)
- Phase 8 depends on Phase 6+7
- Phase 9 depends on all

## Key Architecture Decisions
1. **Cart = Zustand + localStorage** — no server-side cart, sync to DB only at checkout
2. **Mockup order = bundle cart item** — 1 cart item = 1 team order with N players
3. **Custom Mockup = Click-thả real-time** — khách tự do thêm mockup vào vị trí bất kỳ, không giới hạn vị trí cố định
4. **Excel paste = native Clipboard API** — no table library
5. **Logos uploaded to Cloudinary** — URLs stored in cart/order, originals preserved for print
6. **Mandatory login** — Better Auth emailAndPassword + admin plugins; no anonymous/guest checkout

## Reports
- [Brainstorm](../reports/brainstorm-260412-1321-football-uniform-ecommerce.md)
- [Tech Stack Research](../reports/researcher-260412-1321-tech-stack.md)
- [Custom Builder Research](../reports/researcher-260412-1321-custom-builder.md)
- [Auth Feature Brainstorm](../reports/brainstorm-260413-1553-auth-feature.md)

## Validation Log

### Session 1 — 2026-04-12
**Trigger:** Post-plan validation interview
**Questions asked:** 7

#### Confirmed Decisions
1. **[Architecture]** UI component library → **shadcn/ui + Tailwind** — copy-paste, no vendor lock-in
2. **[UX]** Custom Builder layout → **Step wizard** (progress bar + Back/Next) — clear flow, easy per-step validation
3. **[Scope]** Stock tracking MVP → **Skipped** — keep `stock` column in DB, no auto-decrement; shop manages manually
4. **[Scope]** Admin panel → **Order management only** — no product CRUD web UI; use seed + Prisma Studio
   *(Revised Session 2: replaced Prisma Studio with CSV import flow — see Session 2 below)*
5. **[Architecture]** Sleeve logo side → **User-selectable** (left / right / both) — added `side` field per sleeve item
6. **[Scope]** UI language → **Vietnamese only** — no i18n needed
7. **[Scope]** Print service (bring own clothes) → **Full defer to Phase 2** — no contact form in MVP

#### Phase Impact
- phase-01: shadcn/ui added to install steps
- phase-03: sleeve `printConfig` updated with `side` field per item; step wizard UX confirmed
- phase-07: product CRUD removed, files list trimmed, success criteria updated

### Session 3 — 2026-04-13
**Trigger:** Auth feature brainstorm
**Questions asked:** 6

#### Confirmed Decisions
1. **[Scope]** Auth method → **email + password only** (no phone login)
2. **[Scope]** Checkout → **Mandatory login** (reverses Session 1 "guest checkout" decision)
3. **[Scope]** Forgot password → **Email reset link via Resend**
4. **[Scope]** No email verification — register → login immediately
5. **[Architecture]** Admin → same user system, `role = "admin"` (Better Auth admin plugin)
6. **[Plan]** Auth → **new Phase 2**, phases 2–8 renumbered to 3–9

#### Phase Impact
- plan.md: added Phase 2 (Auth), renumbered phases 2–8 → 3–9; key decision 6 updated
- phase-01: remove anonymous plugin, update Better Auth config, Order.userId NOT NULL
- phase-02-auth.md: new file created
- Total effort: +2d (7w → 7w+2d)

---

### Session 2 — 2026-04-13
**Trigger:** Product management flow clarification
**Questions asked:** 5

#### Confirmed Decisions
1. **[Scope]** Product management → **CSV import flow** (replaces Prisma Studio approach)
   - Upload CSV → column mapping UI (auto-suggest, unmapped = empty editable field) → preview grouped by product name → confirm → save DB
   - Duplicate on name collision — no merge/check
2. **[UX]** Unmapped fields → show empty inputs in preview (không ẩn), user điền thủ công
3. **[UX]** Images trong preview → upload trực tiếp per ColorSet (1 hoặc nhiều ảnh), buffer locally → upload Cloudinary khi Confirm
4. **[Architecture]** Grouping — client-side, group by product name (case-insensitive trim); 1 CSV row = 1 ColorSet
5. **[Scope]** Product Edit page riêng (`/admin/products/[id]/edit`) — cho chỉnh chi tiết sau import

#### Phase Impact
- plan.md: Admin scope updated; phase-08 effort 5d (product import + edit)
- phase-08: major expansion — thêm product import + product edit routes/files/steps
- phase-01: seed.ts vẫn giữ cho dev/test data; không còn là primary product management method

### Session 4 — 2026-04-13
**Trigger:** Plan consistency check + clarification session
**Questions asked:** 10

#### Confirmed Decisions
1. **[Payment]** Phase 7 → **COD only** (VNPay/MoMo deferred to Phase 2 post-launch)
2. **[Payment]** Phase 7 effort → **1d** (was 3d with VNPay/MoMo)
3. **[Images]** Image angles per ColorSet → **4 angles**: front, back, sleeve, shorts
4. **[Database]** ColorSet-ProductImage relationship → **ColorSet 1-N ProductImage** (mỗi ColorSet có nhiều ảnh theo góc)
5. **[UX]** Stock display → **Visible to customers** on product detail page (stock field exists but no auto-decrement)
6. **[UX]** Mobile responsive → **From Phase 1** (all pages mobile-friendly, not just Phase 3/4)
7. **[UI]** Font default → **Montserrat** (brand font, not shadcn default Inter)
8. **[UI]** Brand colors theme → **Create src/lib/theme.ts** with custom colors; update tailwind.config.ts
9. **[Phase 5]** Admin Preview → **Full UI implementation** (not just reusable component, integrate with Phase 8 admin order detail)
10. **[CSV Import]** Image mapping → 4 columns: hinh_anh_front, hinh_anh_back, hinh_anh_sleeve, hinh_anh_shorts

#### Phase Impact
- phase-07: rewritten as COD only, effort 3d → 1d
- phase-01: add Montserrat font, brand colors theme config, mobile responsive requirement, 4 image angles
- phase-03: add stock display on product detail page
- phase-04: update image angles to 4 (front/back/sleeve/shorts)
- phase-05: update to full UI implementation (not just component)
- phase-08: add stock display in product list/edit, 4 image angles per ColorSet in CSV import
- Total effort: -2d (7w+2d → 7w) due to COD only simplification
