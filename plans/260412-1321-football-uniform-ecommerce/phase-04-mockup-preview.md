---
phase: 4
title: "2D Mockup Preview + Print Config Visualization"
status: pending
priority: P1
effort: 4d
---

# Phase 4: 2D Mockup Preview + Print Config Visualization

## Context
- Depends on Phase 3 (builder state, print config)
- [Custom Builder Research — 2D Overlay](../reports/researcher-260412-1321-custom-builder.md)

## Overview
CSS overlay approach: position text/logos absolutely over product images. Three views: front, back, shorts. Player dropdown to preview individual names/numbers.

## Key Insights
- CSS overlay = 0 dependencies, GPU-accelerated, instant updates (no debounce needed)
- Each product + color set has images for front/back/shorts angles
- Text overlay: player name, number, team name — positioned via CSS absolute + transform
- Logo overlay: club logo, sponsor, league patch — img tags positioned absolutely
- Canvas export for mockup image (attach to order email) — Phase 5 handles export

## Architecture

```
MockupPreview component:
├── AngleSelector — [Mặt trước] [Mặt sau] [Quần]
├── PlayerSelector — dropdown from player list
├── PreviewContainer (position: relative)
│   ├── ProductImage (base layer)
│   ├── LogoOverlay (absolute positioned img elements)
│   └── TextOverlay (absolute positioned text elements)
└── Disclaimer text
```

### Overlay Positions (per angle)

**Front:**
- Club logo → top-left chest (left: 15%, top: 20%)
- Sponsor logo → center chest (left: 50%, top: 25%, transform: translateX(-50%))
- Small number → top-right chest (right: 15%, top: 20%)

**Back:**
- Team name → top center (top: 15%)
- Number → center (top: 30%, font-size: 4rem)
- Player name → below number (top: 55%)

**Shorts:**
- Number → left or right thigh based on config (top: 20%, left/right: 20%)

## Related Code Files

### Create
- `src/components/custom-builder/mockup-preview.tsx` — main preview component
- `src/components/custom-builder/mockup-overlay-front.tsx` — front view overlays
- `src/components/custom-builder/mockup-overlay-back.tsx` — back view overlays
- `src/components/custom-builder/mockup-overlay-shorts.tsx` — shorts view overlays
- `src/components/custom-builder/angle-selector.tsx` — front/back/shorts tabs
- `src/components/custom-builder/player-selector.tsx` — dropdown to pick player
- `src/lib/mockup-positions.ts` — position constants per angle

### Modify
- `src/components/custom-builder/step-preview.tsx` — replace placeholder with MockupPreview

## Implementation Steps

1. Define position constants in `mockup-positions.ts` — CSS coordinates for each overlay element per angle
2. Build `PreviewContainer` — relative wrapper with fixed aspect ratio (3:4 for jersey, 4:3 for shorts)
3. Build front overlay — conditionally render club logo, sponsor, small number based on printConfig
4. Build back overlay — conditionally render team name, number, player name (order: top to bottom)
5. Build shorts overlay — conditionally render number on selected side
6. Build angle selector — tabs switching between front/back/shorts
7. Build player selector — dropdown from player list, defaults to first player
8. Wire into builder Step 5 — reads form state via useFormContext
9. Add disclaimer: "Màu sắc thực tế có thể khác nhẹ so với màn hình"
10. Responsive: scale preview container proportionally on mobile

## CSS Approach

```css
.mockup-container {
  position: relative;
  width: 100%;
  max-width: 400px;
  aspect-ratio: 3/4;
}
.mockup-container img.base { width: 100%; height: 100%; object-fit: contain; }
.overlay-text {
  position: absolute;
  color: white;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
  font-weight: bold;
  text-align: center;
  pointer-events: none;
}
.overlay-logo {
  position: absolute;
  pointer-events: none;
  object-fit: contain;
}
```

## Todo
- [ ] Position constants for front/back/shorts
- [ ] PreviewContainer with aspect ratio
- [ ] Front overlay (logo + sponsor + small number)
- [ ] Back overlay (team name + number + player name)
- [ ] Shorts overlay (number + side)
- [ ] Angle selector tabs
- [ ] Player selector dropdown
- [ ] Wire into Step 5 of builder
- [ ] Responsive scaling
- [ ] Disclaimer text

## Success Criteria
- Switch angle → correct overlay shows
- Switch player → name/number update instantly
- Toggle print config checkbox → overlay element appears/disappears
- Logo uploads display at correct position
- Works on mobile (scaled down proportionally)

## Risk
- Product images vary in composition → may need per-product position overrides
  - Mitigation: start with generic positions, allow admin to tune later
- Logo aspect ratios vary → use object-fit: contain + max-width/height constraints
