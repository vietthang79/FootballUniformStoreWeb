---
title: "Hiểu biết toàn diện về dự án Football Uniform Store Web"
type: brainstorm
date: 2026-04-13
status: reference
---

# Toàn cảnh dự án: Football Uniform E-Commerce + Custom Mockup Builder

> Tài liệu này thể hiện toàn bộ hiểu biết hiện tại của AI về dự án và mong muốn của chủ sở hữu.

---

## 1. Vấn đề cốt lõi cần giải quyết

**Trước khi có website này:**
- Khách hàng đặt đồng phục đội bóng qua Zalo/Facebook
- Mô tả yêu cầu bằng lời → shop hiểu sai → in xong không đúng ý
- Phải làm lại → tốn thời gian và tiền của cả hai bên

**Giải pháp:**
> Khách tự thiết kế trực tiếp trên web, thấy kết quả ngay trước khi đặt hàng → shop nhận đơn đủ thông tin → in và giao, không cần hỏi qua lại.

---

## 2. Hai luồng bán hàng trên cùng một website

### Luồng 1 — Mua hàng thông thường (Normal E-Commerce)
Dành cho: giày, phụ kiện, áo thi đấu lẻ, đồ không cần in.

```
Xem danh sách → Xem chi tiết → Chọn size/màu → Giỏ hàng → Thanh toán
```

### Luồng 2 — Đặt đồng phục theo yêu cầu (Custom Builder)
Dành cho: bộ đồ bóng đá, áo khoác đội — sản phẩm được shop đánh dấu `customizable = true`.

```
Xem sản phẩm → [Custom] → 6 bước thiết kế → Giỏ hàng → Thanh toán
```

Hai luồng **song song trên cùng web**. Sản phẩm `customizable` hiển thị thêm nút **[Custom]**.

---

## 3. Custom Builder — 6 bước thiết kế

Mở tại `/products/[slug]/custom`. UX: **Step wizard** (progress bar + Back/Next).

| Bước | Nội dung | Điểm quan trọng |
|------|----------|-----------------|
| 1 | Chọn màu sắc | Bộ màu định sẵn (không dùng color picker tự do) |
| 2 | Logo & tên đội | CLB logo, sponsor, giải đấu, flag/patch, tên đội |
| 3 | Danh sách cầu thủ | Bảng inline + **paste từ Excel** (Clipboard API + TSV) |
| 4 | **2D Direct Editor** | Kéo thả trực tiếp, KHÔNG có button zoom/rotate |
| 5 | **2D Mockup Preview** | Overlay lên ảnh thật (CSS absolute positioning) |
| 6 | Lưu & thêm vào giỏ | Lưu design, chia sẻ link, đặt in |

### Bước 4 — 2D Direct Editor (điểm kỹ thuật phức tạp nhất)
- **3 thành phần độc lập**: Name (tên), Number (số áo), Logo (ảnh)
- Mỗi thành phần: position {x,y}, scale {w,h}, rotation 0-360°, zIndex
- **Tương tác**: click chọn → drag di chuyển → resize handles → rotate handle
- **Snap**: góc chuẩn 0°/90°/180°/270° (dừng nhẹ nhưng vẫn mượt)
- **Không có button UI** (không zoom/rotate/resize buttons)
- **No vertical scroll** — fit viewport hoàn toàn

### Bước 5 — 2D Mockup Preview
- Overlay design elements (text + logo) lên ảnh thật sản phẩm (CSS absolute positioning)
- View switcher: [Mặt trước] [Mặt sau] [Quần]
- Player selector: chọn cầu thủ để xem tên/số của người đó
- Read-only — Back để quay về Step 4 chỉnh sửa
- Disclaimer: "Màu sắc thực tế có thể khác nhẹ so với màn hình"

---

