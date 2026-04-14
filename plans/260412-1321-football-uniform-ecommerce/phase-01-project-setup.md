---
phase: 1
title: "Project Setup + DB Schema"
status: pending
priority: P1
effort: 3d
---

# Phase 1: Project Setup + DB Schema

## Context
- [Tech Stack Research](../reports/researcher-260412-1321-tech-stack.md)

## Overview
Bootstrap Next.js 15 project, configure Prisma + PostgreSQL, setup Better Auth, Cloudinary, base layout.

## Requirements
- Next.js 15 App Router with TypeScript
- PostgreSQL via Railway (dev: local Docker or Railway dev)
- Prisma ORM with complete schema
- Better Auth configured (emailAndPassword + admin plugins — full setup in Phase 2); User model extended with shipping fields (Session 8)
- Cloudinary SDK configured
- Tailwind CSS + **shadcn/ui** + base layout (header, footer, responsive)
- **Montserrat font** configured (brand font)
- **Brand colors theme** configured (src/lib/theme.ts with custom colors)
- **Mobile responsive** from Phase 1 (all pages mobile-friendly)
- ESLint + Prettier configured
- Environment variables (.env.example)
<!-- Updated: Session 1 - shadcn/ui added to stack; stock field kept in schema but no auto-decrement logic -->
<!-- Updated: Session 4 - Montserrat font, brand colors theme, mobile responsive from Phase 1 -->
<!-- Updated: Session 8 - ProductVariant FK to colorSetId, ProductImage sortOrder-based (free-form), Player single size field, User shipping fields -->

## DB Schema Design

### Key Schema Changes (Session 8)
1. **ProductVariant**: FK changed from `productId` → `colorSetId` — stock per ColorSet × size × type (override Session 6)
2. **ProductImage**: `angle` enum removed → `sortOrder: Int` only — free-form images, not hard-coded 4 angles
3. **Player**: Single `size` field (required) replaces `jerseySize` + `shortsSize`
4. **User** (Better Auth): Extended with `shippingName`, `shippingPhone`, `shippingAddress` (nullable)
5. **CustomOrder.printConfigJson**: `overlays: OverlayElement[][]` indexed by image slot (not `angles.{front/back/sleeve/shorts}`)

### Core Models

