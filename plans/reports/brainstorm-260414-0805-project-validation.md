# Brainstorm Report: Project Validation — Football Uniform E-Commerce

**Date:** 2026-04-14  
**Session:** 6 (Deep Project Validation)  
**Questions asked:** 14  
**Trigger:** User yêu cầu đọc toàn bộ plan + brainstorm để xác nhận hiểu dự án và phát hiện gaps

---

## Tổng quan

Dự án nhìn chung có kiến trúc solid, 5 session validation trước đã cover nhiều quyết định lớn. Tuy nhiên brainstorm session 6 phát hiện **8 gaps/mâu thuẫn** cần cập nhật vào plan trước khi implement.

---

## Gaps Phát Hiện & Quyết Định

### Gap 1 — isCustomizable: Không có spec rõ cách gán flag khi import
**Vấn đề:** Plan không chỉ rõ admin set `isCustomizable` ở đâu khi import CSV.  
**Lý do quan trọng:** Flag này quyết định sản phẩm có hiện builder sections hay không — nếu thiếu, toàn bộ builder flow bị broken.

**Options đã đánh giá:**
- Toggle trong wizard preview step ✅ (chọn)
- Column trong CSV supplier ✗ (yêu cầu phối hợp supplier)
- Auto-detect theo category ✗ (cứng, khó edge case)
- Chỉ set trong Product Edit ✗ (dễ bỏ sót)

**Quyết định:** Toggle checkbox `[☑ Cho phép tùy chỉnh mockup]` trên header mỗi product group trong **Step 3 - wizard preview**. Admin bật/tắt khi xem preview, không cần sửa CSV supplier.

---

### Gap 2 — CSV Images: Mâu thuẫn giữa Session 2 và Session 4
**Vấn đề:** Session 2: "upload trực tiếp per ColorSet trong preview step". Session 4: "CSV có 4 cột: hinh_anh_front/back/sleeve/shorts". Hai điều mâu thuẫn nhau.

**Quyết định:** **Upload thủ công trong preview step** (Session 2 đúng). Không cần 4 cột ảnh trong CSV. Admin upload ảnh per ColorSet (front/back/sleeve/shorts) trực tiếp trong Step 3 wizard, buffer local → upload Cloudinary khi Confirm.

**Lý do:** CSV là file từ supplier, supplier không cần cung cấp ảnh; ảnh do shop tự chụp/quản lý.

---

### Gap 3 — Mixed Cart: Behavior không được spec
**Vấn đề:** Không có spec khi cart có cả normal items (giày) lẫn đơn đội tùy chỉnh.

**Quyết định:** **Cùng 1 Order**. 1 Order chứa cả OrderItems (sản phẩm thường) và CustomOrders (đơn đội tùy chỉnh). Checkout 1 lần, 1 mã đơn, 1 email.

---

### Gap 4 — ProductVariant Scope
**Vấn đề:** Không rõ ProductVariant gắn với Product hay ColorSet.

**Quyết định:** ProductVariant gắn với **Product** (chung mọi ColorSet). Sizes S/M/L/XL áp dụng cho tất cả màu của sản phẩm. Đơn giản, phù hợp thực tế (áo bóng đá thường có đủ size cho mọi màu).

---

### Gap 5 — Phase 3 vs Phase 6 Scope Overlap
**Vấn đề:** Phase 3 có "checkout form + tạo đơn" + Phase 6 cũng có "order processing" — không rõ ranh giới.

**Quyết định:** 
- **Phase 3:** cart UI + checkout form + POST /api/orders **cho normal items only**
- **Phase 6:** extend POST /api/orders thêm CustomOrder handling + Excel + Email

---

### Gap 6 — Builder: Multiple Logos (THAY ĐỔI LỚN)
**Vấn đề:** Plan hiện tại chỉ có 1 logo upload. Không đủ cho use case thực tế (logo đội ở ngực, logo nhà tài trợ ở tay áo).

**Quyết định:** Multi-logo support.
- **Logo panel** bên trái canvas (sidebar nhỏ)
- Upload nhiều logos (PNG/JPG/SVG, max 5MB mỗi file)
- Drag logo từ panel thả vào canvas per angle
- Không hard-code góc — user tự drag vào vị trí bất kỳ
- Mỗi OverlayElement reference `logoId` cụ thể

