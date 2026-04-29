# Prompts — Generate UI for StarSport (v0 / Lovable)

> Self-contained English prompts for **v0.dev** or **Lovable**. Each prompt embeds all business rules, brand tokens, data types, and design constraints — the AI cannot read repo files, so context is injected inline.
>
> **How to use:**
> 1. Send the **MASTER PROMPT** first to lock the design system + tech stack.
> 2. Then send each **SCREEN PROMPT** (1–4) one at a time, in order. Reference previously generated components by name when needed.
> 3. Screen 3 (Customise Builder) is the heaviest — give v0 the full prompt without trimming.

---

## 🟡 MASTER PROMPT (send first — establishes design system)

```
You are a senior frontend engineer building "StarSport" — a Vietnamese e-commerce platform for custom football team uniforms. Customers design their own jersey mockup (logo + player roster + numbers) on real product photos, then place a bulk order (1+ jerseys per team).

Generate production-quality React + TypeScript code using Next.js 15 App Router, Tailwind CSS, and shadcn/ui. All visible copy must be in Vietnamese (no i18n setup — hardcoded strings are fine).

═══════════════════════════════════════════════════════════
TECH STACK (locked — do not substitute)
═══════════════════════════════════════════════════════════
- Framework:        Next.js 15 App Router (RSC + client islands)
- UI library:       shadcn/ui + Tailwind CSS v3
- Icons:            lucide-react
- Font:             Montserrat (Vietnamese subset) — NOT Inter
- Forms:            react-hook-form + zod
- Cart state:       Zustand + persist middleware (localStorage only — no DB sync)
- Drag/resize/rotate: react-moveable (NOT react-draggable, NOT Fabric.js, NOT Konva)
- Image:            next/image
- Sanitize:         isomorphic-dompurify (for SVG XSS prevention)
- Toast:            sonner

═══════════════════════════════════════════════════════════
BRAND TOKENS (hex values are fixed)
═══════════════════════════════════════════════════════════
- Primary  (Yellow): #FDD017  → main CTAs, hero accent, badges
- Secondary (Red):   #E31E26  → sale, danger, out-of-stock
- Accent (Blue):     #00AEEF  → links, focus rings, info
- Neutral (Gray):    #A7A9AC  → disabled, dividers, helper text
- Background:        #FFFFFF / #F8F9FB
- Text default:      #0F0F10

Add these as Tailwind theme extensions:
  colors: {
    brand: { yellow:'#FDD017', red:'#E31E26', blue:'#00AEEF', gray:'#A7A9AC' }
  }
  fontFamily: { sans: ['Montserrat', 'system-ui', 'sans-serif'] }

Typography rules:
- Headlines: tracking-tight, font-weight 700–800
- Body: line-height 1.6, font-weight 400–500
- ALL CAPS labels (badges, eyebrow text): font-weight 600, letter-spacing 0.05em

═══════════════════════════════════════════════════════════
DESIGN MOOD
═══════════════════════════════════════════════════════════
Inspiration: Nike, Adidas, Puma e-commerce sites. Sporty + energetic + professional. High-contrast hero imagery, generous whitespace, large product photos, subtle micro-interactions.

Component patterns:
- Card:   rounded-xl, shadow-sm, hover:shadow-lg + hover:-translate-y-0.5 transition
- Button: primary = bg-brand-yellow text-black font-semibold; secondary = outline border-brand-red text-brand-red; ghost = text-gray-700
- Badge:  pill, text-xs, uppercase, font-semibold (e.g. "ĐỒNG PHỤC TÙY CHỈNH", "HẾT HÀNG")
- Input:  shadcn defaults, focus ring brand-blue
- Skeleton loaders for every async section
- Empty state:  illustration (use a lucide icon at scale) + headline + CTA

Responsive breakpoints (Tailwind defaults):
- Mobile-first
- md (768px) is the pivot — the custom builder is DESKTOP-ONLY below 768px (show notice instead)
- Catalog / cart / checkout are fully responsive on all sizes

═══════════════════════════════════════════════════════════
DOMAIN DATA TYPES (use exactly these — copy into src/types/)
═══════════════════════════════════════════════════════════
type ProductCategory = 'jersey-club' | 'jersey-national' | 'shoes' | 'accessory';

interface ProductImage {
  id: number;
  url: string;
  sortOrder: number;
  label?: string;        // tab label in builder, e.g. "Mặt trước"
  colorSetId: number;
}

interface ColorSet {
  id: number;
  name: string;          // e.g. "Đỏ-Trắng"
  swatchHex: string;     // for color chip preview
  images: ProductImage[];
}

interface ProductVariant {
  id: number;
  colorSetId: number;
  size: 'S' | 'M' | 'L' | 'XL' | 'XXL';
  stock: number;         // display only — never decremented at order time
}

interface Product {
  id: number;
  slug: string;
  name: string;
  description: string;
  category: ProductCategory;
  basePrice: number;     // VND
  isCustomizable: boolean;
  colorSets: ColorSet[];
  variants: ProductVariant[];
  active: boolean;
  createdAt: string;
}

// ── Custom builder types ──
interface LogoItem {
  id: string;
  url: string;
  filename: string;
  qualityNote?: string;  // auto-set when width<300 OR height<300
}

interface OverlayElement {
  id: string;
  type: 'logo' | 'text';
  logoId?: string;       // when type='logo'
  text?: string;         // when type='text'
  x: number;             // 0..1 of canvas width (left edge)
  y: number;             // 0..1 of canvas height (top edge)
  width: number;         // 0..1 of canvas width
  height: number;        // 0..1 of canvas height
  rotation: number;      // degrees, snap to 0/90/180/270
  zIndex: number;
}

interface CustomConfig {
  teamName?: string;
  logos: LogoItem[];
  overlays: OverlayElement[][];   // indexed by image slot
  players: {
    sortOrder: number;
    playerName?: string;
    playerNumber?: string;
    size: string;                 // REQUIRED — applies to both jersey + shorts
  }[];
}

// ── Cart types ──
interface NormalCartItem {
  type: 'normal';
  id: string;
  productId: number;
  productName: string;
  productImage: string;
  colorSetName: string;
  size: string;
  quantity: number;
  unitPrice: number;
}

interface CustomCartItem {
  type: 'custom';
  id: string;
  productId: number;
  productName: string;
  colorSetId: number;
  colorSetName: string;
  productImages: string[];
  config: CustomConfig;
  unitPrice: number;
  printingSurcharge: number;
  totalPrice: number;
}

type CartItem = NormalCartItem | CustomCartItem;

═══════════════════════════════════════════════════════════
BUSINESS RULES (do not change)
═══════════════════════════════════════════════════════════
1. Stock is DISPLAY-ONLY. Never validate / decrement at checkout. Just disable size chips with tooltip "Hết hàng" when stock=0.
2. Cart is localStorage-only via Zustand persist. No DB sync, no merge endpoint.
3. Checkout requires login. Cart browsing is public.
4. Payment: COD only (Cash on Delivery). VNPay/MoMo are placeholders shown as "Sắp ra mắt" (disabled).
5. Order numbers format: ORD-YYMMDD-NNN (NNN resets daily).
6. Customer email: NO email when placing an order. Only on status changes to "Đang giao" / "Đã giao".
7. Custom builder is desktop-only (<768px shows MobileBuilderNotice).
8. Logo files: PNG/JPG/SVG only, max 5MB. SVG must be sanitized (strip <script>, on*, href="javascript:") before upload. Quality warning badge if width<300 OR height<300 — do NOT block upload.
9. Print fee logic (CRITICAL — depends on actual printing work, not just player count):

   function hasPrintingWork(c: CustomConfig): boolean {
     return c.overlays.some(slot => slot.length > 0)
         || c.players.some(p => p.playerName?.trim() || p.playerNumber?.trim())
         || !!c.teamName?.trim();
   }

   function calculatePrintFee(c: CustomConfig, subtotal: number) {
     if (!hasPrintingWork(c)) return { rate:0, amount:0, label:'Không có in', nudge:null };
     const n = c.players.length;
     if (n >= 10) return { rate:0,    amount:0,            label:'Miễn phí in', nudge:null };
     if (n >= 6)  return { rate:0.10, amount:subtotal*.10, label:'Phí in +10%', nudge:`Thêm ${10-n} bộ để miễn phí in` };
     return                { rate:0.20, amount:subtotal*.20, label:'Phí in +20%', nudge:`Thêm ${6-n} bộ để giảm còn 10%` };
   }

   Print fee is calculated PER custom order (not pooled across multiple custom orders in one cart).

10. Player roster: default 1 row (NOT 6 pre-fill, NOT empty). NO minimum quantity. `size` REQUIRED. `playerName` and `playerNumber` are OPTIONAL.

═══════════════════════════════════════════════════════════
CODE QUALITY RULES
═══════════════════════════════════════════════════════════
- TypeScript strict — no `any`, no `@ts-ignore`
- Files ≤ 200 LOC — split into modules if larger
- File naming: kebab-case (e.g. mockup-canvas.tsx)
- React.memo + useCallback on hot paths (canvas overlays, player list rows)
- No console.log in committed code
- Comments: only WHY when non-obvious. Never WHAT.
- A11y: focus ring visible (brand-blue), aria-labels on icon buttons, keyboard nav (Tab/Enter/Esc)
- Loading skeletons + empty states + error boundaries on every async section
- Toast feedback (sonner) for add-to-cart, upload success, errors

═══════════════════════════════════════════════════════════
ANTI-PATTERNS (do NOT do any of these)
═══════════════════════════════════════════════════════════
- ❌ Wizard / step-by-step builder (use scroll-through sections instead)
- ❌ Fabric.js / Konva / pixi.js (use react-moveable + CSS transforms only)
- ❌ Separate route /custom/[slug] (builder embeds INTO /products/[slug])
- ❌ Hardcoding 4 image angles (front/back/sleeve/shorts) — tabs are dynamic from ProductImage.sortOrder
- ❌ Mixing px and % for overlay coordinates (causes drift on resize)
- ❌ setState inside onDrag handler (causes lag — mutate DOM directly, commit state on onDragEnd)
- ❌ window.innerWidth for canvas sizing (use getBoundingClientRect on the canvas ref)
- ❌ position: fixed on overlays (breaks on scroll)
- ❌ transform-origin: top-left for rotation (use center center)
- ❌ Auto-decrement stock at checkout
- ❌ CartItem DB table or /api/cart/merge endpoint (localStorage only)
- ❌ Email customer when placing order (only on shipping/delivered status)
- ❌ Multi-language / i18n setup (Vietnamese hardcoded only)

═══════════════════════════════════════════════════════════
ACKNOWLEDGE
═══════════════════════════════════════════════════════════
Reply with: "StarSport design system locked. Send screen prompt 1." Then wait for the next prompt.
```

