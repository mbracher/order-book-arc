# Order Book Arc Visualizer — UX Developer Specification

## 1. Overview

The Order Book Arc Visualizer renders a live order book as a half-circle (arc) with interactive order placement, tracking, and trade animation. This document specifies every visual element, interaction, layout rule, and animation in sufficient detail for a UX developer to implement or extend the component.

**Technology:** HTML5 Canvas, vanilla JS. No framework dependencies. Fonts loaded via Google Fonts (JetBrains Mono, Outfit).

**Rendering:** All drawing is done on a single `<canvas>` element using `requestAnimationFrame` at display refresh rate. Book data ticks at a configurable interval (default 280ms).

---

## 2. Layout Architecture

### 2.1 Canvas Coordinate System

```
┌─────────────────────────────────────────────────┐
│                    CONTROLS                      │
├─────────────────────────────────────────────────┤
│                                                  │
│         ┌──── Spread Gap ────┐                   │
│        ╱   (ARC_GAP_DEG=22°) ╲                  │
│       ╱         ◇ trade       ╲                  │
│      ╱     anim appears here   ╲                 │
│  BID ╱                          ╲ ASK            │
│     │   Arc wedges fan outward   │               │
│     │   from inner RADIUS        │               │
│     │                            │               │
│  ───┼────────────────────────────┼───            │
│     │  CY (center Y)            │               │
│     │                            │               │
│     │    Leg bars extend         │               │
│     │    straight down           │               │
│     │                            │               │
│     │    ┌─ Times & Sales ─┐    │               │
│     │    │  B 5 @ 100.05   │    │               │
│     │    │  S 3 @ 100.10   │    │               │
│     │    └─────────────────┘    │               │
│     │                            │               │
│     │  Off-screen order markers  │               │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 2.2 Key Layout Variables

| Variable | Computation | Description |
|---|---|---|
| `W`, `H` | `min(containerWidth, 1200)`, `min(innerHeight - 200, 800)`, clamped ≥ 200 | Canvas logical size |
| `CX` | `W / 2` | Horizontal center |
| `CY` | `H * 0.35` | Vertical center of the arc (not the canvas center) |
| `RADIUS` | `max(40, min(W × 0.20, H × 0.26))` | Inner radius of the arc |
| `MAX_BAR_LEN` | `max(30, min(W × 0.28, RADIUS × 1.6))` | Maximum bar/wedge length |

### 2.3 DPR Handling

Canvas dimensions are multiplied by `devicePixelRatio`. A CSS transform is applied via `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)` so all drawing code uses logical (CSS) pixels.

---

## 3. Arc Geometry

### 3.1 Angular Layout

The arc spans 180° (from π to 0 in canvas terms). It is split:

- **Spread gap:** `ARC_GAP_DEG = 22°` per side from the 12 o'clock position (total gap = 44°).
- **Available arc per side:** `90° - ARC_GAP_DEG = 68°`.
- **Levels on arc:** `min(ARC_MAX_LEVELS=8, totalLevels)`.
- **Slice per level:** `availableDeg / arcLevels`.
- **Wedge gap:** `max(0.3°, sliceDeg × 0.05)` — small separation between wedges.

### 3.2 Wedge Construction

Each wedge is a filled annular sector:

```
Inner arc:  ctx.arc(CX, CY, RADIUS, canvasAngle1, canvasAngle2)
Outer arc:  ctx.arc(CX, CY, outerR, canvasAngle2, canvasAngle1, true)  // reverse
```

Where `outerR = RADIUS + (level.size / maxSize) × MAX_BAR_LEN`.

**Angle mapping:**

- Bid side (left): angles from `90° + startDeg` to `90° + endDeg` (math convention).
- Ask side (right): mirrored — `90° - endDeg` to `90° - startDeg`.
- Canvas angles are negated due to canvas Y-axis inversion.

### 3.3 Leg Bars (Straight Section)

When `levelIndex ≥ arcLevels`, bars are drawn as horizontal rectangles extending outward from the arc's base:

- **Base X:** `CX ± RADIUS` (bid left, ask right).
- **Bar width:** proportional to size, same formula as wedge length.
- **Bar height:** matches arc wedge base width = `RADIUS × sliceRadians`.
- **Vertical spacing:** same as arc gap in pixels = `RADIUS × (wedgeGapDeg × 2 × π / 180)`.
- **First leg Y:** computed from the actual Y-coordinate of the last arc wedge's bottom edge + one gap.

---

## 4. Color System

### 4.1 Price-to-Color Mapping

Every absolute price maps to a fixed hue on the HSL color wheel:

```javascript
function priceToHue(price) {
  const steps = Math.round((price - 90) / REAL_TICK);
  return ((steps * 9.1) % 360 + 360) % 360;
}
```

The color wraps around the full 360° wheel approximately every `360 / 9.1 ≈ 40` ticks (= $2.00 at tick 0.05). This gives enough color variation to distinguish nearby prices while keeping the pattern repeatable.

**Usage:** `hsla(hue, 70%, 56%, alpha)` for fills. Gradient from `alpha=0.9` at inner edge to `alpha=0.35` at outer edge.

### 4.2 UI Colors

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#0a0b0f` | Main background |
| `--bg2` | `#12131a` | Header/controls background |
| `--text` | `#c8cad0` | Default text |
| `--text-dim` | `#5a5d6a` | Secondary text |
| `--text-bright` | `#eef0f6` | Emphasized text |
| `--bid-accent` | `#00e676` | Bid/buy accent |
| `--ask-accent` | `#ff5252` | Ask/sell accent |
| Buy marker | `#00e676` | Buy order triangles |
| Sell marker | `#ff1744` | Sell order triangles |

