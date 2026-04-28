# AI Design Generation Prompt — StarSport Football Uniform Store

**Use:** Paste this prompt vào Stitch / v0.dev / Lovable / Bolt.new / Claude / Cursor để gen UI cho toàn bộ màn hình dự án.

**Source skills:** `ck:ui-ux-pro-max` (50 styles, product→style mapping) + `ck:frontend-design` (anti-AI-slop) + `ck:ui-styling` (shadcn/ui + Tailwind) + `ck:frontend-development` (React patterns).

---

## ⬇️ COPY FROM HERE ⬇️

```
You are a senior UI/UX designer + frontend engineer. Generate production-grade screens for a Vietnamese football uniform e-commerce platform with custom mockup builder.

═══════════════════════════════════════════════════════════════
PROJECT: StarSport — Custom Football Uniform Platform
═══════════════════════════════════════════════════════════════

WHAT IT IS
A Vietnamese e-commerce site selling football team uniforms with a unique custom mockup builder. Customers visually design their team kit (drag logos onto real product photos, paste player roster from Excel, see live mockup) before ordering. Solves Zalo/Facebook ordering miscommunication where shop misinterprets verbal designs.

TARGET USERS
- Vietnamese football team captains, coaches, club owners (age 22-45)
- Order quantity: 6-30 sets per team
- Tech comfort: medium (uses Excel daily, comfortable with online shopping)
- Buying motivation: deliver right design first time, no Zalo back-and-forth

LANGUAGE: Tiếng Việt thuần (no English UI), no i18n.

═══════════════════════════════════════════════════════════════
BRAND IDENTITY (LOCKED — do not change)
═══════════════════════════════════════════════════════════════

Colors:
- Primary  Yellow #FDD017 (StarSport vàng — energetic, optimistic)
- Secondary Red    #E31E26 (Vietnamese football passion)
- Accent   Blue   #00AEEF (trust, sport sky)
- Neutral  Grey   #A7A9AC (supporting)
- Background White #FFFFFF / Off-white #FAFAFA
- Text     Near-black #0F172A / Muted #64748B
- Success  Green  #22C55E (order confirmed, in stock)
- Warning  Amber  #F59E0B (low quality logo, low stock)
- Danger   Red    #DC2626 (cancelled, error — distinct from brand red via tone)

Typography:
- Display & Body: Montserrat (300/400/500/600/700/800/900)
- Use 800-900 for hero headlines, 700 for section titles, 500 for body, 400 for meta
- DO NOT introduce Inter, Roboto, Arial, Open Sans (banned — looks AI-generic)

Aesthetic direction: **Vibrant & Block-based + Motion-Driven** (per UI/UX best practice for sports e-commerce). NOT minimalism (too cold), NOT glassmorphism (overdone), NOT brutalism (kills trust).

Visual cues:
- Bold geometric blocks + diagonal slashes (jersey stripe inspired)
- Generous yellow blocks for hero/CTA, red accents for urgency
- Photography-first: real product photos dominate (no illustrated jerseys)
- Hover lift + 200-300ms ease-out transitions
- Section dividers: angled cuts (skewY -3deg) referencing sports banners

═══════════════════════════════════════════════════════════════
TECH STACK CONSTRAINTS
═══════════════════════════════════════════════════════════════

- Next.js 15 App Router + React Server Components
- TypeScript strict
- Tailwind CSS (utility-first, mobile-first breakpoints sm/md/lg/xl)
- shadcn/ui components (Radix UI primitives — accessible by default)
- Lucide icons (NO emojis, NO Font Awesome)
- react-moveable for drag/resize/rotate in builder canvas
- Zustand for cart state, react-hook-form for forms
- Images via next/image with Cloudinary loader

Component output: TSX files compatible with shadcn/ui structure (`@/components/ui/*` for primitives, `@/components/<feature>/*` for composition).

═══════════════════════════════════════════════════════════════
UX GUIDELINES (CRITICAL — non-negotiable)
═══════════════════════════════════════════════════════════════

Accessibility (WCAG AA minimum):
- Color contrast 4.5:1 for body text, 3:1 for large text
- All interactive elements have visible focus rings (2px primary color, 2px offset)
- Touch targets ≥ 44×44px on mobile
- aria-label for icon-only buttons (e.g., cart icon, hamburger)
- Form inputs paired with <label> via htmlFor
- Skip-to-content link in header for keyboard users

Performance:
- Skeleton loaders for async content (not spinners)
- Reserve image space (aspect-ratio) — no layout jump
- Lazy-load below-fold images
- prefers-reduced-motion: disable parallax, keep instant transitions

Responsive:
- Mobile-first (375px up). Builder is desktop-only ≥768px (show notice on mobile).
- Body text ≥ 16px on mobile (prevent iOS zoom)
- Cart/checkout/listing fully responsive

Interaction:
- Loading buttons disable + show spinner inside button (no global overlay for non-blocking actions)
- Error feedback inline near the offending field, not toasts (toasts for success only)
- Empty states have illustration + clear CTA (e.g., empty cart → "Khám phá sản phẩm")

═══════════════════════════════════════════════════════════════
SCREENS TO GENERATE (priority order)
═══════════════════════════════════════════════════════════════

### Group A — Public marketing & catalog (HIGH priority)

1. **Homepage `/`**
   - Sticky header: logo, nav (Trang chủ, Sản phẩm, Đồng phục, Liên hệ), search icon, cart icon (badge count), user menu (login/avatar)
   - Hero carousel: 2-3 slides, full-width image of football team in uniform, headline overlay (e.g., "Tự thiết kế đồng phục — Nhận hàng đúng ý"), CTA button "Thiết kế ngay" + secondary "Xem sản phẩm"
   - Feature strip: 4 columns (icon + title + 1-line desc) — "Tự thiết kế mockup", "In chất lượng cao", "Giao toàn quốc", "Đơn từ 6 bộ"
   - Featured products: 8 product cards in 4-col grid (mobile: 2-col)
   - Testimonials carousel: 3-5 cards with avatar + name + Vietnamese quote + 5-star
   - CTA banner: yellow background, diagonal slash, headline "Sẵn sàng cho mùa giải mới?" + button
   - Footer: 4 columns (About, Sản phẩm, Hỗ trợ, Liên hệ) + social icons + payment methods (COD, VNPay icons grayed for "sắp có")

2. **Product Listing `/products`**
   - Breadcrumb: Trang chủ / Sản phẩm
   - Hero strip with category title + product count
   - Sidebar (desktop) / Drawer (mobile): filters — category checkboxes, price range slider, "Có thể tùy chỉnh" toggle
   - Top bar: search input, sort dropdown (mới nhất, giá tăng/giảm, nổi bật), result count
   - Product grid: 3-col desktop / 2-col tablet / 1-col mobile with infinite scroll or "Xem thêm" button
   - Product card: image (16:10), badge (top-left "Đồng phục tùy chỉnh" yellow chip if customizable), name, price, color swatches (max 4 chips + "+N" if more), hover → quick-view button slides up
   - Empty state if no results: illustration + "Không tìm thấy sản phẩm" + reset filters button

3. **Product Detail (Normal) `/products/[slug]` — `customizable=false`**
   - Breadcrumb + back button
   - Two-column desktop / stacked mobile:
     - Left: **Image gallery (color-aware)**
       • Main image: large, lazy-loaded, aspect-ratio fixed (4:5 or product's natural), no layout shift
       • Thumbnail grid below: 4-6 thumbs of CURRENTLY SELECTED ColorSet's images (sorted by `ProductImage.sortOrder`)
       • Click thumbnail → swap main image (smooth 200ms cross-fade)
       • Zoom on hover desktop (right-side magnifier or modal lightbox), swipe mobile
       • **Initial state on page load**: show first ColorSet (lowest sortOrder or `colorSets[0]`), main image = that ColorSet's `images[0]`
     - Right: product name (h1, 2xl-3xl), star rating + review count, price (3xl bold)
       • **ColorSet picker (CRITICAL — drives gallery)**: row of image swatches (each shows ColorSet's primary image thumbnail + name label below). Active state: primary border #FDD017 + checkmark icon. Click handler: `setActiveColorSet(id)` → triggers gallery re-render with new ColorSet's images, resets thumbnail selection to first
       • Size picker (chips S/M/L/XL/2XL — show stock per ACTIVE ColorSet × size; disabled with strikethrough + tooltip "Hết hàng" if stock=0)
       • Quantity stepper, "Thêm vào giỏ" button (full-width, primary yellow with red hover), wishlist heart icon
   - Below: tabs (Mô tả | Thông số | Đánh giá)
   - Related products carousel

4. **Product Detail (Customizable) `/products/[slug]` — `customizable=true`** ⭐ HERO SCREEN
   - Same top section as #3 (color-aware gallery + ColorSet picker + price + product name)
   - **CRITICAL — ColorSet drives BOTH gallery AND builder**: single source of truth `activeColorSetId`. Khi user click ColorSet:
     • Gallery thumbnail grid → re-render với ColorSet mới's images
     • Builder Section 3 (Mockup Canvas) tabs → re-render với ColorSet mới's images, **giữ nguyên `overlays[][]` positions** (coords là %, scale tự động)
     • Tab thừa nếu ColorSet mới ít ảnh hơn → ẩn (data preserved trong state, không xóa)
     • Default load: ColorSet đầu tiên (`colorSets[0]`), tab đầu tiên (`images[0]`)
   - BUT instead of size picker → "Bắt đầu thiết kế" intro card with 5 sections cuộn dọc:
     - Section 1: Color set selector (image grid) — same component as Screen 3 ColorSet picker, shared state
     - Section 2: Logo Library — drag-zone "+ Tải logo" button, grid of uploaded logos (thumbnail + filename + delete X), warning badge "⚠ Chất lượng thấp" if <300px
     - Section 3: Mockup Canvas (THE CORE) — split layout: left 25% logo panel (vertical list of LogoItem cards, draggable), right 75% canvas with dynamic tabs above (one tab per ProductImage in active ColorSet, scrollable horizontally if >5), canvas shows real product photo as background with semi-transparent grid overlay, dragged logos appear as bordered elements with corner resize handles + top rotate handle
       **CRITICAL — Coordinate accuracy** (logo MUST follow cursor exactly, không drift, không lệch tỉ lệ):
         • Canvas container: `position: relative`, `aspect-ratio` = ảnh sản phẩm's natural ratio (vd: `aspect-ratio: 4/5`), KHÔNG dùng fixed pixel size
         • Background image: `<img>` với `width: 100%; height: 100%; object-fit: contain` — ảnh fill container, KHÔNG distort
         • Overlay element: `position: absolute`, dùng `left/top` % (KHÔNG pixel) + `width/height` % (KHÔNG pixel) → tự động scale khi container resize
         • `transform-origin: center center` cho rotate (xoay quanh tâm logo, không phải corner)
         • react-moveable config: `<Moveable container={canvasRef.current} keepRatio={true} snappable={false} />` — `keepRatio` để resize không skew, `container` ràng buộc drag trong canvas
         • Coord conversion math (CRITICAL — sai 1 pixel sẽ drift):
           ```typescript
           const canvasRect = canvasRef.current.getBoundingClientRect()
           const xPercent = (mouseClientX - canvasRect.left) / canvasRect.width   // 0-1
           const yPercent = (mouseClientY - canvasRect.top) / canvasRect.height   // 0-1
           // Render: style={{ left: `${xPercent * 100}%`, top: `${yPercent * 100}%` }}
           ```
         • TUYỆT ĐỐI tránh: dùng `window.innerWidth` thay `canvasRect.width`, quên trừ `canvasRect.left/top`, mix `pixel + %` trong cùng element, scroll offset không tính
         • Test thủ công: drag logo từ panel vào góc trên-trái canvas → logo phải nằm sát góc 0,0; drag vào giữa → đúng giữa; resize browser → logo giữ vị trí tương đối
       **CRITICAL — Drag performance** (must implement, do NOT skip):
         • Use `transform: translate3d(x, y, 0)` for GPU acceleration (not `top`/`left`)
         • Add `will-change: transform` on draggable elements when active drag, remove when idle
         • During drag, mutate position via React **ref** (DOM directly), commit to React state ONLY on `onDragEnd` — NEVER setState on every `onDrag` mousemove (causes re-render lag)
         • Wrap `<Moveable>` and target elements in `React.memo`, pass stable callbacks via `useCallback`
         • If still laggy: use `requestAnimationFrame` to throttle position updates
         • Test drag at 60fps in Chrome DevTools Performance panel — must show no scripting bottlenecks > 16ms
       **Rotate snap** (mandatory): use react-moveable `throttleRotate={15}` + custom snap logic — when within ±5° of 0/90/180/270/360, **lock to exact angle** (visual: rotation handle "clicks", brief highlight border in primary color #FDD017). Show angle indicator badge during rotate (e.g., "90°"). Default rotation = 0°.
     - Section 4: Player roster table — columns STT/Tên cầu thủ/Số áo/Size, "+ Thêm cầu thủ" + "Dán từ Excel" buttons, real-time row count → updates Section 5
       **VALIDATION RULES** (must implement exactly, do NOT add minimum):
         • Default 1 row visible (NOT empty, NOT 6 rows pre-filled)
         • **NO minimum quantity** — user can submit with just 1 player. Print fee tiers below are DISCOUNT TIERS, NOT minimum requirements
         • `size` is REQUIRED (red asterisk, validation error if empty)
         • `playerName` is OPTIONAL — if empty → áo trơn không in tên (show muted placeholder "Để trống = không in tên")
         • `playerNumber` is OPTIONAL — if empty → áo trơn không in số (show muted placeholder "Để trống = không in số")
         • Submit button enabled when ≥1 row has size selected (regardless of name/number)
         • Do NOT block submit with messages like "Phải có ít nhất 6 cầu thủ" — wrong, not in spec
     - Section 5: Print fee nudge — 4 trạng thái dựa trên `hasPrintingWork(config)` + `players.length`:
       **State A — Không có in** (KHÔNG có overlay AND KHÔNG có playerName/playerNumber AND KHÔNG có teamName):
         Card neutral xám nhạt: "Không có in — phí in: 0₫. Bạn đang đặt áo trơn." Phí in = 0 BẤT KỂ số bộ.
       **State B — Có in, < 6 bộ** → Card amber: "Phí in +20% (XX,XXX₫). Thêm N bộ để giảm còn 10%"
       **State C — Có in, 6-9 bộ** → Card amber nhẹ: "Phí in +10%. Thêm N bộ để miễn phí in"
       **State D — Có in, ≥10 bộ** → Card green: "Miễn phí in 🎉" (no nudge)
       
       **CRITICAL — đừng tính phí in nếu khách không in gì** (Session 12 rule):
       - "Có in" trigger: ≥1 overlay logo/text trên canvas, HOẶC ≥1 player có name/number, HOẶC có teamName
       - Real-time recompute khi user xóa logo / xóa text / xóa số áo của tất cả players → quay về State A
       - Microcopy emphasize OPPORTUNITY ("Thêm N bộ để miễn phí in") không REQUIREMENT ("Phải có ít nhất 6 cầu thủ" ❌)
   - Mobile: replace entire builder with full-screen notice "Vui lòng dùng máy tính để thiết kế đồng phục" + illustration + button "Xem sản phẩm khác"
   - Sticky bottom bar (desktop only): total price calculation + "Thêm vào giỏ" CTA

### Group B — Cart, Checkout, Order (HIGH priority)

5. **Cart `/cart`**
   - Title "Giỏ hàng (N sản phẩm)"
   - Two-column desktop: items list left, summary panel right (sticky)
   - Cart item row (normal): thumbnail, name, color/size meta, quantity stepper, unit price, subtotal, [Xóa] icon
   - Cart item row (custom team uniform): thumbnail (mockup canvas snapshot), team name + product name, "N cầu thủ — N bộ", color set, expand-to-see player list (collapsible accordion), print fee breakdown sub-row, [Xóa] only (no edit per Session 7)
   - Print fee nudge per custom item (e.g., "Thêm 4 bộ để giảm phí in 10% → miễn phí in")
   - Empty state: illustration + "Giỏ hàng trống" + browse button
   - Summary panel: subtotal, shipping (sẽ tính ở bước sau), print fee per team, total bold, "Tiến hành thanh toán" CTA full-width

6. **Checkout `/checkout`** — requires auth
   - Two-column desktop: form left, order summary right (sticky)
   - Form fields: tên người nhận, SĐT, địa chỉ đầy đủ (textarea — single field, no province dropdown for MVP), ghi chú đơn hàng (optional)
   - Auto-fill from User.shipping* if exists (with "Sửa" link)
   - Payment section: COD radio (only option, with explanation "Thanh toán khi nhận hàng"); VNPay/MoMo grayed with "Sắp ra mắt"
   - "Đặt hàng" CTA button at bottom
   - Order summary right panel: collapsible item list, subtotal/shipping/print fee/total, secure badge

7. **Order Confirmation `/orders/[id]/success`**
   - Centered hero: green checkmark animation, "Đặt hàng thành công!"
   - Order number large + copy-to-clipboard button
   - Summary: total, payment method (COD), shipping ETA estimate
   - Player size summary if any custom orders (e.g., "M×2, L×4, XL×1")
   - CTAs: "Xem đơn hàng" + "Tiếp tục mua sắm"
   - Below: "Email xác nhận đã gửi đến shop, shop sẽ liên hệ bạn trong 24h"

8. **Order History `/orders`**
   - Title + filter chips (Tất cả | Đang xử lý | Đang giao | Đã giao | Đã hủy)
   - Order list: card per order — order number, date, status badge (color-coded), item count, total, "Xem chi tiết" button + "Hủy đơn" button (only if pending)
   - Empty state if no orders

9. **Order Detail `/orders/[id]`**
   - Order number + status timeline (4-step horizontal: Đặt hàng → Đang xử lý → Đang giao → Đã giao)
   - Two-column: items list left, summary + shipping info right
   - For custom orders: expand to show player list + mockup preview thumbnail (read-only, links to full preview)
   - Cancel button if status=pending (with confirmation dialog)

### Group C — Auth (MEDIUM priority)

10. **Login `/login`**
    - Centered card on subtle yellow tinted background, max-width 440px
    - Logo on top, title "Đăng nhập"
    - Email + password fields with validation, "Quên mật khẩu?" link
    - "Đăng nhập" button full-width primary
    - Divider "hoặc" + link "Chưa có tài khoản? Đăng ký"

11. **Register `/register`** — same shell as login, fields: tên, email, password, confirm password
12. **Forgot Password `/forgot-password`** — email field, send reset link, success state confirms email sent
13. **Reset Password `/reset-password?token=...`** — new password + confirm + submit

### Group D — User profile (MEDIUM priority)

14. **Profile `/profile`**
    - Tabs: Thông tin / Địa chỉ / Đổi mật khẩu
    - Forms with current values pre-filled, save button per section

### Group E — Admin (LOWER priority but important)

15. **Admin Dashboard `/admin`** (admin-only)
    - Sidebar navigation: Dashboard, Đơn hàng, Sản phẩm, Người dùng, Cài đặt
    - Top bar: search, notifications, admin avatar
    - Stat cards: doanh thu hôm nay, đơn mới, đơn xử lý, sản phẩm bán chạy
    - Charts: doanh thu 7 ngày (line), top products (horizontal bar)
    - Recent orders table (5 rows)

16. **Admin Order List `/admin/orders`**
    - Filter bar: status tabs, date range, search by order# or phone
    - Data table: order# | khách hàng | total | status (dropdown to update) | actions
    - Bulk select for status batch update
    - Pagination

17. **Admin Order Detail `/admin/orders/[id]`**
    - Full order info, customer info, shipping address
    - Items list — for custom orders show **MockupPreview component** (read-only canvas with overlays, dynamic tabs to switch image slots) + player roster table + downloadable Excel link + downloadable logo files
    - Status update dropdown (triggers customer email on shipping/delivered)
    - Admin note textarea
    - Print invoice button

18. **Admin CSV Import Wizard `/admin/products/import`** (5-step wizard)
    - Step 1: Upload CSV — dropzone, file preview (first 5 rows)
    - Step 2: Column mapping — left list of CSV columns, right list of DB fields, drag-to-map or dropdown, auto-suggest matches, unmapped columns shown grayed
    - Step 3: Preview grouped by product name — cards per product, each with editable fields for unmapped columns, validation errors highlighted in red rows with inline fix
    - Step 4: Image upload per ColorSet — for each product card, expand to ColorSets, dropzone per ColorSet (multiple images, reorder by drag, set sortOrder + label per image)
    - Step 5: Stock matrix — table grid (rows=ColorSets × sizes, cols=jersey/shorts), inline number inputs, "Đặt 0 cho tất cả" / "Sao chép từ ColorSet trên" helpers
    - Footer: Back/Next/Cancel/Confirm buttons, progress indicator at top (5 dots)

19. **Admin Product Edit `/admin/products/[id]/edit`**
    - Tabs: Thông tin chung | Ảnh | Bộ màu | Kho
    - Tab 1: name, slug, description (rich text), category, basePrice, customizable toggle, active toggle
    - Tab 2: image manager — drag-reorder grid, add/delete, label + sortOrder per image, ColorSet assignment dropdown per image
    - Tab 3: ColorSet list — add/edit/delete, color chips picker (primary/secondary hex), images assignment
    - Tab 4: stock per ColorSet × size × type — same matrix as import wizard step 5

═══════════════════════════════════════════════════════════════
SHARED COMPONENTS
═══════════════════════════════════════════════════════════════

- `<Header>` sticky, transparent on hero scroll → solid white after 80px
- `<Footer>` 4-column dark variant with yellow accent border-top
- `<ProductCard>` reusable across listing/featured/related
- `<MockupCanvas>` interactive (Phase 4) + read-only mode (Phase 5 admin)
- `<PrintFeeNudge>` reused in builder + cart
- `<EmptyState>` with illustration prop
- `<StatusBadge>` color-mapped (pending=blue, processing=amber, shipping=accent, delivered=green, cancelled=grey)

═══════════════════════════════════════════════════════════════
DELIVERABLES (per screen)
═══════════════════════════════════════════════════════════════

For EACH screen above, produce:

1. **Wireframe sketch** (ASCII or simple description) showing layout regions
2. **High-fidelity TSX component code** using:
   - Tailwind utility classes (no inline styles unless dynamic)
   - shadcn/ui primitives where applicable (Button, Card, Dialog, Input, Select, Tabs, etc.)
   - Lucide-react icons
   - next/image for images with width/height + alt text
   - Proper semantic HTML (header/main/section/article/aside/footer/nav)
3. **Mobile + desktop responsive variants** (use sm:/md:/lg: breakpoints, mobile-first)
4. **Empty / loading / error states** for data-driven sections
5. **Microcopy in Vietnamese** — natural, friendly tone, no Google-translate stiffness

═══════════════════════════════════════════════════════════════
OUTPUT FORMAT
═══════════════════════════════════════════════════════════════

For each screen:
```
## [Screen N] — [Screen Name] (`/route`)

### Wireframe
[ASCII or description of layout regions]

### Key UX decisions
- [3-5 bullet points explaining choices specific to this screen]

### Component code
```tsx
[Full TSX code, ready to drop into apps/web/src/app/...]
```

### States
- Loading: [describe skeleton]
- Empty: [describe empty state]
- Error: [describe error state]
```

═══════════════════════════════════════════════════════════════
ANTI-PATTERNS — DO NOT DO
═══════════════════════════════════════════════════════════════

❌ Generic Inter/Roboto fonts (USE Montserrat)
❌ Purple→pink gradients (NOT brand-aligned)
❌ Glassmorphism backdrop-blur everywhere (overdone)
❌ Emoji icons in production UI (use Lucide SVG)
❌ Center-everything layouts (boring, not sport energy)
❌ AI-illustrated mascots / cartoon footballs (use real photos)
❌ Spinner in the middle of empty area (use skeleton)
❌ "Lorem ipsum" — write real Vietnamese microcopy
❌ Inline english fragments in Vietnamese UI ("Click here", "Submit")
❌ Generic stock photos of multi-ethnic team in clean studio (look authentic — Vietnamese team on local field)
❌ Light grey on white text (fails contrast)
❌ Hover-only tooltips for critical info (mobile users miss them)

═══════════════════════════════════════════════════════════════
START
═══════════════════════════════════════════════════════════════

Begin with Screen 1 (Homepage). After producing it, ask me to confirm or request adjustments before proceeding to Screen 2. Work screen-by-screen unless I tell you to batch.
```

## ⬆️ COPY TO HERE ⬆️

---

## Hướng dẫn sử dụng

### Option A — Chạy trên v0.dev (recommended cho Next.js + shadcn)
1. Vào v0.dev → New chat
2. Paste toàn bộ prompt trên
3. v0 sẽ gen Screen 1 → review → tiếp tục Screen 2...
4. Mỗi screen v0 cho preview live + code TSX → copy thẳng vào `apps/web/src/app/...`

### Option B — Chạy trên Lovable / Bolt.new
1. Tạo project Next.js mới (chọn template Next.js + shadcn)
2. Paste prompt vào first message
3. Tool sẽ scaffold project + gen UI từng screen

### Option C — Chạy trên Stitch (Google AI design)
1. Bắt đầu project mới
2. Paste prompt → Stitch sinh Figma frames
3. Export sang Figma → dev manually convert sang TSX

### Option D — Chạy trực tiếp trên Claude Code
```
/ck:frontend-design implement [paste prompt above]
```
Claude sẽ chạy `frontend-design` workflow + activate `ui-ux-pro-max` skill, gen code thẳng vào project.

---

## Tip để output chất lượng

**Trước khi paste prompt:**
- Chuẩn bị 3-5 ảnh tham khảo: ảnh đội bóng VN trong đồng phục, ảnh shop bóng đá VN (Đông Á, Akira, Wika), ảnh logo CLB nổi tiếng — feed kèm prompt nếu tool support image input.
- Chia screen thành nhóm nhỏ 3-4 màn nếu tool có context limit (paste section "Group A" trước, sau làm Group B...).

**Sau khi gen xong:**
- Lưu output mỗi screen vào `apps/web/src/app/<route>/page.tsx`
- Verify: chạy `pnpm dev` → check responsive ở DevTools (375 / 768 / 1280) → check Lighthouse a11y score ≥ 90
- Nếu screen không match brand, gửi feedback cụ thể: "Screen 1 hero quá pastel, làm bold yellow #FDD017 hơn, chữ headline 900 weight"

---

## Unresolved (nếu cần refine prompt)

1. Có muốn thêm dark mode variant không? (Hiện chỉ light mode — sport e-commerce thường light first)
2. Animation level: subtle (chỉ hover + page transition) hay rich (parallax + scroll-triggered + GSAP)? Hiện prompt mức "Motion-Driven moderate".
3. Có asset thực tế (logo StarSport, ảnh sản phẩm sample, ảnh đội thực) chưa? Nếu có, đính kèm sẽ giúp AI gen sát brand hơn.
