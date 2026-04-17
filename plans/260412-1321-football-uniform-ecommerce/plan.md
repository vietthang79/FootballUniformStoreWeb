---
title: "Custom Mockup Platform - Football Uniform"
description: "Website custom mockup đồng phục bóng đá - khách tự thêm mockup vào vị trí tùy chọn trên ảnh sản phẩm thật"
status: pending
priority: P1
effort: ~7w
issue: ~
branch: main
tags: [frontend, backend, database, feature, ecommerce]
created: 2026-04-12
---

# Football Uniform E-Commerce + Mockup Builder

## Tech Stack
Next.js 15 App Router | PostgreSQL + Prisma | Better Auth | Zustand | React Hook Form | **shadcn/ui** | Cloudinary | Resend | ExcelJS | COD (MVP) | Vercel + Railway

## Monorepo Structure (pnpm Workspaces)

```bash
FootballUniformStoreWeb/
# Workspace Configuration
```
package.json          # Workspace root with scripts: dev, build, scraper
pnpm-workspace.yaml   # packages: ['apps/*', 'packages/*']

# Applications
```bash
apps/web/             # Next.js 15 Full-stack App (@football-store/web)
  src/
    app/              # App Router (API routes + pages)
    components/       # React components
    lib/              # Database, auth, cloudinary configs
    types/            # Import from @football-store/shared-types
  package.json        # Dependencies: Next.js, Prisma, Better Auth, etc.
  prisma/
    schema.prisma     # Complete database schema
    seed.ts          # Dev/test data

# Shared Packages
```
packages/
  shared-types/       # TypeScript interfaces (@football-store/shared-types)
    src/
      index.ts       # Export: ProductCategory, CsvRow, ScrapedProduct, etc.
    package.json     # Dependencies: TypeScript, Zod
    
  scraper/           # Playwright CLI tool (@football-store/scraper)
    src/
      scraper.ts     # Main scraping logic
      sites/         # Site-specific scrapers
    package.json     # Dependencies: Playwright, @football-store/shared-types
    tsconfig.json    # Node.js target
