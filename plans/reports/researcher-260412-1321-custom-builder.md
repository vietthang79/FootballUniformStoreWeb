# Custom Mockup Builder Research Report: Real-time Positioning

**Date:** April 12, 2026  
**Focus:** Click-thả tự do vào vị trí bất kỳ trên sản phẩm thật  
**Principle:** Real-time positioning + Unlimited positions + 360° preview

---

## 1. 2D MOCKUP OVERLAY: TEXT + LOGO ON PRODUCT IMAGE

### Comparison Matrix

| Approach | Ease | Performance | Preview Share | Maintenance | Best For |
|----------|------|-------------|----------------|-------------|----------|
| **CSS + DOM** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ | ⭐⭐⭐⭐ | Live editing |
| **HTML Canvas** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ | ⭐⭐⭐ | Static export |
| **Fabric.js** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ | ⭐⭐⭐⭐ | Design editor |
| **Konva.js** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ | ⭐⭐⭐ | Complex effects |

### Detailed Analysis

#### CSS + Click-thả Real-time (Recommended for MVP)
**Why:** GPU-accelerated real-time positioning, unlimited positions, no dependencies.

**Implementation:**
- Click-thả để thêm text/logo vào bất kỳ vị trí nào
- Drag & drop để di chuyển elements
- Resize & rotate tự do cho từng element
- Z-index layering cho multiple elements
- Real-time preview với GPU acceleration

**Trade-offs:**
- Không có interactive rotation (nâng cấp sau nếu cần)
- Export image cần Canvas (Phase 5)
- Responsive scaling cần JavaScript tính toán

**Use When:** Custom mockup với vị trí hoàn toàn tự do là yêu cầu chính.

---

#### HTML Canvas (Best for Export/Share)
**Why:** Produces static image file for sharing; good performance for single render.

**Implementation:**
```
1. Load product image via Image() object
2. Draw on canvas: image → text → logo (layer order)
3. Use drawImage() for logo, fillText() for player name/number
4. Export: canvas.toBlob() → download or POST to server
```

**Performance:** Single render ~50-100ms for 600×800 mockup.

**Advantages:**
- Shareable image (Facebook/email preview, Open Graph)
- Full control over rendering order
- No external dependencies

**Limitations:**
- Not interactive (no real-time edit preview)
- Lossy resizing if logo needs to fit different uniform sizes
- Text positioning requires pixel math (manual calculation)

**Use When:** Need to export customized uniform as static image before checkout.

---

#### Fabric.js (Best for Design Editor Feel)
**Why:** Interactive + batteries-included manipulation (drag, resize, rotate text/logo).

**Performance Metrics:**
- Moderate interactive overhead; 30–45 FPS with 100+ objects
- Built-in selection/transform UI reduces code

**Key Features:**
- Drag/drop/resize text and logo interactively
- SVG import (for company logos with vector quality)
- Text styling: fonts, colors, shadows
- Export to PNG/SVG

**Trade-offs:**
- Bundle size: 44.34 kB (gzipped)
- More setup than Canvas
- Overkill for simple name/number only

**Use When:** Builder needs interactive text positioning + logo rotation.

---

#### Konva.js (For Performance-Heavy Scenes)
**Why:** 60 FPS with 1000+ objects; superior to Fabric.js under load.

**Performance Advantage:**
- Selective rendering (only changed areas redraw)
- Layer-based optimization
- Better on mobile

**Best For:** Multi-team uniforms with many text overlays; squad builder (10+ players at once).

**Cost:** More setup, steeper learning curve than Fabric.

---

### Recommendation for Football Uniform Custom Mockup

**MVP Phase:** CSS click-thã + 360° preview + Canvas export
- CSS cho real-time positioning (click-thã tu do)
- 360° preview cho moi goc do san pham
- Canvas cho export/share (final mockup)
- 0 external dependencies, ~200 lines of code

**Key Focus:** Custom mockup voi vi tri hoan toan tu do, khong gioi han vi tri co dinh

**Growth Phase:** Upgrade to Fabric.js neu users request:
- Interactive rotation controls
- Advanced text effects (shadows, gradients)
- Vector logo support

**Don't Use:** Konva.js (overkill tru neu scale len 50+ simultaneous editors).

---

## 2. PASTE EXCEL DATA INTO HTML TABLE

### Recommended Approach: Custom Implementation

**Why:** Straightforward clipboard API; no table library overhead needed.

### Implementation Steps

