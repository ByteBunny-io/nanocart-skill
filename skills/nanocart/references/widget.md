# NanoCart widget — full embed surface

## Script tag

```html
<script src="https://cdn.nanocart.io/v1/nanocart.min.js" data-store-id="YOUR_STORE_ID"></script>
```

Place once per page, before `</body>`. Loads async; scans for button attributes and
re-scans via MutationObserver (SPA-safe). Attributes read from the script tag:

| Attribute | Default | Purpose |
|---|---|---|
| `data-store-id` | (required) | Public store slug. Missing → console error, widget disabled |
| `data-api-url` | `https://api.nanocart.io` | Override for testing only |
| `data-currency` | `USD` | Display currency |
| `data-position` | `right` | Cart drawer side: `left`/`right` |
| `data-hide-badge` | off | `"true"` hides the floating cart badge |
| `data-theme` | `dark` | `dark` or `light` base palette |
| `data-accent-color` | `#4292e7` | Any CSS color for buttons/accents |

## Button attributes (any element; slug = product slug)

| Attribute | Behavior |
|---|---|
| `data-nanocart-buy="slug"` | Add to cart + open drawer (Buy Now) |
| `data-nanocart-add="slug"` | Add to cart + open drawer (Add to Cart) |
| `data-nanocart-product="slug"` | Open product detail modal — images, description, **variant selector** |

**Variants:** `buy`/`add` auto-select the FIRST variant with no prompt. Products with
meaningful options should use `data-nanocart-product`.

Legacy manual attributes on `data-nanocart-add` (no slug): `data-product-id`,
`data-variant-id`, `data-product-name`, `data-variant-name`, `data-product-price`,
`data-product-image`, `data-quantity`, `data-product-type`.

## Web components

- `<nanocart-product slug="..."></nanocart-product>` — standalone product card
  (image, name, price, add button). Multiple can sit in a grid.
- `<nanocart-signup></nanocart-signup>` — email capture (Pro/Expert stores with
  Subscribers enabled; renders nothing otherwise). Attributes: `label` (button text,
  default "Sign Up"), `placeholder` (default "you@example.com"), `success-message`,
  `error-message`, `show-options` (boolean presence — renders the store's configured
  interest checkboxes). Shadow parts for CSS: `form`, `row`, `input`, `button`,
  `options`, `option`, `consent`, `message`.

## Theming — the `--nc-*` variables

The widget renders in shadow DOM on host `#nanocart-widget`. Declare variables on that
selector in page CSS. Precedence (high→low): page CSS on `#nanocart-widget` →
script-tag attributes → portal Widget Appearance settings → defaults.

| Variable | Dark default | Light default |
|---|---|---|
| `--nc-accent` | `#4292e7` | `#4292e7` |
| `--nc-bg` | `#1a1f2e` | `#ffffff` |
| `--nc-surface` | `#242938` | `#f5f5f5` |
| `--nc-border` | `#2d3348` | `#e2e2e2` |
| `--nc-text` | `#e2e8f0` | `#1a1a1a` |
| `--nc-text-muted` | `#94a3b8` | `#555555` |
| `--nc-text-dim` | `#64748b` | `#999999` |
| `--nc-text-strong` | `#ffffff` | `#111111` |
| `--nc-danger` | `#ef4444` | `#ef4444` |
| `--nc-success` | `#22c55e` | `#22c55e` |
| `--nc-radius` | `8px` | `8px` |
| `--nc-font` | system stack | system stack |

Example — match a warm light site:

```css
#nanocart-widget {
  --nc-accent: #e8590c;
  --nc-bg: #fffaf5;
  --nc-radius: 14px;
  --nc-font: 'Inter', sans-serif;
}
```

## Public API (no auth; used by the widget, callable directly)

Base `https://api.nanocart.io`:

- `GET /shop/{storeId}/products` — active products (query: `category`, `sort`
  `newest|price|featured`, `limit` ≤50, `lastKey`)
- `GET /shop/{storeId}/products/{slug}`
- `GET /shop/{storeId}/categories`
- `POST /shop/{storeId}/coupons/validate` — `{code}`
- `GET /shop/{storeId}/payment-methods` — which processors are configured
- `POST /shop/{storeId}/checkout` — `{items:[{productId, variantId?, quantity}], email,
  couponCode?, shippingMethod?: "standard"|"local_pickup", processor?: "stripe"|"paypal"}`
  → `{sessionUrl, orderId}`
- `GET /shop/{storeId}/orders/{orderId}?email=...` — order status (email must match)
- `GET /shop/{storeId}/storefront-config` · `GET /shop/resolve-domain?domain=...`
- `GET /shop/{storeId}/widget-config` · `GET /shop/{storeId}/alerts-config` ·
  `POST /shop/{storeId}/alerts-signup`