**Updated `printConfigJson` data structure:**
```typescript
interface CustomConfig {
  teamName?: string
  logos: Array<{
    id: string           // local ID: 'logo_0', 'logo_1'...
    url: string          // Cloudinary URL sau khi upload
    qualityNote?: string // auto-set nếu < 300px
  }>
  angles: {
    front:   OverlayElement[]
    back:    OverlayElement[]
    sleeve:  OverlayElement[]
    shorts:  OverlayElement[]
  }
  players: Array<{
    sortOrder:    number
    playerName?:  string
    playerNumber?: number
    jerseySize:   string  // required, từ ProductVariant
    shortsSize?:  string
  }>
}

interface OverlayElement {
  id:       string
  type:     'logo' | 'text'
  logoId?:  string   // reference logos[].id (khi type='logo')
  text?:    string   // giá trị text (khi type='text')
  x:        number   // % từ left
  y:        number   // % từ top
  scale:    number   // zoom factor
  rotation: number   // degrees
  width?:   number   // % width (optional, cho resize)
}
```

**⚠️ Security:** SVG cần sanitize (strip `<script>`, event handlers) trước khi upload Cloudinary để tránh XSS.

---

### Gap 7 — Excel Paste: Thiếu spec column detection
**Vấn đề:** Plan chỉ nói "Clipboard API + TSV parser" không spec column order hoặc detection logic.

**Quyết định:** Flexible auto-detect:
1. Parse TSV từ clipboard
2. Pattern matching: cột chứa số nguyên 1–99 → Số áo; cột chứa S/M/L/XL/XXL → Size áo; cột còn lại có text dài → Tên; tương tự cho Size quần
3. Nếu confidence cao (>80%) → điền luôn vào player list
4. Nếu thấp → hiện **quick mapping dialog**: show sample data + dropdown để user confirm/sửa từng cột
5. Kết quả append vào player list, STT tự đánh số

---

### Gap 8 — Cancel Order: Thiếu email notification cho shop
**Vấn đề:** Plan không đề cập shop có nhận email khi khách huỷ đơn.

**Quyết định:** Gửi email cảnh báo shop khi customer cancel (status `pending → cancelled`).
- Subject: `[Đơn huỷ] ORD-YYYYMMDD-NNN — Tên khách`
- Body: thông tin khách, sản phẩm, lý do (nếu có)

---

## Các Quyết Định Nhỏ Xác Nhận

| Topic | Quyết định |
|---|---|
| Admin tài khoản đầu tiên | Seed script khi deploy (`ADMIN_EMAIL` + `ADMIN_PASSWORD` từ .env) |
| Overlay position khi đổi ColorSet | Giữ nguyên — position lưu trong cart item, độc lập với ColorSet |
| Print fee scope | Chỉ tính trên giá CustomOrder. Sản phẩm thường không bị tính phí in |
| Print fee khi nhiều CustomOrder | Tính riêng từng CustomOrder (không pool tổng bộ) |
| Size OOS trong builder dropdown | Show đủ size từ ProductVariant, disable nếu stock=0 (tooltip "Hết hàng") |
| Out of stock normal products | "Hết hàng" + disable button per-size. Sản phẩm vẫn hiện trong catalog |
| Order number reset | Reset hàng ngày: ORD-YYYYMMDD-001, 002... |
| Multi custom orders trong cart | Cho phép nhiều CustomOrders (đặt cho nhiều đội khác nhau) |
| Logo format | PNG/JPG/SVG, max 5MB (SVG phải sanitize) |

---

## Phase Impact

| File | Thay đổi |
|---|---|
| `plan.md` | Fix Session 4 về 4 cột ảnh CSV; thêm session 6 log với 8 decisions |
| `phase-01-project-setup.md` | ProductVariant per Product; schema `printConfigJson` multi-logos; SVG sanitize note |
| `phase-03-product-catalog.md` | Phase 3 scope: cart + checkout normal only; size chips disabled OOS; stock UX "Hết hàng" |
| `phase-04-custom-builder.md` | **MAJOR**: multi-logo panel + drag-drop; Excel flexible detect + mapping dialog; updated `printConfigJson` structure |
| `phase-06-order-processing.md` | Mixed order (normal + custom) in 1 Order; cancel → email shop; print fee per CustomOrder; multi-logo email attachments |
| `phase-08-admin.md` | `isCustomizable` toggle trong wizard preview step; CSV images = manual upload (no URL columns) |

---

## Unresolved Questions
- None — tất cả gaps đã được giải quyết trong session này.
