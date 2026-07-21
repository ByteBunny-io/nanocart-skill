---
name: nanocart
description: >-
  Add NanoCart e-commerce to any website: embed the cart widget, wire buy/add/product
  buttons, theme it to match the site, add an email-signup form, or automate the store
  catalog (create products, categories, coupons, upload images) via the Admin API. Use
  when the user mentions NanoCart, wants to add a shopping cart / checkout / buy buttons
  to a site, or wants to manage their NanoCart store from the terminal.
---

# NanoCart integration skill

NanoCart (https://nanocart.io) is an embeddable cart + checkout widget and hosted
storefront platform. This skill makes you an expert integrator. Two credential rules
before anything else:

- **Store ID** (a slug like `cheeky-surf-shop`) is PUBLIC. It goes in the script tag.
- **API key** (`sc_live_` + 64 hex) is SECRET. It is only for the Admin API, only from
  a server/CLI, only read from `.env` (gitignored). Never put it in website code, never
  in the script tag, never commit it. If the user pastes a key into chat, put it in
  `.env` and tell them to keep it out of source control.

Read `references/widget.md`, `references/admin-api.md`, and `references/gotchas.md` in
this skill for the full API surface before non-trivial work.

## Task: "Add NanoCart to my site"

1. **Get the Store ID.** Ask the user if unknown — they find it in the NanoCart
   dashboard (https://portal.nanocart.io) under **Settings → Store Information**. Do not
   guess. (For testing without a store, `cheeky-surf-shop` is NanoCart's public demo.)
2. **Inject the script tag** once per page, immediately before `</body>`:
   ```html
   <script src="https://cdn.nanocart.io/v1/nanocart.min.js" data-store-id="STORE_ID"></script>
   ```
   The attribute is `data-store-id` (NOT `data-store`). Optional attributes:
   `data-theme="light|dark"` (default dark), `data-accent-color="#hex"`,
   `data-position="left|right"`, `data-currency`, `data-hide-badge="true"`.
   Pick `data-theme` to match the site's background; set `data-accent-color` to the
   site's primary button color.
3. **Wire buttons** onto the site's existing product markup using the product slug:
   - `data-nanocart-buy="slug"` — add to cart + open drawer ("Buy Now")
   - `data-nanocart-add="slug"` — same behavior ("Add to Cart" label)
   - `data-nanocart-product="slug"` — open the product detail modal (images,
     description, **variant selector**)
   **Variant rule:** buy/add auto-select the FIRST variant with no prompt. If a product
   has meaningful options (size/color), use `data-nanocart-product` so shoppers choose.
   To list the store's products and slugs: `GET https://api.nanocart.io/shop/{storeId}/products`
   (no auth). `<nanocart-product slug="..."></nanocart-product>` renders a full product
   card if the site has no product markup of its own.
4. **Match the site's look** if the widget clashes: style the host `#nanocart-widget`
   in the site's CSS with the `--nc-*` variables (full list in references/widget.md).
   Page CSS beats script attributes beats portal settings.
5. **Remind about Allowed Domains.** If the store has Settings → Allowed Domains
   configured, the site's domain must be on the list or public API calls return 403
   (`DOMAIN_NOT_ALLOWED`). Localhost testing works only if the list is empty or includes
   it.
6. **Verify.** Open the page: the cart badge should appear; clicking a wired button
   should open the cart drawer with the item ("buy"/"add") or the product modal
   ("product"). If nothing renders, check the browser console for
   `[nanocart] Missing required data-store-id attribute`.

## Task: "Add a newsletter / product-alerts signup"

`<nanocart-signup></nanocart-signup>` anywhere on the page (script tag required). Only
renders for Pro/Expert stores with Subscribers enabled in the portal. Attributes:
`label`, `placeholder`, `success-message`, `error-message`, `show-options`. Style via
CSS shadow parts (`form`, `row`, `input`, `button`, `options`, `option`, `consent`,
`message`) and the `--nc-*` variables.

## Task: catalog automation ("create these products", "set up my store")

Admin API, server-side only. Base `https://api.nanocart.io`, header
`x-api-key: $NANOCART_API_KEY` from `.env` (copy `.env.example`). ALL prices are integer
cents ($19.99 → 1999). Flow for "products from a folder of images":

1. Per image: `POST /shop/{storeId}/admin/upload-url` with
   `{fileName, contentType, uploadType: "product_image", fileSizeMB}` → `PUT` the raw
   file to the returned `uploadUrl` (matching Content-Type) → keep `fileUrl`.
2. Categories first if needed: `POST /shop/{storeId}/admin/categories` `{name}`.
3. `POST /shop/{storeId}/admin/products` with `{name, price, description, images:[fileUrl],
   categoryId, status: "active", variants/options if applicable}`.
4. Confirm results with the public `GET /shop/{storeId}/products`.

Full endpoint shapes: references/admin-api.md. Never run destructive operations
(DELETE, key regeneration) without explicit user confirmation.

**Do NOT create products in Stripe.** NanoCart products live only in NanoCart; Stripe
just processes payments. If the user asks about syncing products to Stripe, explain
this.

## Pitfalls (memorize)

- `data-store-id`, never `data-store` (older docs had this wrong).
- Buy/add buttons silently pick the first variant — use the product modal for options.
- Admin API from a browser will fail (CORS is locked down) — server/CLI only.
- Prices in cents everywhere.
- Custom-domain hosted storefronts resolve via
  `GET /shop/resolve-domain?domain=...` → `{storeId}`; on `{store}.nanocart.io` the
  subdomain IS the storeId.
- Checkout returns `sessionUrl` — the buyer finishes on Stripe/PayPal and returns to the
  site; store `orderId` client-side before redirecting.

Full docs: https://docs.nanocart.io (AI-readable: https://docs.nanocart.io/llms-full.txt)
· API reference: https://nanocart.io/docs