## 4. Tech Stack đã xác nhận

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 App Router, TypeScript |
| Database | PostgreSQL (Railway) + Prisma ORM |
| Auth | Better Auth + emailAndPassword + admin plugin (mandatory login) |
| State | Zustand + localStorage (cart) |
| Form | React Hook Form + FormProvider |
| UI | shadcn/ui + Tailwind CSS |
| Media | Cloudinary (logo, Excel) |
| Email | Resend + React Email |
| Excel | ExcelJS |
| Mockup render | CSS overlay (client-side only) |
| Payment | **COD only** (Phase 7 MVP) |
| Deploy | Vercel (frontend) + Railway (PostgreSQL) |
| Language | **Tiếng Việt thuần** — không i18n |

---

## 5. Database Schema tóm tắt

```
Product → ProductVariant (size/stock)
        → ProductImage (Cloudinary URLs theo góc + màu)
        → ColorSet (bộ màu định sẵn)

Order (userId NOT NULL — mandatory login)
  → OrderItem (sản phẩm thường)
  → CustomOrder → Player[] (danh sách cầu thủ)
                    → printConfigJson Json? (lưu transform data)
                    → logo URLs, Excel URL
```

**Tồn kho**: Giữ cột `stock` trong DB nhưng **không tự động trừ** — shop quản lý thủ công.

---

## 6. Chính sách phí in (nudge tăng đơn)

| Số bộ | Phụ phí | Nudge hiển thị |
|-------|---------|----------------|
| ≥ 10 | Miễn phí | ✅ "Miễn phí in — áp dụng cho đơn này" |
| 6–9  | +10% | 📦 "Thêm N bộ để miễn phí hoàn toàn!" |
| < 6  | +20% | ⚠️ "Thêm N bộ để giảm xuống +10%" |

**1 bộ = 1 áo + 1 quần** của 1 cầu thủ. Phí tính real-time khi nhập danh sách.

---

## 7. Luồng xử lý đơn hàng

```
Checkout → POST /api/orders
  ├── Tạo Order + OrderItems trong DB
  ├── Cho mỗi CustomOrder:
  │   ├── Tạo CustomOrder + Player[] trong DB
  │   ├── Generate Excel (ExcelJS) → Cloudinary → lưu URL
  │   └── Lưu printConfigJson (transform data)
  ├── COD: confirm ngay → gửi email
  └── Email shop: đơn hàng + Excel đính kèm + logo files + print config
```

**Email shop** gồm: tên khách, SĐT, địa chỉ, file Excel, logo uploads, cấu hình in. Shop đọc → in → giao. Không cần hỏi lại.
**Admin preview**: Sử dụng printConfigJson để render CSS overlay client-side trong dashboard.

---

## 8. Admin Dashboard (scope)

```
/admin          → Stats (đơn hôm nay, tuần, tháng, doanh thu)
/admin/orders   → Danh sách đơn (filter status/date, search by order number)
/admin/orders/[id] → Chi tiết: customer, items, player list, download Excel, logo previews, mockup preview (CSS overlay), cập nhật status
/admin/products → Danh sách sản phẩm (link đến import/edit)
/admin/products/import → CSV Import wizard
/admin/products/[id]/edit → Edit sản phẩm
```

**Product Import**: CSV upload → column mapping → preview → confirm → bulk create.

---

## 9. Phân kỳ triển khai (9 phases, ~6 tuần)

| Phase | Mô tả | Effort | Status |
|-------|-------|--------|--------|
| 1 | Project Setup + DB Schema | 3d | pending |
| 2 | Auth (Better Auth emailAndPassword + admin) | 2d | pending |
| 3 | Product Catalog + Normal E-Commerce | 5d | pending |
| 4 | Custom Mockup Builder UI + Logic | 5d | pending |
| 5 | Mockup Preview Component (Admin) | 1d | pending |
| 6 | Order Processing + Email + Excel | 4d | pending |
| 7 | Payment — COD only | 1d | pending |
| 8 | Admin + Order Management + Product Import | 5d | pending |
| 9 | Polish, SEO, Deploy | 3d | pending |

