# Tech Stack Research Report: Football Uniform E-Commerce Platform

**Date:** 2026-04-12 | **Status:** Research Complete | **Scope:** Full-stack technology evaluation for Vietnamese football uniform e-commerce

---

## Executive Summary

Recommended production-ready stack for solo/small team: **Next.js 15 + Prisma + PostgreSQL + Better Auth + Resend + Cloudinary (free tier) + VNPay + Railway/Vercel combo**. Total infrastructure cost: ~$50-150/month. Development time: 4-8 weeks with proper planning.

---

## 1. Frontend Framework

### Recommendation: **Next.js 15 App Router** ✅

**Why:**
- **SEO domination**: Server Components drastically reduce JS bundle (40-60% reduction). Critical for product discovery.
- **Performance**: RSCs + Server-side rendering = lower Core Web Vitals scores = better Google ranking + faster checkout.
- **DX**: Nested layouts, intuitive routing, built-in Image optimization for product photos.
- **Ecosystem**: Massive community, production e-commerce templates available (Vercel Commerce, Medusa starters).

**Alternatives considered:**
- Remix/SvelteKit: Smaller communities, fewer e-commerce examples.
- React + Vite: No SSR by default, SEO requires extra tooling, more DevOps work.

**Production maturity:** Stable (15.2.4 as of March 2026).

**Cost:** Free (open source).

---

## 2. Database + ORM

### Recommendation: **PostgreSQL + Prisma ORM** ✅

**Why:**
- **Developer experience**: Type-safe queries, auto-generated migrations, intuitive schema syntax.
- **Reliability**: ACID transactions essential for order processing + inventory management.
- **Scale**: Handles millions of SKUs, supports complex queries needed for custom player lists.
- **Prisma Optimize**: AI-driven query analysis detects N+1 problems, slow queries.

**Schema design for your use case:**

```prisma
// Core product structure
model Product {
  id                String    @id @default(cuid())
  name              String
  description       String?
  basePrice         Float
  category          String
  imageUrl          String?   // Cloudinary URL
  sizes             Size[]    // S, M, L, XL, etc.
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  orderItems        OrderItem[]
}

model Size {
  id        String @id @default(cuid())
  name      String // "S", "M", "L", etc.
  stock     Int
  product   Product @relation(fields: [productId], references: [id])
  productId String
}

// Custom order: player list
model CustomOrder {
  id            String @id @default(cuid())
  orderId       String
  order         Order @relation(fields: [orderId], references: [id])
  playerList    PlayerListItem[] // Embedded array
  excelFileUrl  String? // Generated Excel URL
  createdAt     DateTime @default(now())
}

model PlayerListItem {
  id              String @id @default(cuid())
  playerNumber    Int
  playerName      String
  customOrderId   String
  customOrder     CustomOrder @relation(fields: [customOrderId], references: [id])
}

// Order & checkout
model Order {
  id              String @id @default(cuid())
  userId          String  // NOT NULL - mandatory login
  items           OrderItem[]
  customOrder     CustomOrder?
  paymentMethod   String // "vnpay", "momo", "cod"
  paymentStatus   String // "pending", "completed", "failed"
  orderStatus     String // "new", "processing", "shipped", "delivered"
  totalPrice      Float
  shippingAddress String
  email           String
  phone           String
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

model OrderItem {
  id        String @id @default(cuid())
  order     Order @relation(fields: [orderId], references: [id], onDelete: Cascade)
  orderId   String
  product   Product @relation(fields: [productId], references: [id])
  productId String
  quantity  Int
  price     Float // Price at time of order
  size      String
}

// User (optional auth)
model User {
  id        String @id @default(cuid())
  email     String @unique
  name      String?
  orders    Order[]
  createdAt DateTime @default(now())
}
```

**Optimization patterns:**
- Composite types for PlayerListItem (embed directly in CustomOrder) to avoid joins.
- Index on `Order.paymentStatus` for dashboard queries.
- Index on `Product.category` for listing pages.

**Cost:** Free (self-hosted on Railway/Vercel Postgres).

---

## 3. Authentication

### Recommendation: **Better Auth** ✅ (NextAuth.js is EOL for new projects)

**Why:**
- **Active development**: Auth.js team now maintains Better Auth (NextAuth v5+ is legacy).
- **MFA out-of-box**: Email 2FA, passwordless magic links (important for e-commerce trust).
- **Simple setup**: 5-line integration vs NextAuth's 30-line boilerplate.
- **Email + password**: Built-in emailAndPassword plugin for mandatory login.

**Implementation pattern for mandatory login:**

