# NanoCart Admin API — catalog automation reference

**Server-side / CLI only.** Base `https://api.nanocart.io`. Auth header:
`x-api-key: sc_live_...` (from `.env` → `NANOCART_API_KEY`). Browser calls will fail
(CORS locked to the NanoCart portal). ALL prices are integer **cents**.

```bash
curl -s https://api.nanocart.io/shop/$NANOCART_STORE_ID/admin/products \
  -H "x-api-key: $NANOCART_API_KEY" -H "Content-Type: application/json" \
  -X POST -d '{"name":"Classic Tee","price":2400,"status":"active"}'
```

## Products — `POST/PUT/DELETE /shop/{storeId}/admin/products`

Create (POST): `name`* , `price`* (cents). Optional: `description` (HTML ok),
`categoryId`, `compareAtPrice` (> price), `inventory` (null = untracked/unlimited),
`images: [fileUrl, ...]` (upload first — see Uploads), `variants`, `options`,
`productType: "physical"|"digital"` (default physical), `status: "draft"|"active"`
(default draft — set `"active"` to publish), `featured`, `slug` (auto from name),
`taxable` (default true), `tags[]`, `shippingCost` (cents, per_item shipping),
`fulfillmentVendorId` + `vendorSku` + `vendorNotes` (route orders to a fulfillment
vendor — see Fulfillment Vendors; variants may carry their own `vendorSku`).
Returns 201 `{product: {productId, slug, ...}}`.

Variants + options:
```json
{
  "options": [{"name": "Size", "values": ["S", "M", "L"]}],
  "variants": [
    {"name": "S", "price": 2400, "sku": "TEE-S", "inventory": 10, "optionValues": {"Size": "S"}},
    {"name": "M", "price": 2400, "sku": "TEE-M", "inventory": 10, "optionValues": {"Size": "M"}}
  ]
}
```

Digital products: `productType: "digital"` + upload the file with
`uploadType: "digital_file"`; buyers get expiring download links (72h, ≤5 downloads).

Update (PUT): `{productId, ...partial fields}`. Delete (DELETE): `{productId}` —
archives (hidden from public, kept in DB).

## Uploads — `POST /shop/{storeId}/admin/upload-url` (two-step presigned)

Request: `{fileName, contentType, uploadType, fileSizeMB}` where uploadType ∈
`product_image | digital_file | category_image | storefront_image`. Response:
`{uploadUrl, fileUrl, s3Key}`. Then:

```bash
curl -X PUT "$uploadUrl" -H "Content-Type: image/jpeg" --data-binary @photo.jpg
```

Use `fileUrl` in the product's `images[]` / category's `image`. File size is validated
against the store's tier limit.

## Categories — `POST/PUT/DELETE /shop/{storeId}/admin/categories`

Create: `{name, description?, image?, parentId?, sortOrder?, status?: "active"|"hidden"}`.
Update: `{categoryId, ...fields}`. Delete: `{categoryId}` → hidden (children untouched).

## Coupons — `GET/POST/PUT/DELETE /shop/{storeId}/admin/coupons`

Create: `{code, type: "percent_off"|"fixed_amount"|"free_shipping", value,
minOrderAmount?, maxUses?, expiresAt?}`. `value` = 1–100 for percent_off, cents for
fixed_amount. Codes auto-uppercase.

## Settings — `GET/PUT /shop/{storeId}/admin/settings`

PUT one key at a time: `{"settingKey": "...", "value": {...}}`. Keys:
- `shipping_config`: `{method: "flat_rate"|"per_item"|"tiered"|"free", flatRate,
  freeShippingEnabled, freeShippingThreshold, defaultItemShippingCost, tiers[],
  localPickupEnabled, localPickupInstructions}` (Free tier: flat_rate/free only)
- `tax_config`: `{enabled, useStripeTax, defaultRate}` (defaultRate = percent)
- `email_config`: `{fromEmail, fromName}` · `order_config`: `{orderPrefix, nextOrderNumber}`

## Orders (read/manage)

