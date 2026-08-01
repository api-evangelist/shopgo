---
name: Process a ShopGo order (payment, authorization, fulfillment)
description: >-
  Walk a ShopGo order through its lifecycle — retrieve it, record a payment,
  authorize it for fulfillment, and advance its shipment — using the Management API.
api: openapi/shopgo-management-openapi.yml
operations: [testAuthentication, getOrder, createOrderPayment, updateOrder, updateShipment]
---

# Process a ShopGo order

Use the ShopGo Management API (`https://api.shopgo.me`) to take an order from
payment through fulfillment. Requests and responses are `application/json`;
success responses carry `{ "result": "success", "payload": {...} }` and errors
carry `{ "result": "error", "description": "..." }`.

## Authentication
- Send `X-API-KEY: <dashboard user API key>` on every request.
- The user's key must hold the `orders` permission or calls return `403 Forbidden`.
- If the tenant's trial has expired without a subscription, calls return `402 Payment Required`.
- Verify the key first with `testAuthentication` (`GET /v1/management/auth/test`).

## Steps
1. **Retrieve the order** — `getOrder` (`GET /v1/management/order/{number}`). Read
   `total`, `confirmed`, `authorized`, `payments[]`, `shipments[]`.
2. **Record payment** — `createOrderPayment` (`POST /v1/management/order/{number}/payment/`).
   An empty body settles the balance as cash-on-delivery. For a refund, set
   `method: refund` with a negative `total`.
3. **Authorize fulfillment** — `updateOrder` (`PATCH /v1/management/order/{number}`)
   with `{ "authorized": true }`. A shipment can only advance once its order is authorized.
4. **Advance the shipment** — `updateShipment`
   (`PATCH /v1/management/order/{number}/shipment/{id}`) with
   `{ "state": "ready" | "shipped" | "delivered" }`.
5. **(Optional) Cancel** — `updateOrder` with `{ "cancelled": true, "restock": true }`
   to cancel and free reserved stock. Cancellation does not block later refunds.

## Notes
- There is no idempotency-key mechanism — do not blindly retry `createOrderPayment`
  or you may double-charge; re-fetch the order and inspect `payments[]` first.
- `Order.payment_method` is the shopper's chosen method; `Payment.method` is what
  was actually used — they can differ.
- See `conventions/shopgo-conventions.yml` and `errors/shopgo-problem-types.yml`.
