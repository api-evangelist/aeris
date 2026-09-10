---
name: aeris-order-and-activate-a-subscription
description: Order SIMs or eUICC profiles on the Aeris IoT Accelerator platform, track the request, and bring a subscription into service.
api: Aeris IoT Accelerator REST API
base_url: https://iot-api.aeris.com
operations:
  - simOrder
  - simStockOrder
  - euiccOrder
  - euiccStockOrder
  - subscriptionProfileStockOrder
  - getSubscriptionRequest
  - getSubscriptionRequestIds
  - getPendingOrderApprovals
  - approveOrder
  - rejectOrder
  - executeCancelSimOrder
  - SubscriptionStateChange
generated: '2026-09-10'
method: generated
source: openapi/aeris-mds-esb-subscription-management-openapi.yaml, openapi/aeris-subscription-inventory-common-api-openapi.yaml
---

# Order and activate an Aeris subscription

Authenticate first — see `aeris-authenticate-to-the-iot-accelerator-api`.

## 1. Place the order

Pick the operation that matches what you are ordering:

- `simOrder` — `POST /subscriptions/order` — SIMs against an existing specification.
- `simStockOrder` — `POST /subscriptions/stockOrders` — stock replenishment.
- `euiccOrder` — `POST /euiccs/euiccOrder` — eUICC hardware.
- `euiccStockOrder` — `POST /euiccs/stockOrder`.
- `subscriptionProfileStockOrder` — `POST /euiccs/profileOrder` — profiles onto existing eUICCs.
- `euiccsMultiProfileOrder` — `POST /euiccs/multiProfileOrder`.

`simOrderMetaData` (`GET /subscriptions/order/metadata`) and `simStockOrderMetaData` tell you what the
order body must contain before you build one.

**There is no dry-run.** No operation in the Aeris estate accepts a validate-only or preview flag, so
the metadata call is the only rehearsal available.

**There is no idempotency key on this surface.** Only four IoT Watchtower gateway operations accept
`Idempotency-Key`; ordering does not. If a `POST /subscriptions/order` times out, do **not** blind-retry
it — resolve the outcome first with step 2.

## 2. Track the request

Orders are asynchronous. Keep the request id from the response and poll:

- `getSubscriptionRequest` — `GET /subscriptions/requests/{requestId}`
- `getSubscriptionRequestIds` — `POST /subscriptions/requests/find` — recover a request id you lost,
  which is how you disambiguate a timed-out order instead of retrying it.

## 3. Approvals, where they apply

- `getPendingOrderApprovals` — `GET /subscriptions/pendingOrders`
- `approveOrder` — `POST /subscriptions/orders/{orderId}:approve`
- `rejectOrder` — `DELETE /subscriptions/orders/{orderId}`

## 4. Bring the subscription into service

`SubscriptionStateChange` — `POST /requests/state-changes` on the Subscription Management Additional
Functions API — is the general state-change request (activate, suspend, resume, terminate).

## 5. Reversal

`executeCancelSimOrder` — `DELETE /sim-orders/{request-id}` — cancels SIM stock orders, SIM orders,
eUICC stock orders, eUICC orders and subscription profile stock orders.

**Aeris publishes no cancellation window.** The operation exists; how long it keeps working after the
order is placed is not documented anywhere. Treat the window as unknown and cancel promptly rather
than assuming one exists. See `conventions/aeris-conventions.yml` under `reversibility`.

## Rate limits

`iot/api/subscriptions` 10/s, 100/min. `iot/api/sica/subscriptions/sim-orders` 10/s, 100/min.
Read `X-RateLimit-Remaining-Second` and `X-RateLimit-Remaining-Minute` off every response.
Full table: `rate-limits/aeris-rate-limits.yml`.
