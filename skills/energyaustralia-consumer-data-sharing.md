---
name: Read EnergyAustralia customer accounts and billing under CDR consent
description: >-
  Retrieve an EnergyAustralia customer's energy accounts, balances, invoices, billing
  transactions, concessions and payment schedule as an ACCC-accredited Consumer Data Right
  Data Recipient holding that customer's active consent. ACCREDITED-ONLY — this flow
  cannot be run without accreditation, certificates and consent.
api: openapi/energyaustralia-cds-energy-api-openapi.yml
base_url: https://authncdr.energyaustralia.com.au
auth: mTLS + OAuth2/OIDC (FAPI 1.0 Advanced)
gated: true
operations:
  - getCustomer
  - getCustomerDetail
  - listEnergyAccounts
  - getEnergyAccountDetail
  - getEnergyAccountBalance
  - listEnergyAccountBalancesBulk
  - listEnergyAccountBalancesSpecificAccounts
  - getEnergyAccountInvoices
  - listEnergyAccountInvoicesBulk
  - listEnergyInvoicesForSpecificAccounts
  - getBillingForEnergyAccount
  - listEnergyAccountBillingBulk
  - listEnergyAccountBillingForSpecificAccounts
  - getEnergyAccountPaymentSchedule
  - getEnergyAccountConcessions
generated: '2026-07-27'
method: generated
---

# Read EnergyAustralia customer accounts and billing under CDR consent

## Preconditions — read this before writing any code

There is **no developer signup**. Every step below is unreachable without:

1. ACCC accreditation as a Consumer Data Right Data Recipient — or operating as a CDR
   representative / sponsored ADR under an accredited intermediary.
2. Onboarding to the CDR Register, with client **transport and signing certificates**
   issued through the CDR Register PKI.
3. A registered software product.
4. **Per-consumer consent** obtained through the authorisation flow, with an explicit
   sharing duration.

EnergyAustralia's base URI is `https://authncdr.energyaustralia.com.au`, published in the
ACCC CDR Register (`dataHolderBrandId 1cc7833a-b834-ed11-a832-000d3a8830d6`, ABN
99086014968). Anonymous probes of every standards path on that host return HTTP 404 — that
is gating, not absence.

## Step 0 — authorisation

All calls go over **mutual TLS** with a CDR Register-issued client certificate. The access
token is obtained via an OAuth 2.0 authorization code flow under the FAPI 1.0 Advanced
profile: PKCE, Pushed Authorization Requests, signed request objects and `private_key_jwt`
client authentication, with the token bound holder-of-key to your certificate.

Resolve `authorization_endpoint` and `token_endpoint` from the data holder's OIDC discovery
document at `{publicBaseUri}/.well-known/openid-configuration`. **Do not hardcode them** —
that document is only readable over mTLS and its contents were not verifiable anonymously.

Request `openid` plus only the scopes you need. See `scopes/energyaustralia-scopes.yml`.

## Step 1 — confirm the customer (`getCustomer` / `getCustomerDetail`)

`GET /common/customer` at `x-v: 1` — scope `common:customer.basic:read`. Returns name and
occupation for a person, or agent name/role plus organisation name, industry code and
ABN/ACN for a non-individual.

`GET /common/customer/detail` at `x-v: 2` — scope `common:customer.detail:read`. Adds phone,
email, mail address and residential address.

Note the CDS given-name convention: where the holder cannot reliably split given names it
MAY return the whole string in `firstName` with an empty `middleNames` array. Do not assume
`firstName` is a single token.

## Step 2 — list the accounts (`listEnergyAccounts`)

`GET /energy/accounts` at `x-v: 2` — scope `energy:accounts.basic:read`.

Returns `data.accounts[]` with `accountId`, `accountNumber`, `displayName`, `creationDate`,
`plans` and `openStatus`. Paginate with `page` / `page-size` and read
`meta.totalRecords` / `meta.totalPages`.