```typescript
// Mandatory login before checkout
// User must register/login → session created
// On checkout: create Order linked to userId
// Better Auth emailAndPassword + admin plugins

betterAuth({
  plugins: [
    emailAndPassword({ requireEmailVerification: false }),
    admin()
  ]
})
```

**Alternative (NextAuth.js):**
- Requires manual JWT + cookie handling.
- Mature but no longer actively developed for new features.

**Cost:** Free (open source).

---

## 4. File Upload & Image Optimization

### Recommendation: **Cloudinary (free tier) → S3 (production)** ✅

**Cost breakdown:**

| Service | Storage | Bandwidth | Use Case |
|---------|---------|-----------|----------|
| **Cloudinary Free** | 25GB | 25GB/month | MVP/testing |
| **Cloudinary Basic** | $0.37/GB | Auto-optimized CDN | First 12-18 months with <100k visitors |
| **AWS S3** | $0.023/GB | $0.085/GB | Infinite scale, cheaper after 100k visitors |

**Why Cloudinary free tier for start:**
- Automatic resize/optimization on upload (critical for product photos).
- Smart crop for thumbnails (good UX).
- No DevOps: set environment variable, done.

**Logo upload flow:**
1. User uploads logo (React component).
2. Cloudinary resizes to 500x500px, WEBP compression.
3. Returns CDN URL, store in `Product.imageUrl`.
4. Total: 15 minutes setup, validation included.

**When to migrate to S3:**
- Hitting 25GB storage or 100k+ monthly visitors.
- Cost becomes 10-20x cheaper than Cloudinary.

**Cost:** Free → $0 (infrastructure) + $50-100/month bandwidth at scale.

---

## 5. Email Service

### Recommendation: **Resend** ✅ (or Nodemailer for cost-sensitive)

**Comparison:**

| Feature | Resend | SendGrid | Nodemailer |
|---------|--------|----------|-----------|
| **Price** | $20/10k emails | Free (100/day) then $ | Free |
| **Setup time** | 5 mins | 15 mins | 30 mins (SMTP) |
| **Deliverability** | 99.5% | 99.9% | 90-95% |
| **Webhooks** | Built-in | Yes | No |
| **DX** | Excellent | Complex UI | Code-heavy |

**Recommendation logic:**
- **Start:** Resend ($20/month budget-friendly).
- **Scale (10k+/month emails):** SendGrid (dedicated IPs, ISP monitoring).
- **Cheapest:** Nodemailer + AWS SES ($0.10 per 1k emails), but requires SMTP troubleshooting.

**Implementation for order confirmation + Excel:**
```typescript
// Resend API call
await resend.emails.send({
  from: "orders@uniformstore.vn",
  to: customer.email,
  subject: "Order Confirmation",
  html: template, // Use React Email library
  attachments: [{
    filename: "player-list.xlsx",
    content: excelBuffer, // from ExcelJS
  }]
});
```

**Cost:** $20/month (Resend) or free (Nodemailer + SES at $10/month).

---

## 6. Excel Generation

### Recommendation: **ExcelJS** ✅

**Why:**
- Professional formatting (colors, borders, merged cells) for player lists.
- Streaming support for large files (1000+ rows).
- Active maintenance, TypeScript support.
- Can generate & email in same request.

**Comparison:**

| Library | Write | Read | Styling | Streaming | Bundle Size |
|---------|-------|------|---------|-----------|-------------|
| **ExcelJS** | ✅ | ✅ | Rich | ✅ | 500KB |
| **xlsx** | ✅ | ✅ | Basic | ❌ | 800KB |
| **excel4node** | ✅ | ❌ | Good | ❌ | 400KB |

**Player list Excel template:**
```typescript
const workbook = new ExcelJS.Workbook();
const sheet = workbook.addWorksheet("Player List");

sheet.columns = [
  { header: "Number", key: "playerNumber", width: 10 },
  { header: "Player Name", key: "playerName", width: 30 },
];

// Add header styling
sheet.getRow(1).font = { bold: true, color: { argb: "FFFFFFFF" } };
sheet.getRow(1).fill = { type: "pattern", pattern: "solid", fgColor: { argb: "FF0070C0" } };

// Add data rows
customOrder.playerList.forEach(p => {
  sheet.addRow({ playerNumber: p.playerNumber, playerName: p.playerName });
});

const buffer = await workbook.xlsx.writeBuffer();
// Email this buffer as attachment
```

**Cost:** Free (open source npm package).

---

## 7. Payment Gateway (Vietnam)

### Recommendation: **VNPay + MoMo + COD (fallback)** ✅

**Market context:**
- VNPay: ~45% market share, best QR code integration, supports e-wallet redirect.
- MoMo: ~30% market share, fastest growing, excellent API.
- COD: Still ~25% of online orders in Vietnam (important for trust).

**Integration approach:**

