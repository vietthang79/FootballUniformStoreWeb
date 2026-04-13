---
title: "Hiểu biết toàn diện về dự án StarSport"
type: brainstorm
date: 2026-04-13
status: reference
session: "Brainstorm + Q&A xác nhận — 2026-04-13 14:35"
---

# StarSport — Football Uniform E-Commerce + Custom Mockup Builder

> Tài liệu này thể hiện toàn bộ hiểu biết của AI về dự án, tổng hợp từ:
> plan files, phase files, validation session 1 (2026-04-12), Q&A session (2026-04-13).
> Mọi quyết định đã được xác nhận với chủ sở hữu.

---

## 1. Vấn đề gốc rễ

**Trước khi có website:**
- Khách đặt đồng phục qua Zalo/Facebook
- Mô tả yêu cầu bằng lời → shop hiểu sai → in xong không đúng ý
- Làm lại → mất thời gian và tiền cả hai bên

**Giải pháp:**
> Khách **tự thiết kế** trực tiếp trên web, thấy kết quả trước khi đặt.
> Shop nhận đơn **đầy đủ thông tin** (file Excel + mockup PNG + logo files) → in và giao, không hỏi qua lại.

---

## 2. Thương hiệu

- **Tên shop:** StarSport
- **Ngôn ngữ UI:** Tiếng Việt thuần (không i18n)
- **Thị trường:** Việt Nam
- **Màu sắc brand:**
  - Primary: Đỏ #E31E26
  - Secondary: Vàng #FDD017
  - Accent: Xanh dương #00AEEF, Xám #A7A9AC

---

## 3. Hai luồng bán hàng song song

### Luồng 1 — Mua hàng thông thường
Dành cho: giày, phụ kiện, áo lẻ, đồ không cần in.

```
Xem danh sách → Chi tiết → Chọn size/màu → Giỏ → Thanh toán
```

### Luồng 2 — Đặt đồng phục Custom
Dành cho: bộ đồ bóng đá, áo khoác đội (`customizable = true`).

```
Xem sản phẩm → [Custom] → 6 bước thiết kế → Giỏ → Thanh toán
```

**Sản phẩm `customizable`** hiện thêm nút **[Custom]** bên cạnh **[Thêm vào giỏ]**.

---

## 4. Custom Builder — 6 bước

Route: `/products/[slug]/custom`
UX: **Step wizard** (progress bar + Back/Next). Single React Hook Form + FormProvider xuyên suốt.

| Bước | Tên | Nội dung | Điểm kỹ thuật |
|------|-----|----------|---------------|
| 1 | ColorSetSelector | Chọn bộ màu sản phẩm | Grid ảnh **thật** từng bộ màu admin đã định sẵn, click chọn. Không có color picker tự do — chỉ chọn trong các màu admin đã upload ảnh cho sản phẩm đó |
| 2 | LogoUploader | Upload logo + nhập tên đội | Cloudinary upload, auto note chất lượng kém |
| 3 | PlayerListEditor | Bảng cầu thủ | useFieldArray + **paste từ Excel** (Clipboard API + TSV) |
| 4 | PrintConfigEditor | **2D Direct Editor** | Kéo/resize/rotate — **không có UI buttons** |
| 5 | MockupPreview | **2D Overlay Preview** | Design overlay lên ảnh thật (CSS absolute) |
| 6 | AddToCart | Thêm vào giỏ | Lưu CustomCartItem vào Zustand |

### Bước 4 — 2D Direct Editor (điểm kỹ thuật phức tạp nhất)

**3 thành phần độc lập:** Name (tên), Number (số áo), Logo (ảnh)

Mỗi element có: `position {x%, y%}`, `scale {w, h px}`, `rotation 0-360°`, `zIndex`

**Tương tác:**
- Click chọn → hiện bounding box + handles
- Drag → di chuyển (giới hạn trong canvas)
- Corner handles → resize giữ aspect ratio
- Edge handles → scale 1 chiều
- Rotate handle → xoay tự do + **snap nhẹ** tại 0°/90°/180°/270°
- Hiện góc hiện tại (vd "45°") khi drag

**Luật UI:**
- ❌ Không có button zoom/rotate/resize
- ✅ Chỉ tương tác trực tiếp trên element
- ✅ No vertical scroll — fit viewport
- ✅ **Touch events bắt buộc** (mobile từ Phase 3)
- ✅ Cursor thay đổi theo context (move/resize/rotate)

**View toggle:** [Mặt trước] [Mặt sau] [Tay áo] [Quần]

### Bước 1 — ColorSetSelector (chi tiết)

- Admin thêm sản phẩm → thêm các `ColorSet`, mỗi set có:
  - `name`: tên bộ màu (ví dụ: "Xanh Navy + Trắng")
  - `images[]`: ảnh thật sản phẩm theo góc (front, back, sleeve, shorts) — upload lên Cloudinary
  - `colorPrimary` / `colorSecondary`: hex để hiển thị chip màu trong UI (phụ, không dùng để render)
