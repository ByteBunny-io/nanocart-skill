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
`taxable` (default true), `tags[]`, `shippingCost` (cents, per_item shipping).
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

## Safety rules

- Ask before DELETE operations or `regenerate-key` (regenerating invalidates the old
  key immediately).
- Never echo the API key into committed files, logs, or frontend bundles.
- Products created as `draft` are invisible to the public API — set `active` when the
  user wants them live.