---

## 5. Text & Labels

### 5.1 Typography

| Context | Font | Weight | Size |
|---|---|---|---|
| Header title | Outfit | 600 | 18px |
| Spread value | Outfit | 600 | 14px |
| Control labels | JetBrains Mono | 400 | 10-11px |
| Price in arc wedge | JetBrains Mono | 500 | `max(6, min(11, barBaseWidth × 0.45))` |
| Price in leg bar | JetBrains Mono | 500 | `max(7, min(11, barH × 0.48))` |
| Size labels | JetBrains Mono | 400 | `max(6, min(9, barBaseWidth × 0.36))` |
| Order size (triangle) | JetBrains Mono | 600 | `max(6, triSize × 0.9)` |
| Trade tape entries | JetBrains Mono | 500 | 10px |

### 5.2 Label Positioning

**Arc price labels:**
- Positioned at radius `RADIUS + fontSize × 1.8 + 4` along the wedge's midAngle.
- Rotated along the **radial** (bar) direction: `rotation = -midAngle`, clamped to ±90° for readability.

**Leg price labels:**
- Positioned 8px inside from the base edge.
- Horizontal (no rotation). Right-aligned for bids, left-aligned for asks.

**Size labels (arc):**
- Positioned 14px beyond the outer edge of the wedge, same radial rotation.

**Size labels (legs):**
- Positioned 6px beyond the bar's outer end. Same alignment as price labels.

---

## 6. Tick & Price System

### 6.1 Decimal Arithmetic

All price calculations use integer-based rounding to avoid floating-point drift:

```javascript
function roundTick(price, tick) {
  const m = Math.round(1 / tick);
  return Math.round(price * m) / m;
}
```

### 6.2 Real Tick vs Display Tick

| Parameter | Default | Purpose |
|---|---|---|
| `REAL_TICK` | 0.05 | The instrument's actual minimum price increment |
| `displayTick` | 0.05 | Visual grouping level (must be a multiple of REAL_TICK) |

When `displayTick > REAL_TICK`, raw book entries are bucketed:
- **Bids:** bucket = `floor(price / displayTick) × displayTick`
- **Asks:** bucket = `ceil(price / displayTick) × displayTick`
- Sizes within a bucket are **summed**.

Price formatting adapts to display tick precision: `price.toFixed(decimals(displayTick))`.

---

## 7. Order System

### 7.1 Order Data Model

```typescript
interface Order {
  id: number;          // Auto-incrementing unique ID
  price: number;       // Limit price, snapped to displayTick
  side: 'buy' | 'sell';
  size: number;        // Remaining unfilled quantity
}
```

### 7.2 Order Placement

**Via click:** hit-test against `hitZones[]` array rebuilt each frame. Arc zones use polar coordinate test (distance from center + angle range). Leg zones use rectangular bounds test.

**Via manual entry:** price is snapped to nearest `displayTick`. Size defaults to 1 if not specified.

### 7.3 Order Grouping for Display

Before rendering, orders are grouped by `side + price`. Multiple orders at the same price show as a single marker with accumulated `totalSize`.

### 7.4 Fill Detection

Checked every frame against the current grouped book:

