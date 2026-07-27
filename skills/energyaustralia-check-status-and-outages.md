---
name: Check EnergyAustralia CDR status and scheduled outages
description: >-
  Read the machine-readable health and scheduled-outage surface for the EnergyAustralia
  Consumer Data Right brand path before and during any integration run. Public and
  unauthenticated. This is the only operational-health contract published under the
  EnergyAustralia name — there is no human status page.
api: openapi/energyaustralia-cds-common-api-openapi.yml
base_url: https://cdr.energymadeeasy.gov.au/energyaustralia/cds-au/v1
auth: none
operations:
  - getStatus
  - getOutages
generated: '2026-07-27'
method: generated
---

# Check EnergyAustralia CDR status and scheduled outages

## Why this exists

EnergyAustralia publishes no status page. The Consumer Data Standards Common API defines
two Discovery endpoints that every data holder implementation must serve, and the AER
Energy Made Easy gateway serves them on the EnergyAustralia brand path. Poll these instead
of guessing.

Both are public. Do not authenticate.

## Step 1 — health check (`getStatus`)

`GET /discovery/status` with header `x-v: 1`.

`data.status` is an enum:

| Value | Meaning | What to do |
|---|---|---|
| `OK` | Implementation fully functional | Proceed |
| `PARTIAL_FAILURE` | One or more endpoints unexpectedly unavailable | Proceed with retries; expect gaps |
| `SCHEDULED_OUTAGE` | An advertised outage is in effect | Check `getOutages`, back off until the window closes |
| `UNAVAILABLE` | Full implementation unexpectedly unavailable | Stop; back off |

`data.updateTime` tells you how fresh the status is. Treat a stale `updateTime` as a signal
in its own right.

## Step 2 — scheduled outages (`getOutages`)

`GET /discovery/outages` with header `x-v: 1`.

Returns `data.outages[]`, each with an outage time, duration, whether it is partial, and an
explanation. An empty array means no outage is scheduled.

Use this to schedule bulk harvests around planned windows rather than discovering them as
5xx responses.

## Operating guidance

- Call `getStatus` **before** starting a long paginated walk of `listEnergyPlans`, and again
  if you begin seeing 5xx responses mid-walk. It distinguishes "the holder is down" from
  "my request is malformed".
- Both endpoints are cheap and unauthenticated. There is no reason not to gate a run on them.
- These endpoints cover the **AER gateway** serving the EnergyAustralia brand path. They do
  **not** cover the accredited-only consumer surface at
  `https://authncdr.energyaustralia.com.au`, which serves its own Discovery endpoints to
  accredited recipients over mutual TLS.

## Errors

Same envelope as everywhere in CDR: `application/json` with an `errors[]` array carrying
`urn:au-cds:error:` codes. A missing or non-integer `x-v` returns HTTP 400
`Header/InvalidVersion`; an unsupported version returns HTTP 406
`Header/UnsupportedVersion`. See `errors/energyaustralia-problem-types.yml`.

## Verified

`GET /discovery/status` returned HTTP 200 with `data.status` `OK` and `updateTime`
`2026-07-27T12:14:46.428Z`; `GET /discovery/outages` returned HTTP 200 with an empty
`outages` array. Both re-verified 2026-07-27.
