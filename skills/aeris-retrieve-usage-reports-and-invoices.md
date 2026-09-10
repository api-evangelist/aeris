---
name: aeris-retrieve-usage-reports-and-invoices
description: Pull organization usage, traffic and invoice data out of the Aeris IoT Accelerator Business Analytics Report API.
api: Aeris IoT Accelerator REST API
base_url: https://iot-api.aeris.com/iot/api/business-analytics-service/v1
operations:
  - QueryReports
  - DownloadReport
  - QueryInvoices
  - DownloadInvoice
  - DeleteInvoice
  - QueryUsage
  - DownloadUsage
  - TopUsageSubscriptions
generated: '2026-09-10'
method: generated
source: openapi/aeris-business-analytics-report-api-openapi.yaml
---

# Retrieve Aeris usage reports and invoices

Bearer JWT. Authenticate first — see `aeris-authenticate-to-the-iot-accelerator-api`.

## The pattern is list-then-download

Every family here works the same way: a query operation returns metadata with an id, and a second
operation downloads the artifact by that id.

- `QueryReports` — `GET /reports` → `DownloadReport` — `GET /reports/{id}`
- `QueryInvoices` — `GET /invoices` → `DownloadInvoice` — `GET /invoices/{id}`
- `QueryUsage` — `GET /usages` → `DownloadUsage` — `GET /usages/{id}`

Downloads come back as `application/octet-stream`. Do not assume JSON.

## Aggregated traffic

`TopUsageSubscriptions` — `GET /aggregated-traffic-usages/top-usage-subscriptions` — the highest-usage
subscriptions in the organization, without pulling a full report.

## Destructive operation

`DeleteInvoice` — `DELETE /invoices` — **has no published reversal and no idempotency key.** Nothing in
the contract or the docs describes restoring a deleted invoice. Confirm the target with `QueryInvoices`
before calling it.

## Rate limits

Every path in this group is **5 requests per second, 50 per minute**:

- `iot/api/business-analytics-service/v1/reports`
- `iot/api/business-analytics-service/v1/invoices`
- `iot/api/business-analytics-service/v1/usages`
- `iot/api/business-analytics-service/v1/aggregated-traffic-usages/top-usage-subscriptions`

These are report endpoints, so a natural "download everything for the month" loop will throttle.
Read `X-RateLimit-Remaining-Minute` and pace to it.

## Scheduled alternative

The legacy AerTraffic Reports API (`https://aertrafficapi.aeris.com/v1`) supports scheduled report
templates and archives, which the REST Business Analytics API does not. Docs only, no contract:
https://support.aeris.net/hc/en-us/articles/360037394973-AerTraffic-Reports-API
