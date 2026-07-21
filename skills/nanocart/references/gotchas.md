# NanoCart gotchas — the mistakes integrators actually make

1. **`data-store-id`, not `data-store`.** The widget only reads `data-store-id`. A
   missing/wrong attribute logs `[nanocart] Missing required data-store-id attribute on
   the script tag.` and the widget stays dead. (Some older docs showed `data-store` —
   wrong.)

2. **Never put the API key in the script tag or any frontend code.** The script tag
   takes the public Store ID. The `sc_live_...` key is full admin access; it belongs in
   `.env` on a server. If you see a site with the key in HTML, flag it and tell the user
   to regenerate the key in Settings → API Keys.

3. **Variant products + buy/add buttons:** `data-nanocart-buy`/`data-nanocart-add`
   silently add the FIRST variant. No prompt. Use `data-nanocart-product` (detail modal
   with variant selector) whenever options matter.

4. **Admin API is server-side only.** Browser calls fail CORS (admin origin is locked to
   the NanoCart portal, and preflights don't reliably allow `x-api-key`). Use curl, a
   build script, or a backend.

5. **Allowed Domains 403s.** If Settings → Allowed Domains is non-empty, public API
   calls from any origin NOT on the list return
   `403 {"code": "DOMAIN_NOT_ALLOWED"}`. Add every domain where the widget runs,
   including `www.` variants. Empty list = all origins allowed. Server-to-server calls
   (no Origin header) always pass.

6. **Prices are integer cents.** Everywhere. $19.99 = `1999`. Sending `19.99` creates a
   20-cent product.

7. **Don't create products in Stripe.** Stripe only processes payments; the catalog
   lives in NanoCart. Stripe's onboarding suggests adding products — merchants should
   skip that step.

8. **Custom domains:** hosted storefronts on custom domains resolve
   `GET /shop/resolve-domain?domain=shop.example.com` → `{storeId}`, then use per-store
   endpoints. On `{store}.nanocart.io` the subdomain IS the storeId.

9. **Checkout flow:** `POST /shop/{storeId}/checkout` returns `{sessionUrl, orderId}`.
   Persist `orderId` (localStorage) BEFORE redirecting to `sessionUrl` — the return
   redirect relies on Referer/Origin and the success page will want the orderId for
   `GET /shop/{storeId}/orders/{orderId}?email=...`.

10. **Draft vs active:** admin-created products default to `draft` and won't appear in
    the public products list. Set `status: "active"` to publish.

11. **Signup element renders nothing** on Free/Standard stores or when Subscribers is
    disabled in the portal — that's by design, not a bug. Pro/Expert + enabled required.