- Khách thấy grid ảnh front của từng bộ màu → click chọn 1
- **Không có color picker tự do** — chỉ chọn trong những màu admin đã setup
- Bộ màu được chọn → dùng ảnh thật tương ứng làm nền cho Step 4 (editor) và Step 5 (mockup preview)

### Bước 5 — 2D Mockup Preview ✅ CONFIRMED (không có Three.js)

- Overlay design elements lên **ảnh thật sản phẩm** (CSS absolute positioning)
- View switcher: [Mặt trước] [Mặt sau] [Quần]
- Player selector dropdown: xem tên/số của từng cầu thủ
- Read-only — Back để quay về Step 4
- Disclaimer: *"Màu sắc thực tế có thể khác nhẹ so với màn hình"*

### Font chữ
Dùng font mặc định của dự án (không có font selector cho khách).

---

## 5. Tech Stack (confirmed)

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 App Router + TypeScript |
| Database | PostgreSQL (Railway) + Prisma ORM |
| Auth | Better Auth + emailAndPassword + admin plugin (mandatory login) |
| State | Zustand + localStorage (cart) |
| Form | React Hook Form + FormProvider |
| UI | shadcn/ui + Tailwind CSS ||
| Media | Cloudinary (logo, Excel) |
| Email | Resend + React Email |
| Excel | ExcelJS |
| Mockup render | CSS overlay (client-side for admin preview) |
| Payment | **COD only** (Phase 7 MVP) |VP) |
| Deploy | Vercel (frontend) + Railway (PostgreSQL) |

**Payment update:** VNPay + MoMo dVfer NPay + MoMo defer sang Phase 2 sau khi shop vào hoạt động.

---

## 6. Database Schema (updated)

```
Product → ProductVariant (size/stock — không auto-decrement)
        → ProductImage (Cloudinary URLs theo góc + màu, liên kết với ColorSet)
        → ColorSet (bộ màu sản phẩm do admin định sẵn: tên + ảnh thật theo góc + hex chip UI)

Order (userId NOT NULL — mandatory login)
  → OrderItem (sản phẩm thường)
  → CustomOrder → Player[] (danh sách cầu thủ)
                    → printConfigJson Json? ← ✅ (lưu đầy đủ transform)
                    → boolean flags (giữ lại cho quick query)
                    → logo URLs, Excel URL, mockup URLs
```

**Lỗ hổng đã vá:** `CustomOrder` bổ sung `printConfigJson Json?` lưu toàn bộ `ElementTransform` (position/scale/rotation) của tất cả elements. Field này là **bắt buộc** để tái tạo chính xác design của khách trong admin preview.

**Tồn kho:** Giữ cột `stock` trong DB nhưng **không tự động trừ** — shop quản lý thủ công.

---

## 7. Chính sách phí in (nudge tăng đơn)

| Số bộ | Phụ phí | Nudge |
|-------|---------|-------|
| ≥ 10 | Miễn phí | ✅ "Miễn phí in!" |
| 6–9  | +10% | "Thêm N bộ để miễn phí!" |
| < 6  | +20% | "Thêm N bộ để giảm xuống 10%" |

**1 bộ = 1 áo + 1 quần** của 1 cầu thủ. Tính real-time tại Step 3.

---

## 8. Luồng xử lý đơn hàng

```
Checkout → POST /api/orders
  ├── Tạo Order + OrderItems trong DB
  ├── Với mỗi CustomOrder:
  │   ├── Tạo CustomOrder + Players trong DB
  │   ├── Generate Excel (ExcelJS) → Cloudinary → lưu URL
  │   └── Generate mockup PNG (Canvas) → Cloudinary → lưu URL
  ├── COD: confirm ngay → gửi email
  └── Email shop: tên/SĐT/địa chỉ + Excel đính kèm + logos + mockup + cấu hình in
```

**Email shop nhận:** đầy đủ thông tin để in và giao mà không cần hỏi thêm.

---

## 9. Admin Dashboard (scope)

```
/admin          → Stats: đơn hôm nay, tuần, tháng, doanh thu
/admin/orders   → Danh sách đơn (filter status/date, search)
/admin/orders/[id] → Chi tiết: customer info, items, player list,
                     download Excel, logo previews, cập nhật status
/admin/products → Danh sách sản phẩm (link đến import/edit)
/admin/products/import → CSV Import wizard (PapaParse client-side)
/admin/products/[id]/edit → Edit sản phẩm (images, colorsets, price)
```

**Product Import:** CSV upload → column mapping → preview → confirm → bulk create Products + ColorSets + ProductImages.
**Email:** Chỉ để nhận thông báo đơn, mọi quản lý qua Admin Dashboard.

---

## 10. Phân kỳ triển khai (9 phases)