---

## 🟢 SCREEN 1 PROMPT — Homepage `/`

```
Generate the StarSport homepage at `app/page.tsx` (RSC) plus child components.

Sections in order:

1. <NavHeader /> (sticky, top)
   - Logo "StarSport" (text + flame/lightning lucide icon, brand-yellow)
   - Nav links: Trang chủ · Sản phẩm · Giới thiệu · Liên hệ
   - Right: <SearchPopover /> trigger, <CartIcon /> with item count badge (red dot), Auth: "Đăng nhập" button OR <UserDropdown /> with avatar
   - Mobile: hamburger drawer (shadcn Sheet)

2. <HeroCarousel /> (client island — uses shadcn Carousel + embla autoplay)
   - 3 slides, full-bleed images of football jerseys in action (use Unsplash photo URLs: https://images.unsplash.com/photo-1551958219-acbc608c6377, https://images.unsplash.com/photo-1517466787929-bc90951d0974, https://images.unsplash.com/photo-1577223625816-7546f13df25d)
   - Each slide: dark overlay (bg-black/40), centered content with eyebrow tag ("ĐỒNG PHỤC ĐỘI BÓNG"), big headline (text-5xl md:text-7xl, font-extrabold, drop-shadow), 1-line subhead, primary CTA "Thiết kế ngay →" linking to /products
   - Slide indicators (dots) bottom center, arrow buttons left/right (visible on hover)
   - Auto-advance every 6s, pause on hover

3. <FeaturesSection />
   - Grid of 4 cards (1 col mobile, 2 col tablet, 4 col desktop)
   - Icons: PenTool, Award, Truck, Users (from lucide-react)
   - Titles: "Tự thiết kế mockup", "In chất lượng cao", "Giao hàng toàn quốc", "Hỗ trợ đội nhỏ từ 6 bộ"
   - Each card: icon in brand-yellow circle (size-12), title (font-bold), 2-line description
   - Section padding: py-16 md:py-24

4. <FeaturedProductsSection /> (mock 8 products inline — see MOCK DATA below)
   - Section title "Sản phẩm nổi bật" + subtitle "Áo đấu chính hãng — Tùy chỉnh theo đội của bạn"
   - Grid: 4 cols desktop, 2 cols tablet, 1 col mobile
   - Each <ProductCard /> shows: image (4:5 aspect ratio, object-cover), badge "ĐỒNG PHỤC TÙY CHỈNH" (top-left, brand-yellow) when isCustomizable, name, color chips (max 4 visible + "+N"), price formatted as Vietnamese đồng (e.g. "450.000₫")
   - Hover: card lifts (translate-y -0.5, shadow-lg), image zoom (scale 1.05)
   - Click → navigate to /products/[slug]
   - Bottom CTA: "Xem tất cả sản phẩm →" link

5. <TestimonialsSection /> (5 hardcoded Vietnamese reviews)
   - Background: subtle brand-yellow tint (bg-brand-yellow/5)
   - Carousel of testimonial cards (shadcn Carousel, 1 visible mobile, 2 tablet, 3 desktop)
   - Each card: 5-star rating (lucide Star, brand-yellow filled), quote (italic), customer first name + team name
   - Sample reviews (use these):
     a) "Đặt 22 bộ cho FC Cường Phát, in nét, giao đúng hẹn. Sẽ quay lại!" — Anh Tuấn, FC Cường Phát
     b) "Trang custom dễ dùng, kéo logo thả là xong. Đội mình ưng ý lắm." — Minh Quân, Cường Quyết FC
     c) "Vải đẹp, form chuẩn, giá hợp lý. Recommend cho đội phong trào." — Đức Huy, Saigon Stars
     d) "Đợt giải khu phố, đặt 12 bộ, in tên tiếng Việt có dấu vẫn nét." — Hoàng Long, Phong Thủy FC
     e) "Shop tư vấn nhiệt tình, hỗ trợ chỉnh logo nét hơn miễn phí." — Trung Kiên, Bình Minh Team

6. <Footer />
   - 4 columns: About | Mua sắm | Hỗ trợ | Kết nối
   - Bottom strip: "© 2026 StarSport — Đồng phục bóng đá chuyên nghiệp"

═══ MOCK DATA (inline at top of page.tsx) ═══
const MOCK_PRODUCTS: Product[] = [
  { id:1, slug:'ao-dau-mu-23-24', name:'Áo đấu Manchester United 23/24', description:'...', category:'jersey-club', basePrice:450000, isCustomizable:true, active:true, createdAt:'2026-01-15', colorSets:[{id:1,name:'Đỏ-Trắng',swatchHex:'#DA291C',images:[{id:1,url:'https://images.unsplash.com/photo-1577223625816-7546f13df25d',sortOrder:0,label:'Mặt trước',colorSetId:1},{id:2,url:'https://images.unsplash.com/photo-1551958219-acbc608c6377',sortOrder:1,label:'Mặt sau',colorSetId:1}]}], variants:[{id:1,colorSetId:1,size:'S',stock:10},{id:2,colorSetId:1,size:'M',stock:15},{id:3,colorSetId:1,size:'L',stock:8},{id:4,colorSetId:1,size:'XL',stock:0}] },
  // generate 7 more similar entries — vary names (Real Madrid, Liverpool, Arsenal, Việt Nam, áo trọng tài, giày đinh, găng tay), categories, isCustomizable mix (5 customizable, 3 not), prices 200k-800k
];

═══ DELIVERABLES ═══
- app/page.tsx (RSC, composes sections)
- components/layout/nav-header.tsx (client — needs auth state)
- components/layout/footer.tsx
- components/home/hero-carousel.tsx (client)
- components/home/features-section.tsx
- components/home/featured-products-section.tsx
- components/home/testimonials-section.tsx (client — uses Carousel)
- components/product/product-card.tsx (reused on listing page)
- components/cart/cart-icon.tsx (client — reads Zustand)
- lib/format-vnd.ts (e.g. `formatVnd(450000) → "450.000₫"`)

Ensure mobile responsive on all sections. Use next/image with placeholder="blur" + blurDataURL for hero images.
```

