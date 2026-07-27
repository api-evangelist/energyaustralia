---
name: Read EnergyAustralia electricity service points, interval usage and DER data
description: >-
  Walk from an EnergyAustralia customer's consented accounts to their electricity service
  points (NMIs), then to interval meter reads and Distributed Energy Resource register
  records — solar PV, batteries and inverters. ACCREDITED-ONLY. Much of this data is held
  by AEMO as designated secondary data holder and proxied by the retailer.
api: openapi/energyaustralia-cds-energy-api-openapi.yml
base_url: https://authncdr.energyaustralia.com.au
auth: mTLS + OAuth2/OIDC (FAPI 1.0 Advanced)
gated: true
operations:
  - getEnergyAccountDetail
  - listElectricityServicePoints
  - getElectricityServicePointDetail
  - getElectricityServicePointUsage
  - listElectricityUsageBulk
  - listElectricityUsageForServicePoints
  - getElectricityDERForServicePoint
  - listElectricityDERBulk
  - listElectricityDERForSpecificServicePoints
generated: '2026-07-27'
method: generated
---

# Read EnergyAustralia service points, usage and DER data

## Preconditions

Same gate as all consumer data: ACCC accreditation as a Data Recipient, CDR
Register-issued mTLS certificates, a registered software product, and per-consumer consent.
See `energyaustralia-consumer-data-sharing.md` for the authorisation flow.

**Know where the data comes from.** Service point standing data, metering data and the DER
register are held by **AEMO** as designated secondary data holder. EnergyAustralia proxies
those requests. You call one API; two organisations back it. Expect the secondary-sourced
calls to be slower and more prone to transient `Unavailable` conditions than the
retailer's own account data.

## Step 1 — find the NMIs

Two routes:

- From accounts: `getEnergyAccountDetail` (`GET /energy/accounts/{accountId}`, `x-v: 4`)
  returns `plans[].servicePointIds` — the NMIs linked to each plan on the account.
- Directly: `listElectricityServicePoints` (`GET /energy/electricity/servicepoints`,
  `x-v: 2`) — scope `energy:electricity.servicepoints.basic:read`. Returns every consented
  service point with its id, classification, jurisdiction and status.

`servicePointId` **is** the National Metering Identifier — the physical connection point
identifier used across the Australian electricity market. It is the join key for everything
in this skill.

## Step 2 — service point detail (`getElectricityServicePointDetail`)

`GET /energy/electricity/servicepoints/{servicePointId}` at `x-v: 2` — scope
`energy:electricity.servicepoints.detail:read`.

This is the NMI standing data: meters, registers and related market participants. Note the
scope escalation from `.basic:read` to `.detail:read` — request the narrower scope if you
only need the list.

## Step 3 — interval usage

Scope `energy:electricity.usage:read` for all three shapes:

| Shape | Operation | Path |
|---|---|---|
| Single | `getElectricityServicePointUsage` | `GET /energy/electricity/servicepoints/{servicePointId}/usage` |
| Bulk | `listElectricityUsageBulk` | `GET /energy/electricity/servicepoints/usage` |
| Specific | `listElectricityUsageForServicePoints` | `POST /energy/electricity/servicepoints/usage` |

Usage is windowed by date parameters. Supplying an out-of-range or wrongly-typed date
returns HTTP 400 `urn:au-cds:error:cds-all:Field/InvalidDateTime` — this is one of the ten
operations that declare that code, so validate your window before calling.

`EnergyUsageRead` records are interval or accumulated reads. Interval data is high volume:
paginate with `page` / `page-size` (max 1000), read `meta.totalRecords` first, and prefer
the bulk or specific-accounts shape over a loop of single calls.

## Step 4 — DER register

Scope `energy:electricity.der:read`:

| Shape | Operation | Path |
|---|---|---|
| Single | `getElectricityDERForServicePoint` | `GET /energy/electricity/servicepoints/{servicePointId}/der` |
| Bulk | `listElectricityDERBulk` | `GET /energy/electricity/servicepoints/der` |
| Specific | `listElectricityDERForSpecificServicePoints` | `POST /energy/electricity/servicepoints/der` |

`EnergyDerRecord` describes registered solar PV, battery and inverter devices at the
connection point. This is the CDR's answer to what other markets expose via IEEE 2030.5 or
Green Button — **EnergyAustralia publishes neither**; the DER register endpoints are the
only device-level surface it has.

## Step 5 — POST vs GET

The `POST` variants are **queries, not mutations**. They exist only to carry a long
`data.servicePointIds[]` array in the body. They are idempotent; there is no idempotency key
in this API.

Their error semantics differ from the GET variants:

- POST (id in body) → **HTTP 422**
  `urn:au-cds:error:cds-energy:Authorisation/InvalidServicePoint` (permanent) or
  `.../UnavailableServicePoint` (transient).
- GET (id in path) → **HTTP 404** for the same two conditions.

Handle both status codes for the same semantic condition.

## Volume and throttling

Interval usage across many NMIs over a long window is the heaviest thing you can ask this
API for. Gate the run on `getStatus` (see
`energyaustralia-check-status-and-outages.md`), honour `Retry-After` on HTTP 429, and split
long date windows rather than requesting `page-size` above 1000 — that returns HTTP 400
`Field/InvalidPageSize`.

## Conventions

`x-v` required per endpoint; `x-fapi-interaction-id` played back for correlation; errors as
an `errors[]` array in `application/json`, not RFC 9457. See
`conventions/energyaustralia-conventions.yml` and
`errors/energyaustralia-problem-types.yml`.
