# EnergyAustralia (energyaustralia)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

EnergyAustralia is one of Australia's "big three" gentailers — a vertically integrated electricity and gas business owned by Hong Kong's CLP Group and headquartered in Melbourne, generating from Yallourn, Mount Piper and Hallett alongside wind assets and retailing to more than 1.7 million residential and business customers across Victoria, New South Wales, Queensland, South Australia and the ACT. Its API posture is entirely a function of the Consumer Data Right rather than of any product strategy: it is a designated and live CDR energy data holder, but the only anonymously reachable contract carrying its name is the Product Reference Data surface, and that is hosted by the Australian Energy Regulator's Energy Made Easy CDR gateway, not by EnergyAustralia. Open on plans, closed on everything else.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/energyaustralia/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/energyaustralia/refs/heads/main/apis.yml)

## Tags

- Energy
- Australia
- Utilities
- Electricity
- Gas
- Energy Retailer
- Consumer Data Right
- CDR
- Product Reference Data
- Smart Metering
- Energy Markets
- Renewables

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### EnergyAustralia CDR Energy Plans API

Public, unauthenticated Consumer Data Right energy Product Reference Data for the EnergyAustralia brand, conforming to the Australian Consumer Data Standards energy schemas. Unlike CDR banking — where every bank self-hosts its own product endpoint — energy product data is served centrally by the Australian Energy Regulator's Energy Made Easy CDR gateway on a per-retailer path. Confirmed live on 2026-07-27: `GET /energy/plans` returned HTTP 200 with response header `x-v: 1` and `meta.totalRecords` of 1122 published EnergyAustralia plans (265 of them `GAS` fuelType), and `GET /energy/plans/{planId}` returned HTTP 200 at `x-v: 3`. No API key, no registration, no consent required.

- **Human URL:** [https://www.energymadeeasy.gov.au/frequently-asked-questions/how-can-i-get-access-to-the-energy-made-easy-plan-data](https://www.energymadeeasy.gov.au/frequently-asked-questions/how-can-i-get-access-to-the-energy-made-easy-plan-data)
- **Base URL:** `https://cdr.energymadeeasy.gov.au/energyaustralia/cds-au/v1`

#### Tags

- Consumer Data Right
- CDR
- Product Reference Data
- Energy
- Electricity
- Gas
- Australia

#### Properties

- [OpenAPI](openapi/energyaustralia-cds-energy-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#energy-apis)
- [Documentation](https://www.energymadeeasy.gov.au/frequently-asked-questions/how-can-i-get-access-to-the-energy-made-easy-plan-data)
- [API Standards](https://consumerdatastandardsaustralia.github.io/standards/)

### EnergyAustralia CDR Discovery Status API

The Consumer Data Standards Common API discovery endpoints served for the EnergyAustralia brand path on the AER Energy Made Easy CDR gateway. Public and unauthenticated. Confirmed live on 2026-07-27: `GET /discovery/status` returned HTTP 200 with `data.status: "OK"`, and `GET /discovery/outages` returned HTTP 200 with an empty outages array.

- **Human URL:** [https://consumerdatastandardsaustralia.github.io/standards/#common-apis](https://consumerdatastandardsaustralia.github.io/standards/#common-apis)
- **Base URL:** `https://cdr.energymadeeasy.gov.au/energyaustralia/cds-au/v1`

#### Tags

- Consumer Data Right
- CDR
- Discovery
- Status
- Australia

#### Properties

- [OpenAPI](openapi/energyaustralia-cds-common-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#common-apis)
- [API Standards](https://consumerdatastandardsaustralia.github.io/standards/)

### EnergyAustralia CDR Energy Consumer Data Sharing API

The mandated, authenticated Consumer Data Right consumer data-sharing surface — energy accounts, invoices, billing, balances, concessions, payment schedules, electricity service points, interval usage and DER register data — implemented to the Consumer Data Standards CDR Energy API. EnergyAustralia is a live registered CDR energy data holder; the ACCC CDR Register returned its brand entry on 2026-07-27 with `publicBaseUri: https://authncdr.energyaustralia.com.au`. That host is live but not anonymously browsable: every standards path probed returned HTTP 404. Access requires ACCC accreditation as a Data Recipient, mTLS certificates issued through the CDR Register, and a FAPI 1.0 OAuth2/OIDC authorisation with customer consent. **There is no developer signup.**

- **Human URL:** [https://www.energyaustralia.com.au/home/help-support/faqs/consumer-data-right](https://www.energyaustralia.com.au/home/help-support/faqs/consumer-data-right)
- **Base URL:** `https://authncdr.energyaustralia.com.au`

#### Tags

- Consumer Data Right
- CDR
- Energy
- Usage Data
- Billing
- DER
- Smart Metering
- Australia

#### Properties

- [OpenAPI](openapi/energyaustralia-cds-energy-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://consumerdatastandardsaustralia.github.io/standards/#energy-apis)
- [API Standards](https://consumerdatastandardsaustralia.github.io/standards/)
- [CDR Register](https://api.cdr.gov.au/cdr-register/v1/energy/data-holders/brands/summary)

## Common Properties

- [Website](https://www.energyaustralia.com.au/)
- [Consumer Data Right](https://www.energyaustralia.com.au/home/help-support/faqs/consumer-data-right)
- [API Standards](https://consumerdatastandardsaustralia.github.io/standards/)
- [CDR Register](https://api.cdr.gov.au/cdr-register/v1/energy/data-holders/brands/summary)
- [LinkedIn](https://www.linkedin.com/company/energyaustralia)

## Maintainers

- Kin Lane — kin@apievangelist.com
