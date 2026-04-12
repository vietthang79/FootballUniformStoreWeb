# Research: Fabric.js v6 for Sports Jersey Mockup Customizer

**Date**: 2026-04-12 | **Context**: Football Uniform Store Web

## Executive Summary

Fabric.js v6 is **production-ready** for jersey mockup customizers in Next.js. Fabric's zone-based overlay system, text positioning, and canvas-to-image export align perfectly with your requirements. **Recommendation: Use Fabric.js v6** for object-oriented canvas manipulation and built-in filtering; consider Konva.js only if performance with 100+ objects becomes critical.

---

## 1. Fabric.js v6 in Next.js: SSR & Client-Side Loading

### SSR Compatibility Status
- **v6 improvement**: Introduced `env` concept with `getEnv()` for browser-specific logic, making SSR significantly easier than v5.
- **Status**: Fabric.js v6 now **partially supports SSR** (better than previous versions, though client-side rendering is still recommended for canvas operations).

### Recommended Import Pattern (Next.js 13+ App Router)

```typescript
// components/jersey-customizer.tsx (Client Component)
'use client';

import dynamic from 'next/dynamic';
import { useEffect, useState } from 'react';

// Dynamic import with ssr: false for canvas operations
const FabricCanvas = dynamic(() => import('./fabric-canvas'), {
  ssr: false,
  loading: () => <div>Loading customizer...</div>,
});

export default function JerseyCustomizer() {
  const [isMounted, setIsMounted] = useState(false);

  useEffect(() => {
    setIsMounted(true);
  }, []);

  return isMounted ? <FabricCanvas /> : null;
}
```

```typescript
// components/fabric-canvas.tsx (Actual Fabric Logic)
'use client';

import { Canvas, Image as FabricImage, Text } from 'fabric';
import { useEffect, useRef } from 'react';

export default function FabricCanvasComponent() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const fabricCanvasRef = useRef<Canvas | null>(null);

  useEffect(() => {
    if (!canvasRef.current) return;

    // Initialize Fabric Canvas (only runs on client)
    const fabricCanvas = new Canvas(canvasRef.current, {
      width: 400,
      height: 600,
      backgroundColor: '#fff',
    });

    fabricCanvasRef.current = fabricCanvas;

    return () => {
      fabricCanvas.dispose();
    };
  }, []);

  return <canvas ref={canvasRef} />;
}
```

**Key Points**:
- Use `'use client'` directive (required for Next.js App Router)
- Wrap Fabric components in `next/dynamic` with `ssr: false`
- Initialize canvas in `useEffect` (client-side only)
- Fabric v6 supports modern ES modules: `import { Canvas } from 'fabric'`

---

## 2. Zone-Based Overlay Pattern: Static Positions for Jersey Customization

### Implementation Approach

Define **zones** as static coordinate objects on your product image:

```typescript
interface CustomizationZone {
  id: string;
  name: string; // 'nameZone', 'numberZone', 'logoZone', etc.
  x: number;
  y: number;
  width: number;
  height: number;
  type: 'text' | 'image'; // What can be placed here
  constraints?: {
    lockRotation?: boolean;
    lockScaling?: boolean;
    maxScale?: number;
  };
}

const jerseyZones: CustomizationZone[] = [
  {
    id: 'zone-name',
    name: 'Player Name',
    x: 100,
    y: 250,
    width: 200,
    height: 40,
    type: 'text',
    constraints: { lockRotation: true, lockScaling: false },
  },
  {
    id: 'zone-number',
    name: 'Player Number',
    x: 150,
    y: 320,
    width: 100,
    height: 120,
    type: 'text',
    constraints: { lockRotation: true },
  },
  {
    id: 'zone-logo',
    name: 'Team Logo',
    x: 50,
    y: 80,
    width: 80,
    height: 80,
    type: 'image',
    constraints: { lockRotation: false },
  },
];

// Add overlay zones to canvas (visual guides during edit mode)
function addZoneOverlays(canvas: Canvas, zones: CustomizationZone[]) {
  zones.forEach((zone) => {
    const rect = new fabric.Rect({
      left: zone.x,
      top: zone.y,
      width: zone.width,
      height: zone.height,
      fill: 'rgba(0, 0, 255, 0.1)',
      stroke: '#0066ff',
      strokeWidth: 1,
      selectable: false,
      evented: false, // Don't respond to mouse events
      name: `zone-${zone.id}`,
    });
    canvas.add(rect);
  });
}
```