```prisma
model Product {
  id            Int      @id @default(autoincrement())
  name          String
  slug          String   @unique
  description   String?
  basePrice     Float
  category      String   // "uniform-set", "jersey", "shorts", "jacket", "shoes", "accessory"
  customizable  Boolean  @default(false)
  active        Boolean  @default(true)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  variants      ProductVariant[]
  images        ProductImage[]
  colorSets     ColorSet[]
  orderItems    OrderItem[]
  customOrders  CustomOrder[]
}

model ProductVariant {
  id        Int     @id @default(autoincrement())
  colorSetId Int
  colorSet  ColorSet @relation(fields: [colorSetId], references: [id], onDelete: Cascade)
  size      String  // "S", "M", "L", "XL", "2XL"
  type      String  // "jersey", "shorts"
  stock     Int     @default(0)

  @@unique([colorSetId, size, type])
}

model ProductImage {
  id        Int     @id @default(autoincrement())
  productId Int
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  url       String  // Cloudinary URL
  colorSetId Int?
  colorSet  ColorSet? @relation(fields: [colorSetId], references: [id])
  sortOrder Int     @default(0)  // Session 8: free-form images, not hard-coded angles
}

// ColorSet = bộ màu sản phẩm do admin định sẵn (không phải color picker tự do).
// Relationship: ColorSet 1-N ProductImage (mỗi ColorSet có nhiều ảnh tự do, indexed by sortOrder).
// Relationship: ColorSet 1-N ProductVariant (stock per ColorSet riêng lẻ — Session 8 override).
// Admin upload ảnh thật cho mỗi bộ màu (không hard-code 4 góc).
// Khách chỉ được chọn trong các ColorSet admin đã tạo cho sản phẩm đó.
// primary/secondary hex chỉ dùng để hiện chip màu trong UI — KHÔNG dùng để render màu.
model ColorSet {
  id          Int     @id @default(autoincrement())
  productId   Int
  product     Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  name        String  // "Xanh Navy + Trắng"
  slug        String
  primary     String  // hex — chỉ để hiện chip màu trong UI
  secondary   String  // hex — chỉ để hiện chip màu trong UI
  images      ProductImage[] // ảnh thật sản phẩm tự do cho bộ màu này (sortOrder-based)
  variants    ProductVariant[] // stock per ColorSet + size + type

  @@unique([productId, slug])
}

model Order {
  id              Int      @id @default(autoincrement())
  orderNumber     String   @unique // human-readable: ORD-20260412-001
  userId          String   // NOT NULL — mandatory login (Phase 2)
  customerName    String
  customerEmail   String
  customerPhone   String
  shippingAddress String
  // Session 8: User model extended with shippingName, shippingPhone, shippingAddress (nullable) — auto-fill checkout
  note            String? // Customer note for the order
  adminNote       String? // Note for shop admin (e.g., logo quality issues)

  subtotal        Float
  printingSurcharge Float @default(0)
  shippingFee     Float   @default(0)
  totalPrice      Float

  paymentMethod   String  // "cod" (MVP - VNPay/MoMo deferred to Phase 2)
  paymentStatus   String  @default("pending") // "pending", "paid", "failed", "refunded"
  orderStatus     String  @default("pending") // "pending", "processing", "shipping", "delivered", "cancelled"

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  items           OrderItem[]
  customOrders    CustomOrder[]

  @@index([paymentStatus])
  @@index([orderStatus])
}

model OrderItem {
  id        Int     @id @default(autoincrement())
  orderId   Int
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId Int
  product   Product @relation(fields: [productId], references: [id])
  quantity  Int
  unitPrice Float
  size      String
  colorSetName String?

  @@index([orderId])
}

model CustomOrder {
  id              Int      @id @default(autoincrement())
  orderId         Int
  order           Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId       Int
  product         Product  @relation(fields: [productId], references: [id])
  colorSetName    String
  // teamName optional — fallback to product.name in email/Excel if empty
  teamName        String?
  logoQualityNote String? // Auto-generated note for low quality logos

  // Full print config — single source of truth for builder state + admin preview
  // Session 7: removed redundant logoUrl columns + boolean print config columns — printConfigJson is the only source
  // Session 8: overlays stored as 2D array indexed by image slot (free-form, not fixed angles)
  // Structure:
  // {
  //   "teamName": "FC Sao Vàng",
  //   "logos": [
  //     { "id": "logo_1", "url": "https://cloudinary.com/...", "name": "Club Logo" }
  //   ],
  //   "overlays": [
  //     [{ "logoId": "logo_1", "x": 45.2, "y": 30.1, "w": 10.5, "h": 8.3, "rotation": 0 }],  // image slot 0
  //     [],  // image slot 1
  //     [...]  // image slot N
  //   ]
  // }
  // OverlayElement: { type: "logo"|"text", logoId?: string, text?: string, x: %, y: %, w: %, h: %, rotation: number, zIndex: number }
  // logos[] supports multi-logo (club logo, sponsor, flag patch, etc.)
  printConfigJson     Json?

  // Generated files
  excelFileUrl        String?

  playerCount         Int
  printingSurcharge   Float   @default(0)

  players             Player[]
  createdAt           DateTime @default(now())
}

model Player {
  id            Int         @id @default(autoincrement())
  customOrderId Int
  customOrder   CustomOrder @relation(fields: [customOrderId], references: [id], onDelete: Cascade)
  sortOrder     Int
  playerName    String?     // optional — nếu trống thì áo trơn không in tên
  playerNumber  String?     // optional — nếu trống thì áo trơn không in số
  size          String      // Session 8: required — single size for both jersey + shorts (XS/S/M/L/XL/XXL)

  @@index([customOrderId])
}
```

## Implementation Steps

### Monorepo Setup (do first)
1. Init workspace root: create `package.json` (name: `football-uniform-store`, private, engines node≥20 pnpm≥9) + `pnpm-workspace.yaml` (`packages: ['apps/*', 'packages/*']`)
2. Create `packages/shared-types/` — `package.json` (name: `@football-store/shared-types`) + `src/index.ts` exporting `ProductCategory`, `ScrapedProduct`, `ScrapedColorSet`, `CsvRow`, `SiteConfig`, `SiteScraper` interfaces
3. Create `apps/web/` dir — run `npx create-next-app@latest apps/web` with TypeScript, Tailwind, App Router, src/ directory; update its `package.json` name to `@football-store/web`, add dep `@football-store/shared-types: workspace:*`

