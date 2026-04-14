# Phase 11: Data Scraper Tool

**Context:** [Brainstorm Report](../reports/brainstorm-260414-1022-data-scraper-tool.md) | [Plan](./plan.md)

## Overview

- **Priority:** P2 (dev/demo data)
- **Status:** pending
- **Effort:** ~4d
- **Goal:** Scrape product data from 4 Vietnamese football sites → CSV → import via admin wizard

## Requirements

- Scrape: name, description, price, category, colors, sizes, front image
- Download images + upload to Cloudinary, save URL in CSV
- CSV format matches phase-08 import wizard (English column names)
- Rate limiting + stealth to avoid blocks
- CLI flags: `--site`, `--limit`, `--skip-images`, `--category`

## Architecture

```
packages/scraper/
├── src/
│   ├── core/
│   │   ├── playwright-base.ts       # Browser, rate limiter, retry, robots.txt check
│   │   ├── cloudinary-uploader.ts   # Upload buffer → URL, batch concurrency=3
│   │   ├── image-downloader.ts      # Download image → buffer, retry 3x
│   │   └── csv-writer.ts            # Append rows, finalize file
│   ├── sites/
│   │   ├── soccerstore-vn.ts        # Priority 1 — WooCommerce, cleanest
│   │   ├── sport9-vn.ts             # Priority 2
│   │   ├── beck-vn.ts               # Priority 3
│   │   └── decathlon-vn.ts          # Priority 4 — CDN, slower
│   ├── config/
│   │   └── sites.config.ts          # URLs, selectors, delay config per site
│   └── run.ts                       # CLI entry
├── package.json                     # name: @football-store/scraper
└── tsconfig.json
```

## CSV Schema (English columns)

```
name,description,price,category,colorName,primaryColor,secondaryColor,sizes,quantity,frontImageUrl
```

| Column | Type | Notes |
|--------|------|-------|
| `name` | string | Product name |
| `description` | string | Product description |
| `price` | number | Integer VND (strip đ/./,) |
| `category` | string | `jersey \| shorts \| uniform-set \| shoes \| accessory` |
| `colorName` | string | Color set name (e.g. "Navy Blue + White") |
| `primaryColor` | string | Hex if site has swatch CSS, else empty |
| `secondaryColor` | string | Hex if site has swatch CSS, else empty |
| `sizes` | string | Comma-separated: `S,M,L,XL` |
| `quantity` | number | Default 100 if not shown on site |
| `frontImageUrl` | string | Cloudinary URL (first product image) |

## TypeScript Types (packages/shared-types)

```ts
export type ProductCategory = 'jersey' | 'shorts' | 'uniform-set' | 'shoes' | 'accessory'

export interface ScrapedColorSet {
  colorName: string
  primaryColor?: string   // hex or undefined
  secondaryColor?: string
  frontImageUrl: string
}

export interface ScrapedProduct {
  name: string
  description: string
  price: number           // integer VND
  category: ProductCategory
  colorSets: ScrapedColorSet[]
  sizes: string[]         // ['S', 'M', 'L', 'XL']
}

export interface CsvRow {
  name: string
  description: string
  price: number
  category: ProductCategory
  colorName: string
  primaryColor: string
  secondaryColor: string
  sizes: string           // comma-separated: 'S,M,L,XL'
  quantity: number
  frontImageUrl: string
}

export interface SiteSelectors {
  productList: string
  productLink: string
  name: string
  description: string
  price: string
  sizes: string
  colorSwatches: string
  frontImage: string
}

export interface SiteConfig {
  baseUrl: string
  categoryUrls: string[]
  selectors: SiteSelectors
  delayMin: number        // ms
  delayMax: number        // ms
  maxRetries: number
}

export interface SiteScraper {
  listProductUrls(limit?: number): Promise<string[]>
  scrapeProduct(url: string): Promise<ScrapedProduct>
}
```

## Target Sites (5 sites)

| Site | Anti-bot | Priority | Notes |
|------|----------|----------|-------|
| soccerstore.vn | Low | 1 | WooCommerce, cleanest data |
| sport9.vn | Low | 2 | Simple VN site |
| beck.vn | Low | 3 | VN site |
| decathlon.vn | Medium | 4 | CDN, longer delays |
| ~~prodirectsport.com~~ | High | Skipped | Cloudflare — blocked |
| ~~unisportstore.com~~ | High | Skipped | Cloudflare — blocked |

## CLI (English flags)

```
pnpm scraper [options]
  --site=all|soccerstore,sport9,beck,decathlon
  --limit=N                   (default: unlimited)
  --category=jersey,shorts,uniform-set,shoes,accessory
  --output=./output/products-YYYYMMDD.csv
  --skip-images               (save original URL instead of uploading to Cloudinary)
```

