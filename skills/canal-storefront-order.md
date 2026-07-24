---
name: Place and track a Rokt Catalog (Canal) order from a Storefront
description: >-
  Record a customer order that contains Catalog-sourced products, quote shipping,
  and track fulfillment/cancellation as a Storefront (partner) integration.
api: openapi/canal-openapi-original.yml
operations:
  - shipping_rates_create
  - orders_create_or_get_create
  - orders_retrieve
  - orders_cancel_create
  - fulfillments_list
---

# Place and track a Storefront order

Base URL `https://api.shopcanal.com/platform`. Authenticate every request with the
`X-CANAL-APP-ID` and `X-CANAL-APP-TOKEN` headers (from the Developer tab). These
operations are **Storefront-only**.

## Steps

1. **Quote shipping** — `POST /shipping-rates/` (`shipping_rates_create`) with the
   customer's `shipping_address` and `line_items[]` (each `variant_id` UUID +
   `quantity`). Present the returned `rates[]` at checkout.
2. **Create the order idempotently** — `POST /orders/create_or_get/`
   (`orders_create_or_get_create`) with the full order payload including your unique
   `order_name`. Because it is keyed on `order_name`, retrying after a network error
   returns the existing order (200) instead of creating a duplicate. Handle `202`
   (dispatch deferred by `order_forwarding_delay`) as success.
3. **Read order state** — `GET /orders/{id}/` (`orders_retrieve`) to inspect
   `fulfillment_status` and `financial_status`.
4. **Track shipments** — `GET /fulfillments/` (`fulfillments_list`) for tracking
   numbers created by the Supplier.
5. **Cancel if needed** — `POST /orders/{id}/cancel/` (`orders_cancel_create`);
   emits an `order/cancel` webhook.

## Rules
- Idempotency: always use `orders_create_or_get_create`, not bare `orders_create`, for retry safety.
- A `422` means a variant is paused/unlisted — re-check availability before ordering.
- Subscribe to `order/update` and `fulfillment/create` webhooks (HMAC-SHA256, `X-CANAL-EVENT-HASH`) instead of polling.
