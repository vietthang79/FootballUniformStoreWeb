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
- Better Auth configured (emailAndPassword + admin plugins — full setup in Phase 2)
- Cloudinary SDK configured
- Tailwind CSS + **shadcn/ui** + base layout (header, footer, responsive)
- **Montserrat font** configured (brand font)
- **Brand colors theme** configured (src/lib/theme.ts with custom colors)
- **Mobile responsive** from Phase 1 (all pages mobile-friendly)
- ESLint + Prettier configured
- Environment variables (.env.example)
<!-- Updated: Validation Session 1 - shadcn/ui added to stack; stock field kept in schema but no auto-decrement logic -->
<!-- Updated: Session 2026-04-13 - Montserrat font, brand colors theme, mobile responsive from Phase 1, 4 image angles -->

## DB Schema Design

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
  productId Int
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  size      String  // "S", "M", "L", "XL", "2XL"
  type      String  // "jersey", "shorts"
  stock     Int     @default(0)

  @@unique([productId, size, type])
}

model ProductImage {
  id        Int     @id @default(autoincrement())
  productId Int
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  url       String  // Cloudinary URL
  angle     String  // "front", "back", "sleeve", "shorts" (4 angles per ColorSet)
  colorSetId Int?
  colorSet  ColorSet? @relation(fields: [colorSetId], references: [id])
  sortOrder Int     @default(0)
}

// ColorSet = bộ màu sản phẩm do admin định sẵn (không phải color picker tự do).
// Relationship: ColorSet 1-N ProductImage (mỗi ColorSet có nhiều ảnh theo 4 góc: front/back/sleeve/shorts).
// Admin upload ảnh thật cho mỗi bộ màu theo từng góc.
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
  images      ProductImage[] // ảnh thật sản phẩm theo 4 góc cho bộ màu này (front/back/sleeve/shorts)

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
  // Structure: { teamName, logos: [{ id, url, filename, qualityNote? }], angles: { front: OverlayElement[], back: [], sleeve: [], shorts: [] }, players: [...] }
  // OverlayElement: { id, type: "logo"|"text", logoId?: string, text?: string, x: %, y: %, width: %, height: %, rotation: number, zIndex: number }
  // Positions stored as % for responsiveness. logos[] supports multi-logo (club, sponsor, flag patch, etc.)
  // Session 7: removed redundant logoUrl columns + boolean print config columns — printConfigJson is the only source
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
  playerNumber  Int?        // optional — nếu trống thì áo trơn không in số
  jerseySize    String      // required — để sản xuất (XS/S/M/L/XL/XXL)
  shortsSize    String?     // optional

  @@index([customOrderId])
}
```

## Implementation Steps

1. `npx create-next-app@latest` with TypeScript, Tailwind, App Router, src/ directory
2. Install deps: `prisma @prisma/client better-auth zustand react-hook-form cloudinary resend exceljs`; init shadcn/ui: `npx shadcn@latest init`
3. `npx prisma init` — configure schema above
4. Setup `.env.example` with all required vars
5. Configure **Montserrat font**: add to `tailwind.config.ts` and `src/app/layout.tsx`
6. Create **brand colors theme**: `src/lib/theme.ts` with brand colors (Vàng #FDD017 primary, Đỏ #E31E26 secondary, Xanh dương #00AEEF accent, Xám #A7A9AC accent); update `tailwind.config.ts`
7. Create base layout: `src/app/layout.tsx` with header (logo, nav, cart icon) + footer (mobile responsive)
8. Run `npx @better-auth/cli generate` → adds User/Session/Account/Verification tables to schema.prisma. Then configure Better Auth base — `src/lib/auth.ts` (plugins configured in Phase 2)
9. Configure Prisma client — `src/lib/db.ts`
10. Configure Cloudinary — `src/lib/cloudinary.ts`
11. Create `src/lib/constants.ts` — sizes, categories, printing fee tiers
12. Seed script — `prisma/seed.ts` with sample products + color sets (dev/test only — production products managed via CSV import in Phase 8)
13. Verify: `npm run dev` → homepage renders, DB connected, mobile responsive

## Files to Create
- `src/app/layout.tsx` — root layout (with Montserrat font, mobile responsive)
- `src/app/page.tsx` — homepage
- `src/lib/db.ts` — Prisma client singleton
- `src/lib/auth.ts` — Better Auth config
- `src/lib/cloudinary.ts` — Cloudinary config
- `src/lib/theme.ts` — brand colors constants
- `src/lib/constants.ts` — sizes, categories, printing fee tiers
- `tailwind.config.ts` — update with Montserrat font + brand colors
- `prisma/schema.prisma` — full schema
- `prisma/seed.ts` — sample data
- `.env.example`

## Todo
- [ ] Create Next.js 15 project
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
