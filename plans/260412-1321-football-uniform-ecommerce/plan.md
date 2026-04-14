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
- **Brand colors**: Vàng #FDD017 (primary), Đỏ #E31E26 (secondary), Xanh dương #00AEEF (accent), Xám #A7A9AC (accent)
- **Builder UX**: ~~Step wizard~~ → Sections trên trang product detail (any-order, no wizard)
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

### Session 5 — 2026-04-14
**Trigger:** Deep review brainstorm — project understanding validation
**Questions asked:** 12

#### Confirmed Decisions
1. **[Brand]** Colors labels were swapped → Vàng #FDD017 (primary), Đỏ #E31E26 (secondary)
2. **[UX]** Builder UX overrides Session 1 — NOT a step wizard → **sections on product detail page** `/products/[slug]`. User can interact with any section in any order.
3. **[UX]** Mockup preview = **Tab switcher 4 góc** (Mặt trước / Mặt sau / Tay áo / Quần), mỗi tab có CSS overlay drag-drop riêng, lưu vị trí per-angle.
4. **[Architecture]** Builder tech = **CSS overlay** (confirmed, no Fabric.js)
5. **[UX]** Mobile: **Desktop-only** for builder (show notice < 768px); catalog + checkout vẫn responsive
6. **[Architecture]** Product types: `isCustomizable` flag trên Product model (true = hiện builder sections)
7. **[UX]** Player list luôn hiện (default 1 dòng); **Size bắt buộc**, Tên + Số áo tùy chọn
8. **[Scope]** Print fee **vào MVP** (Phase 3 nudge + Phase 6 server-side validation)
9. **[UX]** Admin order preview: CSS overlay mockup (read-only) + bảng cầu thủ + logo link
10. **[Shipping]** Khách nhập địa chỉ đầy đủ; StarSport tự giao hoặc qua VC (không tích hợp API)
11. **[Order]** 4 trạng thái: `pending → processing → shipping → delivered` (+ `cancelled`)
12. **[Order]** Khách tự hủy được khi status = `pending`; sau đó chỉ qua shop

#### Phase Impact
- plan.md: fix brand colors; override Session 1 wizard decision
- phase-01: fix color labels; update Player model (name/number optional); printConfigJson per-angle structure; order status "pending" default
- phase-03: builder is embedded in `/products/[slug]`; add print fee nudge in cart; add cancel order (pending only)
- phase-04: **COMPLETE REWRITE** — remove wizard, implement product-detail sections + 4-angle tab CSS overlay
- phase-05: `<MockupPreview />` built in Phase 4 (interactive); Phase 5 integrates read-only into admin
- phase-06: add cancel order endpoint; confirm 2-sheet Excel; print fee server-side validation
- phase-08: add cancel order admin display

---

### Session 6 — 2026-04-14
**Trigger:** Deep project validation brainstorm — verify understanding + phát hiện gaps
**Questions asked:** 14

#### Confirmed Decisions
1. **[UX/Admin]** `isCustomizable` flag → **toggle checkbox trong wizard preview step** (per product group header). Không cần cột trong CSV supplier.
2. **[CSV Import]** Image mapping → **upload thủ công per ColorSet** trong wizard preview step. Session 4 "4 cột ảnh" bị override — ảnh do shop tự chụp/upload, không từ CSV.
3. **[Architecture]** Mixed cart → **1 Order** chứa cả OrderItems (thường) + CustomOrders (đội). Checkout 1 lần, 1 mã đơn.
4. **[Architecture]** ProductVariant scope → **per Product** (chung mọi ColorSet). Sizes áp dụng cho tất cả màu.
5. **[Phase split]** Phase 3 = cart + checkout + POST /api/orders (normal items only); Phase 6 = extend API thêm CustomOrder handling.
6. **[Builder — MAJOR]** Multi-logo support: logo panel bên trái canvas, drag logos vào canvas per angle. `CustomConfig.logos[]` thay `logoUrl`. `OverlayElement` reference `logoId`. Logo formats: PNG/JPG/SVG max 5MB (SVG phải sanitize XSS).
7. **[Builder]** Excel paste → **flexible auto-detect** cột (pattern matching). Confidence thấp → hiện quick mapping dialog.
8. **[Order]** Cancel đơn → **gửi email cảnh báo shop** (subject: "[Đơn huỷ] ORD-xxx — Tên khách").
9. **[Order]** Order number reset **theo ngày** (NNN reset về 001 mỗi ngày mới).
10. **[Order]** Print fee tính **riêng từng CustomOrder** (không pool tổng số bộ).
11. **[UX]** Overlay position **giữ nguyên** khi khách đổi ColorSet.
12. **[UX]** Builder size dropdown: show đủ sizes từ ProductVariant, **disable nếu stock=0**.
13. **[UX]** Out of stock normal products: "Hết hàng" + disable button per-size.
14. **[Admin]** Admin tài khoản đầu tiên: **seed script** khi deploy.