### Constraint-Based Editing

Fabric v6 supports per-object constraints:

```typescript
function addTextToZone(
  canvas: Canvas,
  zone: CustomizationZone,
  textContent: string
) {
  const text = new Text(textContent, {
    left: zone.x,
    top: zone.y,
    fontFamily: 'Arial',
    fontSize: 32,
    fontWeight: 'bold',
    fill: '#000',
    name: zone.id,
  });

  // Apply constraints
  if (zone.constraints?.lockRotation) text.setControlsVisibility({ mtr: false });
  if (zone.constraints?.lockScaling === false) text.lockScaling = false;

  canvas.add(text);
  canvas.renderAll();
}
```

---

## 3. Draggable & Resizable Image Objects (Logos)

Fabric.js v6 handles image manipulation natively:

```typescript
async function addLogoToZone(
  canvas: Canvas,
  zone: CustomizationZone,
  imageUrl: string
) {
  const img = await FabricImage.fromURL(imageUrl, {
    crossOrigin: 'anonymous',
  });

  img.scaleToWidth(zone.width);
  img.set({
    left: zone.x,
    top: zone.y,
    name: zone.id,
  });

  // Prevent rotation if zone locks it
  if (zone.constraints?.lockRotation) {
    img.setControlsVisibility({ mtr: false });
  }

  canvas.add(img);
  canvas.renderAll();
}

// Enable drag & drop for uploaded logos
export function setupImageUpload(
  canvas: Canvas,
  inputElement: HTMLInputElement,
  zone: CustomizationZone
) {
  inputElement.addEventListener('change', async (e) => {
    const file = (e.target as HTMLInputElement).files?.[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = async (event) => {
      if (typeof event.target?.result !== 'string') return;
      await addLogoToZone(canvas, zone, event.target.result);
    };
    reader.readAsDataURL(file);
  });
}
```

---

## 4. Text Rendering: Fonts, Positioning, Jersey Name+Number

### Font Loading (Critical for Consistency)

**Issue**: Fabric caches bounding box values before fonts load, causing cursor/positioning bugs.

```typescript
// Use CSS Font Loading API BEFORE creating text
async function ensureFontLoaded(fontFamily: string) {
  const font = new FontFace(
    fontFamily,
    `url('https://fonts.googleapis.com/css2?family=${fontFamily}:wght@700')`
  );
  await font.load();
  document.fonts.add(font);
}

// Load font, THEN create text object
async function addJerseyNumber(
  canvas: Canvas,
  zone: CustomizationZone,
  number: string
) {
  await ensureFontLoaded('Courier Prime');

  const text = new Text(number, {
    left: zone.x,
    top: zone.y,
    fontFamily: 'Courier Prime',
    fontSize: 100, // Large, bold numerals
    fontWeight: 'bold',
    fill: '#fff',
    stroke: '#000',
    strokeWidth: 2,
  });

  canvas.add(text);
  canvas.renderAll();
}
```

### Precise Text Positioning (Known Issue)

**Warning**: Fabric's `getBoundingRect()` can be inaccurate due to variable padding depending on font size/angle.

```typescript
// Workaround: Use textlines array for manual positioning
function getAccurateTextBounds(text: Text) {
  const lines = text.textLines;
  const lineHeights = text.getHeightOfLine(0);
  // Manual calculation more accurate than getBoundingRect()
  return {
    height: lines.length * lineHeights,
    width: Math.max(...lines.map((line) => text.getLineWidth(line))),
  };
}

// Center text in zone accurately
function centerTextInZone(text: Text, zone: CustomizationZone) {
  text.centerH = true; // Horizontal center
  text.top = zone.y + zone.height / 2 - text.height / 2;
}
```

### Recommended Font Stack for Jersey
- **Primary**: Courier Prime (monospace, bold numerals)
- **Fallback**: Arial Black, Trebuchet MS
- **Load from**: Google Fonts (free, reliable)

---

## 5. Canvas-to-Image Export (Order Confirmation)

### PNG/JPG Export