```typescript
// Payment method selection at checkout
enum PaymentMethod {
  VNPAY = "vnpay",
  MOMO = "momo",
  COD = "cod"
}

// VNPay flow
if (paymentMethod === "vnpay") {
  const vnpUrl = createVNPayUrl({
    amount: order.totalPrice,
    orderId: order.id,
    returnUrl: "https://yoursite.com/payment-callback"
  });
  redirect(vnpUrl); // User enters bank details at VNPay portal
}

// MoMo flow
if (paymentMethod === "momo") {
  const momoResponse = await createMomoPayment({
    amount: order.totalPrice,
    orderId: order.id
  });
  redirect(momoResponse.paymentUrl); // User confirms in MoMo app
}

// COD: just mark order as "pending_payment"
if (paymentMethod === "cod") {
  order.paymentStatus = "pending_payment";
  order.paymentMethod = "cod";
  await db.order.update(order);
  // Email confirmation, await payment on delivery
}
```

**Libraries:**
- **VNPay**: `vnpay` npm package (open-source, simple API).
- **MoMo**: Official `momo-api` (well-documented).
- Both support webhooks for payment confirmation.

**Cost:** 
- VNPay: 0.5-1.5% transaction fee (merchant account required).
- MoMo: 1% transaction fee.
- COD: No fee (payment collected offline).

**Resources:**
- VNPay Docs: https://vnpay.js.org/en/
- MoMo Docs: https://developers.momo.vn/v3/

---

## 8. Deployment

### Recommendation: **Vercel (frontend) + Railway (database/backend)** ✅

**Reasoning:**

| Aspect | Vercel | Railway | VPS |
|--------|--------|---------|-----|
| **Next.js native** | Perfect fit | Good | Manual |
| **Database included** | Via Postgres plugin | Native PostgreSQL | Manual setup |
| **Cold starts** | Minimal (edge) | 0ms (always warm) | N/A |
| **Scaling** | Auto | Auto | Manual |
| **Cost ($)** | 0-50/mo | 10-50/mo | 9-30/mo |
| **For solo dev** | Best | Best | If DevOps comfortable |

**Recommended setup:**

```
Frontend: Vercel
- Next.js 15 app
- Auto-deploys on git push
- Free tier works for <100k req/month
- Cost: $0-20/month

Database: Railway PostgreSQL
- Managed PostgreSQL instance
- Automatic backups, monitoring
- Easy Prisma integration
- Cost: ~$10/month base + usage

Backend: Same Vercel deployment
- API routes in /app/api/*
- Serverless functions handle:
  - Payment webhooks (VNPay/MoMo)
  - Excel generation
  - Email sending
- No extra cost

Alternative: Vercel + Vercel Postgres (if want single platform)
- Cost: +$15/month (expensive for DB)
- Fewer features than Railway
```

**Why NOT pure VPS:**
- You (solo dev) manage all DevOps: SSL, backups, upgrades, scaling.
- Not suitable for 4-8 week timeline.
- Better once you need extreme cost optimization (100k+ monthly orders).

**GitHub integration:**
```bash
# 1. Git push to main branch
# 2. Vercel auto-builds + deploys
# 3. Railway auto-runs database migrations
# 4. Zero-downtime deployment
```

**Cost breakdown (monthly):**
- Vercel: $0-20
- Railway PostgreSQL: $10-15
- Cloudinary: $0 (free tier) → $20-50 at scale
- Resend email: $20
- VNPay/MoMo: transaction fees only (0.5-1.5%)
- **Total: $30-55/month** (beautiful for MVP)

---

## Architecture Diagram (High-Level)

```
┌─────────────────────────────────────────────────────────┐
│                    BROWSER (User)                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────┐
│         VERCEL (Next.js 15 + API Routes)                │
│  ┌──────────────────┐  ┌──────────────────┐             │
│  │  App Router      │  │  API Routes      │             │
│  │  (RSCs)          │  │  - /checkout     │             │
│  │  - Products      │  │  - /payment      │             │
│  │  - Cart          │  │  - /webhooks     │             │
│  │  - Checkout      │  │  - /email        │             │
│  └──────────────────┘  └──────────────────┘             │
└─────────┬──────────────────────┬────────────────────────┘
          │                      │
    ┌─────▼──────┐         ┌─────▼──────┐
    │  RAILWAY   │         │ CLOUDINARY │
    │            │         │   (Images) │
    │ PostgreSQL │         └────────────┘
    │  + Prisma  │
    └────────────┘
         │
    ┌────▼────────────────────────────┐
    │   Data Models (Prisma)           │
    │  - Products + Sizes              │
    │  - Orders + OrderItems           │
    │  - CustomOrders + PlayerLists    │
    │  - Users (Better Auth)           │
    └─────────────────────────────────┘
```