```javascript
// Listen for paste on table container
table.addEventListener('paste', async (e) => {
  e.preventDefault();
  
  // Get TSV from clipboard
  const text = e.clipboardData.getData('text/plain');
  const rows = text.trim().split('\n');
  
  // Parse tab-separated values
  const data = rows.map(row => row.split('\t'));
  
  // Insert into table DOM (or update form fields)
  populateTable(data);
});
```

**Why This Works:**
- Excel/Sheets auto-convert to TSV on copy (tabs = columns)
- Clipboard API supported in all modern browsers
- ~20 lines of code

### Library Comparison

| Library | Use Case | Size | Overhead |
|---------|----------|------|----------|
| **Custom** | Player list paste | ~30 lines | None |
| **TanStack Table** | Complex table UI + sorting/filtering | 15.2 kB | Requires UI building |
| **AG Grid** | Enterprise Excel-like grid | 298 kB | Overkill; costs $5k/year |
| **Tabulator** | Built-in paste parser | ~50 kB | Good middle ground |
| **Handsontable** | True Excel-in-browser | 200+ kB | Only if formulas needed |

### When to Use Each

**Custom (Recommended):** Simple player list (name, number, size) → straight to order form.
- "Paste roster from Excel, then add numbers/sizes"
- Minimal validation needed

**Tabulator:** If need visible editable grid with validation UI.
- `clipboardPasteParser` can customize parsing
- Built-in paste event handling

**Skip AG Grid/Handsontable:** Unless client specifically requests spreadsheet-like interaction (formulas, cell validation dropdowns). Cost/complexity not justified for simple "paste name+number" workflow.

### Parsing Multi-Format Paste

Excel can paste as:
1. **text/plain** (TSV) — preferred, always works
2. **text/html** (table HTML) — fallback for complex formatting
3. **text/rtf** (rich text) — ignore, use SheetJS if needed

```javascript
const clipboardData = e.clipboardData;

// Try plain text first (fastest)
let text = clipboardData.getData('text/plain');
if (!text) {
  // Fallback to HTML if available
  const html = clipboardData.getData('text/html');
  text = parseHTMLTable(html);
}
```

### Validation & Error Handling

After paste, validate before inserting:
```javascript
const isValidRow = (row) => {
  // row[0] = name, row[1] = number, row[2] = size
  return row[0]?.trim() && !isNaN(row[1]) && ['XS','S','M','L','XL'].includes(row[2]);
};

const validRows = data.filter(isValidRow);
if (data.length !== validRows.length) {
  alert(`${data.length - validRows.length} rows had invalid data`);
}
```

---

## 3. IMAGE UPLOAD + VALIDATION

### Client-Side Validation Checklist

✅ **Dimension Check** (before upload)
✅ **Preview** (show user before submit)
✅ **Compression** (reduce file size, optional)
✅ **Format Validation** (JPEG/PNG only)

### Implementation

```javascript
// 1. Validate dimensions
const validateImageDimensions = (file) => {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        const isValid = img.width >= 500 && img.height >= 500;
        resolve({
          width: img.width,
          height: img.height,
          isValid,
          aspectRatio: (img.width / img.height).toFixed(2)
        });
      };
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  });
};

// 2. Optional: Compress before upload
const compressImage = async (file, maxWidth = 800, maxHeight = 600) => {
  const canvas = document.createElement('canvas');
  const img = new Image();
  
  img.onload = () => {
    let width = img.width;
    let height = img.height;
    
    // Calculate scaled dimensions maintaining aspect ratio
    if (width > maxWidth) {
      height = (height * maxWidth) / width;
      width = maxWidth;
    }
    if (height > maxHeight) {
      width = (width * maxHeight) / height;
      height = maxHeight;
    }
    
    canvas.width = width;
    canvas.height = height;
    canvas.getContext('2d').drawImage(img, 0, 0, width, height);
    
    // Return blob
    canvas.toBlob((blob) => {
      console.log(`Compressed: ${file.size} → ${blob.size} bytes`);
    });
  };
  
  img.src = URL.createObjectURL(file);
};
```

### When to Compress

| Scenario | Compress? | Quality Impact |
|----------|-----------|----------------|
| Logo (vector) | No | Canvas blur = unacceptable |
| Team photo | Yes | Acceptable for preview |
| Player headshots | Maybe | Depends on print quality |
| Social preview | Yes | 500×500 sufficient |

**⚠️ Warning:** Canvas resizing is **lossy**. Use only for non-critical assets (previews, thumbnails). For print-quality logos, skip compression.