### Next.js App Setup (inside apps/web/)
4. Install deps (from `apps/web/`): `prisma @prisma/client better-auth zustand react-hook-form cloudinary resend exceljs`; init shadcn/ui: `npx shadcn@latest init`
5. `npx prisma init` — configure schema above
6. Setup `.env.example` with all required vars
7. Configure **Montserrat font**: add to `tailwind.config.ts` and `src/app/layout.tsx`
8. Create **brand colors theme**: `src/lib/theme.ts` with brand colors (Vàng #FDD017 primary, Đỏ #E31E26 secondary, Xanh dương #00AEEF accent, Xám #A7A9AC accent); update `tailwind.config.ts`
9. Create base layout: `src/app/layout.tsx` with header (logo, nav, cart icon) + footer (mobile responsive)
10. Run `npx @better-auth/cli generate` → adds User/Session/Account/Verification tables to schema.prisma. Then extend User model with `shippingName`, `shippingPhone`, `shippingAddress` (nullable) for auto-fill checkout. Configure Better Auth base — `src/lib/auth.ts` (plugins configured in Phase 2)
11. Configure Prisma client — `src/lib/db.ts`
12. Configure Cloudinary — `src/lib/cloudinary.ts`
13. Create `src/lib/constants.ts` — sizes, categories, printing fee tiers
14. Seed script — `prisma/seed.ts` with sample products + color sets (dev/test only — production products managed via CSV import in Phase 8)
15. Verify: `pnpm dev` (from root) → homepage renders, DB connected, mobile responsive

## Files to Create

**Workspace root:**
- `package.json` — workspace root
- `pnpm-workspace.yaml`

**packages/shared-types:**
- `packages/shared-types/package.json`
- `packages/shared-types/src/index.ts` — exported interfaces

**apps/web:**
- `apps/web/src/app/layout.tsx` — root layout (with Montserrat font, mobile responsive)
- `apps/web/src/app/page.tsx` — homepage
- `apps/web/src/lib/db.ts` — Prisma client singleton
- `apps/web/src/lib/auth.ts` — Better Auth config
- `apps/web/src/lib/cloudinary.ts` — Cloudinary config
- `apps/web/src/lib/theme.ts` — brand colors constants
- `apps/web/src/lib/constants.ts` — sizes, categories, printing fee tiers
- `apps/web/tailwind.config.ts` — update with Montserrat font + brand colors
- `apps/web/prisma/schema.prisma` — full schema
- `apps/web/prisma/seed.ts` — sample data
- `apps/web/.env.example`

## Todo
- [ ] Create pnpm workspace root (package.json + pnpm-workspace.yaml)
- [ ] Create packages/shared-types with exported TS interfaces
- [ ] Create Next.js 15 project in apps/web/
- [ ] Install all dependencies
- [ ] Write Prisma schema
- [ ] Run initial migration
- [ ] Configure Montserrat font (tailwind.config.ts + layout.tsx)
- [ ] Create src/lib/theme.ts with brand colors
- [ ] Update tailwind.config.ts with brand colors
- [ ] Configure Better Auth base (plugins + auth pages in Phase 2)
- [ ] Setup Cloudinary SDK
- [ ] Create base layout (header + footer + nav, mobile responsive)
- [ ] Create seed script with 5-10 sample products
- [ ] Verify dev server runs clean + mobile responsive

## Success Criteria
- `npm run dev` works
- `npx prisma db push` succeeds
- Seed data visible in Prisma Studio (dev/test only — production products via CSV import in Phase 8)
- Base layout renders on all routes
- Montserrat font applied globally
- Brand colors configured in theme
- Mobile responsive (test on mobile viewport)

## Risk
- Better Auth version — pin version; full plugin setup in Phase 2
- Railway PostgreSQL connection string format — test early

## Security Notes
- Upload API (Phase 4): validate MIME type + max 5MB; **SVG files must be sanitized** (strip `<script>`, event handlers, `href` with `javascript:`) before uploading to Cloudinary to prevent XSS
- ProductVariant is scoped per **Product** (shared across all ColorSets) — do not add ColorSet FK to ProductVariant