**Dependencies**: 1→2→3→4→5→6→7→8→9 (tuần tự)

---

## 10. Các quyết định kiến trúc đã xác nhận (Validation Session 1 — 2026-04-12)

| # | Quyết định | Lý do |
|---|-----------|-------|
| 1 | shadcn/ui + Tailwind | Copy-paste, no vendor lock-in |
| 2 | Step wizard (progress bar + Back/Next) | Clear flow, easy per-step validation |
| 3 | Không tracking tồn kho MVP | Đơn giản, shop tự quản lý |
| 4 | Admin chỉ quản lý đơn hàng | Không cần product CRUD web UI |
| 5 | Tay áo logo: user chọn trái/phải/cả hai | Linh hoạt, tránh tranh cãi |
| 6 | Tiếng Việt thuần | Không cần i18n |
| 7 | Dịch vụ in riêng (tự mang đồ) | **Defer hoàn toàn sang Phase 2** |
| 8 | Guest checkout | Better Auth anonymous plugin |
| 9 | Cart = Zustand + localStorage | Sync DB chỉ tại checkout |

---

## 11. Những điểm phức tạp cần chú ý

### A. 2D Editor direct manipulation
Không dùng thư viện canvas sẵn có (Fabric.js, Konva) vì muốn UX tùy chỉnh cao. Phải tự implement:
- Mouse event handling (mousedown/mousemove/mouseup)
- Transform matrix cho resize/rotate
- Snap logic cho rotation
- Bounding box + handles rendering
- GPU acceleration với CSS transforms

### B. Excel paste (Clipboard API)
- Paste TSV từ Excel/Google Sheets/Numbers
- Format có thể khác nhau giữa apps
- Validate và map đúng cột (tên, số, size áo, size quần)

### C. Logo quality handling
- Cảnh báo nếu ảnh < 300x300px
- Tự động thêm note: *"nếu được thì shop làm nét lại (nếu có thể)"*
- Chấp nhận logo chất lượng kém, không block checkout

### D. Tên cầu thủ quá dài
- Giới hạn ký tự
- Cảnh báo trong preview nếu text overflow

---

## 12. Giá trị mang lại

| Đối tượng | Lợi ích |
|-----------|---------|
| Khách hàng | Thấy kết quả trước khi đặt, chủ động thiết kế, không sợ shop làm sai |
| Shop | Nhận đơn đầy đủ thông tin, không cần hỏi qua lại, giảm sai sót và làm lại |
| Kinh doanh | Nudge tự nhiên tăng số bộ/đơn → tăng doanh thu in |

---

## 13. Phạm vi Phase 2 (chưa triển khai)

- Dịch vụ in theo yêu cầu riêng (khách tự mang đồ)
- Có thể: product CRUD web UI cho admin
- Có thể: i18n nếu mở rộng thị trường

---

## 14. Hiện trạng dự án (2026-04-13)

- **Plan**: Đã hoàn thành planning đầy đủ (9 phases)
- **Code**: **Chưa có code nào được viết** — tất cả phases đều `status: pending`
- **index.html**: Chỉ có HTML placeholder (chưa phải Next.js project)
- **Bước tiếp theo**: Bắt đầu Phase 1 — Bootstrap Next.js 15 project

---

## Câu hỏi còn mở

1. **Ảnh sản phẩm**: Chưa có. Cần ảnh thật từng góc (front/back/shorts) cho từng bộ màu để dùng làm nền preview. Có thể dùng placeholder tạm khi dev.
2. **VNPay/MoMo**: Chưa đăng ký. Registration có thể mất vài ngày → nên bắt đầu sớm trước Phase 2 (sau MVP).
3. **Demo data**: Càng nhiều càng tốt nhưng chưa cần ngay — seed data sẽ tạo sau khi Phase 1 setup xong.
4. **Domain**: Deploy lên domain riêng hay subdomain Vercel trước?
