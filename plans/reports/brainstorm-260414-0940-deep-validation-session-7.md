---
title: "Session 7 — Deep Validation & Gap Analysis"
type: brainstorm
date: 2026-04-14
questions: 14
---

# Session 7 — Deep Validation & Gap Analysis

**Trigger:** Post-review brainstorm — xác nhận lại hiểu dự án + phát hiện gaps còn lại

---

## Confirmed Decisions

### 1. [DB Schema] CustomOrder — xóa cột logo riêng lẻ
**Decision:** Xóa `clubLogoUrl`, `sponsorLogoUrl`, `leagueLogoUrl`, `flagPatchUrl` và tất cả boolean print config columns (`printClubLogo`, `printSponsorLogo`, `printSmallNumber`, `printLeagueLogo`, `printSleeveSponsor`, `printFlagPatch`, `printTeamName`, `printBackNumber`, `printPlayerName`, `printShortsNumber`, `shortsNumberSide`) khỏi CustomOrder model.
**Reason:** Session 6 đã quyết định dùng `logos[]` array + `OverlayElement` trong `printConfigJson`. Các cột riêng lẻ là redundant, gây confuse, tăng migration burden.
**CustomOrder chỉ giữ:** `printConfigJson`, `teamName`, `logoQualityNote`, `excelFileUrl`, `playerCount`, `printingSurcharge`.

### 2. [Phase split] Phase 6 = API only, Phase 8 = Admin UI
**Decision:** Phase 6 chỉ implement API endpoints (POST /api/orders, PATCH /api/orders/[id]/cancel, email logic, Excel). Admin UI (order list, order detail, status update) thuộc Phase 8.
**Impact:** Phase 6 files list cần trim bỏ admin UI components/pages.

### 3. [UX] Nút [Sửa] trong cart — bỏ khỏi MVP
**Decision:** Không có nút Sửa. Khách xóa item rồi vào lại /products/[slug] custom lại từ đầu.
**Reason:** Builder embedded trong /products/[slug], không có clean way để pre-populate từ cart. Tránh over-engineering MVP.
**Impact:** phase-03, phase-04 cập nhật — xóa mọi đề cập đến [Sửa] button và edit flow.

### 4. [Architecture] Order number race condition — fix bằng unique + retry
**Decision:** `orderNumber` đã có `@@unique`. Thêm try/catch trong `order-number.ts`: nếu unique constraint violation → retry (max 3 lần). Đủ cho MVP traffic thấp.

### 5. [DB Schema] ProductImage giữ cả 2 FK (productId + colorSetId)
**Decision:** Giữ nguyên. productId cho phép query trực tiếp images theo product mà không cần join qua ColorSet. Thêm index là đủ.

### 6. [Project state] Chỉ có index.html tĩnh, Next.js chưa khởi tạo
**Decision:** Phase 1 chưa bắt đầu. index.html là mockup concept, không dùng làm reference. Homepage Next.js làm mới hoàn toàn.

### 7. [Auth] Better Auth schema dùng Prisma adapter generate
**Decision:** Chạy `npx @better-auth/cli generate` để tạo User/Session/Account/Verification tables. Không tự viết thủ công.

### 8. [Brand colors] Fix bug phase-01 step 6
**Decision:** Sửa lại: Vàng #FDD017 (primary), Đỏ #E31E26 (secondary). Phase-01 step 6 viết ngược.

### 9. [UX] Print fee nudge per CustomOrder (không pool)
**Decision:** Mỗi CustomCartItem có nudge riêng dựa trên player count của đơn đó. Cart hiển thị nudge per-item. Đúng với Session 6.

### 10. [Services] Giữ Railway + Cloudinary + Resend
**Decision:** Không thay đổi stack. Railway cho PostgreSQL, Cloudinary cho tất cả file storage (logo + product images), Resend cho email.

### 11. [Admin] Tạo admin user thủ công sau deploy
**Decision:** Register bình thường qua /register, sau đó update role='admin' trực tiếp trong DB lần đầu (qua Railway console hoặc Prisma Studio dev). Không cần seed script phức tạp.

### 12. [UX] teamName fallback = tên sản phẩm
**Decision:** Khi khách không nhập tên đội, dùng `product.name` làm fallback trong email subject, Excel header, admin preview. Ví dụ: "Áo CLB Season 2026 — 12 bộ".

### 13. [Homepage] Làm mới hoàn toàn
**Decision:** index.html không dùng làm reference. Thiết kế theo brand guidelines (Montserrat, brand colors, shadcn/ui).

### 14. [Phase 9] Scope: SEO/OG + Performance + Admin analytics
**Decision:** Phase 9 bao gồm: meta tags + og:image per product, Core Web Vitals (Next.js Image, lazy load, bundle size), admin analytics dashboard (doanh thu theo ngày/tuần, sản phẩm bán chạy). Không có PWA/offline.

---

## Phase Impact

### phase-01
- Fix step 6: sửa label màu (Vàng #FDD017 primary, Đỏ #E31E26 secondary)
- Thêm: chạy `npx @better-auth/cli generate` cho User/Session/Account tables
- Update DB schema: xóa redundant columns khỏi CustomOrder (xem Decision 1)
- Ghi chú: admin user tạo thủ công sau deploy (không cần seed user)

### phase-03
- Xóa: [Sửa] button và edit flow trong cart
- Update: cart custom item chỉ có [Xóa] button
- Update: print fee nudge hiển thị per-CustomOrder riêng lẻ

### phase-04
- Xóa: "Edit flow: re-open builder from cart with existing config" trong Steps + Todo + Success Criteria
- Update: Add to Cart → redirect to /cart (không có edit path)

### phase-06
- Trim files list: xóa admin UI components/pages (thuộc Phase 8)
- Giữ: API routes, email logic, Excel generator, order-number.ts (với retry)
- Update teamName fallback logic trong email templates

### phase-08
- Clarify: Phase 8 nhận admin UI từ Phase 6 (không duplicate API routes)
- Thêm: Admin analytics trong dashboard (doanh thu, top products)

### phase-09 (tạo mới)
- Scope: SEO meta tags + og:image, Core Web Vitals, admin analytics

---

## Remaining Gaps (nhỏ, không block)

1. **ColorSet hex chips**: primary/secondary hex chỉ để hiện color chip trong UI. Nếu admin nhập sai hex → chip render sai màu. Không cần validate MVP nhưng nên có color picker trong product edit.
2. **Resend 100 emails/day limit**: Nếu có 100+ đơn trong 1 ngày → email bị drop. Acceptable for MVP.
3. **SVG sanitize thư viện**: Nên dùng `dompurify` (Node.js: `isomorphic-dompurify`) thay tự viết — battle-tested hơn.
4. **Excel upload to Cloudinary**: Cloudinary không tối ưu cho non-image files. Có thể dùng raw upload resource_type='raw'. Cần test.
