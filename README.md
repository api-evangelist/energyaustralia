# EnergyAustralia (energyaustralia)

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
