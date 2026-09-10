---
name: aeris-send-sms-to-a-device
description: Send a mobile-terminated SMS to a device on the Aeris network, poll its delivery status, and read inbound messages.
api: Aeris IoT Accelerator SMS Messaging API
base_url: https://sms.iot-api.aeris.com/dcpapi/smsmessaging/v1
operations:
  - sendSMS
  - querySMS
  - retrieveSMS
generated: '2026-09-10'
method: generated
source: openapi/aeris-sms-messaging-api-openapi.yaml
---

# Send an SMS to a device on the Aeris network

This API is **HTTP Basic**, not bearer. It is the one IoT Accelerator group that does not take a JWT.

The contract states it is based on the **OneAPI SMS interface / OMA REST NetAPIs v1**, so if you
already have a OneAPI client the resource shape will be familiar.

## 1. Send

`sendSMS` — `POST /outbound/{senderAddress}/requests`

`senderAddress` uses the `tel:` URI form, e.g. `tel:12345`. The response carries a `resourceURL` that
already contains the `requestId` — keep it, it is the handle for step 2.

## 2. Poll delivery

`querySMS` — `GET /outbound/{senderAddress}/requests/{requestId}/deliveryInfos`

Returns per-recipient delivery info. SMS on a cellular IoT network is asynchronous and devices are
frequently unreachable, so poll rather than assuming the send succeeded.

## 3. Read inbound messages

`retrieveSMS` — `GET /inbound/registrations/{registrationID}/messages`

Returns queued mobile-originated messages for a registration.

## Reversibility

**None.** A sent SMS cannot be recalled. There is no cancel, void or reverse operation on this API,
and no idempotency key — so a retried `sendSMS` sends a second message. If a send times out, poll
`querySMS` with the `requestId` before retrying.

## Alternative surface

The Aeris-native **AerFrame** API (`https://api.aerframe.aeris.com`) covers the same job on the older
Aeris network with an application/notification-channel model and callbacks, plus MO-SMS acknowledgement
and network registration reset. Aeris publishes no machine-readable contract for it — docs only:
https://support.aeris.net/hc/en-us/articles/360036914654-AerFrame-Device-Communication-and-Control-API