### Library Alternative: Compressor.js

If compression needed:
```javascript
import Compressor from 'compressorjs';

new Compressor(file, {
  quality: 0.8,
  maxWidth: 800,
  maxHeight: 600,
  success(result) {
    console.log('Compressed:', result); // Blob
  }
});
```

**Pros:** Handles quality automatically.  
**Cons:** Added dependency (15 kB).

### Recommended Flow

1. User uploads logo
2. Validate dimensions → show error if <500×500
3. Preview before adding to cart (no compression)
4. Store original file in cart
5. On server: resize for different uniform sizes if needed

---

## 4. CART DATA STRUCTURE & PERSISTENCE

### State Management Stack: Zustand + localStorage

**Why Zustand:** Lightweight (1.16 kB), zero boilerplate, persist middleware built-in.

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CustomOrder {
  id: string;
  teamName: string;
  players: {
    name: string;
    number: number;
    size: 'XS' | 'S' | 'M' | 'L' | 'XL';
  }[];
  mockupDesign: {
    logoUrl: string; // CloudFlare R2 URL or base64
    logoX: number;
    logoY: number;
    logoScale: number;
    textOverlays: {
      position: 'chest_left' | 'chest_center' | 'back_top' | 'back_number' | 'sleeve_left' | 'sleeve_right';
      text: string;
      font: string;
      color: string;
    }[];
  };
  uniforms: {
    style: 'jersey' | 'shorts' | 'combo';
    quantity: number;
  };
  createdAt: string;
}

interface CartStore {
  items: CustomOrder[];
  addOrder: (order: CustomOrder) => void;
  removeOrder: (id: string) => void;
  updateOrder: (id: string, order: Partial<CustomOrder>) => void;
  clearCart: () => void;
}

const useCartStore = create<CartStore>()(
  persist(
    (set) => ({
      items: [],
      
      addOrder: (order) => set((state) => ({
        items: [...state.items, { ...order, id: Date.now().toString() }]
      })),
      
      removeOrder: (id) => set((state) => ({
        items: state.items.filter(item => item.id !== id)
      })),
      
      updateOrder: (id, updates) => set((state) => ({
        items: state.items.map(item =>
          item.id === id ? { ...item, ...updates } : item
        )
      })),
      
      clearCart: () => set({ items: [] })
    }),
    {
      name: 'football-uniform-cart', // localStorage key
      storage: localStorage,
      partialize: (state) => ({
        items: state.items.map(item => ({
          ...item,
          // Exclude non-serializable fields
          // logoUrl handled separately
        }))
      })
    }
  )
);
```

### Serialization for Complex Types

**Problem:** Image URLs or File objects can't JSON.stringify.

**Solutions:**

1. **Store URLs only** (recommended)
   ```typescript
   logoUrl: 'https://r2.example.com/logo-123.png' // persists fine
   ```

2. **Use base64 for small images**
   ```typescript
   const toBase64 = (file: File) => {
     return new Promise((resolve) => {
       const reader = new FileReader();
       reader.onload = () => resolve(reader.result);
       reader.readAsDataURL(file);
     });
   };
   
   // Store as string
   logoUrl: 'data:image/png;base64,iVBOR...'
   ```
   **⚠️ Warning:** Base64 = 33% larger, avoid for images >100kB.

3. **For complex types (Map, Set, Date):** Use superjson
   ```typescript
   import superjson from 'superjson';
   
   {
     storage: {
       getItem: (key) => {
         const item = localStorage.getItem(key);
         return item ? superjson.parse(item) : null;
       },
       setItem: (key, value) => {
         localStorage.setItem(key, superjson.stringify(value));
       }
     }
   }
   ```

### Redux vs Zustand vs Context API (2026 Analysis)

| Solution | Cart Use Case | Pros | Cons |
|----------|---------------|------|------|
| **Zustand** | ✅ Best choice | Lightweight, persist middleware, zero boilerplate | Limited DevTools |
| **Redux (RTK)** | ⚠️ If team needs it | Powerful middleware ecosystem, time-travel debug | 40 kB overhead, setup cost |
| **Context API** | ❌ Don't use | Built-in to React | Re-renders entire app on cart change |

**Zustand Persistence Middleware Details:**
```typescript
persist(store, {
  name: 'store-key',
  storage: localStorage,
  version: 1, // for migrations
  migrate: (persistedState, version) => {
    if (version === 0) {
      // Upgrade old schema
      return { ...persistedState, newField: 'default' };
    }
    return persistedState;
  },
  partialize: (state) => ({
    // Only persist certain fields
    items: state.items,
    // Don't persist: UI state, temporary values
  })
});
```

### Data Persistence Strategy

**On Browser Reload:**
1. Zustand loads from localStorage automatically
2. Cart items restored within 10ms
3. No need for splash screen or loading delay

**On Server:**
Send cart to `/api/orders` only on checkout:
```typescript
const checkout = async () => {
  const items = useCartStore((state) => state.items);
  const response = await fetch('/api/orders', {
    method: 'POST',
    body: JSON.stringify({ items, timestamp: Date.now() }),
    headers: { 'Content-Type': 'application/json' }
  });
  // Clear cart after successful order
  useCartStore.setState({ items: [] });
};
```

---

## 5. FORM STATE MANAGEMENT: MULTI-STEP BUILDER

### Recommendation: React Hook Form

**Why:** Superior performance, smaller bundle, active maintenance.

| Metric | React Hook Form | Formik |
|--------|-----------------|--------|
| Bundle Size | 12.12 kB | 44.34 kB |
| Dependencies | 0 | 7 |
| Last Commit | Active (2026) | 1 year ago |
| Re-render Count | Minimal | Higher in large forms |

### Multi-Step Builder Implementation

```typescript
import { useForm, FormProvider, useFormContext } from 'react-hook-form';