`GET /shop/{storeId}/admin/orders` (filters/pagination), `PUT` to update status,
`POST /shop/{storeId}/admin/orders/{orderId}/resend-confirmation`.

## Fulfillment Vendors — `GET/POST /shop/{storeId}/admin/vendors` (Pro/Expert)

Custom partners (local print shop, drop shipper) that get an emailed **order sheet**
when their routed products sell. Vendor emails **never contain prices** — the template
system has no price tags by design.

Create: `{name*, email: {to*: ["addr"], cc?: []}, subjectTemplate?, templateHtml?,
includeShipping? (default true), includeCustomerContact? (default false), notes?,
status?: "active"|"paused"}`. Max 10 vendors, 5 To + 5 Cc each.

- `templateHtml` empty = standard order sheet. Custom HTML **must** include
  `{{items_table}}` or an `{{#items}}…{{/items}}` block (else 400
  `NO_ITEMS_PLACEHOLDER`). Tags: `{{order.number}}` `{{order.date}}` `{{store.name}}`
  `{{store.logo}}` `{{shipping_address}}` `{{vendor.name}}` `{{vendor.notes}}`; inside
  item blocks: `{{item.quantity}}` `{{item.name}}` `{{item.variant}}`
  `{{item.vendorSku}}` `{{item.sku}}` `{{item.notes}}` `{{item.image}}`.
- `GET/PUT/DELETE /admin/vendors/{vendorId}` — **PUT is full replace**: GET first,
  modify, PUT the complete record, or omitted fields (like `templateHtml`) are reset.
- `POST /admin/vendors/draft` `{description, currentHtml?}` → AI-drafted HTML (free).
- `POST /admin/vendors/{vendorId}/test` → sample order sheet emailed to the MERCHANT.
- `POST /admin/orders/{orderId}/send-vendor` `{vendorId}` → manual send/re-send.
- Route products via the product fields above; orders then dispatch automatically at
  payment. Failures appear on the order's `podFulfillments` (provider `"vendor"`) and
  retry via `POST /admin/orders/{orderId}/retry-pod` `{"provider": "vendor"}`.

## Donations — `GET/POST /shop/{storeId}/admin/donations/campaigns` (every plan)

Campaigns power `data-nanocart-donate="slug"` and `<nanocart-donate-stats>`.
Create: `{campaignId* (public slug, immutable), name*, description?, buttonText?,
successUrl?, suggestedOneTime?: [cents], suggestedMonthly?: [cents] (≤8 each,
100–999999), allowCustomAmount? (one-time only), allowOneTime?, allowRecurring?,
defaultFrequency?, goalAmountCents?, goalMetric?: "month_total"|"mrr"|"all_time",
statsConfig?: {showCount, showRaised, showMonthly, showGoal, showButton},
hideBranding? (Pro/Expert), status?: "active"|"paused"}`.

- Caps: 1 campaign Free/Standard, 10 Pro/Expert (403 TIER_LIMIT); duplicate slug 409.
- `GET/PUT/DELETE .../campaigns/{campaignId}` — **PUT is full replace** (GET → modify
  → PUT; counters + createdAt preserved server-side, slug immutable).
- `statsConfig` controls the PUBLIC stats widget — disabled fields never leave the
  API, so count-only-no-money is enforceable.
- Records: `GET /admin/donations?kind=one_time|recurring|recurring_payment&limit&lastKey`;
  supporters: `GET /admin/donations/supporters`; report:
  `GET /admin/donations/report?from&to` (range fold + live stats + per-campaign).
- Public (no auth): `GET /shop/{storeId}/donation-config/{slug}`,
  `GET .../donation-stats/{slug}`, `POST .../donate` `{campaignId, amountCents,
  frequency, email?, processor?}` → `{sessionUrl}`. `_default` slug = oldest active
  campaign. Monthly = Stripe only + requires the 4 subscription webhook events on the
  merchant's Stripe endpoint.

## Safety rules

- Ask before DELETE operations or `regenerate-key` (regenerating invalidates the old
  key immediately).
- Never echo the API key into committed files, logs, or frontend bundles.
- Products created as `draft` are invisible to the public API — set `active` when the
  user wants them live.
