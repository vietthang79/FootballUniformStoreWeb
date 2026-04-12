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
- Better Auth with anonymous plugin (guest checkout)
- Cloudinary SDK configured
- Tailwind CSS + **shadcn/ui** + base layout (header, footer, responsive)
- ESLint + Prettier configured
- Environment variables (.env.example)
<!-- Updated: Validation Session 1 - shadcn/ui added to stack; stock field kept in schema but no auto-decrement logic -->

## DB Schema Design

### Core Models

```prisma
model Product {
  id            String   @id @default(cuid())
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
  id        String  @id @default(cuid())
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  size      String  // "S", "M", "L", "XL", "2XL"
  type      String  // "jersey", "shorts"
  stock     Int     @default(0)

  @@unique([productId, size, type])
}

model ProductImage {
  id        String  @id @default(cuid())
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  url       String  // Cloudinary URL
  angle     String  // "front", "back", "shorts", "detail"
  colorSetId String?
  colorSet  ColorSet? @relation(fields: [colorSetId], references: [id])
  sortOrder Int     @default(0)
}

model ColorSet {
  id          String  @id @default(cuid())
  productId   String
  product     Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  name        String  // "Xanh Navy + Trắng"
  slug        String
  primary     String  // hex
  secondary   String  // hex
  images      ProductImage[]

  @@unique([productId, slug])
}

model Order {
  id              String   @id @default(cuid())
  orderNumber     String   @unique // human-readable: ORD-20260412-001
  userId          String?
  sessionId       String?  // guest checkout
  customerName    String
  customerEmail   String
  customerPhone   String
  shippingAddress String
  note            String?
  adminNote       String? // Note for shop admin (e.g., logo quality issues)

  subtotal        Float
  printingSurcharge Float @default(0)
  shippingFee     Float   @default(0)
  totalPrice      Float

  paymentMethod   String  // "vnpay", "momo", "cod"
  paymentStatus   String  @default("pending") // "pending", "paid", "failed", "refunded"
  orderStatus     String  @default("new") // "new", "confirmed", "processing", "shipped", "delivered", "cancelled"

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  items           OrderItem[]
  customOrders    CustomOrder[]

  @@index([paymentStatus])
  @@index([orderStatus])
}

model OrderItem {
  id        String  @id @default(cuid())
  orderId   String
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id])
  quantity  Int
  unitPrice Float
  size      String
  colorSetName String?

  @@index([orderId])
}

model CustomOrder {
  id              String   @id @default(cuid())
  orderId         String
  order           Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId       String
  product         Product  @relation(fields: [productId], references: [id])
  colorSetName    String
  teamName        String?

  // Logo URLs (Cloudinary)
  clubLogoUrl     String?
  sponsorLogoUrl  String?
  leagueLogoUrl   String?
  flagPatchUrl    String?

  // Logo quality notes
  logoQualityNote String? // Auto-generated note for low quality logos

  // Print config — front
  printClubLogo       Boolean @default(false) // ngực trái
  printSponsorLogo    Boolean @default(false) // giữa ngực
  printSmallNumber    Boolean @default(false) // ngực phải

  // Print config — sleeve
  printLeagueLogo     Boolean @default(false)
  printSleeveeSponsor Boolean @default(false)
  printFlagPatch      Boolean @default(false)

  // Print config — back
  printTeamName       Boolean @default(false)
  printBackNumber     Boolean @default(false)
  printPlayerName     Boolean @default(false)

  // Print config — shorts
  printShortsNumber   Boolean @default(false)
  shortsNumberSide    String? // "left", "right"

  // Generated files
  excelFileUrl        String?
  mockupFrontUrl      String?
  mockupBackUrl       String?

  playerCount         Int
  printingSurcharge   Float   @default(0)

  players             Player[]
  createdAt           DateTime @default(now())
}

model Player {
  id            String      @id @default(cuid())
  customOrderId String
  customOrder   CustomOrder @relation(fields: [customOrderId], references: [id], onDelete: Cascade)
  sortOrder     Int
  playerName    String
  playerNumber  Int
  jerseySize    String      // "S", "M", "L", "XL"
  shortsSize    String      // "S", "M", "L", "XL"

  @@index([customOrderId])
}
```

## Implementation Steps

1. `npx create-next-app@latest` with TypeScript, Tailwind, App Router, src/ directory
2. Install deps: `prisma @prisma/client better-auth zustand react-hook-form cloudinary resend exceljs`; init shadcn/ui: `npx shadcn@latest init`
3. `npx prisma init` — configure schema above
4. Setup `.env.example` with all required vars
5. Create base layout: `src/app/layout.tsx` with header (logo, nav, cart icon) + footer
6. Configure Better Auth — `src/lib/auth.ts`
7. Configure Prisma client — `src/lib/db.ts`
8. Configure Cloudinary — `src/lib/cloudinary.ts`
9. Seed script — `prisma/seed.ts` with sample products + color sets
10. Verify: `npm run dev` → homepage renders, DB connected

## Files to Create
- `src/app/layout.tsx` — root layout
- `src/app/page.tsx` — homepage
- `src/lib/db.ts` — Prisma client singleton
- `src/lib/auth.ts` — Better Auth config
- `src/lib/cloudinary.ts` — Cloudinary config
- `src/lib/constants.ts` — sizes, categories, printing fee tiers
- `prisma/schema.prisma` — full schema
- `prisma/seed.ts` — sample data
- `.env.example`

## Todo
- [ ] Create Next.js 15 project
- [ ] Install all dependencies
- [ ] Write Prisma schema
- [ ] Run initial migration
- [ ] Configure Better Auth with anonymous plugin
- [ ] Setup Cloudinary SDK
- [ ] Create base layout (header + footer + nav)
- [ ] Create seed script with 5-10 sample products
- [ ] Verify dev server runs clean

## Success Criteria
- `npm run dev` works
- `npx prisma db push` succeeds
- Seed data visible in Prisma Studio
- Base layout renders on all routes

## Risk
- Better Auth anonymous plugin API may change — pin version
- Railway PostgreSQL connection string format — test early
