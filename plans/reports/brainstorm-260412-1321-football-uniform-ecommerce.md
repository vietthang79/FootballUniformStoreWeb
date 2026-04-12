# Brainstorm Report — Football Uniform E-Commerce + Custom Builder

**Date:** 2026-04-12  
**Project:** FootballUniformStoreWeb  
**Type:** Business Brainstorm

---

## Vấn đề cốt lõi cần giải quyết

Khách hàng đặt đồng phục đội bóng qua Zalo/Facebook → mô tả bằng lời → shop hiểu sai → in xong không đúng ý → phải làm lại → tốn thời gian và tiền của cả hai bên.

**Mục tiêu:** Khách tự thiết kế trực tiếp trên web, thấy kết quả ngay trước khi đặt hàng → shop nhận đơn đủ thông tin → in và giao, không cần hỏi qua lại.

---

## Hai luồng bán hàng song song

### Luồng 1 — Mua hàng thông thường
Dành cho: giày, phụ kiện, áo thi đấu lẻ, đồ không cần in.

```
Xem danh sách → Xem chi tiết → Chọn size/màu → Giỏ hàng → Thanh toán
```

### Luồng 2 — Đặt đồng phục theo yêu cầu (Custom Builder)
Dành cho: bộ đồ bóng đá, áo khoác đội.

```
Xem sản phẩm → Bấm [Custom] → Thiết kế → Xem trước kết quả → Giỏ hàng → Thanh toán
```

Hai luồng **cùng tồn tại trên một trang web**, không tách rời. Sản phẩm nào có tính năng Custom thì hiển thị thêm nút [Custom] bên cạnh nút [Thêm giỏ hàng].

---

## Ai thấy nút Custom?

Sản phẩm được shop đánh dấu là "có thể Custom" từ trang quản lý. Frontend tự động hiển thị/ẩn nút dựa trên đánh dấu này.

- Danh sách sản phẩm: mỗi card bộ đồ/áo khoác có nút [Custom] nhỏ
- Trang chi tiết sản phẩm: hai nút rõ ràng — [Thêm giỏ hàng] và [Custom theo yêu cầu]

---

## Custom Builder — Thiết kế từng bước

Mở trên cùng trang hoặc trang riêng, được nạp sẵn thông tin sản phẩm khách đang xem.

### Bước 1 — Chọn màu sắc
- Mỗi sản phẩm có sẵn **bộ màu định sẵn** (ví dụ: Xanh Navy + Trắng, Đỏ + Đen...)
- Khách chọn → ảnh sản phẩm đổi theo bộ màu đó (ảnh thật, chụp sẵn)
- Không dùng color picker tự do → tránh tranh cãi màu sau khi in

### Bước 2 — Logo & tên đội
| Mục | Mô tả |
|---|---|
| Logo đội / CLB | Upload ảnh (PNG nền trong, khuyến nghị ≥ 500px) |
| Logo nhà tài trợ | Upload ảnh (optional) |
| Logo giải đấu | Upload ảnh (optional) |
| Flag / Patch tay áo | Upload ảnh (optional) |
| Tên đội | Nhập chữ (optional) |

Hệ thống cảnh báo nếu ảnh nhỏ hơn 300x300px. Hướng dẫn ngắn ngay tại chỗ upload.

### Bước 3 — Danh sách cầu thủ
Bảng nhập thông tin, mỗi dòng là 1 cầu thủ:

| STT | Tên cầu thủ | Số áo | Size áo | Size quần |
|---|---|---|---|---|
| 1 | Nguyễn Văn A | 10 | L | M |
| 2 | Trần Văn B | 7 | XL | XL |

**Tính năng quan trọng:** Hỗ trợ copy-paste trực tiếp từ Excel vào bảng. Đội nào cũng có danh sách Excel sẵn rồi — không bắt nhập tay từng người.

Số bộ được tính tự động → hiển thị phí in ngay lập tức (xem phần Phí in).

### Bước 4 — Cấu hình in

**Mặt trước áo:**
- [ ] Logo đội / CLB — ngực trái (optional)
- [ ] Logo nhà tài trợ (Sponsor) — giữa ngực (optional)
- [ ] Số nhỏ — ngực phải (optional)