type UniformBuilderForm = {
  step1_team: {
    teamName: string;
    sport: string;
  };
  step2_design: {
    colorPrimary: string;
    colorSecondary: string;
    logoFile?: FileList;
  };
  step3_players: {
    playerCount: number;
    players: { name: string; number: number; size: string }[];
  };
};

export function UniformBuilder() {
  const methods = useForm<UniformBuilderForm>({
    mode: 'onChange',
    defaultValues: {
      step1_team: { teamName: '', sport: 'football' },
      step2_design: { colorPrimary: '#000', colorSecondary: '#fff' },
      step3_players: { playerCount: 0, players: [] }
    }
  });
  
  const [currentStep, setCurrentStep] = useState(1);
  
  const onSubmit = (data) => {
    if (currentStep < 3) {
      setCurrentStep(currentStep + 1);
    } else {
      // Send to cart
      useCartStore.getState().addOrder({
        teamName: data.step1_team.teamName,
        design: { colorPrimary: data.step2_design.colorPrimary, ... },
        players: data.step3_players.players,
        ...
      });
    }
  };
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        {currentStep === 1 && <Step1TeamInfo />}
        {currentStep === 2 && <Step2Design />}
        {currentStep === 3 && <Step3Players />}
        
        <button type="submit">
          {currentStep < 3 ? 'Next' : 'Add to Cart'}
        </button>
      </form>
    </FormProvider>
  );
}

// Reusable step component
function Step1TeamInfo() {
  const { register, formState: { errors } } = useFormContext<UniformBuilderForm>();
  
  return (
    <fieldset>
      <input
        {...register('step1_team.teamName', {
          required: 'Team name is required',
          minLength: { value: 3, message: 'Min 3 chars' }
        })}
        placeholder="Team Name"
      />
      {errors.step1_team?.teamName && (
        <span>{errors.step1_team.teamName.message}</span>
      )}
    </fieldset>
  );
}
```

### Key Advantages for Multi-Step Forms

1. **Automatic Field State Tracking**
   - No manual state management per field
   - Validation errors from schema

2. **useFieldArray for Dynamic Players**
   ```typescript
   const { fields, append, remove } = useFieldArray({
     control,
     name: 'step3_players.players'
   });
   
   return (
     <>
       {fields.map((field, index) => (
         <div key={field.id}>
           <input {...register(`step3_players.players.${index}.name`)} />
           <button onClick={() => remove(index)}>Remove</button>
         </div>
       ))}
       <button onClick={() => append({ name: '', number: 0, size: 'M' })}>
         Add Player
       </button>
     </>
   );
   ```

3. **Integration with Zustand Cart**
   ```typescript
   const onFinalSubmit = async (data) => {
     const order = transformFormDataToOrder(data);
     useCartStore.getState().addOrder(order);
     navigate('/checkout');
   };
   ```

### When NOT to Use Formik

- Formik not actively maintained (last commit 1 year ago)
- Larger bundle (44 kB vs 12 kB)
- More re-renders in large forms
- More boilerplate for multi-step (FormikStep wrapper needed)

---

## 6. REAL-TIME PREVIEW UPDATE: DEBOUNCE + CANVAS/CSS

### Challenge
User types player name → canvas/CSS updates → should feel responsive but not thrash browser.

### Solution: Debounced Canvas/CSS Update

```typescript
import { useCallback, useRef, useEffect } from 'react';

