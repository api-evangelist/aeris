---
name: aeris-investigate-a-device-connectivity-problem
description: Work out why an Aeris-connected device is offline — inventory, state, location, signalling history — and raise an incident if it is not the device.
api: Aeris IoT Accelerator REST API
base_url: https://iot-api.aeris.com
operations:
  - listdevices
  - deviceDetails
  - getSubscriptionDetails
  - querySubscriptionHistory
  - get-cell-global-identity-of-subscription
  - TopUsageSubscriptions
generated: '2026-09-10'
method: generated
source: openapi/aeris-subscription-device-api-openapi.yaml, openapi/aeris-mds-esb-subscription-management-openapi.yaml, openapi/aeris-subscription-location-api-openapi.yaml, openapi/aeris-subscription-signalling-events-api-openapi.yaml, openapi/aeris-incidents-external-api-openapi.yaml, openapi/aeris-business-analytics-report-api-openapi.yaml
---

# Investigate a device connectivity problem

A device is not reporting. Work outward from the subscription record before blaming the network.

Authenticate first — most steps below are Bearer, but Subscription Management is `X-Access-Token`.
See `aeris-authenticate-to-the-iot-accelerator-api`.

## 1. Confirm the subscription exists and is in the state you think

`deviceDetails` — `GET /devices/{idType}/{value}` — address the device by IMSI, ICCID or MSISDN.
`listdevices` — `GET /devices` — for the surrounding inventory.

Subscriptions are addressed polymorphically: nearly every path takes a `subscription-id-type` of
IMSI, ICCID or MSISDN plus a `subscription-id`. Getting the type wrong returns `404`, not a validation
error, so check the type before concluding the device is unknown.

## 2. Check what changed

`querySubscriptionHistory` — `GET /subscriptions/{id}/history` — did someone suspend it, move it to a
different package, or change its region?

The Subscription Change History API (`iot/api/subscriptions/changes`) covers the same ground for a
batch of subscriptions.

## 3. Check where it last was

`get-cell-global-identity-of-subscription` — `GET /locations/cell-global-identities` — returns the
Cell Global Identity for the subscription. A stale or absent CGI is a different problem from a device
that is attached but silent.

## 4. Check signalling and usage

The signalling events surface (`iot/api/xdr/events`) and signalling usages surface
(`iot/api/xdr/reports`) carry the per-subscription network record. Note these are the tightest limits
on the platform — **5 requests per second, 25 per minute** — so batch rather than looping per device.

`TopUsageSubscriptions` — `GET /aggregated-traffic-usages/top-usage-subscriptions` — useful in the
opposite direction, when a device is consuming far more than expected.

## 5. Raise it

If the subscription is healthy and the network is not, the Incident Management API
(`iot/api/ts`) creates and tracks a ticket. Aeris publishes no public status page, so this
authenticated surface is the only way to find out whether an outage is known.

## Watch the limits

Diagnostic work is loop-shaped and this platform is per-path rate limited. Read
`X-RateLimit-Remaining-Second` and `X-RateLimit-Remaining-Minute` on every response and back off before
you hit `429`. Full table: `rate-limits/aeris-rate-limits.yml`.
