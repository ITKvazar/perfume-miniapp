# CLAUDE.md

## Project Overview

**Perfume Mini-App** ("Ароматная Публика") is a Telegram Mini-App for an e-commerce perfume store. It is a single-page application built with vanilla HTML, CSS, and JavaScript — no build tools, no frameworks, no package manager.

The primary UI language is **Russian**.

## Repository Structure

```
perfume-miniapp/
├── index.html          # Main production file (current version, ~1780 lines)
├── index1.html         # Previous iteration (similar to index.html)
├── index2.html         # Simplified experimental version (~170 lines)
├── index3.html         # Alternative simplified version (~270 lines)
├── index-gpt.html      # ChatGPT-generated variant (~1750 lines)
└── CLAUDE.md           # This file
```

**`index.html`** is the canonical, production-ready file. The other HTML files are legacy iterations kept for reference.

## Architecture

### Monolithic Single-File App

Everything lives in `index.html`:
- **Lines 1–7**: Document head, meta tags, Telegram SDK script tag
- **Lines 8–974**: `<style>` block — all CSS (~960 lines)
- **Lines 976–1158**: HTML body — structural markup
- **Lines 1160–1780**: `<script>` block — all JavaScript (~620 lines)

There is no build step, no bundling, no compilation. The file is served directly as static HTML.

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 (semantic) |
| Styling | CSS3 (custom properties, flexbox, grid, backdrop-filter) |
| Logic | Vanilla JavaScript (ES6+) |
| Platform | Telegram Mini-App SDK (`telegram-web-app.js`) |
| Backend | Google Apps Script (webhook endpoint) |
| Persistence | `localStorage` (favorites, order history) |

### No External Dependencies

- No `package.json`, no npm/yarn
- No TypeScript, no transpilation
- No CSS preprocessors
- No framework (React, Vue, etc.)
- Only external script: Telegram Web App SDK loaded via CDN

## State Management

Global mutable `state` object at `index.html:1172`:

```javascript
let state = {
    products: [...],           // Hardcoded product catalog (6 items)
    cart: {},                  // SKU -> quantity mapping
    favorites: [],             // Array of favorite SKUs (persisted to localStorage)
    orders: [],                // Order history (persisted to localStorage)
    currentCategory: 'all',    // Active filter: 'all' | 'women' | 'men' | 'unisex'
    currentSort: 'default',    // Sort mode: 'default' | 'new' | 'price-asc' | 'price-desc'
    searchQuery: '',           // Current search string
    currentProduct: null,      // Currently open product in modal
    bannerIndex: 0             // Active banner slide index
};
```

State changes trigger imperative re-renders via functions like `renderProducts()`, `renderCart()`, `renderFavorites()`.

Persisted via `localStorage`:
- `favorites` — array of SKU strings
- `orders` — array of order objects

## Navigation

Tab-based SPA navigation via `showPage(page)` function (`index.html:1622`). No URL routing.

Four pages controlled by bottom nav:
1. **Main** (`mainPage`) — Catalog with banners, collections, product grid
2. **Favorites** (`favoritesPage`) — Saved products
3. **Cart** (`cartPage`) — Shopping cart with checkout form
4. **Profile** (`profilePage`) — Telegram user info and order history

Product details display in a full-screen modal (`productModal`).

## Key Functions Reference

| Function | Line | Purpose |
|----------|------|---------|
| `renderProducts()` | 1336 | Filters, sorts, and renders product grid |
| `renderFilteredProducts()` | 1362 | Renders a given product array into the grid |
| `openProduct(sku)` | 1429 | Opens product detail modal |
| `quickAdd(sku)` | 1504 | Adds item to cart with haptic feedback |
| `updateQuantity(sku, delta)` | 1522 | Adjusts cart item quantity |
| `renderCart()` | 1529 | Renders cart items, total, and checkout form |
| `toggleFavorite(sku)` | 1575 | Toggles favorite status, persists to localStorage |
| `renderFavorites()` | 1593 | Renders favorites grid |
| `showPage(page)` | 1622 | Page navigation (main/favorites/cart/profile) |
| `updateBadges()` | 1653 | Updates cart/favorites count badges and Telegram MainButton |
| `submitOrder()` | 1712 | Validates form, POSTs order to Google Apps Script |
| `initBanner()` | 1291 | Initializes banner carousel with auto-rotation |
| `initImageSwipe(sku, count)` | 1403 | Sets up touch swipe on product image galleries |
| `loadUserData()` | 1681 | Loads Telegram user info into profile page |
| `renderOrders()` | 1690 | Renders order history from localStorage |

## CSS Design System

CSS custom properties defined in `:root` (`index.html:11`):

