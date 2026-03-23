# Order Book Arc Visualizer — Product Owner Guide

## What This Document Covers

This guide explains the core trading concepts behind the Order Book Arc Visualizer, what it does, why it exists, and how its features map to real-world trading workflows. No prior trading knowledge is assumed.

---

## Part 1: Trading Fundamentals

### What Is an Order Book?

An order book is the list of all outstanding buy and sell orders for a financial instrument (a stock, bond, future, cryptocurrency, etc.) at any given moment. Think of it as a live queue of people waiting to trade.

Every order book has two sides:

- **Bid side (buyers):** People willing to buy. Each bid says "I want to buy X units at price Y." The highest bid is called the **best bid** — the most someone is currently willing to pay.
- **Ask side (sellers):** People willing to sell. Each ask says "I want to sell X units at price Z." The lowest ask is called the **best ask** — the cheapest price someone is willing to sell at.

### The Spread

The **spread** is the gap between the best bid and the best ask. For example, if the best bid is 99.95 and the best ask is 100.05, the spread is 0.10. No one is currently willing to trade inside this gap — it represents the price "no man's land."

A narrow spread means the market is liquid and active. A wide spread suggests less activity or higher uncertainty.

### How Trades Happen

A trade occurs when a buyer and seller agree on a price. In practice this means:

- A **buy order** fills (executes) when the best ask drops to or below the buy order's limit price. The buyer gets what they wanted at the seller's price.
- A **sell order** fills when the best bid rises to or at the sell order's limit price. The seller gets the buyer's price.

Orders can also **partially fill** — if you want to buy 10 units but only 6 are available at your price, you get 6 now and the remaining 4 stay as an open order waiting for more sellers.

### Tick Size

Prices don't move in arbitrary decimals. Every instrument has a **tick size** — the minimum price increment. If the tick size is 0.05, valid prices are 100.00, 100.05, 100.10, etc. You cannot place an order at 100.03.

### Display Tick (Grouping)

When watching a fast market, individual ticks can be overwhelming. A **display tick** groups multiple price levels together. For example, with a real tick of 0.05 and a display tick of 0.10, the prices 100.00 and 100.05 merge into one row showing their combined size. This simplifies the view without losing important information.

### Depth / Levels

The **depth** of the book is how many price levels away from the best bid/ask you can see. "10 levels" means you see the 10 best prices on each side. Deeper views show more of the queue but add visual complexity.

---

## Part 2: The Problem We're Solving

### Traditional Order Book Displays

The standard way to display an order book is a vertical table: bids on the left, asks on the right, with horizontal bars representing volume at each price. This works, but it has a critical flaw:

**When prices move, you cannot visually track them.**

In a fast market, every tick shifts all the numbers. A price that was row 3 is now row 5. Your eye loses track. You're reading numbers, not patterns. This is cognitively expensive and slow — exactly when speed matters most.

### The Color-Wheel Insight

Our key innovation: **assign each absolute price a fixed color from the color spectrum.** Price 100.00 is always the same shade of green. Price 100.50 is always the same shade of blue. These colors never change.

When the market moves up, you see all colors shifting downward on the visualization. When it moves down, they shift up. This turns price movement into **visible motion** — something the human visual system processes instantly, without reading a single number.

### The Arc Layout

Instead of a flat table, we arrange the order book as a **half-circle (arc)**:

- Bid prices fan out to the left from the top of the arc.
- Ask prices fan out to the right from the top of the arc.
- The spread gap sits at the apex — the visual focal point.
- If there are more levels than fit on the curve, they extend straight down as "legs."

This layout provides several advantages over a flat table: the spread is the natural focal point at the top, both sides are visually symmetrical and easy to compare, the wedge shape naturally conveys that prices further from the spread are "deeper" in the book, and the arc layout creates a distinctive, memorable visual that traders can scan in milliseconds.

---

## Part 3: Feature Walkthrough

### 3.1 The Arc Visualization