function UniformPreview() {
  const [playerName, setPlayerName] = useState('');
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const debounceTimerRef = useRef<NodeJS.Timeout | null>(null);
  
  // Debounced redraw
  const redrawCanvas = useCallback(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d')!;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Draw uniform background
    ctx.drawImage(uniformImage, 0, 0);
    
    // Draw player name
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 24px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(playerName, canvas.width / 2, 100);
  }, [playerName]);
  
  // Debounce: delay redraw until user stops typing
  const debouncedRedraw = useCallback(() => {
    if (debounceTimerRef.current) {
      clearTimeout(debounceTimerRef.current);
    }
    
    debounceTimerRef.current = setTimeout(() => {
      redrawCanvas();
    }, 300); // Wait 300ms after last keystroke
  }, [redrawCanvas]);
  
  const handleNameChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setPlayerName(e.target.value);
    debouncedRedraw(); // Queue redraw
  };
  
  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (debounceTimerRef.current) {
        clearTimeout(debounceTimerRef.current);
      }
    };
  }, []);
  
  return (
    <>
      <input
        value={playerName}
        onChange={handleNameChange}
        placeholder="Enter player name"
      />
      <canvas ref={canvasRef} width={400} height={600} />
    </>
  );
}
```

### Alternative: Lazy Custom Hook

```typescript
// useDebounce.ts
export function useDebounce<T>(value: T, delay: number) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage in component
function UniformPreview() {
  const [playerName, setPlayerName] = useState('');
  const debouncedName = useDebounce(playerName, 300);
  
  useEffect(() => {
    // Redraw only when debounced value changes
    redrawCanvas(debouncedName);
  }, [debouncedName]);
  
  return <input onChange={(e) => setPlayerName(e.target.value)} />;
}
```

### CSS Overlay (Simpler, Faster)

If using CSS instead of Canvas:
```typescript
// CSS updates are GPU-accelerated, no debounce needed
<div className="uniform-preview">
  <img src="uniform.png" alt="uniform" />
  <div className="player-name" style={{ opacity: playerName ? 1 : 0 }}>
    {playerName}
  </div>
</div>

// CSS
.player-name {
  position: absolute;
  top: 100px;
  left: 50%;
  transform: translateX(-50%); // GPU accelerated
  font-size: 24px;
  font-weight: bold;
  color: white;
  transition: opacity 0.2s;
}
```

### Performance Comparison

| Approach | Delay | Re-render Count | FPS | Best For |
|----------|-------|-----------------|-----|----------|
| No debounce | 0 ms | Per keystroke | 30 FPS | Small previews |
| 300ms debounce | 300 ms | Per name | 60 FPS | Canvas-heavy |
| CSS overlay | <1 ms | Per keystroke | 60 FPS | Live preview |

**Recommendation:** CSS overlay (no debounce) for MVP; debounce Canvas only if preview gets slow (1000+ objects).

---

## TECHNICAL STACK SUMMARY

### Frontend Architecture

```
React 18
├── React Hook Form (form state) → 12 kB
├── Zustand (cart state) → 1 kB
├── Context API (theme/auth only)
├── Canvas/CSS (2D mockup overlay)
└── Custom Clipboard Handler (paste data)

