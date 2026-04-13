---
name: "CSV Product Import Brainstorm"
description: "Brainstorm report cho CSV import flow quản lý sản phẩm"
type: brainstorm
date: 2026-04-13
---

# Brainstorm Report — CSV Product Import Flow

## Problem Statement
Project cần quản lý sản phẩm nhưng không muốn build manual CRUD web UI. Supplier xuất file CSV với format có thể thay đổi mỗi lần. Cần flow import linh hoạt cho admin upload CSV, map columns, preview, và bulk import sản phẩm vào database.

## Requirements

| Requirement | Decision |
|---|---|
| CSV parsing | PapaParse (client-side) |
| Column mapping | UI với auto-suggest fuzzy match |
| Unmapped fields | Hiện empty input trong preview (không ẩn) |
| Grouping logic | Client-side, group by product name (case-insensitive trim) |
| 1 CSV row = | 1 ColorSet (không phải 1 Product) |
| Duplicate handling | Tạo mới trên name collision (không merge/check) |
| Image upload | Per ColorSet trong preview, buffer locally, upload Cloudinary khi Confirm |
| Product edit | Trang riêng `/admin/products/[id]/edit` sau import |

## Evaluated Approaches

### Option A: CSV Import Wizard (Recommended)
**Pros:**
- Linh hoạt với các format CSV khác nhau
- Auto-suggest mapping giảm manual work
- Preview grouped giúp admin spot errors trước import
- Image upload per ColorSet trong preview → upload Cloudinary batch khi confirm
- Client-side grouping → no server load

**Cons:**
- Phải build UI tương đối phức tạp (3 steps: upload/mapping → preview → confirm)
- File CSV lớn có thể làm chậm client-side parsing

### Option B: Prisma Studio (Rejected)
**Pros:**
- Built-in UI
- Không cần dev effort

**Cons:**
- Không phù hợp cho bulk import
- Không có CSV parsing
- Không có column mapping
- Admin phải nhập thủ công từng sản phẩm

**Verdict:** Option A — CSV Import Wizard.

## CSV Import Flow

```
[1] Upload CSV (PapaParse — client-side)
        ↓
[2] Column Mapping UI
    CSV Col       → DB Field (auto-suggest by name fuzzy match)
    "ten_sp"      → name
    "gia_von"     → basePrice
    "mau_sac"     → colorSet.name
    "so_luong"    → stock
    "hinh_anh"    → imageUrl
    (unmapped)    → field visible with empty input
        ↓
[3] Client-side grouping by product name (case-insensitive trim)
    → ProductDraft[]: 1 Product = N ColorSets
        ↓
[4] Preview Table (grouped, all fields shown, inline editable)
    ▼ Áo CLB A  [category] [description]
      Xanh Navy | vốn:[__] | bán:[__] | stock:[__] | 🖼 [upload ảnh]
      Đỏ Trắng  | vốn:[__] | bán:[250k]| stock:[8] | 🖼 [thumb1][+]
        ↓
[5] Confirm
    → Upload imageFiles[] → Cloudinary (parallel, blob URLs locally until here)
    → POST /api/admin/products/import
    → prisma.$transaction → Product[] + ColorSet[] + ProductImage[]
    → Redirect /admin/products
```

## Data Structure

### CSV Row (Flat)
```csv
ten_sp,category,description,mau_sac,gia_von,gia_ban,so_luong,hinh_anh
"Áo CLB A","uniform-set","Áo bóng đá CLB","Xanh Navy",150000,250000,10,"url1"
"Áo CLB A","uniform-set","Áo bóng đá CLB","Đỏ Trắng",150000,250000,8,"url2"
```

### Grouped ProductDraft
```typescript
interface ProductDraft {
  name: string // "Áo CLB A"
  category: string
  description: string
  colorSets: ColorSetDraft[]
}

interface ColorSetDraft {
  name: string // "Xanh Navy"
  slug: string // "xanh-navy"
  basePrice: number
  sellPrice: number
  stock: number
  images: File[] // buffer locally until confirm
}
```

## Column Mapping Algorithm

Fuzzy match CSV header → DB field name:
```typescript
function suggestDbField(csvCol: string): string | null {
  const mappings = {
    'ten_sp': 'name',
    'ten': 'name',
    'product_name': 'name',
    'gia_von': 'basePrice',
    'cost': 'basePrice',
    'gia_ban': 'sellPrice',
    'price': 'sellPrice',
    'mau_sac': 'colorSet.name',
    'color': 'colorSet.name',
    'so_luong': 'stock',
    'quantity': 'stock',
    'hinh_anh': 'imageUrl',
    'image': 'imageUrl',
  };
  
  const normalized = csvCol.toLowerCase().replace(/[_\s]/g, '');
  return mappings[normalized] || null;
}
```

## Image Upload Strategy

**Trong preview step:**
- User upload images per ColorSet (1 hoặc nhiều ảnh theo góc: front, back, sleeve, shorts)
- Store as `File[]` in component state
- Generate blob URL for preview display

**Khi confirm:**
- Upload tất cả `File[]` lên Cloudinary song song
- Get URLs → map vào ProductImage records
- Nếu upload fail → báo lỗi, cho retry

**4 góc ảnh per ColorSet:**
- front: mặt trước
- back: mặt sau
- sleeve: tay áo
- shorts: quần

CSV mapping có thể là:
- `hinh_anh_front` → ProductImage angle='front'
- `hinh_anh_back` → ProductImage angle='back'
- `hinh_anh_sleeve` → ProductImage angle='sleeve'
- `hinh_anh_shorts` → ProductImage angle='shorts'

## Implementation Considerations

**Phase impact:**
- Phase 8: Thêm CSV import flow (3d effort)
- Phase 1: seed.ts vẫn giữ cho dev/test data, không còn primary method

**Tech stack:**
- PapaParse cho CSV parsing (client-side)
- Fuzzy string matching cho auto-suggest
- React Hook Form cho preview table state
- Cloudinary SDK cho image upload

**Performance:**
- CSV lớn (>500 rows) → consider streaming parse
- Image upload song song → Promise.all với concurrency limit

## Success Criteria
- Admin upload CSV → map columns → preview grouped products
- Admin upload images per ColorSet trong preview
- Confirm → products visible in `/admin/products` và storefront
- Admin edit product details post-import via `/admin/products/[id]/edit`

## Risks
| Risk | Level | Mitigation |
|------|-------|------------|
| Supplier CSV format varies | Medium | Column mapping UI handles this — user re-maps each import |
| Product name inconsistency causes wrong grouping | Low | Show grouped preview clearly; user can spot and edit |
| Cloudinary upload failure mid-confirm | Low | Wrap in try/catch; report failed images, allow retry |
| Large CSV (>500 rows) → slow preview | Unlikely | PapaParse streaming + virtual scroll if needed |

## Next Steps
1. Update Phase 8 plan với CSV import flow details
2. Create components: column-mapping-step, preview-table, product-group-row, colorset-row
3. Implement fuzzy match logic
4. Implement grouping logic
5. Wire up Cloudinary upload on confirm

---
*Session 2 — 2026-04-13 | Questions asked: 5*