---

## 🟢 SCREEN 2 PROMPT — Product Listing `/products`

```
Generate the StarSport product listing page at `app/products/page.tsx` with search + filters + grid.

Layout: 2-column desktop (sticky filters left 280px, grid right). Stack vertically on mobile (filters in shadcn Sheet drawer, triggered by "Bộ lọc" button).

═══ FEATURES ═══

1. URL params drive state (use Next.js useSearchParams):
   - ?q=...           → search by name (case-insensitive contains)
   - ?category=...    → ProductCategory filter
   - ?priceMin=...&priceMax=...
   - ?customizable=true|false
   - ?sort=newest|price-asc|price-desc
   AND-logic across all filters.

2. <FilterSidebar /> (sticky top-24):
   - Search input with magnifying-glass icon (debounce 300ms before updating URL)
   - Section "Danh mục" — checkbox list: Áo CLB, Áo đội tuyển, Giày, Phụ kiện
   - Section "Khoảng giá" — shadcn Slider, min 0 max 1,500,000 step 50,000, dual handle
   - Section "Loại sản phẩm" — radio: Tất cả / Đồng phục tùy chỉnh / Sản phẩm thường
   - "Xóa bộ lọc" ghost button at bottom (visible only when any filter active)

3. <ProductGrid />:
   - Header strip: "Hiển thị X sản phẩm" + sort dropdown (right) — Mới nhất / Giá tăng dần / Giá giảm dần
   - Grid: 4 cols desktop, 3 cols tablet, 2 cols mobile (xs: 1 col)
   - Reuse <ProductCard /> from Screen 1
   - Pagination at bottom (shadcn Pagination, 12 per page)

4. Empty state: when 0 results, show centered illustration (lucide PackageX icon size-16 brand-gray) + "Không tìm thấy sản phẩm phù hợp" headline + "Thử thay đổi bộ lọc hoặc xóa bộ lọc để xem tất cả" + "Xóa bộ lọc" button

5. Loading state: skeleton grid (12 cards, animate-pulse, same dimensions as ProductCard)

═══ MOCK DATA ═══
Reuse MOCK_PRODUCTS from Screen 1 but expand to 24 entries to test pagination. Mix categories evenly. Use varied Unsplash football jersey/shoe photo URLs.

═══ DELIVERABLES ═══
- app/products/page.tsx (client component — needs URL params + Zustand cart)
- components/products/filter-sidebar.tsx
- components/products/product-grid.tsx
- components/products/sort-dropdown.tsx
- components/products/empty-state.tsx
- lib/filter-products.ts (pure function: takes products[] + URLSearchParams → filtered+sorted+paginated array)

Filter logic must be pure + testable. Sort by createdAt for "newest". Stock per (ColorSet+size) shown in product detail (Screen 3), not on cards.
```

---

## 🟢 SCREEN 3 PROMPT — Product Detail `/products/[slug]` (DUAL MODE — most complex)

> ⚠️ This is the heaviest screen. Send the entire prompt below to v0/Lovable in one go. Do not trim.