**Tay áo:**
- [ ] Logo giải đấu (optional)
- [ ] Logo sponsor nhỏ (optional)
- [ ] Flag / Patch (optional)

**Mặt sau áo** — tick cái nào muốn in, thứ tự luôn từ trên xuống:
- [ ] Tên đội (trên cùng)
- [ ] Số (giữa)
- [ ] Tên cầu thủ (dưới cùng)

Combo phổ biến:
- Tên cầu thủ + Số + Tên đội → in 3 dòng
- Tên cầu thủ + Số → in 2 dòng
- Tên đội + Số → in 2 dòng (không in tên cá nhân)

**Quần:**
- [ ] In số quần — chọn bên: ● Trái  ○ Phải

### Bước 5 — Xem trước kết quả (Mockup)
- Chọn góc xem: [Mặt trước] [Mặt sau] [Quần]
- Chọn xem của cầu thủ nào: dropdown danh sách
- Tên, số, logo hiển thị overlay lên ảnh thật của sản phẩm
- *Lưu ý nhỏ dưới mockup: "Màu sắc thực tế có thể khác nhẹ so với màn hình"*

### Bước 6 — Thêm vào giỏ hàng
Giỏ hàng hiển thị đơn custom như một gói:
> **Bộ đồ Barcelona (Xanh Navy) — Đội ABC FC — 12 bộ**  
> [Chỉnh sửa] [Xóa]

Nút Chỉnh sửa mở lại builder với toàn bộ thông tin đã nhập, không mất dữ liệu.

---

## Chính sách phí in

| Số bộ trong đơn | Phụ phí |
|---|---|
| Từ 10 bộ trở lên | **Miễn phí in** |
| 6 – 9 bộ | +10% tổng đơn hàng |
| Dưới 6 bộ | +20% tổng đơn hàng |

**1 bộ = 1 áo + 1 quần** (của 1 cầu thủ)

### Nudge tăng đơn tự động
Khi khách nhập danh sách cầu thủ, hệ thống hiển thị ngay:
- 5 bộ → `⚠️ Đang +20% phí in — thêm 1 bộ để giảm xuống +10%`
- 8 bộ → `📦 Đang +10% phí in — thêm 2 bộ để miễn phí hoàn toàn!`
- 10+ bộ → `✅ Miễn phí in — áp dụng cho đơn này`

---

## Shop nhận đơn như thế nào?

Khi khách thanh toán thành công, hệ thống tự động gửi email cho shop gồm:
- Thông tin đơn hàng (tên khách, SĐT, địa chỉ)
- Bảng danh sách cầu thủ (file Excel đính kèm)
- File logo đã upload (đính kèm)
- Cấu hình in: màu, vị trí logo, mặt sau in gì, quần in không
- Ảnh mockup đã xem trước
- Tổng tiền và phụ phí in (nếu có)

Shop mở email → đọc → in → đóng gói → giao. Không cần hỏi thêm gì.

---

## Dịch vụ in theo yêu cầu riêng (in lên đồ tự mang)

**Chưa triển khai trong giai đoạn này.** Để sau khi hoàn thiện luồng bán online.

---

## Rủi ro cần xử lý

| Rủi ro | Cách xử lý |
|---|---|
| Khách upload logo chất lượng thấp | Cảnh báo ngay + hướng dẫn chuẩn file |
| Kỳ vọng màu vs màu thật khi in | Disclaimer + ảnh mẫu sản phẩm đã in thật |
| Khách muốn sửa sau khi vào giỏ | Nút "Chỉnh sửa" giữ nguyên toàn bộ data |
| Tên cầu thủ quá dài, không vừa áo | Giới hạn ký tự + cảnh báo trong preview |

---

## Tóm tắt giá trị mang lại

- **Khách:** Thấy kết quả trước khi đặt, chủ động thiết kế, không sợ shop làm sai
- **Shop:** Nhận đơn đầy đủ thông tin, không cần hỏi qua lại, giảm sai sót và làm lại
- **Kinh doanh:** Nudge tự nhiên tăng số bộ mỗi đơn, tăng doanh thu in

---

*Báo cáo này là tài liệu business, không đề cập chi tiết kỹ thuật. Xem kế hoạch triển khai tại `plans/260412-1321-football-uniform-ecommerce/`*
