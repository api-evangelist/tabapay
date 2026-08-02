---
name: Run a 3D Secure card transaction with TabaPay
description: Authenticate a cardholder with the 3DS API (init, lookup, authenticate) and feed the results into an authorized pull Transaction.
api: openapi/tabapay-openapi.yml
operations: [3dsinit, 3dslookup, 3dsauthenticate, transactionCreate, transactionCapture]
generated: '2026-07-21'
method: generated
---

# Run a 3D Secure transaction

3DS reduces fraud liability by authenticating the cardholder with their issuer (EMV 3-D Secure via
Cardinal Commerce). Enablement is per-client — request it from TabaPay Support first.

1. **Initialize** — `3dsinit` (`POST /v2/clients/{ClientID}/3ds/init`) starts the authentication
   session; run device data collection between init and lookup.
2. **Lookup** — `3dslookup` (`POST /v2/clients/{ClientID}/3ds/lookup`) checks card enrollment and
   returns challenge information when required (browser flow shows the challenge to the customer).
3. **Authenticate** — `3dsauthenticate` (`POST /v2/clients/{ClientID}/3ds/authenticate`) returns
   `status`, `ECI`, `codeCAVV`/UCAF data, and `3dsVersion`.
4. **Transact** — pass results into `transactionCreate` (`POST /v1/clients/{ClientID}/transactions`)
   REFORMATTED per the provider contract:
   - `ECI`: strip the leading zero (`05` → `5`)
   - `version`: first digit only (`2.2.0` → `2`)
5. **Auth & capture (optional)** — create as an authorization, then `transactionCapture`
   (`PATCH /v1/clients/{ClientID}/transactions/{TransactionID}`) to capture the authorized amount.

Errors: SC/EC/EM envelope (errors/tabapay-problem-types.yml); check 3DS challenge results and ECI
values against the 3DS reference pages. Sandbox: dedicated 3DS test cards trigger frictionless,
challenge, and failure scenarios — amount triggers from the domestic test-card page may not apply.