```
Buy order fills when:   bestAsk ≤ order.price
Sell order fills when:  bestBid ≥ order.price
```

**Partial fills:** 30% probability when `order.size > 1`. Fill size = `max(1, floor(size × random(0.3–0.8)))`. Remainder stays as active order with reduced size.

---

## 8. Order Marker Rendering

### 8.1 Triangle Shape

Equilateral-ish triangle, drawn at a given position and angle:

```
Tip:          (size, 0)
Bottom-left:  (-size×0.6, -size×0.65)
Bottom-right: (-size×0.6, +size×0.65)
```

Rotated by `-angle` (canvas convention). Rendered with fill + shadow glow (`shadowBlur: 8`).

`triSize = max(6, barH × 0.4)`

### 8.2 Marker Placement Rules

| Order Location | Triangle Position | Points Toward | Size Label | Price Label |
|---|---|---|---|---|
| **On arc** (price visible in arc) | Inside arc at `RADIUS - triSize - 4` along wedge midAngle | Outward (toward bar) | Further inside arc, radially rotated | — (bar's own price label suffices) |
| **On leg** (price visible in legs) | Inside leg base at `baseX ± (triSize + 4)` | Toward bar (away from center) | Next to triangle, further inside | — (bar's own price label suffices) |
| **In spread** (price between bestBid and bestAsk) | In gap, angle interpolated proportionally between bid/ask edges | Outward into gap | Inside arc, radially rotated | Horizontal, directly above triangle |
| **Off-screen** (beyond visible depth) | Below last visible bar, stacked vertically | Same as leg direction | Next to triangle | Aligned like leg bar prices |

### 8.3 Colors

- Buy orders: `#00e676` (green)
- Sell orders: `#ff1744` (red)

---

## 9. Trade Animation System

### 9.1 Animation Phases

Each fill creates a `tradeAnim` object that progresses through two phases:

#### Phase 1: Appear (t: 0 → 1)

- **Position:** At the spread gap, centered at `(CX, CY - RADIUS + 16)`.
- **Visual:** Radial glow burst scales from 0 to 20px. Text `"size @ price"` scales from 0 to 12px font.
- **Color:** Green for buys, red for sells (same as markers).
- **Duration:** `t` increments by `ANIM_SPEED = 0.04` per frame ≈ 25 frames ≈ 0.4s at 60fps.
- **Transition:** At `t ≥ 1`, switches to phase 2.

#### Phase 2: Drop (t: 0 → 1)

- **Motion:** Text drops from spread position to `tapeStartY = CY + 14` using `easeOutBounce` easing.
- **Visual:** 11px bold monospace showing `"size @ price"`.
- **Duration:** Same speed as phase 1.
- **On complete:** Entry added to `tradeTape[]` (unshift), oldest entries trimmed beyond `MAX_TAPE_ENTRIES = 10`.

### 9.2 Bounce Easing Function

Standard bounce-out with 4 bounces:

```javascript
function easeOutBounce(t) {
  if (t < 1/2.75)      return 7.5625 * t * t;
  if (t < 2/2.75)      { t -= 1.5/2.75;   return 7.5625*t*t + 0.75; }
  if (t < 2.5/2.75)    { t -= 2.25/2.75;  return 7.5625*t*t + 0.9375; }
  t -= 2.625/2.75;     return 7.5625*t*t + 0.984375;
}
```

---

## 10. Times & Sales Tape

### 10.1 Layout

- **Position:** Centered horizontally at `CX`, starting at `tapeStartY = CY + 14`.
- **Title:** "TIMES & SALES" in Outfit 8px, 20% white opacity, 10px above first entry.
- **Line height:** 18px per entry.
- **Max entries:** 10 (configurable via `MAX_TAPE_ENTRIES`).

### 10.2 Entry Rendering

Each entry shows: `{B|S}  {size} @ {price}`

- **Background:** Colored pill (110×15px, 3px radius) at 12% opacity of the side color.
- **Text:** 10px JetBrains Mono 500, full side color.
- **Fade:** `alpha = entryAge × max(0.15, 1 - index × 0.08)`. Newer entries are brighter.
- **Age animation:** Each entry's `age` ramps from 0 to 1 at +0.05 per frame, creating a fade-in effect when a new trade lands.

---

## 11. Interaction Specification

### 11.1 Click-to-Order

**Cursor:** `crosshair` over the canvas.

**Hit detection** is performed against the `hitZones[]` array, which is rebuilt every frame:

- **Arc zones:** Polar coordinate test. Compute `distance` from `(CX, CY)` and `angle = atan2(dy, dx)`. Check `innerR ≤ dist ≤ outerR` and `angle ∈ [a1rad, a2rad]`.
- **Leg zones:** Simple rectangle bounds check against stored `{x, y, w, h}`.

**Result:** Clicking a bid bar places a buy order; clicking an ask bar places a sell order. Size uses `defaultOrderSize`.

### 11.2 Manual Order Entry

| Element | Behavior |
|---|---|
| Price input | Text field. On focus, pre-fills with current best bid if empty. Value snapped to `displayTick` on submit. |
| Size input | Text field. Defaults to "1". |
| BUY button | Submits as buy order. Green styling. |
| SELL button | Submits as sell order. Red styling. |
| Enter key | Submits as buy. |
| Shift+Enter | Submits as sell. |
| Confirmation | Flash message shows for 2s: "{BUY|SELL} {size} @ {price}" in side color. |

### 11.3 Controls

| Control | Options | Default | Effect |
|---|---|---|---|
| Pause/Play | ⏸ / ▶ | Playing | Stops/resumes book ticking |
| Step | ⏭ | — | Advances one tick, auto-pauses |
| Speed | 0.5× / 1× / 3× | 1× | Multiplier on tick interval |
| Levels | 10 / 15 / 20 | 10 | Number of visible price levels per side |
| Display Tick | 0.05 / 0.10 / 0.25 | 0.05 | Price grouping granularity |
| Order Size | 1 / 5 / 10 | 1 | Default size for click-placed orders |

---

## 12. Simulated Market Data

The current implementation uses a random-walk simulation for development and demo purposes. This section documents the simulation so it can be replaced with real data feeds.

### 12.1 Mid-Price Walk

Each tick: 35% chance move up one `REAL_TICK`, 35% chance move down, 30% chance stay. This creates a slight upward drift to keep things interesting.

### 12.2 Book Generation

Each tick regenerates the full raw book:
- Depth: `ceil((NUM_LEVELS × displayTick) / REAL_TICK) + 6` levels.
- Size formula: `max(0.1, 2 + depth_index × 0.5 + random(0–12))`.
- This creates the typical book shape: thin near the top, thicker deeper.

### 12.3 Replacing with Real Data

To connect real data, replace `tickRawBook()` with a function that populates `rawBids` and `rawAsks` Maps (keyed by price → size) from a WebSocket feed or REST poll. The grouping, rendering, and order systems will work unchanged.

The interface expects:

```typescript
// Populate these Maps with real data
rawBids: Map<number, number>  // price → total size
rawAsks: Map<number, number>  // price → total size
midPrice: number              // current mid-price
```

---

## 13. Performance Notes

- All rendering uses a single canvas — no DOM manipulation per frame.
- Hit zones are flat arrays scanned linearly (typically 10–20 entries — negligible).
- Shadow blur is used sparingly (wedge glow, triangle glow) — if performance is an issue on low-end devices, reduce or eliminate `shadowBlur`.
- `requestAnimationFrame` drives the render loop; data ticks are decoupled and time-gated.
- The grouped book computation runs every frame — for very large raw books (1000+ entries), consider caching until raw data changes.

---

## 14. Extension Points

### 14.1 Adding New Display Tick Options

Add a button in the controls HTML and a corresponding `setDisplayTick(value, buttonId)` call. The value must be a multiple of `REAL_TICK`.

### 14.2 Adding Order Types

Currently only limit orders are supported. To add market orders, skip the price (use best bid/ask), and trigger immediate fill. The animation system will handle the rest.

### 14.3 Customizing Trade Animation

- **Speed:** Adjust `ANIM_SPEED` (higher = faster).
- **Easing:** Replace `easeOutBounce` with any `t → t` easing function.
- **Tape length:** Adjust `MAX_TAPE_ENTRIES`.

### 14.4 Real-Time Data Integration

Replace the simulation functions (`seedBook`, `tickRawBook`) with WebSocket handlers that update `rawBids`, `rawAsks`, and `midPrice`. Call `groupedBook()` on each update. The rendering loop will pick up changes automatically.

### 14.5 Multiple Instruments

To support multiple instruments, encapsulate the entire state (book, orders, trades, config) in a class. Instantiate one per instrument with its own canvas. The color mapping will automatically differentiate price ranges.
