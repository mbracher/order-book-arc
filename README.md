# Order Book Arc

A real-time order book visualizer that maps every price level to a fixed color on a 360-degree hue wheel -- so you can track price movement, depth shifts, and fills visually without reading a single number. The book is rendered as an interactive half-circle arc with bids fanning left and asks fanning right. Streams live BTC/USDT data from the Binance WebSocket API.

**[Live Demo](https://order-book-arc.vercel.app)**

![Demo](docs/order-book-arc-demo.gif)

## Features

- **Arc visualization** -- order book depth rendered as colored wedges fanning out from a central spread gap
- **Hue-mapped pricing** -- each price level maps to a fixed position on a 360-degree color wheel, so you can track price movement visually without reading numbers
- **Order placement** -- click the arc or use the manual entry panel to place buy/sell limit orders
- **Order tracking** -- open orders shown as triangle markers; fills trigger animated trade indicators
- **Times & Sales tape** -- scrolling list of recent trades with side, price, and size
- **Adjustable controls** -- pause/play, speed (0.5x/1x/3x), depth levels (10/15/20), tick grouping (0.05/0.10/0.25)

## Getting Started

No build step required. Open `index.html` in any modern browser.

```sh
# or serve locally
npx serve .
```

## Deployment

Deploy to [Vercel](https://vercel.com) by importing the repo -- it auto-detects the static `index.html` with zero configuration.

## Docs

- [Product Owner Guide](docs/product-owner-guide.md) -- trading concepts and feature walkthrough
- [UX Developer Spec](docs/ux-spec.md) -- layout, rendering, and interaction specification

## Built with Claude

This project was developed entirely through conversation with [Claude](https://claude.ai) by Anthropic -- from the original sketch below to the final interactive prototype. The product owner guide, UX spec, and all code were generated collaboratively using Claude.

![Original sketch](docs/order-book-arc-sketch.png)

## License

Dual-licensed under [MIT](LICENSE) or [Apache-2.0](LICENSE), at your option.