`accountId` is the key for every step that follows.

## Step 3 — account detail (`getEnergyAccountDetail`)

`GET /energy/accounts/{accountId}` at `x-v: 4` — scope `energy:accounts.detail:read`.

Adds `plans[].servicePointIds` (the NMIs — your bridge to usage and DER data),
`plans[].planDetail` and `plans[].authorisedContacts`.

## Step 4 — the money

All four use scope `energy:billing:read`:

- **Balance** — `GET /energy/accounts/{accountId}/balance`
- **Invoices** — `GET /energy/accounts/{accountId}/invoices`. Each `EnergyInvoice` carries a
  `period` (mandatory when usage or demand charges are present), usage charges split by fuel,
  and an optional `payOnTimeDiscount`.
- **Billing transactions** — `GET /energy/accounts/{accountId}/billing` at `x-v: 3`.
  Discriminated by `transactionUType`: `usage`, `demand`, `onceOff`, `otherCharges`,
  `payment`. Switch on it; the sibling fields are conditional.
- **Payment schedule** — `GET /energy/accounts/{accountId}/payment-schedule`, scope
  `energy:accounts.paymentschedule:read`. Discriminated by `paymentScheduleUType`:
  `cardDebit`, `directDebit`, `digitalWallet`, `manualPayment`. Treat `bsb` and
  `accountNumber` on `directDebit` as sensitive; they are unmasked unless `isTokenised` is
  true.

Concessions: `GET /energy/accounts/{accountId}/concessions`, scope
`energy:accounts.concessions:read`.

## Step 5 — bulk vs specific-accounts

Every account-scoped resource has three shapes. Choose deliberately:

| Shape | Operation pattern | When |
|---|---|---|
| Single | `getEnergyAccountBalance` | One known account |
| Bulk | `listEnergyAccountBalancesBulk` (`GET /energy/accounts/balances`) | Every consented account |
| Specific | `listEnergyAccountBalancesSpecificAccounts` (`POST /energy/accounts/balances`) | A named subset |

The same triple exists for invoices (`getEnergyAccountInvoices` /
`listEnergyAccountInvoicesBulk` / `listEnergyInvoicesForSpecificAccounts`) and billing
(`getBillingForEnergyAccount` / `listEnergyAccountBillingBulk` /
`listEnergyAccountBillingForSpecificAccounts`).

**The POST variants are queries, not mutations.** POST is used only to carry a long
`data.accountIds[]` array in the request body. They are naturally idempotent; there is no
idempotency key in this API and none is needed. Prefer the specific-accounts POST over N
single calls — it is one round trip and one rate-limit charge.

## Error handling that matters here

The id-in-body POST operations return **HTTP 422**, not 404, for a bad id:

- `urn:au-cds:error:cds-energy:Authorisation/InvalidEnergyAccount` — permanent, do not retry.
- `urn:au-cds:error:cds-energy:Authorisation/UnavailableEnergyAccount` — transient, retry later.

The GET-with-id-in-path operations return the same two conditions as **HTTP 404**. Handle
both status codes for the same semantic condition.

Consent failures are 403: `Authorisation/RevokedConsent`, `Authorisation/InvalidConsent`,
`Authorisation/AdrStatusNotActive`. On `RevokedConsent`, stop and delete the data you hold —
do not retry. Full catalog in `errors/energyaustralia-problem-types.yml`.

## Conventions that apply to every call

- `x-v` required, per endpoint, values listed above. HTTP 406 on unsupported.
- `x-fapi-interaction-id` — send an RFC 4122 UUID, it is played back; log it.
- `x-fapi-auth-date` required for all resource calls.
- `x-fapi-customer-ip-address` and `x-cds-client-headers` for customer-present calls only.
- Errors are `application/json` with an `errors[]` array, **not** RFC 9457.
- HTTP 429 returns `Retry-After`; honour it.

See `conventions/energyaustralia-conventions.yml`.