#### Phase Impact
- plan.md: fix Session 4 CSV image decision; thêm multi-logo decision
- phase-01: update printConfigJson comment → multi-logo structure (`logos[]` + `logoId` in OverlayElement)
- phase-03: Phase 3 checkout scope clarified (normal only); size OOS UX (disable + "Hết hàng"); stock UX
- phase-04: **MAJOR** — multi-logo panel sidebar; updated data types (LogoItem, OverlayElement.logoId); flexible Excel detect + mapping dialog; SVG sanitize security
- phase-06: mixed order handling (normal + custom same Order); cancel email to shop; multi-logo email attachments; order number daily reset note; print fee per CustomOrder
- phase-08: `isCustomizable` toggle trong wizard preview; CSV images = manual upload (no CSV URL columns)

### Session 7 — 2026-04-14
**Trigger:** Deep validation + gap analysis — rà soát toàn bộ plan trước khi implement
**Questions asked:** 14

#### Confirmed Decisions
1. **[DB Schema]** Xóa redundant columns khỏi CustomOrder (`clubLogoUrl`, `sponsorLogoUrl`, `leagueLogoUrl`, `flagPatchUrl`, tất cả boolean print config) — chỉ dùng `printConfigJson` làm single source of truth
2. **[Phase split]** Phase 6 = API only (orders, email, Excel); Admin UI (order list, order detail, status update) → Phase 8
3. **[UX]** Bỏ nút [Sửa] trong cart khỏi MVP — chỉ [Xóa]. Khách custom lại từ /products/[slug]
4. **[Architecture]** Order number race condition → fix bằng unique constraint + retry (max 3 lần)
5. **[DB Schema]** ProductImage giữ cả 2 FK (productId + colorSetId) — giữ nguyên
6. **[Project state]** Next.js chưa khởi tạo — Phase 1 bắt đầu từ đầu; index.html không dùng làm reference
7. **[Auth]** Better Auth schema dùng `npx @better-auth/cli generate` → Prisma adapter
8. **[Brand colors]** Fix bug phase-01 step 6: Vàng #FDD017 (primary), Đỏ #E31E26 (secondary)
9. **[UX]** Print fee nudge per-CustomOrder riêng lẻ (không pool)
10. **[Services]** Giữ Railway + Cloudinary + Resend (không thay đổi)
11. **[Admin]** Tạo admin user thủ công: register bình thường → update role='admin' trong DB lần đầu
12. **[UX]** teamName fallback = product.name (trong email subject, Excel, admin preview)
13. **[Homepage]** Làm mới hoàn toàn, không dùng index.html làm reference
14. **[Phase 9]** Scope: SEO/OG per product + Core Web Vitals + admin analytics (doanh thu, top products). Không PWA.

#### Phase Impact
- phase-01: fix brand color labels; thêm `npx @better-auth/cli generate`; xóa redundant CustomOrder columns khỏi schema
- phase-03: print fee nudge per-CustomOrder; cart custom item chỉ có [Xóa]
- phase-04: xóa [Sửa] + edit flow; teamName fallback = product.name
- phase-06: trim admin UI files/steps (→ Phase 8); thêm retry logic cho order-number.ts; teamName fallback trong email templates
- phase-08: clarify không duplicate API routes từ Phase 6; thêm analytics vào dashboard

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
10. **[CSV Import]** Image mapping → **manual upload per ColorSet** trong wizard preview step (không cần cột ảnh trong CSV supplier — ảnh do shop tự upload)

#### Phase Impact
- phase-07: rewritten as COD only, effort 3d → 1d
- phase-01: add Montserrat font, brand colors theme config, mobile responsive requirement, 4 image angles
- phase-03: add stock display on product detail page
- phase-04: update image angles to 4 (front/back/sleeve/shorts)
- phase-05: update to full UI implementation (not just component)
- phase-08: add stock display in product list/edit, manual image upload (4 angles) per ColorSet trong wizard preview step
- Total effort: -2d (7w+2d → 7w) due to COD only simplification