```
Generate the StarSport product detail page at `app/products/[slug]/page.tsx`. This is a DUAL-MODE page that renders DIFFERENTLY based on `product.isCustomizable`. There is NO separate /custom/[slug] route.

═══════════════════════════════════════════════════════════
MODE A: isCustomizable = false (shoes, accessories — simple)
═══════════════════════════════════════════════════════════

Layout: 2-column desktop (gallery left, info right), stacked mobile.

Left column:
- <ProductGallery /> — main image (4:5 aspect, zoom on hover) + thumbnail grid below (4-5 thumbs, click to swap main, active thumb has brand-yellow ring)

Right column:
- Breadcrumb: Trang chủ / Sản phẩm / [category] / [name]
- Product name (text-3xl font-bold)
- Price (text-2xl brand-red, formatted VND)
- Description (prose paragraph)
- <ColorSetSelector /> — image swatches (size-16 rounded), active = brand-yellow ring + checkmark
- <SizeSelector /> — chip row (S/M/L/XL/XXL). Each chip shows size letter; if stock=0 → grayed out, line-through, tooltip "Hết hàng" on hover, NOT clickable
- Quantity stepper (-/+ buttons + input, min 1 max 99)
- Primary CTA button (full-width, h-12, brand-yellow): "Thêm vào giỏ hàng" — disabled until size selected. On click: add to Zustand cart, show sonner toast "Đã thêm vào giỏ hàng", do NOT navigate.
- Secondary CTA below: "Mua ngay" (outline brand-red) — adds to cart + redirects to /checkout
- Info accordion: Chi tiết sản phẩm / Hướng dẫn bảo quản / Chính sách đổi trả

═══════════════════════════════════════════════════════════
MODE B: isCustomizable = true — FULL CUSTOM BUILDER
═══════════════════════════════════════════════════════════

⚠️ DESKTOP-ONLY. On <768px viewport, render <MobileBuilderNotice /> ONLY:
   - Centered layout, py-24
   - Icon: Monitor (lucide), size-16 brand-blue
   - Headline: "Vui lòng dùng máy tính để thiết kế đồng phục"
   - Body: "Trình thiết kế mockup yêu cầu màn hình lớn để đảm bảo chính xác từng pixel."
   - Secondary: "Bạn vẫn có thể xem ảnh sản phẩm và bộ màu bên dưới ↓" (then render gallery + color set selector read-only)

Above 768px, render the full builder as scroll-through sections (NOT a wizard — customer can interact with sections in any order).

Page state shape (use Zustand or useReducer at page level):
  activeColorSetId: number   // single source of truth, default = product.colorSets[0].id
  activeImageSlot: number    // tab index, default 0
  config: CustomConfig       // see types in master prompt

═══ SECTION 1 — Top hero strip ═══
- Breadcrumb (same as Mode A)
- Product name + price
- Eyebrow tag: "ĐỒNG PHỤC TÙY CHỈNH" (brand-yellow pill)

═══ SECTION 2 — <ColorSetSelector /> ═══
- Horizontal row of image swatches (size-20)
- Click → setActiveColorSetId
- Behavior: gallery main image swaps; canvas tabs re-render for new ColorSet's images; overlays[][] is PRESERVED (% coords auto-scale to new image dimensions); if new ColorSet has fewer images, extra tabs are hidden (data preserved in array — never delete)

═══ SECTION 3 — <ProductGallery /> (dual-mode display) ═══
Toggle between "Xem ảnh" (thumbnail grid) and "Thiết kế" (tabs) modes via segmented control at top.
- Browse mode: thumbnail grid below main image
- Builder mode: tabs row above canvas (linked to activeImageSlot state)

═══ SECTION 4 — <LogoLibrary /> + Team Name ═══
2-column split:

Left card (60%): <LogoUploader />
- Dropzone (shadcn) — accepts PNG/JPG/SVG, max 5MB
- Drag-and-drop zone with lucide UploadCloud icon, size-12, brand-blue
- "Kéo thả logo vào đây hoặc bấm để chọn" + helper "PNG, JPG, SVG · tối đa 5MB"
- Validation:
  * MIME type check (image/png, image/jpeg, image/svg+xml only)
  * File size check (≤ 5MB)
  * Width/height check via Image() onload — if width<300 OR height<300 → set qualityNote: "Logo dưới 300px — shop sẽ làm nét lại nếu có thể"
  * SVG: sanitize via isomorphic-dompurify (strip <script>, on*, javascript:) BEFORE storing the data URL
- After upload, append LogoItem to config.logos and show sonner toast "Đã tải logo lên"

Right card (40%): <LogoLibraryPanel />
- Section title "Thư viện logo (N)"
- Thumbnail grid (3 cols) of uploaded LogoItems
- Each thumb: size-20, rounded, ring on hover, draggable=true (HTML5 drag for canvas drop)
- Bottom badge "⚠ Chất lượng thấp" (brand-red pill, tiny) if qualityNote exists
- [×] remove button (top-right, appears on hover)
- Empty state: "Chưa có logo nào" + arrow pointing to uploader

Below the 2-column split: input "Tên đội (in trên áo)" — optional, max 30 chars, helper "Tên đội sẽ in cùng số áo của từng cầu thủ"

═══ SECTION 5 — <MockupCanvas /> (CORE — drag/resize/rotate) ═══

⚠️ INTERACTION FLOW (MANDATORY — implement ALL 10 steps, this is what v0 commonly skips):
  1. User drops logo from panel onto canvas → create OverlayElement → IMMEDIATELY setSelectedId(newId) so handles appear without an extra click
  2. Click an overlay → select it (selectedId = el.id) → react-moveable wraps it → 8 resize handles + rotation knob become visible
  3. Click empty area of canvas (not on any overlay) → setSelectedId(null) → handles disappear
  4. SELECTED state visual: dashed brand-blue (#00AEEF) outline 2px on overlay element itself (separate from Moveable's handles). Add via [data-selected="true"] CSS attribute, NOT a wrapper div (would offset coordinates)
  5. Drag the overlay body → move (% coords)
  6. Drag a corner handle → resize with keepRatio=true (Shift to ignore ratio, optional)
  7. Drag the rotation knob → rotate with MAGNETIC SNAP (see below)
  8. Keyboard (when an overlay is selected and canvas has focus):
     - Delete / Backspace → remove the overlay
     - Escape → deselect
     - Arrow keys → nudge by 1% (Shift+Arrow = 5%)
     - [ / ] → send back / bring forward (zIndex)
  9. Double-click a 'text' overlay → inline edit text (contentEditable or popover input)
  10. Hover any overlay (when not selected) → cursor: move + faint brand-blue outline (1px solid, opacity 50%)

Layout: 2-column [LogoPanel left 240px | Canvas right flex-1]

Left: compact <LogoLibraryPanel /> (vertical list, drag source). Selected element actions appear here when an overlay is selected on canvas: [Bring to front] [Send to back] [Duplicate] [Delete]. Wire each button to selectedId state — disabled when selectedId === null.

Right: <ImageTabSwitcher /> + Canvas

ImageTabSwitcher:
- Dynamic tabs: one per ProductImage in activeColorSet.images, sorted by sortOrder
- Tab label: ProductImage.label OR fallback "Ảnh ${index+1}"
- Active tab: brand-yellow underline
- 2-10 tabs depending on data — NEVER hardcode "front/back/sleeve/shorts"

Canvas DOM structure (mandatory — pixel-accurate dragging requires this exact setup):

  <div ref={canvasRef}
       className="relative w-full select-none border rounded-lg overflow-hidden bg-gray-50"
       style={{ aspectRatio: `${imgNaturalW}/${imgNaturalH}` }}
       onDragOver={(e) => e.preventDefault()}
       onDrop={handleLogoDropFromPanel}>
    <img src={activeImage.url}
         className="absolute inset-0 w-full h-full object-contain pointer-events-none"
         alt="" />
    {config.overlays[activeImageSlot]?.map(el => (
      <OverlayElement key={el.id}
                      element={el}
                      canvasRef={canvasRef}
                      isSelected={selectedId === el.id}
                      onSelect={() => setSelectedId(el.id)} />
    ))}
    {selectedId && <Moveable target={`#overlay-${selectedId}`} ... />}
  </div>

