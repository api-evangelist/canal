---
name: Manage a Rokt Catalog (Canal) supplier catalog and fulfill orders
description: >-
  Publish products and variants, keep inventory in sync, and create/update
  fulfillments as a Brand (supplier) integration.
api: openapi/canal-openapi-original.yml
operations:
  - products_create
  - variants_update
  - variants_create
  - fulfillments_create
  - fulfillments_update
  - refunds_create
---

# Manage a supplier catalog and fulfill orders

Base URL `https://api.shopcanal.com/platform`. Authenticate with `X-CANAL-APP-ID`
and `X-CANAL-APP-TOKEN`. These operations are **Supplier-only**.

## Steps

1. **Publish a product** — `POST /products/` (`products_create`) with `title`,
   `body_html`, and at least one entry in `variants[]` (each with `price`, `sku`,
   `inventory_quantity`, `option1`). Returns the product with Catalog UUIDs.
2. **Add variants** — `POST /variants/` (`variants_create`) with the parent
   `product_id`.
3. **Sync price / inventory** — `PATCH /variants/{id}/` (`variants_update`);
   price and inventory changes propagate asynchronously to connected Storefronts.
4. **Fulfill an order** — when you receive an `order/create` webhook, ship it and
   `POST /fulfillments/` (`fulfillments_create`) with `order_id`, tracking fields,
   and the `line_items[]` (Catalog line-item `id` + `quantity`). Emits a
   `fulfillment/update` webhook and syncs tracking back to the Storefront.
5. **Update tracking** — `PATCH /fulfillments/{id}/` (`fulfillments_update`).
6. **Refund** — `POST /refunds/` (`refunds_create`) with `order_id` and
   `line_items[]`; watch for `ITEMS_ALREADY_REFUNDED` / `SK_REFUND_FAILED` error codes.

## Rules
- Deleting the last variant of a product is rejected — delete the product instead.
- Errors are a custom envelope (`error_code` + `message`), not RFC 9457.
- Refund quantities may not exceed the currently refundable quantity per line item.