| Element | What It Shows |
|---|---|
| Colored wedges (arc) | Each price level's volume, color-coded by absolute price |
| Colored bars (legs) | Deeper levels that extend straight down below the arc |
| Wedge length | The relative size (volume) at that price — longer means more orders |
| Price labels | The price at each level, inside the bar near its base |
| Size labels | The volume at each level, outside the bar tip |
| Spread gap | The empty space at the top between best bid and best ask |
| BID / ASK labels | Indicate which side is which |

### 3.2 Controls

**Speed controls** allow pausing, slowing (0.5×), normal (1×), or speeding up (3×) the simulated market. The **Step** button advances exactly one tick and pauses — essential for detailed analysis.

**Levels** (10/15/20) controls how deep into the book you see.

**Display Tick** (0.05/0.10/0.25) controls price grouping. Higher values aggregate more prices into each bar, simplifying the view.

**Order Size** (1/5/10) sets the default size when clicking a bar to place an order.

### 3.3 Order Placement

Users can place orders in two ways:

1. **Click a bar** — clicking any bid bar places a buy order at that price; clicking any ask bar places a sell order. Uses the default order size.
2. **Manual entry** — type a specific price and size, then press BUY or SELL. Keyboard shortcuts: Enter = buy, Shift+Enter = sell.

### 3.4 Order Tracking (Triangles)

Every open order is represented by a **triangle marker**:

- **Green triangles** = buy orders
- **Red triangles** = sell orders
- The **size** is displayed next to the triangle

Triangle positioning depends on where the order sits relative to the current book:

| Situation | Marker Position |
|---|---|
| Order price is on the visible arc | Inside the arc, pointing outward at that wedge |
| Order price is on the visible legs | Inside the leg area, next to that bar |
| Order price is in the spread | In the gap at the top, positioned proportionally between best bid and best ask |
| Order price is beyond visible depth | Below the last bar, showing its price explicitly |

Multiple orders at the same price **combine** into a single triangle showing the total accumulated size.

### 3.5 Trade Execution & Animation

When an order fills, a three-phase animation plays:

1. **Flash at spread** — the trade appears at the top of the arc (the spread gap) with a burst effect showing the filled size and price.
2. **Drop animation** — the trade text falls with a bounce easing down into the center.
3. **Times & Sales tape** — the trade lands in a scrolling list in the center of the arc, showing the most recent trades (max 10).

**Partial fills** are supported: if only part of an order can execute, the filled portion animates as a trade and the remaining size stays as an active order.

### 3.6 Times & Sales Tape

The center of the arc displays the most recent trades as a vertical list. Each entry shows the side (B for buy, S for sell), the size, and the price, color-coded green or red. Older entries fade gradually. This gives the user an immediate history of their execution activity.

---

## Part 4: Use Cases

### For Traders
Quickly assess market direction by watching color flow. Place and track orders visually. See fills happen in real-time with clear feedback.

### For Trading Desk Managers
Monitor market microstructure at a glance. The arc layout is scannable from a distance — useful on trading floor screens.

### For Training / Education
The step-by-step mode and visual order tracking make this an excellent tool for teaching new traders how order books work, how orders fill, and what spread dynamics look like.

### For Product Demos
The distinctive visual style makes the product memorable in demos and presentations, standing out from standard table-based order book displays.

---

## Part 5: Glossary

| Term | Definition |
|---|---|
| **Order Book** | The list of all pending buy and sell orders for an instrument |
| **Bid** | A buy order — an offer to purchase at a specific price |
| **Ask** | A sell order — an offer to sell at a specific price |
| **Best Bid** | The highest current buy price |
| **Best Ask** | The lowest current sell price |
| **Spread** | The difference between best ask and best bid |
| **Tick Size** | The minimum price increment allowed |
| **Display Tick** | A grouping size for visual simplification (multiples of tick size) |
| **Depth / Levels** | How many price levels are shown from the best price |
| **Fill / Execution** | When an order matches and a trade occurs |
| **Partial Fill** | When only part of an order's size gets executed |
| **Limit Order** | An order to buy/sell at a specific price or better |
| **Times & Sales** | A chronological list of executed trades |