OverlayElement positioning (ALL coords as %, NEVER mix px):

  <div id={`overlay-${el.id}`} ref={ref}
       className="absolute will-change-transform cursor-move"
       style={{
         left:    `${el.x * 100}%`,
         top:     `${el.y * 100}%`,
         width:   `${el.width * 100}%`,
         height:  `${el.height * 100}%`,
         transform: `rotate(${el.rotation}deg) translate3d(0,0,0)`,
         transformOrigin: 'center center',
       }}>
    {el.type === 'logo'
      ? <img src={logoMap[el.logoId].url} className="w-full h-full object-contain pointer-events-none" />
      : <div className="w-full h-full flex items-center justify-center font-bold text-2xl">{el.text}</div>}
  </div>

react-moveable config (CRITICAL — use this exact setup for performance):

  <Moveable
    target={selectedTargetRef}
    container={canvasRef.current}
    draggable={true}
    resizable={true}
    rotatable={true}
    keepRatio={true}
    throttleRotate={15}
    snappable={true}
    onDrag={(e) => {
      // Mutate DOM directly — DO NOT setState here
      const rect = canvasRef.current!.getBoundingClientRect();
      e.target.style.left = `${(e.left / rect.width) * 100}%`;
      e.target.style.top  = `${(e.top  / rect.height) * 100}%`;
    }}
    onDragEnd={(e) => {
      // Commit to React state ONLY here
      const rect = canvasRef.current!.getBoundingClientRect();
      updateOverlay(activeImageSlot, selectedId, {
        x: e.lastEvent.left / rect.width,
        y: e.lastEvent.top  / rect.height,
      });
    }}
    onResize={(e) => {
      e.target.style.width  = `${e.width}px`;
      e.target.style.height = `${e.height}px`;
      e.target.style.transform = e.drag.transform;
    }}
    onResizeEnd={(e) => {
      const rect = canvasRef.current!.getBoundingClientRect();
      updateOverlay(activeImageSlot, selectedId, {
        width:  e.lastEvent.width  / rect.width,
        height: e.lastEvent.height / rect.height,
      });
    }}
    onRotate={(e) => {
      const { angle, isSnapped } = applyMagneticSnap(e.rotation);
      e.target.style.transform = `rotate(${angle}deg)`;
      e.target.dataset.snapped = isSnapped ? 'true' : 'false';
      if (isSnapped) showSnapBadge(angle);   // see helper below
    }}
    onRotateEnd={(e) => {
      const { angle } = applyMagneticSnap(e.lastEvent.rotation);
      updateOverlay(activeImageSlot, selectedId, { rotation: angle });
      hideSnapBadge();
    }}
  />

═══ MAGNETIC ROTATION SNAP (CRITICAL — what makes the UX feel premium) ═══

Spec:
- Snap angles: 0°, 90°, 180°, 270° (360° normalizes to 0°)
- Attraction zone: ±8° around each snap angle (wider than the previous 5° — easier to feel the magnet)
- Dwell behavior: when the rotation enters the attraction zone, LOCK to the exact snap angle and HOLD for 100ms before allowing exit. This produces the "khựng lại 1 tí" feel — the rotation visibly sticks at the cardinal angle for ~100ms even if the user keeps moving the cursor. After 100ms dwell, if cursor is still inside the zone the angle stays snapped; if cursor moved outside the zone it releases smoothly.
- Use throttleRotate={3} on Moveable (degree-stepped rotation feels smoother than continuous)

Implementation (place near top of MockupCanvas component):

  const SNAP_ANGLES = [0, 90, 180, 270];
  const SNAP_THRESHOLD = 8;
  const DWELL_MS = 100;
  const lastSnapEnterRef = useRef<{ angle: number; t: number } | null>(null);

  const applyMagneticSnap = useCallback((raw: number): { angle: number; isSnapped: boolean } => {
    const norm = ((raw % 360) + 360) % 360;
    const hit = SNAP_ANGLES.find(s => Math.abs(norm - s) <= SNAP_THRESHOLD || Math.abs(norm - (s + 360)) <= SNAP_THRESHOLD);
    if (hit !== undefined) {
      const now = performance.now();
      // First entry into this snap angle's zone — record entry time
      if (!lastSnapEnterRef.current || lastSnapEnterRef.current.angle !== hit) {
        lastSnapEnterRef.current = { angle: hit, t: now };
      }
      // Within dwell window OR cursor still inside zone → stay locked
      return { angle: hit, isSnapped: true };
    }
    // Outside any zone — but if we just snapped within DWELL_MS, hold the lock briefly
    if (lastSnapEnterRef.current && performance.now() - lastSnapEnterRef.current.t < DWELL_MS) {
      return { angle: lastSnapEnterRef.current.angle, isSnapped: true };
    }
    lastSnapEnterRef.current = null;
    return { angle: norm, isSnapped: false };
  }, []);