Progress log: `[soccerstore] 12/50 — "Nike Dri-FIT Training Jersey"`
End summary: `✓ 45 scraped | ✗ 5 failed → packages/scraper/output/failed-urls.json`

## Implementation Steps

### Step 1 — Monorepo Package Setup (0.5d)
1. Create `packages/scraper/package.json`:
   ```json
   {
     "name": "@football-store/scraper",
     "private": true,
     "scripts": { "start": "tsx src/run.ts" },
     "dependencies": {
       "@football-store/shared-types": "workspace:*",
       "playwright-extra": "^4.x",
       "puppeteer-extra-plugin-stealth": "^2.x",
       "axios": "^1.x",
       "p-limit": "^5.x",
       "cloudinary": "^2.x"
     }
   }
   ```
2. Create `packages/scraper/tsconfig.json` extending root tsconfig
3. Add to root `package.json` scripts: `"scraper": "pnpm --filter scraper start"`
4. Add to `.gitignore`: `packages/scraper/output/`, `packages/scraper/cache/`

### Step 2 — Shared Types (0.5d)
1. Add all interfaces above to `packages/shared-types/src/index.ts`
2. `packages/scraper/src/config/sites.config.ts` — fill selectors + config for 4 sites after DOM inspection

### Step 3 — Core Utilities (1d)
1. `playwright-base.ts`:
   - `launchBrowser()`: chromium + stealth plugin, headless
   - `randomDelay(min, max)`: `await sleep(random(min, max))`
   - `retry(fn, max, backoff)`: exponential backoff 30s → 2m → 10m
   - `checkRobotsTxt(domain, path)`: fetch + parse → `allowed: boolean`
2. `image-downloader.ts`: axios GET binary, retry 3x
3. `cloudinary-uploader.ts`: `v2.uploader.upload_stream()`, batch p-limit(3)
4. `csv-writer.ts`: fs write stream, header on init, `appendRow()`, `finalize()`

### Step 4 — Site Scrapers (1.5d)
Each scraper implements `SiteScraper` interface. Priority: soccerstore → sport9 → beck → decathlon.

When scraping product:
- Normalize price: strip "đ", ".", "," → parse as integer
- Map category based on URL path or breadcrumb
- Extract hex: `page.evaluate()` → `getComputedStyle(swatchEl).backgroundColor`
- If no hex found: leave `primaryColor`/`secondaryColor` empty

### Step 5 — CLI run.ts (0.5d)
- Parse CLI args with `process.argv`
- Orchestrate: list URLs → scrape → download image → upload Cloudinary → append CSV row
- Write failed URLs to `output/failed-urls.json`

## Todo List

- [ ] Add dependencies in packages/scraper/package.json
- [ ] Setup tsconfig + .gitignore
- [ ] Add shared interfaces to packages/shared-types/src/index.ts
- [ ] Implement core/playwright-base.ts
- [ ] Implement core/image-downloader.ts
- [ ] Implement core/cloudinary-uploader.ts
- [ ] Implement core/csv-writer.ts
- [ ] Inspect soccerstore.vn DOM → fill selectors in sites.config.ts
- [ ] Implement sites/soccerstore-vn.ts
- [ ] Inspect sport9.vn DOM → fill selectors
- [ ] Implement sites/sport9-vn.ts
- [ ] Implement sites/beck-vn.ts
- [ ] Inspect decathlon.vn DOM → fill selectors
- [ ] Implement sites/decathlon-vn.ts
- [ ] Implement run.ts CLI
- [ ] Test: `pnpm scraper --site=soccerstore --limit=5`
- [ ] Verify CSV in Excel + admin import wizard

## Success Criteria

- `pnpm scraper --site=soccerstore --limit=20` runs without errors
- CSV is UTF-8, price is integer VND
- Front image displays from Cloudinary URL
- CSV imports successfully via /admin/products/import wizard
- ≥50 products from ≥2 sites

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|-----------|
| DOM selector changes | High | Log errors clearly, easy re-config in sites.config.ts |
| decathlon.vn Cloudflare tightening | Medium | Increase delays, fallback skip site |
| Cloudinary quota exceeded | Low | Free 25 credits/month ≈ 200 images, sufficient for MVP |
| Price format inconsistent | Medium | Normalize pipeline tested with fixture data |

## Security

- Do not commit `.env` with Cloudinary credentials
- SVG images from scrape must be sanitized before upload (server-side DOMPurify)
- Do not publish scraper output CSV (may contain copyrighted content)

## Dependencies

- Phase 08 CSV import wizard must be implemented before Step 5 validation
- `.env` needs: `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`
- `packages/shared-types` must be built before scraper package