```css
--primary: #8B5CF6;        /* Purple — main brand color */
--primary-light: #A78BFA;
--secondary: #EC4899;      /* Pink — accent */
--bg: #F9FAFB;             /* Light gray background */
--surface: #FFFFFF;         /* Card/surface white */
--text: #111827;            /* Primary text */
--text-secondary: #6B7280;  /* Secondary text */
--border: #E5E7EB;          /* Borders */
--success: #10B981;         /* Green — success states */
--men: #3B82F6;             /* Blue — men's category */
--women: #EC4899;           /* Pink — women's category */
--unisex: #8B5CF6;          /* Purple — unisex category */
```

Design conventions:
- Mobile-first, responsive layout
- Glassmorphism effects (`backdrop-filter: blur(20px)`)
- Telegram-style UI (rounded pills, blur headers)
- Standard padding: `16px`
- Border radius: `12px`–`16px` for cards, `20px`–`50%` for pills/badges
- System font stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto`

## Backend Integration

**Endpoint**: Google Apps Script webhook (hardcoded at `index.html:1162`)

**Order submission** (`submitOrder()` at line 1712):
- Method: `POST`
- Body: JSON with `name`, `phone`, `address`, `comment`, `items[]`, `total`, `userId`, `username`
- No authentication headers
- Response: `{ success: true }` or `{ error: "..." }`

## Telegram Integration

Uses the [Telegram Web App SDK](https://core.telegram.org/bots/webapps):
- `tg.ready()` / `tg.expand()` — initialization
- `tg.setHeaderColor()` / `tg.setBackgroundColor()` — theming
- `tg.HapticFeedback.impactOccurred('light')` — tactile feedback on cart/favorite actions
- `tg.HapticFeedback.notificationOccurred('success')` — order confirmation feedback
- `tg.MainButton` — shows cart total, navigates to cart on tap
- `tg.showAlert()` — native alert dialogs
- `tg.initDataUnsafe.user` — reads Telegram user data for profile

## Product Data Model

Products are hardcoded in the `state.products` array (`index.html:1173`):

```javascript
{
    sku: string,          // Unique ID (e.g., 'p001')
    title: string,        // Product name
    price: number,        // Current price in RUB
    oldPrice?: number,    // Original price (for discount display)
    category: string,     // 'women' | 'men' | 'unisex'
    images: string[],     // Array of image URLs
    description: string,  // Product description (Russian)
    characteristics: {},  // Key-value pairs (volume, type, manufacturer, year)
    isNew: boolean,       // NEW badge flag
    tags: string[]        // Collection tags: 'new', 'spring', 'summer', 'gift'
}
```

## Development Workflow

### Running Locally

No build step needed. Serve the file with any static HTTP server:

```bash
# Python
python3 -m http.server 8080

# Node.js (npx)
npx serve .
```

Then open `http://localhost:8080` in a browser. For full Telegram integration, use [BotFather](https://core.telegram.org/bots/webapps#launching-mini-apps) to set up a test bot with a web app URL.

### Testing

No automated tests exist. Test manually in:
1. A mobile browser (responsive layout)
2. Telegram app (full Mini-App features: haptics, MainButton, user data)

### Linting / Formatting

No linting or formatting tools are configured. When editing:
- Use 4-space indentation (matches existing code)
- Follow existing camelCase for JS functions and variables
- Follow existing kebab-case for CSS classes
- Keep UPPERCASE for constants (e.g., `API_URL`)

## Conventions for AI Assistants

1. **Single-file architecture**: All changes to the app go in `index.html`. CSS in the `<style>` block, JS in the `<script>` block, markup in `<body>`.
2. **No build system**: Do not introduce npm, webpack, vite, or any build tooling unless explicitly requested.
3. **Vanilla JS only**: Do not introduce frameworks (React, Vue, etc.) or libraries unless explicitly requested.
4. **Russian UI text**: All user-facing strings are in Russian. Maintain this convention.
5. **Product data is hardcoded**: The product catalog lives in `state.products`. To add/remove products, edit this array directly.
6. **Preserve Telegram SDK usage**: Haptic feedback, MainButton, and user data integration are integral to the UX.
7. **localStorage persistence**: Favorites and orders persist via localStorage. Cart is session-only (intentional).
8. **Legacy files**: `index1.html`, `index2.html`, `index3.html`, and `index-gpt.html` are historical. Do not modify them unless asked.
9. **No secrets in code**: The Google Apps Script URL is hardcoded. If moving to a more sensitive backend, use environment variables or Telegram bot token validation.
10. **Mobile-first**: All UI changes must work on mobile screens (320px+). The app runs inside Telegram's webview.