Visual feedback when isSnapped === true:
- The overlay element gets [data-snapped="true"] attribute → CSS rule applies brand-yellow outline:
    [data-snapped="true"] { outline: 2px solid #FDD017; outline-offset: 2px; transition: outline-color 120ms ease; }
- Floating angle badge component <SnapBadge angle={angle} /> renders above the canvas (position: absolute, top: -28px relative to overlay center) with text like "0°" / "90°" / "180°" / "270°", bg-brand-yellow text-black px-2 py-0.5 rounded-full text-xs font-semibold. Fades out 200ms after isSnapped becomes false (use a timeout in showSnapBadge / hideSnapBadge helpers).
- Optional micro-haptic feel: brief CSS transition on the overlay's transform (transition: transform 80ms ease-out) ONLY on the snap frame — toggle a class for 1 frame then remove. Don't apply this transition globally or it'll lag during free rotation.

Drop-from-panel handler (MUST auto-select the new overlay so resize/rotate handles appear immediately — without this, customers report "logo uploaded but no controls"):
  function handleLogoDropFromPanel(e: React.DragEvent) {
    e.preventDefault();
    const logoId = e.dataTransfer.getData('logoId');
    if (!logoId) return;
    const rect = canvasRef.current!.getBoundingClientRect();
    const x = (e.clientX - rect.left) / rect.width  - 0.075;  // center the 15% logo on cursor
    const y = (e.clientY - rect.top)  / rect.height - 0.075;
    const newId = crypto.randomUUID();
    addOverlay(activeImageSlot, {
      id: newId,
      type: 'logo',
      logoId,
      x: Math.max(0, Math.min(0.85, x)),
      y: Math.max(0, Math.min(0.85, y)),
      width: 0.15,
      height: 0.15,
      rotation: 0,
      zIndex: nextZIndex(),
    });
    // CRITICAL: auto-select the new overlay so Moveable handles render on next paint
    requestAnimationFrame(() => setSelectedId(newId));
  }

Click outside any overlay → setSelectedId(null) → deselect.

Keyboard handler (attach to canvas wrapper with tabIndex={0}, focus on mount):
  function handleCanvasKeyDown(e: React.KeyboardEvent) {
    if (!selectedId) return;
    const step = e.shiftKey ? 0.05 : 0.01;
    switch (e.key) {
      case 'Delete':
      case 'Backspace':  e.preventDefault(); removeOverlay(activeImageSlot, selectedId); setSelectedId(null); return;
      case 'Escape':     setSelectedId(null); return;
      case 'ArrowLeft':  e.preventDefault(); nudgeOverlay(selectedId, -step, 0); return;
      case 'ArrowRight': e.preventDefault(); nudgeOverlay(selectedId,  step, 0); return;
      case 'ArrowUp':    e.preventDefault(); nudgeOverlay(selectedId, 0, -step); return;
      case 'ArrowDown':  e.preventDefault(); nudgeOverlay(selectedId, 0,  step); return;
      case '[':          bringForward(selectedId, -1); return;
      case ']':          bringForward(selectedId,  1); return;
    }
  }

Performance rules — must implement:
1. GPU layer: transform: translate3d + will-change: transform
2. DOM-mutation during drag, setState only on dragEnd / resizeEnd / rotateEnd
3. React.memo wrap OverlayElement
4. useCallback for all event handlers
5. Validate 60fps target in Chrome DevTools Performance recording

Coordinate accuracy — must avoid:
- ❌ window.innerWidth (use canvasRect.width)
- ❌ pageX/pageY (use clientX/Y — viewport-aware)
- ❌ Forgetting to subtract canvasRect.left/top
- ❌ Mixing px and % in same element style
- ❌ position: fixed on overlays
- ❌ transform-origin: top-left (use center center)

═══ SECTION 6 — <PlayerListEditor /> ═══

Card containing:
- Title "Danh sách cầu thủ" + counter "(N cầu thủ)"
- 2 buttons top-right: [+ Thêm cầu thủ] (brand-yellow) | [Paste từ Excel] (outline brand-blue, with Clipboard icon)
- Table:
  | STT | Tên cầu thủ (optional) | Số áo (optional) | Size * | (×)|
  - STT auto-generated
  - Tên: text input, max 20 chars, sanitize XSS (strip < > on blur)
  - Số áo: text input, regex /^\d{0,3}$/, max value 99
  - Size: shadcn Select dropdown — options from product.variants.filter(v => v.colorSetId === activeColorSetId), label = size, disabled if stock=0 (with "Hết hàng" suffix in option label)
  - (×): trash icon button (only visible if rows > 1) — soft confirm via popover

Validation:
- Default: 1 row on mount (NOT 6, NOT empty)
- NO minimum quantity (1 player order is valid)
- size REQUIRED (red asterisk + bottom error message if empty on submit)
- playerName + playerNumber both OPTIONAL

[Paste từ Excel] handler:
  async function handlePasteFromExcel() {
    const text = await navigator.clipboard.readText();
    const lines = text.trim().split(/\r?\n/);
    const rows = lines.map(line => line.split(/\t/));
    // Auto-detect columns:
    //   - Column with majority of values matching /^\d{1,3}$/ → "Số áo"
    //   - Column with values matching /^(S|M|L|XL|XXL)$/i → "Size"
    //   - Column with majority alphabetic strings (>3 chars) → "Tên cầu thủ"
    // Confidence ≥ 80% (≥80% of values match pattern) → auto-map
    // Otherwise → open <ColumnMappingDialog />
    if (autoDetected) {
      appendRows(parsedRows);
      toast.success(`Đã thêm ${parsedRows.length} cầu thủ`);
    } else {
      openMappingDialog({ rows, sampleData });
    }
  }

<ColumnMappingDialog />:
- shadcn Dialog
- Title "Ánh xạ cột từ Excel"
- Body: preview table of first 3 rows + dropdown above each column to assign meaning (Tên / Số áo / Size / Bỏ qua)
- "Áp dụng" + "Hủy" buttons

═══ SECTION 7 — <PrintFeeNudge /> ═══

Sticky card (or large card pinned above add-to-cart) showing 4 states based on calculatePrintFee result:

State A — !hasPrintingWork:
  bg: bg-gray-100, icon: ShirtIcon (lucide) gray
  Title: "Bạn đang đặt áo trơn"
  Body: "Không có chữ in / logo / số áo. Phí in: 0₫"

State B — count < 6:
  bg: bg-amber-50 border-amber-200, icon: AlertTriangle amber
  Title: "Phí in +20% ({formatVnd(amount)})"
  Body: "Thêm {6-count} bộ để giảm phí in xuống 10%"
  Progress bar: count / 10

State C — count 6..9:
  bg: bg-amber-50/60, icon: TrendingUp amber
  Title: "Phí in +10% ({formatVnd(amount)})"
  Body: "Thêm {10-count} bộ để được miễn phí in"
  Progress bar: count / 10

State D — count ≥ 10:
  bg: bg-emerald-50 border-emerald-200, icon: PartyPopper emerald
  Title: "Miễn phí in 🎉"
  Body: "Đơn của bạn được miễn phí in toàn bộ"

The fee recomputes in real-time on every: overlay add/remove, player name/number change, teamName change, player row add/remove.

═══ SECTION 8 — Sticky bottom bar (price + add to cart) ═══

position: sticky bottom-0, z-40, bg-white border-t shadow-2xl
Layout: 2 columns
- Left: Price breakdown
  Số bộ: {players.length}
  Đơn giá: {formatVnd(unitPrice)} × {players.length} = {formatVnd(subtotal)}
  Phí in: {formatVnd(printFee.amount)} ({printFee.label})
  TỔNG: {formatVnd(total)} (text-2xl brand-red font-bold)
- Right: Add to Cart button (h-14, brand-yellow, font-bold, text-lg)
  "Thêm vào giỏ hàng — {players.length} bộ"
  Disabled if: any player missing size, OR no logos AND hasPrintingWork is false but customer wants printing (no — actually allow blank áo trơn submit, just enforce size)
  On click:
    1. Validate all sizes present
    2. Build CustomCartItem:
       { type:'custom', id: crypto.randomUUID(), productId, productName, colorSetId: activeColorSetId, colorSetName, productImages: activeColorSet.images.map(i=>i.url), config, unitPrice, printingSurcharge: printFee.amount, totalPrice: total }
    3. cartStore.addCustomItem(item)
    4. toast.success(`Đã thêm ${players.length} bộ vào giỏ`)
    5. router.push('/cart')

═══ DELIVERABLES ═══
- app/products/[slug]/page.tsx (client — full state lives here)
- components/product/product-gallery.tsx (dual-mode: grid + tabs)
- components/product/color-set-selector.tsx
- components/product/size-selector.tsx
- components/product/breadcrumb.tsx
- components/custom-builder/mobile-builder-notice.tsx
- components/custom-builder/logo-uploader.tsx
- components/custom-builder/logo-library-panel.tsx
- components/custom-builder/mockup-canvas.tsx
- components/custom-builder/overlay-element.tsx
- components/custom-builder/image-tab-switcher.tsx
- components/custom-builder/player-list-editor.tsx
- components/custom-builder/player-row.tsx
- components/custom-builder/column-mapping-dialog.tsx
- components/custom-builder/print-fee-nudge.tsx
- components/custom-builder/sticky-add-to-cart.tsx
- lib/svg-sanitizer.ts (uses isomorphic-dompurify)
- lib/clipboard-parser.ts (TSV parse + column auto-detect)
- lib/printing-fee.ts (hasPrintingWork + calculatePrintFee — exported)
- stores/cart-store.ts (Zustand + persist, with both NormalCartItem + CustomCartItem)
- types/custom-config.ts

Use ONE long mock product with isCustomizable=true at top of file:
  const MOCK_PRODUCT: Product = {
    id: 1,
    slug: 'ao-doi-bong-pro-2026',
    name: 'Áo đội bóng StarSport Pro 2026',
    description: 'Áo đấu chuyên nghiệp, vải Coolmax thoáng khí, có thể tùy chỉnh logo + tên + số.',
    category: 'jersey-club',
    basePrice: 250000,
    isCustomizable: true,
    active: true,
    createdAt: '2026-04-01',
    colorSets: [
      { id:1, name:'Đỏ-Trắng', swatchHex:'#E31E26', images:[
        { id:1, url:'https://images.unsplash.com/photo-1577223625816-7546f13df25d', sortOrder:0, label:'Mặt trước', colorSetId:1 },
        { id:2, url:'https://images.unsplash.com/photo-1551958219-acbc608c6377', sortOrder:1, label:'Mặt sau', colorSetId:1 },
        { id:3, url:'https://images.unsplash.com/photo-1517466787929-bc90951d0974', sortOrder:2, label:'Tay áo', colorSetId:1 },
        { id:4, url:'https://images.unsplash.com/photo-1556906903-5e1f6f0c4d6f', sortOrder:3, label:'Quần', colorSetId:1 },
      ]},
      { id:2, name:'Xanh-Đen', swatchHex:'#00AEEF', images:[
        { id:5, url:'https://images.unsplash.com/photo-1517649763962-0c623066013b', sortOrder:0, label:'Mặt trước', colorSetId:2 },
        { id:6, url:'https://images.unsplash.com/photo-1521223890158-f9f7c3d5d504', sortOrder:1, label:'Mặt sau', colorSetId:2 },
      ]},
    ],
    variants: [
      { id:1, colorSetId:1, size:'S', stock:10 },
      { id:2, colorSetId:1, size:'M', stock:15 },
      { id:3, colorSetId:1, size:'L', stock:8 },
      { id:4, colorSetId:1, size:'XL', stock:0 },
      { id:5, colorSetId:1, size:'XXL', stock:3 },
      { id:6, colorSetId:2, size:'S', stock:5 },
      { id:7, colorSetId:2, size:'M', stock:12 },
      { id:8, colorSetId:2, size:'L', stock:0 },
      { id:9, colorSetId:2, size:'XL', stock:7 },
    ],
  };

EVERY feature listed above must be implemented and working. No placeholders. No "TODO" comments. The customer must be able to:
- Upload multiple logos with quality warning
- Drag a logo from the panel onto the canvas
- Drag/resize/rotate the logo with snap-to-90°
- Switch image tabs and have independent overlays per tab
- Switch ColorSet and have overlays preserved (data-wise, not visually if image dims differ — auto-scale via %)
- Add 1+ player rows manually OR paste from Excel with auto-detection
- See real-time print fee changes across all 4 states
- Click "Add to cart" → see CustomCartItem in Zustand store → land on /cart

Use shadcn/ui primitives (Dialog, Tabs, Select, Slider, Checkbox, Tooltip, Sonner, Sheet, Card, Button, Input). Style with Tailwind only — no global CSS overrides except font import.
```

---

## 🟢 SCREEN 4 PROMPT — Cart `/cart` + Checkout `/checkout`

```
Generate the StarSport cart and checkout pages.

═══════════════════════════════════════════════════════════
PAGE 1: app/cart/page.tsx
═══════════════════════════════════════════════════════════

Layout: 2-column desktop (items list left flex-1, summary right 380px sticky), stacked mobile.

Reads from Zustand cart store (mixed NormalCartItem + CustomCartItem).

═══ Items list ═══

For NormalCartItem render <NormalCartRow />:
- Thumbnail (size-24, rounded)
- Right of thumb: name (font-semibold), "Bộ màu: {colorSetName} · Size: {size}", quantity stepper [- N +] (min 1 max 99), unit price × qty = subtotal (right-aligned)
- Far right: trash icon (×) with confirm popover

For CustomCartItem render <CustomCartRow />:
- Thumbnail (size-24, first image of productImages)
- Top-left badge "ĐỒNG PHỤC TÙY CHỈNH" (brand-yellow pill)
- Bundle summary block:
  Title: {config.teamName || productName} — {config.players.length} bộ
  Subline: "Bộ màu: {colorSetName}"
  Player size summary (chips): "M×3", "L×6", "XL×3"  (computed by groupBy size)
  Logo count chip: "🎨 {config.logos.length} logo"
  Print fee label: <PrintFeeNudge /> compact variant inline
- Right: total price (formatted VND, brand-red font-bold)
- Trash icon × ONLY (no edit button — to modify, customer must delete and recreate from /products/[slug])

Empty state: when cart is empty
- Centered, py-32
- Icon: ShoppingCart (lucide) size-20 brand-gray
- Headline: "Giỏ hàng của bạn đang trống"
- Body: "Khám phá hàng trăm mẫu áo đấu và phụ kiện chuyên nghiệp"
- CTA: brand-yellow button "Khám phá sản phẩm" → /products

═══ Cart summary (right, sticky top-24) ═══
- Title "Tóm tắt đơn hàng"
- Line items:
  Tạm tính ({total items}): {formatVnd(subtotal)}
  Phí in (per custom order, listed individually):
    "Đơn '{teamName1}': {formatVnd(fee1)}"
    "Đơn '{teamName2}': {formatVnd(fee2)}"
  Phí vận chuyển: "Tính khi giao hàng" (italic gray)
- Divider
- Total: TỔNG CỘNG {formatVnd(grandTotal)} (text-2xl brand-red bold)
- CTA button (full-width, h-12, brand-yellow): "Thanh toán →"
  Click handler: if user not logged in, redirect /login?redirect=/checkout; else router.push('/checkout')
- Helper text below: "Đơn hàng được giao hàng toàn quốc · Thanh toán khi nhận hàng (COD)"
- Trust badges row: lucide Truck, Shield, Phone icons + "Miễn phí đổi size · Bảo mật · Hotline 24/7"

═══════════════════════════════════════════════════════════
PAGE 2: app/checkout/page.tsx
═══════════════════════════════════════════════════════════

Auth gate: if no session, server-side redirect to /login?redirect=/checkout.

Layout: 2-column desktop (form left flex-1, summary right 380px sticky).

═══ <CheckoutForm /> ═══
react-hook-form + zod schema:

  const schema = z.object({
    fullName: z.string().min(2, 'Vui lòng nhập họ tên').max(50),
    phone: z.string().regex(/^(0|\+84)[0-9]{9,10}$/, 'Số điện thoại không hợp lệ'),
    address: z.string().min(10, 'Vui lòng nhập địa chỉ đầy đủ'),
    note: z.string().max(500).optional(),
    paymentMethod: z.literal('COD'),
  });

Auto-fill from a mock user object:
  const MOCK_USER = { shippingName: 'Nguyễn Văn An', shippingPhone: '0901234567', shippingAddress: '123 Đường Lê Lợi, Quận 1, TP.HCM' };

Card 1: "Thông tin giao hàng"
- 2-column grid (1-col mobile): fullName | phone
- Full-width: address (textarea, 3 rows)
- Full-width: note (textarea, 2 rows, placeholder "Ví dụ: Giao giờ hành chính, gọi trước khi đến...")

Card 2: "Phương thức thanh toán"
- Radio group:
  ✅ COD — Thanh toán khi nhận hàng (default, selected)
  ⏸ VNPay — Sắp ra mắt (disabled, opacity-50, lock icon)
  ⏸ MoMo — Sắp ra mắt (disabled)
  ⏸ Chuyển khoản ngân hàng — Sắp ra mắt (disabled)
- Below COD: helper text "Bạn sẽ thanh toán bằng tiền mặt cho nhân viên giao hàng. Đơn hàng được giao trong 3-5 ngày làm việc."

Card 3 (collapsible): "Mã giảm giá" — placeholder, just renders an input + "Áp dụng" button (does nothing, marked "Sắp ra mắt").

═══ Order summary (right) ═══
Same structure as Cart summary, but with bigger CTA:
- "Đặt hàng — {formatVnd(grandTotal)}" (h-14, brand-yellow, font-bold)
- Loading state during submit (spinner + "Đang xử lý...")
- Helper below: "Bằng việc đặt hàng, bạn đồng ý với Điều khoản & Chính sách bảo mật của StarSport"

Submit handler (mock for v0/Lovable):
  async function onSubmit(data) {
    setSubmitting(true);
    await new Promise(r => setTimeout(r, 1500));  // simulate API
    const orderId = `ORD-${formatYYMMDD()}-${String(Math.floor(Math.random()*999)+1).padStart(3,'0')}`;
    cartStore.clear();
    router.push(`/orders/${orderId}?just-placed=true`);
  }

═══════════════════════════════════════════════════════════
PAGE 3: app/orders/[id]/page.tsx (Confirmation)
═══════════════════════════════════════════════════════════

Show:
- Big success card top:
  ✅ icon (CheckCircle2 emerald, size-20)
  "Đặt hàng thành công!"
  "Mã đơn: {orderId}" (text-3xl font-mono font-bold)
  "Cảm ơn bạn — shop sẽ liên hệ xác nhận trong 24h."
- 2-column block:
  Left card: Thông tin giao hàng (name, phone, address, note)
  Right card: Phương thức thanh toán (COD)
- Order items list (compact, no editing):
  For each NormalCartItem: thumb + name + qty + total
  For each CustomCartItem: thumb + bundle title + player size summary chips + total
- Total summary at bottom (subtotal + per-custom print fees + grand total)
- 2 CTA buttons: "Xem đơn hàng của tôi" (outline) → /orders | "Tiếp tục mua sắm" (brand-yellow) → /products
- Cancel order button — visible ONLY if status === 'pending':
  "Hủy đơn hàng" (ghost text-brand-red) — opens shadcn AlertDialog confirm "Bạn có chắc muốn hủy đơn?"
  On confirm: PATCH /api/orders/:id/cancel (mock — just update local state to 'cancelled')

NO mockup canvas rendering on this page. Player size summary only.

═══ DELIVERABLES ═══
- app/cart/page.tsx
- app/checkout/page.tsx
- app/orders/[id]/page.tsx
- components/cart/normal-cart-row.tsx
- components/cart/custom-cart-row.tsx
- components/cart/cart-summary.tsx
- components/cart/empty-cart-state.tsx
- components/checkout/checkout-form.tsx
- components/checkout/payment-method-selector.tsx
- components/orders/order-success-banner.tsx
- components/orders/order-items-summary.tsx
- components/orders/cancel-order-dialog.tsx
- lib/group-players-by-size.ts
- lib/format-order-id.ts

Reuse <PrintFeeNudge /> from Screen 3 in compact variant for per-custom-order rows.
```

---

## 📋 TIPS WHEN PROMPTING v0 / LOVABLE

1. **Send Master Prompt first**, wait for "design system locked" acknowledgment, THEN send screen prompts.
2. **Lovable**: it can generate full Next.js app — you can paste the master + all 4 screens together as a single requirements doc, then ask it to scaffold the project.
3. **v0**: it generates one component / page at a time. Send screens one by one. Use "Refine" to ask for fixes. For screen 3, you may need to break it into 3 follow-up prompts (1: structure + gallery + logo upload, 2: canvas + react-moveable, 3: player roster + print fee + add-to-cart).
4. **Mock data discipline**: keep all mock data inline at the top of the page file (`const MOCK_PRODUCTS = [...]`). Do NOT let v0 invent its own product types — paste the type definitions from the Master Prompt every time it forgets.
5. **Library installs**: v0 / Lovable auto-install shadcn but may miss `react-moveable`, `isomorphic-dompurify`, `zustand`, `react-hook-form`, `zod`, `sonner`. Confirm in the project's package.json after generation.
6. **Iteration prompts** (when the AI gets something wrong):
   - "The overlay drift bug — coords are mixed px and %. Refactor so all 4 sides (left/top/width/height) are %. Use canvasRef.getBoundingClientRect() inside react-moveable handlers."
   - "Print fee is wrong when overlays.length === 0 but players have names. Re-read hasPrintingWork() — it must return true if ANY of: overlays present, any player has name OR number, OR teamName is set."
   - "Tabs are hardcoded to 4 angles. Make them dynamic from `activeColorSet.images.sort(by sortOrder).length` — could be 2 to 10 tabs."
   - "After dropping a logo on the canvas there are no resize/rotate handles — Moveable isn't wrapping the new overlay. Fix: in handleLogoDropFromPanel, after addOverlay, call requestAnimationFrame(() => setSelectedId(newId)) to auto-select. Also wrap the canvas in tabIndex={0} and add Delete/Escape/Arrow key handlers tied to selectedId."
   - "Rotation snaps instantly without the 'magnetic dwell' feel. Replace the snap logic with applyMagneticSnap() helper (see Section 5): widen threshold to ±8°, hold the snapped angle for 100ms after entering the zone, set [data-snapped='true'] for the brand-yellow outline + floating angle badge. Use throttleRotate={3} on Moveable."
   - "Selected overlay has no visible outline of its own — only the Moveable handles. Add [data-selected='true'] CSS rule on the overlay element with 2px dashed brand-blue outline so the customer can see what's selected even before interacting with handles."

---

## Unresolved Questions

1. Should v0/Lovable also generate `/orders/list` (order history) and `/login` `/register` pages? Master prompt currently scopes only the 4 screens requested — those are out of scope but commonly auto-generated.
2. Do we want to provide actual Cloudinary asset URLs for hero images, or keep Unsplash placeholders until shop provides photos? (Currently: Unsplash.)
3. Lovable can wire Supabase auth — do we want it to, or keep auth as a `MOCK_USER` stub for the design preview phase?