```typescript
function exportJerseyDesign(
  canvas: Canvas,
  format: 'png' | 'jpg' = 'png'
): Blob {
  // Hide editing overlays before export
  canvas.getObjects().forEach((obj) => {
    if ((obj as any).name?.startsWith('zone-')) {
      obj.visible = false;
    }
  });

  canvas.renderAll();

  // Export as data URL
  const dataUrl = canvas.toDataURL({ format });

  // Show overlays again
  canvas.getObjects().forEach((obj) => {
    if ((obj as any).name?.startsWith('zone-')) {
      obj.visible = true;
    }
  });

  // Convert data URL to Blob
  return dataURLToBlob(dataUrl);
}

function dataURLToBlob(dataUrl: string): Blob {
  const [header, data] = dataUrl.split(',');
  const bstr = atob(data);
  const n = bstr.length;
  const u8arr = new Uint8Array(n);
  for (let i = 0; i < n; i++) {
    u8arr[i] = bstr.charCodeAt(i);
  }
  const mimeMatch = header.match(/:(.*?);/);
  return new Blob([u8arr], { type: mimeMatch?.[1] || 'image/png' });
}

// Save to local file or upload
async function downloadDesign(blob: Blob, filename = 'jersey-design.png') {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
```

---

## 6. Alternative: Konva.js vs Fabric.js Comparison

| Feature | Fabric.js | Konva.js |
|---------|-----------|---------|
| **Performance** | Good (suitable for <200 objects) | Excellent (optimized for 1000+) |
| **Object Handling** | Rich filters, effects, SVG support | Scene graph, groups, layers |
| **React Integration** | Manual + useRef | Native (react-konva) |
| **Export** | PNG, JPG, SVG | PNG only, no SVG |
| **Bundle Size** | ~300KB | ~200KB |
| **Use Case** | Jersey customizers, image editors | Game dev, animations |

**For jersey customizer**: Fabric.js is the better choice unless you need:
- Animations with 100+ moving objects
- Real-time collaboration with frequent updates
- Game-like interactivity

---

## Known Issues & Workarounds

| Issue | Severity | Workaround |
|-------|----------|-----------|
| Text positioning inaccuracy (browser-dependent) | Medium | Use manual `textLines` calculation, avoid `getBoundingRect()` |
| Font caching before load | High | Always `await FontFace.load()` before creating Text objects |
| Image loading crossOrigin | Medium | Set `crossOrigin: 'anonymous'` on `fromURL()` |
| SSR canvas rendering | Low | Use `next/dynamic` with `ssr: false` |

---

## Implementation Checklist

- [ ] Install: `npm install fabric`
- [ ] Create zone definitions (coordinates mapping to jersey areas)
- [ ] Implement font preloader using CSS Font Loading API
- [ ] Build canvas wrapper with `'use client'` directive
- [ ] Add text/image overlays per zone
- [ ] Implement export-to-PNG with overlay hiding
- [ ] Test text positioning across browsers
- [ ] Handle image upload with FileReader API
- [ ] Add undo/redo via Fabric's built-in history or custom stack

---

## Conclusion

**Fabric.js v6 recommendation**: Use it for jersey customization. Strengths in object manipulation, text rendering, and export align with requirements. Main gotchas are font loading timing and text positioning precision—both manageable with provided patterns. Avoid Konva.js unless performance with many objects becomes a bottleneck (unlikely for jersey editor).

---

## Unresolved Questions

1. Do you need undo/redo functionality? (Requires custom history stack or Fabric's transaction API)
2. Will users upload multiple image logos, or predefined team logos only?
3. What jersey image formats/sizes (responsive design across devices)?
4. Export format preference: PNG for web preview, PDF for printing?

Sources:
- [Step by step on how to setup fabric.js in the next.js app - DEV Community](https://dev.to/ziqin/step-by-step-on-how-to-setup-fabricjs-in-the-nextjs-app-3hi3)
- [Fabric.js Documentation](https://fabricjs.com/docs/)
- [Building a Real-time Collaborative Whiteboard with Next.js and Fabric.js — Medium](https://medium.com/@adredars/building-a-real-time-collaborative-whiteboard-frontend-with-next-js-7c6b2ef1e072)
- [React: Comparison of JS Canvas Libraries (Konvajs vs Fabricjs) - DEV Community](https://dev.to/lico/react-comparison-of-js-canvas-libraries-konvajs-vs-fabricjs-1dan)
- [Fabric.js vs Konva - StackShare](https://stackshare.io/stackups/fabricjs-vs-konva)
- [How to render text — Fabric.js Wiki](https://github.com/fabricjs/fabric.js/wiki/How-to-render-text)
- [Konva.js vs. Fabric.js: Choosing Your Canvas Companion - Oreate AI Blog](https://www.oreateai.com/blog/konvajs-vs-fabricjs-choosing-your-canvas-companion-9d255e8dbd093ab89c868295b2d20187)
