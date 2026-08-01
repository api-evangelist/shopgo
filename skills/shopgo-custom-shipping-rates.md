---
name: Provide custom ShopGo shipping rates via webhook
description: >-
  Configure and implement the ShopGo calculate-shipping-rates webhook to return
  custom shipping rates at checkout.
api: openapi/shopgo-management-openapi.yml
operations: [getWebhookUrl, changeWebhookUrl]
---

# Provide custom ShopGo shipping rates

ShopGo's `calculate-shipping-rates` webhook lets you override shipping rates at
checkout. It is a synchronous callback: ShopGo POSTs the current rates and
checkout addresses to your URL and uses the `[ShippingRate]` array you return.

## Configure the webhook URL
- Read the current URL — `getWebhookUrl`
  (`GET /v1/management/settings/webhook/calculate-shipping-rates`).
- Set it — `changeWebhookUrl`
  (`PATCH /v1/management/settings/webhook/calculate-shipping-rates`) with
  `{ "url": "https://your-service/webhooks/shipping" }`.
- Requires the `settings` permission; `X-API-KEY` on every request.

## Implement the endpoint
1. Receive `{ "rates": [ShippingRate], "checkout": { "shipping": Address, "destination": Address } }`.
2. Return an updated `[ShippingRate]` body. Each `ShippingRate` has `id`, `price`
   (Money), `duration` (`value` + `unit` of days/minutes/hours), bilingual `name`
   and `description`, and a machine-friendly `provider` (e.g. `dhl`).
3. You may return the list unchanged, modify it, or replace it entirely.

## Notes
- The response body — not a stored resource — determines the rates shown.
- See `asyncapi/shopgo-webhooks.yml` for the full object shapes.
