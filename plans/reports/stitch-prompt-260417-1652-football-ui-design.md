# Stitch / AI Design Prompt — Football Uniform Custom Mockup Platform

**Target tool:** Google Stitch (or similar AI UI/Figma generator)
**Scope:** 2 screens (Homepage + Custom Builder)
**Style:** Modern Sports Energetic
**Copy language:** Vietnamese
**Output:** Desktop-first (1440×900), responsive variants optional

---

## 📋 The Prompt (Copy below this line into Stitch)

---

Design a professional, modern sports e-commerce web application for **StarSport** — a Vietnamese football uniform customization platform. Users upload their club logos, drag them onto real product photos, and order batch team uniforms. Generate **2 high-fidelity desktop screens** (1440×900) with production-ready polish.

### 🎯 Brand Identity

- **Name:** StarSport (đồng phục bóng đá tuỳ chỉnh)
- **Positioning:** Premium yet accessible — serious football teams, amateur clubs, corporate teams in Vietnam
- **Voice:** Confident, energetic, trustworthy — "đồng phục chất lượng, tự do thiết kế"
- **Target users:** Team captains, club managers, event organizers (ages 22-45, sports enthusiasts)

### 🎨 Design System (use exactly)

**Colors:**
- Primary (energy/CTA): `#FDD017` (saturated warm yellow — jersey gold)
- Secondary (power/accent): `#E31E26` (bold Vietnamese red)
- Info/link: `#00AEEF` (electric sky blue)
- Neutral text: `#1A1A1A` (near black), `#6B7280` (muted gray)
- Surface: `#FFFFFF` primary, `#F9FAFB` elevated, `#111827` dark sections
- Borders/dividers: `#E5E7EB`

**Gradients (use sparingly for impact):**
- Hero overlay: `linear-gradient(135deg, #E31E26 0%, #FDD017 100%)` (red→yellow diagonal)
- CTA hover: `linear-gradient(90deg, #FDD017 0%, #F5A623 100%)`

**Typography:**
- Family: **Montserrat** (400, 500, 600, 700, 800, 900)
- Display (H1): 72px / 80px line-height / 900 weight / -0.02em tracking / uppercase
- H2: 48px / 56px / 800 / -0.01em
- H3: 32px / 40px / 700
- Body L: 18px / 28px / 400
- Body: 16px / 24px / 400
- Caption: 13px / 18px / 500 / uppercase / 0.08em tracking

**Visual motifs (Sports Energetic):**
- Diagonal slashes and angled section dividers (15° skew)
- Bold drop shadows on CTAs (offset 0 8px 24px `#E31E26`/20)
- Chunky rounded corners (12px cards, 8px buttons, 24px hero elements)
- Jersey number typography (massive outlined numerals as decorative backgrounds)
- Halftone dot patterns subtle in dark sections
- Motion-implying photography (athletes mid-action, dynamic crops)
- Badge elements with chevron/shield shapes

