---
name: Compare EnergyAustralia energy plans
description: >-
  Retrieve and compare EnergyAustralia's published electricity and gas plans from the
  public Consumer Data Right Product Reference Data surface. No key, no registration and
  no consent required — this is the one EnergyAustralia surface any agent can call today.
api: openapi/energyaustralia-cds-energy-api-openapi.yml
base_url: https://cdr.energymadeeasy.gov.au/energyaustralia/cds-au/v1
auth: none
operations:
  - listEnergyPlans
  - getEnergyPlanDetail
generated: '2026-07-27'
method: generated
---

# Compare EnergyAustralia energy plans

## What this surface is

EnergyAustralia's plan data is **not hosted by EnergyAustralia**. In CDR banking each bank
stands up its own public product endpoint; in CDR energy the Australian Energy Regulator is
the designated data holder for product data and serves every retailer from one gateway on a
per-brand path. The base URL is therefore
`https://cdr.energymadeeasy.gov.au/energyaustralia/cds-au/v1`.

This is public. Do not attempt to authenticate. Do not send a bearer token.

## Before you start

Two rules govern every call:

1. **`x-v` is mandatory on every request.** It is the *payload* version, and it is set
   **per endpoint**, not per API. Omitting it, or sending a non-integer, returns HTTP 400
   `urn:au-cds:error:cds-all:Header/InvalidVersion`.
2. **Versions differ between endpoints.** `listEnergyPlans` answers at `x-v: 1`.
   `getEnergyPlanDetail` requires `x-v: 3` — sending `x-v: 1` returns HTTP 406
   `urn:au-cds:error:cds-all:Header/UnsupportedVersion` with the supported minimum in the
   detail. If you are unsure, send `x-min-v` alongside `x-v` and let the holder pick.

## Step 1 — list the plans (`listEnergyPlans`)

`GET /energy/plans` with header `x-v: 1`.

Useful query parameters, all declared in the spec:

| Parameter | Values | Default |
|---|---|---|
| `type` | `STANDING`, `MARKET`, `REGULATED`, `ALL` | `ALL` |
| `fuelType` | `ELECTRICITY`, `GAS`, `DUAL`, `ALL` | `ALL` |
| `effective` | `CURRENT`, `FUTURE`, `ALL` | `CURRENT` |
| `updated-since` | DateTimeString | — |
| `brand` | string | — |
| `page` | PositiveInteger | — |
| `page-size` | PositiveInteger, max 1000 | — |

Read `meta.totalRecords` and `meta.totalPages` to size the walk, then follow `links.next`
until it is absent. Requesting `page-size` above 1000 returns HTTP 400
`Field/InvalidPageSize`; requesting a page beyond the range returns HTTP 422
`Field/InvalidPage`.

Each `data.plans[]` entry carries `planId`, `effectiveFrom`, `effectiveTo`, `type`,
`fuelType`, `brand`, `brandName`, `customerType` and `geography`. Filter on
`brand == "energyaustralia"` if you are working from a wider dataset.

## Step 2 — fetch the detail (`getEnergyPlanDetail`)

`GET /energy/plans/{planId}` with header `x-v: 3`.

`planId` values are fully qualified and include an origin suffix — for example
`ENE831725MRE15@EME`. URL-encode the `@`.

The response carries `electricityContract` and/or `gasContract` (conditional on
`fuelType`), each an `EnergyPlanContractV3` holding the tariff periods, discounts,
incentives, fees, eligibility rules, solar feed-in tariffs, controlled load and GreenPower
charges. This is where the comparable economics live — the list response does not contain
pricing.

An unknown or retired `planId` returns HTTP 404
`urn:au-cds:error:cds-all:Resource/Invalid`, which is *permanent*. Do not retry it.

## Step 3 — comparing

To compare like for like, group by `fuelType` and `geography` before comparing tariff
periods. Plans are jurisdiction-scoped: an offer available in the Ausgrid network is not
comparable to one in SA Power Networks. `type` matters too — a `REGULATED` plan is a
default offer, not a market offer.

## Error handling

All errors return `application/json` with an `errors[]` array — **not** RFC 9457
`application/problem+json`. Each element has `code` (a `urn:au-cds:error:` URN), `title`
and `detail`. See `errors/energyaustralia-problem-types.yml` for the full catalog.

Retry policy: `Retry-After` is honoured on HTTP 429 and is exposed to browser clients via
CORS. `Resource/Invalid` is permanent; `Resource/Unavailable` is transient.

## Tracing

Send an RFC 4122 UUID in `x-fapi-interaction-id` and the gateway plays it back on the
response. If you do not send one it mints one. Log it — it is the only correlation handle
you have.

## What you cannot do here

`/energy/accounts` and every other consumer path returns HTTP 404 on this gateway. The AER
serves product data only. For anything about a real customer see
`energyaustralia-consumer-data-sharing.md`.