No: Redux, Formik, Fabric.js (MVP), Handsontable
```

### Specific Library Choices

| Feature | Library | Size | Why |
|---------|---------|------|-----|
| Form builder | React Hook Form | 12 kB | Active, performant, zero deps |
| Cart state | Zustand + persist | 1 kB | Lightweight, localStorage built-in |
| 2D overlay | CSS + Canvas | 0 kB | No deps; upgrade to Fabric if interactive |
| Paste handler | Native API | 0 kB | Built into browsers, 30 lines of code |
| Image validate | Native API | 0 kB | FileReader + Image object |

**Total External Deps:** 2 (React Hook Form + Zustand = ~13 kB gzipped)

---

## IMPLEMENTATION PRIORITY

### Phase 1 (Week 1-2): MVP Mockup
- [ ] CSS overlay preview (live)
- [ ] HTML Canvas export (static image)
- [ ] React Hook Form multi-step
- [ ] Zustand cart + localStorage
- [ ] Native paste handler
- [ ] Image upload + dimension validation

### Phase 2 (Week 3): Polish
- [ ] Debounce preview updates
- [ ] Error handling for invalid pastes
- [ ] Compress image on upload (optional)
- [ ] Unit tests

### Phase 3 (Week 4+): Growth
- [ ] Upgrade CSS/Canvas to Fabric.js (if users request interactive positioning)
- [ ] Add batch operations (paste 10 players at once)
- [ ] Social sharing (meta tags for Open Graph)

---

## LOGO QUALITY HANDLING

### Business Requirement
- Accept low-quality logos (customer experience priority)
- Auto-generate note: "nêu dâ thì shop làm nét lai (nêu co the)"
- No rejection of orders due to logo quality

### Technical Implementation

```typescript
// Logo quality detection
function detectLogoQuality(file: File): {
  isLowQuality: boolean;
  note?: string;
} {
  const img = new Image();
  img.src = URL.createObjectURL(file);
  
  return new Promise(resolve => {
    img.onload = () => {
      const { width, height } = img;
      const isLowQuality = width < 300 || height < 300;
      
      resolve({
        isLowQuality,
        note: isLowQuality ? "nếu được thì shop làm nét lai (nêu co the)" : undefined
      });
    };
  });
}
```

### Storage Strategy
- Store original file (no compression)
- Add quality note to order metadata
- Admin can review and enhance logos manually

## UNRESOLVED QUESTIONS

1. **Logo storage:** Upload to R2/S3 or embed as base64? (Affects serialization strategy)
2. **Print quality:** Do logos need 300 DPI or web-quality (72 DPI) sufficient?
3. **Team sizes:** Max players per order? (Impacts form complexity)
4. **Browser support:** IE11 or modern only? (Affects Canvas vs CSS choice)
5. **Real-time collaboration:** Do multiple users need same cart? (Zustand is single-browser only)

---

## SOURCES

### 2D Overlay Research
- [Konva.js vs Fabric.js Technical Comparison](https://dev.to/xingjian_hu_123dc779cbcac/konvajs-vs-fabricjs-in-depth-technical-comparison-and-use-case-analysis-3k7l)
- [Canvas Libraries Comparison 2026](https://www.pkgpulse.com/blog/fabricjs-vs-konva-vs-pixijs-canvas-2d-graphics-libraries-2026)
- [Image Overlay CSS Guide](https://cloudinary.com/guides/image-effects/image-overlay-css)

### Paste/Excel Integration
- [SheetJS Clipboard Documentation](https://docs.sheetjs.com/docs/demos/local/clipboard/)
- [MDN Element Paste Event](https://developer.mozilla.org/en-US/docs/Web/API/Element/paste_event)
- [Tabulator Clipboard Documentation](https://tabulator.info/docs/4.0/clipboard)
- [Working with Pasted Content in JavaScript](https://www.raymondcamden.com/2024/07/03/working-with-pasted-content-in-javascript/)

### Image Validation & Compression
- [Client-Side Image Compression DEV Community](https://dev.to/ramko9999/client-side-image-compression-on-the-web-26j7)
- [Compressor.js Library](https://fengyuanchen.github.io/compressorjs/)

### Form State Management
- [React Hook Form vs Formik 2026 Comparison](https://www.refine.dev/blog/react-hook-form-vs-formik/)
- [Form Libraries Comparison Smashing Magazine](https://www.smashingmagazine.com/2023/02/comparing-react-form-libraries/)

### Cart State & Persistence
- [Redux vs Zustand vs Context API 2026](https://medium.com/@sparklewebhelp/redux-vs-zustand-vs-context-api-in-2026-7f90a2dc3439)
- [State Management Comparison OneUptime](https://oneuptime.com/blog/post/2026-01-15-choose-react-state-management-context-redux-zustand/view)
- [Zustand Persist Middleware Guide](https://sanjewa.com/blogs/zustand-persistence-middleware-guide/)
- [Taking Zustand Further: Persist & DevTools](https://medium.com/@skyshots/taking-zustand-further-persist-immer-and-devtools-explained-ab4493083ca1)

### Debounce & Performance
- [Debouncing in React](https://www.developerway.com/posts/debouncing-in-react)
- [useDebounce React Hook](https://usehooks.com/usedebounce)

### Table Libraries
- [TanStack Table vs AG Grid 2026 Comparison](https://www.simple-table.com/blog/tanstack-table-vs-ag-grid-comparison)
- [AG Grid Alternatives 2026](https://www.thefrontendcompany.com/posts/ag-grid-alternatives)
