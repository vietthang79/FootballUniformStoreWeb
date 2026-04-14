# Brainstorm Report: Data Scraper Tool — Phase 11

**Date:** 2026-04-14
**Trigger:** User request — cần product data phong phú cho e-commerce demo/test
**Session type:** Brainstorm + Decision

---

## Problem Statement

Project football uniform e-commerce cần dữ liệu sản phẩm thực tế (tên, giá, màu sắc, size, ảnh) để populate DB cho dev/demo. Manual entry quá chậm. Giải pháp: scrape từ các sites bán đồ thể thao VN.

---

## Target Sites

| Site | Anti-bot | Notes |
|------|----------|-------|
| soccerstore.vn | Thấp | WooCommerce, data bóng đá đúng nhất |
| sport9.vn | Thấp | Site VN đơn giản |
| beck.vn | Thấp | Site VN |
| decathlon.vn | Trung bình | CDN/Cloudflare Basic |
| ~~prodirectsport.com~~ | Cao | Cloudflare — skip |
| ~~unisportstore.com~~ | Cao | Cloudflare — skip |

---

## Decisions Made

| # | Quyết định | Lý do |
|---|-----------|-------|
| 1 | Node.js + Playwright | Cùng ecosystem Next.js, tái dụng Cloudinary SDK |
| 2 | Output: CSV → Admin wizard import | Review/edit thủ công trước khi vào DB |
| 3 | Images: download + upload Cloudinary | Chỉ ảnh đầu (front), 3 góc còn lại admin fill wizard |
| 4 | Anti-bot: Playwright Stealth + random delay 1.5-6s | Đủ cho 4 sites VN |
| 5 | Location: `scripts/scraper/` trong repo | Tiện dùng chung tsconfig/deps |
| 6 | Scope: áo, quần, bộ, giày, phụ kiện | Tất cả danh mục bóng đá |

---

## Architecture

```
scripts/scraper/
├── core/playwright-base.ts      # Browser, rate limiter, retry
├── core/cloudinary-uploader.ts  # Upload ảnh → Cloudinary URL
├── core/image-downloader.ts     # Download image buffer
├── core/csv-writer.ts           # CSV format phase-08
├── sites/soccerstore-vn.ts
├── sites/sport9-vn.ts
├── sites/beck-vn.ts
├── sites/decathlon-vn.ts
├── config/sites.config.ts       # Selectors, rate limits per site
├── types/scraper-types.ts
└── run.ts                       # CLI entry point
```

## CSV Schema

```
ten_sp,mo_ta,gia_ban,danh_muc,mau_sac,mau_chinh,mau_phu,size,so_luong,anh_truoc
```

3 cột `anh_sau/anh_tay/anh_quan` để trống — admin fill qua wizard.

## Anti-bot Strategy

- Playwright stealth plugin (ẩn headless fingerprint)
- Random delay min/max per site (không cố định)
- Exponential backoff khi gặp 429/block
- User-agent rotation (pick once per session, không per-request)
- Robots.txt compliance check

## Effort

~3-4 ngày: 0.5d setup, 1d core, 1.5d scrapers, 0.5d CLI + test

---

## Unresolved Questions

- Hex màu: nếu site không có swatch CSS → admin fill thủ công (acceptable)
- Cloudinary quota: free 25 credits/month — đủ cho ~200 ảnh MVP