```

## Workspace Scripts:
```json
{
  "scripts": {
    "dev": "pnpm --filter web dev",
    "build": "pnpm --filter shared-types build && pnpm --filter web build",
    "scraper": "pnpm --filter scraper dev",
    "db:push": "pnpm --filter web db:push",
    "db:seed": "pnpm --filter web db:seed"
  }
}
```

## Package Dependencies:
- `@football-store/web` depends on `@football-store/shared-types: workspace:*`
- `@football-store/scraper` depends on `@football-store/shared-types: workspace:*`
- Build order: shared-types first, then web/scraper in parallel

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
| 3 | Homepage (Marketing Landing Page Only) | pending | 1.5d | [phase-03](./phase-03-homepage.md) |
| 4 | Custom Mockup Builder UI + Logic | pending | 5d | [phase-04](./phase-04-custom-builder.md) |
| 5 | Admin Preview Component | pending | 1d | [phase-05](./phase-05-mockup-preview.md) |
| 6 | Order Processing + Email + Excel | pending | 4d | [phase-06](./phase-06-order-processing.md) |
| 7 | Payment Integration (COD only) | pending | 1d | [phase-07](./phase-07-payment.md) |
| 8 | Admin + Order Management + Product Import | pending | 5d | [phase-08](./phase-08-admin.md) |
| 9 | Polish, SEO, Deploy | pending | 3d | [phase-09](./phase-09-polish-deploy.md) |
| 10 | Product Catalog + Cart + Checkout | pending | 6d | [phase-10](./phase-10-product-catalog.md) |
| 11 | Data Scraper Tool (Local Dev Seed) | pending | 4d | [phase-11](./phase-11-data-scraper.md) |
| 12 | Payment Gateways (VNPay/MoMo) — Post-MVP | deferred | TBD | [phase-12](./phase-12-payment-gateways.md) |

## Dependencies
- Phase 2 depends on Phase 1
- Phase 3 depends on Phase 1+2 (DB for featured products, auth for nav user state)
- Phase 4 depends on Phase 2 (auth required for checkout)
- Phase 5 depends on Phase 4 (product data)
- Phase 6 depends on Phase 5 (builder state)
- Phase 7 depends on Phase 5+6 (custom order data)
- Phase 8 depends on Phase 7 (order flow)
- Phase 9 depends on Phase 7+8
- Phase 10 depends on all

## Key Architecture Decisions
1. **Cart = Zustand + localStorage + DB sync on login** — guest browse with localStorage, merge to server `CartItem` table upon login (Session 10)
2. **Mockup order = bundle cart item** — 1 cart item = 1 team order with N players
3. **Custom Mockup = `react-moveable` overlay** (Session 10 override) — drag + resize + rotate + snap. Coords % (0-1) to scale across ColorSet switches
4. **Image slots = dynamic tabs** (Session 10) — tabs generated từ `ProductImage.sortOrder`, không cứng 4 góc
5. **Excel paste = native Clipboard API** — no table library
6. **Logos uploaded to Cloudinary** — per CustomCartItem scope, không persist cross-session. Quality badge nếu < 300x300px
7. **Stock = display only** — không validate/decrement, admin quản lý thủ công (Session 1 + 10)
8. **Email = fire-and-forget** — trust Resend, không retry/log (Session 10)
9. **Mandatory login** — Better Auth emailAndPassword + admin plugins; no anonymous/guest checkout
10. **Admin user = seed script** — đọc ADMIN_EMAIL/PASSWORD từ `.env` lần đầu deploy (Session 10)

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

### Session 8 — 2026-04-14
**Trigger:** Gap analysis brainstorm — rà soát toàn bộ plan trước khi implement
**Questions asked:** 15

#### Confirmed Decisions
1. **[DB Schema]** Player model: 1 field `size` duy nhất (required), xóa `shortsSize` — áo + quần cùng size
2. **[Email]** Email scope final: Shop nhận (new order + cancel); Khách KHÔNG nhận khi đặt; Khách CÓ nhận khi status → "Đang giao"/"Đã giao"
3. **[DB Schema]** ProductImage: xóa `angle` enum, thay bằng `sortOrder: Int` — ảnh tự do, không hard-code 4 góc
4. **[DB Schema]** ProductVariant: đổi FK từ `productId` → `colorSetId` — stock per màu riêng lẻ (override Session 6)
5. **[Architecture]** printConfigJson: `angles.{front/back/sleeve/shorts}` → `overlays: OverlayElement[][]` (indexed by image slot)
6. **[UX]** ColorSet switch: giữ overlay positions, đổi ảnh nền; tab thừa bị ẩn (không xóa data)
7. **[UX]** Gallery dual-mode: thumbnail grid (normal view) + tabs (builder view), cùng 1 trang `/products/[slug]`
8. **[UX]** Order confirmation page: mã đơn + tổng tiền + player size summary (M×2, L×4...) — không có mockup canvas
9. **[DB Schema]** User model thêm: `shippingName`, `shippingPhone`, `shippingAddress` (nullable) — auto-fill checkout
10. **[UX]** Logo quality: badge "⚠ Chất lượng thấp" + note tự động, không block upload
11. **[UX]** Search + Filter: AND logic — search within category (`?q=...&category=...`)
12. **[Feature]** Status email to customer: gửi email khi admin chuyển → "Đang giao" và "Đã giao"
13. **[Scope]** Product edit page: FULL scope — info + ảnh (reorder/add/delete) + ColorSet + stock per variant
14. **[Scope]** Orphaned logos Cloudinary: chấp nhận bỏ qua cho MVP
15. **[Priority]** Phase 11 Scraper: nâng lên P1 (cần trước launch); CSV format cập nhật (1 row = ColorSet × size)

#### Phase Impact
- phase-01: schema changes (ProductVariant FK, angle→sortOrder, User shipping fields, Player 1 size)
- phase-03/10: order confirmation UX, auto-fill shipping, search+filter AND
- phase-04: overlays[][] structure, gallery dual-mode, ColorSet switch UX, logo quality badge
- phase-06: email scope final (no customer email on order, yes on status)
- phase-08: product edit FULL, status PATCH triggers customer email, import update for ColorSet-based variants
- phase-11: P1 priority, CSV format update

---
### Session 9 — 2026-04-14
**Trigger:** Pre-implementation gap analysis — review toàn bộ plan + resolve conflicts
**Questions asked:** 9

#### Confirmed Decisions
1. **[DB Schema]** ProductVariant FK → `colorSetId` (Session 8 wins over Session 6) — stock riêng từng màu
2. **[DB Schema]** ProductImage thêm `label: String?` — admin đặt tên tab trong builder
3. **[UX]** Overlay indexed by slot index, tab thừa ẩn khi ColorSet switch (data preserved)
4. **[Phase split]** Phase 3 vs 10: giữ nguyên cả 2, khi implement check cái nào đã làm thì skip
5. **[UX]** Normal product detail: cùng `/products/[slug]`, if/else layout dựa vào `isCustomizable`
6. **[Phase 11]** Upgrade P2 → P1, start song song với Phase 1-2 (cần data trước launch)
7. **[Email]** Admin cancel KHÔNG gửi email cho khách — shop tự liên hệ qua Zalo/điện thoại
8. **[Builder]** Drag-drop library: `react-draggable` (không dùng react-moveable)
9. **[Cart]** Zustand store: 2 arrays riêng biệt (`items[]` + `customOrders[]`)

#### Phase Impact
- phase-01: thêm `ProductImage.label: String?`
- phase-04: đổi library sang `react-draggable`; tabs labeled by `ProductImage.label`; xóa `shortsSize` khỏi Player type
- phase-06: admin cancel không gửi email cho khách (no change needed — đã đúng)
- phase-11: priority P2 → P1
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

### Session 10 — 2026-04-17
**Trigger:** Pre-implementation gap resolution — review toàn bộ plan + 25 gaps phát hiện
**Questions asked:** 20

#### Confirmed Decisions

**A. Builder & Canvas (Phase 4)**
1. **[Library]** Override Session 9 → **`react-moveable`** (drag + resize + rotate + snap), không dùng `react-draggable`
2. **[ColorSet switch]** Overlay coords dạng `%` (x/y/w/h 0-1), giữ nguyên khi switch — scale tự động theo dimensions ảnh mới
3. **[Image slots]** **Dynamic tabs** theo `ProductImage.sortOrder`, 2-10 ảnh đều OK. Override Phase 5 assumption 4 góc
4. **[Logo library]** Scope **per CustomCartItem** only — đóng tab mất logos. Không user-persist
5. **[Logo quality]** Auto-detect `width < 300 OR height < 300` → badge "⚠ Chất lượng thấp" + note

**B. Stock & Order Flow**
6. **[Stock]** Display only — UI disable size khi stock=0, KHÔNG validate/decrement khi order
7. **[Cart sync]** Thêm `CartItem` table. Merge localStorage + server cart khi login (union)
8. **[Order number]** Retry 3 lần → fail = return 500, client hiện "Thử lại sau"
9. **[Print fee]** Skip edge case recalc — pricing rules stable MVP

**C. Admin & CSV Import (Phase 8)**
10. **[Admin setup]** Seed script đọc `ADMIN_EMAIL` + `ADMIN_PASSWORD` từ `.env` — tự tạo lần đầu deploy
11. **[CSV invalid]** Preview step highlight bad rows + inline editable — admin fix manual trước confirm
12. **[CSV stock]** Wizard step cuối: admin nhập stock per (ColorSet × size) manual trong table

**D. Email & Failure (Phase 6)**
13. **[Email fail]** Fire-and-forget — không retry/log/dashboard. Trust Resend
14. **[Cancel email]** Shop only (current). Khách thấy status `cancelled` trong profile
15. **[Excel URL]** Public Cloudinary URL + UUID slug — accept low security risk

**E. Phase Structure**
16. **[Phase 3]** Trim scope → **chỉ homepage** (hero, featured products, testimonials). Effort 2d → 1.5d
17. **[Phase 11]** Local-only one-shot — không deploy, không cron
18. **[Phase 12 NEW]** Payment gateways placeholder post-MVP (VNPay/MoMo/Bank)

**F. Mobile & Cleanup**
19. **[Mobile builder]** Giữ desktop-only < 768px (notice). Catalog/cart/checkout responsive
20. **[Cleanup]** Xóa `index.html` + `components/custom-builder/` — Next.js clean slate

#### Phase Impact
- plan.md: Session 10 log, phase table update (Phase 3 1.5d, Phase 10 6d, Phase 12 new), Key Architecture Decisions rewritten
- phase-01: add `CartItem` model, seed admin script, cleanup step
- phase-03: remove cart/checkout/listing → chỉ marketing landing
- phase-04: `react-moveable` override, dynamic tabs, logo quality check, % coords
- phase-05: dynamic tabs (không hardcode 4 angles)
- phase-06: remove email retry, order number fail = 500, Excel UUID upload
- phase-08: CSV wizard stock-per-size step, bad rows highlight + inline fix
- phase-10: expand scope (listing + filters + detail + cart + checkout + sync API), 5d → 6d
- phase-11: mark local-only dev tool
- phase-12: NEW placeholder file

---

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