| # | Phase | Effort | Status | Thay đổi từ Q&A |
|---|-------|--------|--------|-----------------|
| 1 | Project Setup + DB Schema | 3d | pending | Thêm pintCfigJon à Cu
| 2 | Auth (Better Auth emailAndPassword + admin) | 2d | pending | ✅ MỚI — mandatory login |
| 3 | Product Catalog + Normal E-Commerce | 5d | pending | |
| 4 | Custom Mockup Builder UI + Logic | 5d | pending | Totch eveJtsàbắ  buộu|
| 5 | Mockup Preview Component (Admin) | 1d | pending | Client-side CSS overlayr(khôa
| 6 | Order Processing + Email + Excel | 4d | pending | |
| 7 | Payment — **COD only** | 1d | pending | VNPay/MoMo defer Phase 2 |
| 8 | Admin + Order Management + Product Import | 5d | pending | Thêm CSV import flow |
| 9 | Polish, SEO, Deploy | 3d | pending | |

**Dependencies:** 1→2→3→4→5→6→7→8→9 (tuần tự)
**Effort tổng:** ~6 tuần

---

## 11. Kiến trúc cart & order

- **Cart = Zustand + localStorage** — không có server-side cart
- **Sync DB tại checkout** — khi khách submit đơn
- **CustomCartItem** = bundle: 1 item = 1 đội với N cầu thủ
- **Mandatory login** — Better Auth emailAndPassword, bắt buộc đăng nhập trước checkout
- **Cart [Sửa] button** — mở lại builder với pre-filled data từ CustomCartItem

---
## 12. Logo & ảnh sản phẩm

| Loại | Trạng thái | Xử lý |
|------|-----------|-------|
| Ảnh sản phẩm thật | Chưa có | Dùng placeholder trong dev |
| Logo khách upload | Runtime | Upload Cloudinary, cảnh báo nếu < 300px |
| Mockup PNG | Runtime (generated) | Canvas render → Cloudinary |
| Excel file | Runtime (generated) | ExcelJS → Cloudinary |

**Logo quality policy:** Chấp nhận logo chất lượng kém, tự động thêm note:
*"nếu được thì shop làm nét lại (nếu có thể)"* — không block checkout.

---

## 13. index.html hiện tại

File `index.html` giữ lại làm **tham khảo design**.
Next.js homepage (`/app/page.tsx`) sẽ follow style/layout của file này.
Design tokens (màu sắc, typography, shadow) tham khảo `:root` variables trong file.

---e.
**F
## 14. Quyết định kiến trúc đã xác nhận (tổng hợp)

| # | Quyết định | Lý do |
|---|-----------|-------|
| 1 | shadcn/ui + Tailwind | Copy-paste, no vendor lock-in |
| 2 | Step wizard | Clear flow, easy per-step validation |
| 3 | Không auto-decrement stock | Đơn giản MVP, shop tự quản lý |
| 4 | Admin: Order management + CSV product import | Không cần manual product CRUD web UI |
| 5 | Sleeve logo: user chọn trái/phải/cả hai | Linh hoạt |
| 6 | Tiếng Việt thuần | Không cần i18n |
| 7 | Dịch vụ in riêng | Defer sang Phase 2 |
| 8 | Mandatory login (Better Auth emailAndPassword) | Bắt buộc đăng nhập trước checkout |
| 9 | Cart = Zustand + localStorage | Đơn giản, no race conditions |
| 10 | printConfigJson Json? | Lưu đầy đủ transform cho admin preview |
| 11 | Mockup: Client-side CSS overlay | Loại bỏ Three.js + server-side canvas |
| 12 | Mobile touch events bắt buộc từ Phase 3 | Khách VN dùng mobile nhiều |
| 13 | Font: Dùng font mặc định dự án | Đơn giản hóa |
| 14 | COD on rPspo  iveo defer Ph1se 2 |
| 15 | Brand Montserrat | Braldrs: Đỏ|
|314,|àBragd colors: src/lib/theme.ts #FDỏ #E31E26, Và0g1#FDD017, Xanh dươn7 #00AEEF, XámD#A7A9ACtrên logo StarSport |
5 Đơn giản,
---6 | 4 image angles per ColorSet Front, back, sleeve,shots cho disply + builer|
| 17 | ColorSet 1-N ProdutImage | Mỗi CSetcónhiềuảh theo óc|
| 8Stockhiển hị cho khách | T productdetai pae |
| 19 | Phase 5: Full UIimplemention | Không chỉ eusable comnen

## 15. Câu hỏi còn mở

1. **Số lượng sản phẩm ban đầu:** Bao nhiêu sản phẩm seed data để test?
2. **Ảnh sản phẩm:** Khi nào có ảnh thật để replace placeholder?
3. **Domain:** Deploy lên domain riêng hay Vercel subdomain trước?
4. **Sandbox payment:** Khi nào đăng ký VNPay/MoMo sandbox để chuẩn bị Phase 2?
5. **Giới hạn tên cầu thủ:** Tối đa bao nhiêu ký tự cho player name + player number display?
Sesson4—2026-04-13 (Plan consistency check + clarification session)*
---
