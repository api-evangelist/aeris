---
name: aeris-authenticate-to-the-iot-accelerator-api
description: Obtain and reuse an Aeris IoT Accelerator access token, choosing the right mechanism for the API group you are calling.
api: Aeris IoT Accelerator REST API
base_url: https://iot-api.aeris.com
operations:
  - POST /iot/api/auth/token
  - GET /iot/api/auth/certs
  - reissueToken
  - formLoginV1_1
generated: '2026-09-10'
method: generated
source: openapi/aeris-auth-3.0-openapi.yaml, openapi/aeris-mds-esb-subscription-management-openapi.yaml, https://iotdeveloper.aeris.net/hc/en-us/articles/25348523998748-API-Quick-start-guide, https://iotdeveloper.aeris.net/hc/en-us/articles/25348574275868-JWT-Authentication-Best-Practices
---

# Authenticate to the Aeris IoT Accelerator API

Aeris does not use one authentication mechanism. Which one you need depends on the API group,
and picking the wrong one is the most common first failure on this platform.

## 1. Decide which mechanism the target API group uses

| API group | Mechanism |
|---|---|
| Shared bundle, Enterprise Management, eUICC Setup, Device Reconnect, Subscription Change History, Subscription management additional functions, Search Subscription Details, Incident ticketing, Custom fields, Subscription location, Subscription signalling usages, Subscription signalling events, Devices eUICC inventory | Bearer JWT — `POST /token` |
| User Administration, Consumer Connectivity, Subscription Management, Device Localization | `X-Access-Token` — `POST /login` |
| SMS Messaging | HTTP Basic |
| Enterprise Provisioning for Marketplace HUB | Bearer JWT — `POST /auth/login` |
| Service Portal SOAP | WS-Security `UsernameToken` in the SOAP header |
| AerAdmin, AerFrame, AerTraffic (Aeris-native) | `?apiKey=` query parameter |

The full table is in `authentication/aeris-authentication.yml` under `docs_findings.by_api_group`.

## 2. Get a bearer token

`POST https://iot-api.aeris.com/iot/api/auth/token` with the credentials your Connectivity Service
Provider issued. The response carries the JWT in `access_token`. Send it as `Authorization: Bearer <token>`.

There is no self-serve signup. Credentials come from the Service Provider at subscription time, and
the same credentials work for both the Service Portal UI and the API.

## 3. Reuse the token — do not re-authenticate per request

The JWT payload carries an `exp` claim (UNIX seconds). Decode it, cache the token, and only call the
auth endpoint again once `exp` has passed. Aeris documents this explicitly: `/iot/api/auth` is itself
rate limited to **5 requests per second and 60 per minute**, so a client that authenticates before
every call will throttle itself.

Useful claims on the token: `organization_ids` and `enterprise_group_ids` scope what you can see, and
`groups` carries the permission set. A `404` on an identifier you believe is valid usually means it
sits outside your `organization_ids` scope, not that it does not exist.

`GET /iot/api/auth/certs` returns the signing certificates if you want to verify the token yourself.

## 4. For X-Access-Token groups

Call the group's own `POST /login` (`formLoginV1_1`) and send the returned token in the
`X-Access-Token` header. `reissueToken` (`POST /token-reissue`) refreshes it.

## Errors

- `401` — token missing, malformed or expired. Get a new one; do not retry the same token.
- `403` — token is valid but lacks the OAuth scope or organization membership. See `scopes/aeris-scopes.yml`.
- `429` — you are re-authenticating too often. See step 3.

Full catalogue: `errors/aeris-problem-types.yml`.