**External services:**
- **Better Auth**: Email + password authentication, stored in PostgreSQL.
- **Resend**: Email API (order confirmations + Excel attachments).
- **ExcelJS**: Server-side generation (no external call needed).

---

## Implementation Timeline (Solo/Small Team)

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| **Setup** | Days 1-2 | Next.js + Prisma + PostgreSQL + Better Auth |
| **Products** | Days 3-4 | Product listing, search, categories |
| **Cart + Checkout** | Days 5-6 | Mandatory login, Better Auth integration |
| **Payment** | Days 7 | COD integration (VNPay/MoMo deferred to Phase 2) |
| **Custom Orders** | Days 8-10 | Player list form + Excel generation |
| **Email** | Days 11 | Order confirmation + Excel attachment via Resend |
| **Deployment** | Days 12-13 | Vercel + Railway production setup |
| **Testing** | Days 14-15 | E2E tests, UAT |

**Total: 2-3 weeks core development + 1 week polish = ~1 month MVP.**

---

## Cost Summary (First Year)

| Component | Setup | Monthly | Annual |
|-----------|-------|---------|--------|
| **Domain** | $0-10 | $10-12 | $120 |
| **Vercel** | Free | $5-20 | $60-240 |
| **Railway PostgreSQL** | Free | $10-15 | $120-180 |
| **Cloudinary** | Free | $0-50 | $0-600 |
| **Resend Email** | Free | $20 | $240 |
| **VNPay/MoMo** | ~$200 merchant setup | 0% (fees only) | $0 |
| **Development Tools** | - | $0-40 | $0-480 |
| **Hosting Total** | ~$200 | $45-117 | $540-1,860 |

**Minimum viable:** $540/year (domain + Vercel + Railway + Resend).  
**Comfortable production:** $1,200-1,800/year.

---

## Key Decisions Summary

| Decision | Choice | Why |
|----------|--------|-----|
| Framework | Next.js 15 | Best SEO + performance for e-commerce |
| Database | PostgreSQL + Prisma | Type-safe, scales, excellent DX |
| Auth | Better Auth | Email + password, active development |
| Images | Cloudinary free → S3 | Free start, no DevOps, upgradeable |
| Email | Resend | Clean API, great DX, affordable |
| Excel | ExcelJS | Professional formatting, streaming |
| Payment | COD only (MVP) | Simple for MVP, VNPay/MoMo deferred to Phase 2 |
| Hosting | Vercel + Railway | Zero DevOps, auto-scaling, solo dev friendly |

---

## Unresolved Questions

1. **Order shipping management**: Should integrate with Vietnam logistics APIs (Giao Hàng Nhanh/ViettelPost) for auto label generation?
2. **Inventory sync**: Real-time stock across multiple warehouses (Vietnam + overseas)?
3. **Admin dashboard**: Custom Next.js admin panel (email notifications + dashboard for order management)
4. **Multi-language**: Vietnamese + English from start, or Vietnamese-only MVP?
5. **SEO content**: Product descriptions auto-translated or manually curated?
6. **Warranty/returns**: Complex return flow for customized jerseys?
7. **Analytics**: Plausible/Mixpanel integration for tracking conversion funnel?

---

## Sources

- [Next.js 15 E-Commerce Guide](https://www.elevaseo.com/en/blog/headless/nextjs-ecommerce-guide)
- [Next.js Commerce Templates](https://vercel.com/templates/next.js/nextjs-commerce)
- [Prisma ORM Documentation](https://www.prisma.io/)
- [Better Auth Documentation](https://better-auth.com/docs)
- [Better Auth vs NextAuth Comparison](https://indie-starter.dev/blog/next-auth-js-vs-better-auth)
- [Cloudinary vs S3 Cost Analysis](https://cloudinary.com/guides/ecosystems/cloudinary-vs-s3)
- [Resend vs SendGrid Comparison](https://forwardemail.net/en/blog/resend-vs-sendgrid-email-service-comparison)
- [ExcelJS Documentation](https://www.npmjs.com/package/exceljs)
- [VNPay Integration](https://vnpay.js.org/en/)
- [MoMo Developer API](https://developers.momo.vn/v3/)
- [Vercel vs Railway Deployment](https://designrevision.com/blog/vercel-vs-railway)
- [Prisma Query Optimization](https://www.prisma.io/docs/orm/prisma-client/queries/query-optimization-performance)
- [Solopreneur Tech Stack 2026](https://prometai.app/blog/solopreneur-tech-stack-2026)
- [E-Commerce Tech Stack for Startups](https://wearebrain.com/blog/best-ecommerce-tech-stack-for-startups-2026/)

