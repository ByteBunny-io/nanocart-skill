# NanoCart — instructions for AI coding agents (Codex, Cursor, and others)

You are integrating NanoCart (https://nanocart.io), an embeddable cart + checkout
widget with an Admin API. Credential rules: the **Store ID** (slug) is public; the
**API key** (`sc_live_...`) is secret — `.env` only, server-side only, never in
frontend code or the script tag.

## Add the widget to a site

1. Script tag once per page, before `</body>` — the attribute is `data-store-id`:
   `<script src="https://cdn.nanocart.io/v1/nanocart.min.js" data-store-id="STORE_ID"></script>`
   Optional: `data-theme="light|dark"`, `data-accent-color="#hex"`, `data-position`,
   `data-hide-badge="true"`.
2. Buttons on existing markup (slug = product slug from the store):
   `data-nanocart-buy="slug"` (add + open cart), `data-nanocart-add="slug"` (same),
   `data-nanocart-product="slug"` (detail modal WITH variant selector).
   IMPORTANT: buy/add auto-select the first variant silently — use the product modal
   for products with size/color options.
   `<nanocart-product slug="..."></nanocart-product>` renders a standalone card;
   `<nanocart-signup></nanocart-signup>` renders email capture (Pro+ stores).
3. Theme via page CSS on the shadow host:
   `#nanocart-widget { --nc-accent: ...; --nc-bg: ...; --nc-radius: ...; --nc-font: ...; }`
   (also: --nc-surface, --nc-border, --nc-text, --nc-text-muted, --nc-text-dim,
   --nc-text-strong, --nc-danger, --nc-success). Page CSS > script attrs > portal.
4. If the store restricts Allowed Domains (portal Settings), the site's domain must be
   listed or public calls 403 (DOMAIN_NOT_ALLOWED).
5. Verify: cart badge appears; wired button opens the drawer/modal. Console error
   "[nanocart] Missing required data-store-id" means the script tag is wrong.

## Manage the catalog (Admin API — server-side/CLI only; browser calls fail CORS)

Base https://api.nanocart.io, header `x-api-key: $NANOCART_API_KEY`. Prices in CENTS.
- Upload image: POST /shop/{storeId}/admin/upload-url
  {fileName, contentType, uploadType:"product_image", fileSizeMB} -> PUT file to
  uploadUrl -> use fileUrl.
- Create product: POST /shop/{storeId}/admin/products
  {name, price, description?, images?, categoryId?, variants?, options?,
   productType?: "physical"|"digital", status: "active"} (default draft = hidden).
- Categories: POST /shop/{storeId}/admin/categories {name}. Coupons:
  POST /shop/{storeId}/admin/coupons {code, type, value}.
- Public sanity check (no auth): GET /shop/{storeId}/products.
- Ask before DELETEs or key regeneration. Never create products in Stripe — the
  catalog lives in NanoCart; Stripe only processes payments.

Full docs: https://docs.nanocart.io · AI-readable: https://docs.nanocart.io/llms-full.txt
· API reference: https://nanocart.io/docs