**Component style (follow shadcn/ui + Radix patterns):**
- Buttons: solid filled primary (#FDD017 bg, #1A1A1A text), outlined secondary, ghost tertiary
- Cards: `rounded-xl`, `border`, subtle shadow, hover lift `translate-y-[-4px]`
- Inputs: 48px height, `rounded-lg`, focus ring in primary color
- Tabs: underlined active state with primary color 3px border-bottom

---

## 🖥️ SCREEN 1: Homepage (Marketing Landing)

**Route:** `/`
**Viewport:** 1440 × (scroll)
**Goal:** Convert visitors to product browsers. Communicate brand energy + core value (custom team uniforms fast).

### Section 1.1 — Top Navigation (sticky, 80px tall)

```
[StarSport Logo]  [Trang chủ] [Sản phẩm] [Tuỳ chỉnh đồng phục] [Liên hệ]     [🔍] [Giỏ hàng ▸ 2] [Tài khoản ▾]
```
- Logo: bold wordmark "STARSPORT" in black with red accent dot
- Links: 15px / 600 weight / black / 32px gap
- Cart button: yellow background pill with count badge in red
- Account: avatar circle if logged in, else "Đăng nhập" link

### Section 1.2 — Hero (600px tall, full-bleed)

Full-width image carousel with 3 slides. First slide:
- **Background:** Photo of football team wearing custom jerseys mid-celebration (dark overlay 40% black gradient)
- **Foreground content** (left-aligned, 60% width):
  - Eyebrow: "ĐỒNG PHỤC BÓNG ĐÁ TUỲ CHỈNH" — 13px uppercase yellow
  - Headline (H1 massive): "THIẾT KẾ NGAY\nGIAO NHANH TRONG 7 NGÀY" — 80px, white, tight, 2 lines
  - Subtitle: "Upload logo đội, kéo thả lên áo, đặt hàng chỉ với 6 bộ." — 20px, white/90%
  - CTA row (2 buttons side by side):
    - Primary: "BẮT ĐẦU THIẾT KẾ →" — yellow bg, black text, 56px tall, bold
    - Secondary: "Xem sản phẩm" — white outlined ghost
  - Trust row below: "✓ Giao toàn quốc  ✓ Hỗ trợ từ 6 bộ  ✓ In chất lượng cao" — 14px white
- **Right side:** Diagonal cut showing mockup of a red/yellow jersey with visible custom logos applied — floating with shadow
- **Carousel dots** bottom-center, yellow active

### Section 1.3 — Stats Ribbon (120px tall, bg #111827 dark)

4-column stats with jersey-number styling (massive outlined numerals behind):
- `500+` Đội đã đặt hàng
- `10,000+` Áo đã in
- `7 ngày` Giao hàng
- `4.9/5` Đánh giá
Numbers in yellow 56px / 900, labels below in white 14px uppercase tracking.

### Section 1.4 — Features (4-column grid, padding 96px vertical)

Section header centered:
- Eyebrow: "TẠI SAO CHỌN STARSPORT" (yellow, 13px uppercase)
- H2: "Công cụ thiết kế mạnh mẽ" (48px, black)

4 feature cards (equal width, gap 24px):
1. **🎨 Tự do thiết kế** — "Kéo thả logo, text, số áo lên ảnh sản phẩm thật. Preview ngay tức thì."
2. **⚡ Nhanh chóng** — "Từ upload logo đến đặt hàng chỉ trong 5 phút."
3. **✂️ Chất lượng cao** — "In chuyển nhiệt cao cấp, đường may tỉ mỉ, bảo hành 12 tháng."
4. **🚚 Giao toàn quốc** — "Ship 64 tỉnh thành. Free ship cho đơn từ 10 bộ."

Each card: white bg, rounded-xl, 32px padding, yellow icon circle (64px), title 20px / 700, desc 15px / gray, hover scale 1.02 + shadow lift.

### Section 1.5 — Featured Products (8-item grid, 4 cols × 2 rows)

Section header:
- Eyebrow: "SẢN PHẨM NỔI BẬT"
- H2: "Thiết kế hot nhất tháng"
- Right side: "Xem tất cả →" link

Product card (reusable):
```
┌──────────────────┐
│ [IMG 1:1 ratio]  │  ← product photo on gray bg, "TUỲ CHỈNH" badge top-left yellow
│                  │
├──────────────────┤
│ Áo CLB Phoenix   │  ← 18px / 700
│ Từ 250.000₫      │  ← 16px / 600 red
│ ●●● (3 màu)      │  ← color chips row 14px
│ [Tuỳ chỉnh ngay] │  ← 40px yellow button full-width
└──────────────────┘
```
Hover: image zoom 1.05, shadow deepen, button scale.

### Section 1.6 — How It Works (4-step process, diagonal section)

Background: Dark (#111827) with subtle red-yellow gradient overlay. Section has 15° angled top cut.
Header white: "CÁCH THỨC HOẠT ĐỘNG"

4 horizontal steps with connector arrows:
1. `01` Chọn sản phẩm
2. `02` Upload logo & thiết kế
3. `03` Nhập danh sách cầu thủ
4. `04` Đặt hàng & giao nhận

Each step: huge yellow outlined "01" numeral 120px, title below 24px / 700 white, description 15px gray.

### Section 1.7 — Testimonials (3-card carousel)

Section header centered on light gray bg (#F9FAFB):
- Eyebrow: "KHÁCH HÀNG NÓI GÌ"
- H2: "Đội bóng tin tưởng chúng tôi"

3 testimonial cards:
- Customer photo circle (64px) + name + team label ("FC Sao Vàng - Captain")
- 5-star yellow rating
- Quote (18px italic): "Giao hàng đúng hẹn, chất lượng in sắc nét, đội mình hài lòng!"
- Badge chip showing product bought ("Áo CLB · 15 bộ")

### Section 1.8 — CTA Banner (full-width, 320px tall)

Diagonal red-yellow gradient bg. Center content:
- H2 white: "Sẵn sàng thiết kế đồng phục cho đội của bạn?"
- Subtitle: "Bắt đầu miễn phí. Không cần tài khoản."
- Single massive CTA: "THIẾT KẾ NGAY →" black text, white pill bg, 64px tall, huge shadow

### Section 1.9 — Footer (dark bg #1A1A1A, 320px tall)

4-column layout:
- Column 1: StarSport logo (yellow) + tagline "Đồng phục bóng đá chất lượng, thiết kế tự do" + social icons (Facebook, Instagram, Zalo, TikTok)
- Column 2: "Sản phẩm" — Áo đồng phục, Quần, Găng tay, Giày
- Column 3: "Hỗ trợ" — Hướng dẫn đặt hàng, Chính sách đổi trả, Bảo hành, Liên hệ
- Column 4: "Liên hệ" — Hotline with phone icon, Email, Địa chỉ, MST

Bottom bar: "© 2026 StarSport. Thiết kế & phát triển bởi StarSport Team." — 13px gray, centered

---

## 🎨 SCREEN 2: Custom Builder (Product Detail — customizable=true)

**Route:** `/products/[slug]` (customizable products only)
**Viewport:** 1440 × (scroll)
**Goal:** Power user interface for team uniform designers. Must feel like a professional tool (Canva/Figma-level UX) while staying approachable.

### Layout: Two-column, 50/50 split below top nav

Reuse top navigation from homepage.

### Section 2.1 — Breadcrumb + Product Header (88px tall)

```
Trang chủ / Áo đồng phục / Áo CLB Phoenix 2026

[H2] Áo CLB Phoenix 2026                              [Tag: TUỲ CHỈNH] [Tag: MỚI]
     ★★★★★ 4.8 (127 đánh giá)
```

### Section 2.2 — LEFT COLUMN: Product Gallery + Canvas Preview (sticky, 50% width)

**Top card (aspect 1:1, max 600px):** Main product image display showing current design state. This is the INTERACTIVE CANVAS.

Canvas chrome:
- Top bar: "Xem trước thiết kế" label + [zoom out ⊖] [100%] [zoom in ⊕] [⛶ fullscreen]
- Canvas area: white bg, subtle checkered pattern, CENTER current ProductImage with OVERLAY ELEMENTS (logos positioned by user)
- Selected overlay: yellow dashed border with corner resize handles (8px circles) + top rotate handle (attached to line) + center-top delete X button in red circle

**Below canvas — dynamic image tabs (one per product image, scrollable if >5):**
```
[●Mặt trước] [Mặt sau] [Tay trái] [Tay phải] [Quần trước] [Quần sau]
```
Active tab: yellow underline 3px + bold label. Inactive: gray.

**Below tabs — ColorSet Selector row:**
"Chọn màu sắc:" + color option cards (each 80×80, rounded-lg, border):
- Thumbnail image of the product in that colorway
- Name label: "Xanh Navy", "Đỏ Trắng", "Vàng Đen"
- Active state: yellow border 3px + checkmark badge top-right

### Section 2.3 — RIGHT COLUMN: Design Controls (scrollable, 50% width)

#### Block A — Team Name Input
- Label: "Tên đội (tuỳ chọn)"
- Input: full-width, 48px tall, placeholder "VD: FC Sao Vàng"
- Helper text: "Nếu không nhập, sẽ dùng tên sản phẩm."

#### Block B — Logo Library Panel (most important block)

Header: "Thư viện Logo" + count badge "3/10"
Dropzone/upload button full-width:
```
┌─────────────────────────────────────┐
│       📁 Kéo thả logo vào đây       │
│       hoặc bấm để chọn file          │
│  PNG, JPG, SVG · tối đa 5MB mỗi file │
└─────────────────────────────────────┘
```
Yellow dashed border, 120px tall, center-aligned, icon + text.

Uploaded logos grid (2 cols, 8px gap, each 96×96):
```
┌──────┐┌──────┐
│ [🛡️] ││ [🦁] │
│ logo1││ logo2│
│  [×] ││  [×] │
└──────┘└──────┘
[+ Thêm]
```
Each item: white bg, rounded-lg, 1px border, logo thumbnail centered, filename truncated below 11px, hover shows delete X + "drag vào canvas" tooltip.

**Quality warning badge on low-res logos:** yellow `⚠ Chất lượng thấp` pill top-right

#### Block C — Player List (collapsible, expanded by default)

Header: "Danh sách cầu thủ" + count badge "12 cầu thủ" + collapse arrow

Action row:
- `[+ Thêm cầu thủ]` yellow outlined button
- `[📋 Dán từ Excel]` gray ghost button

Table (dense, 40px row height):
```
┌───┬──────────────────┬────────┬──────┬──┐
│STT│ Tên cầu thủ      │ Số áo  │ Size │  │
├───┼──────────────────┼────────┼──────┼──┤
│ 1 │ [Nguyễn Văn A]   │ [7]    │ [M▾] │✕ │
│ 2 │ [Trần Văn B]     │ [10]   │ [L▾] │✕ │
│ 3 │ [Lê Văn C]       │ [23]   │ [XL▾]│✕ │
└───┴──────────────────┴────────┴──────┴──┘
```
Inputs inline editable, size dropdown shows "M (còn 8)" / "XXL (Hết hàng)" disabled state.

#### Block D — Print Fee Nudge (card with dynamic color)

Current state (12 players → free tier):
- Green bg card, checkmark icon
- "✅ Miễn phí in! Đơn 10+ bộ không mất phí."

Alternative state (4 players):
- Yellow tinted bg
- "💡 Thêm 2 bộ để giảm phí in xuống 10%"
- Progress bar showing tier

#### Block E — Price Summary Card (sticky bottom-right)

```
Đơn giá:                  250.000₫
× 12 bộ:                3.000.000₫
Phí in (Miễn phí):              0₫
─────────────────────────────────
Tổng cộng:              3.000.000₫

[  THÊM VÀO GIỎ HÀNG  ]  ← 56px yellow button full-width
```

### Section 2.4 — Below the fold: Product Info Tabs

Reuse shadcn Tabs component:
- Tabs: [Mô tả] [Chất liệu] [Chính sách đổi trả] [Đánh giá (127)]
- Content area per tab below

### Section 2.5 — Related Products (4-card horizontal scroll)

Similar to homepage featured products section but 4 items wide.

### Section 2.6 — Footer (reuse from homepage)

---

## ⚙️ Design Tokens to Output (if tool supports)

Export as Figma variables / design tokens JSON:
- Color primitives (all hex values above)
- Spacing scale: 4, 8, 12, 16, 24, 32, 48, 64, 96, 128 (px)
- Radius scale: 4, 8, 12, 16, 24 (px)
- Shadow scale: sm, md, lg, xl, 2xl (standard Tailwind)
- Type scale (as above)

## 📱 Responsive notes (if tool generates responsive variants)

- **Desktop (≥1024px):** as designed above
- **Tablet (768-1023px):** Builder right column stacks below canvas; homepage grids become 2-col
- **Mobile (<768px):** Builder shows notice "Vui lòng dùng desktop để thiết kế" with illustration + product gallery continues normally; homepage single-column

## ✨ Quality bar (MUST hit)

1. **Photography over illustrations** — hero, product cards, testimonials all use realistic (or realistic-looking) football team photos
2. **Type hierarchy razor-sharp** — every heading, body, caption clearly differentiated
3. **CTAs unmistakable** — yellow primary buttons always highest contrast, never mix tones
4. **Vietnamese diacritics rendered correctly** — no broken characters (ọ, ệ, ờ, ấ, ỗ)
5. **Empty states thoughtful** — builder right column has placeholder state before logos uploaded
6. **Loading/skeleton not needed** — generate the fully populated state
7. **No Lorem Ipsum** — all copy in Vietnamese as shown

## 🚫 Do NOT

- Do not use generic stock icons — use football-specific where possible (jersey, shield, whistle, ball, cleats, trophy)
- Do not use light orange instead of yellow #FDD017 — this is the brand gold, must match exactly
- Do not use pink/purple — off-brand
- Do not center-align body paragraphs — only headers
- Do not overuse gradients — apply only to hero overlay + CTA banner
- Do not design the mobile builder — show the "desktop-only" notice instead

---

## 📦 Output Deliverable

Produce **one Figma file** with:
- Page 1: "01 Homepage" (full scroll, sections stacked)
- Page 2: "02 Custom Builder" (product detail customizable, full scroll)
- Page 3: "Design System" — color swatches, type samples, button states, card variants, form inputs
- All text layers in Vietnamese
- All components properly named (`Button/Primary`, `Card/Product`, etc.)
- Auto-layout frames throughout
- Export-ready at 1x, 2x, 3x

---

**End of Prompt.**

---

## 📖 Notes for User

1. **Copy from "The Prompt" section** down to "End of Prompt." and paste into Stitch / AI designer
2. Stitch may not honor every spec literally — iterate by feeding back specific blocks that miss
3. After generation, use `/ck:ui-ux-pro-max` skill + shadcn MCP to audit & refine code implementation
4. For frontend coding: `/ck:frontend-design` to map Figma → React+Tailwind, `/ck:ui-styling` for shadcn component wiring
5. For UX review: `/ck:web-design-guidelines` to check accessibility + heuristics
