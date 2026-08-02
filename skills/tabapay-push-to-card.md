---
name: Send an instant payout (push to card) with TabaPay
description: Pay out funds to a recipient's debit card — verify push eligibility and availability, tokenize the card, create a push Transaction, and handle exceptions.
api: openapi/tabapay-openapi.yml
operations: [cardQuery, accountCreate, transactionCreate, transactionRetrieve, queryfx]
generated: '2026-07-21'
method: generated
---

# Send an instant payout (push to card)

Authentication: `Authorization: Bearer <token>` on your client-specific `https://{FQDN}` base URL.
Bodies must be compact JSON.

1. **Verify the destination card** — `cardQuery` (`POST /v1/clients/{ClientID}/cards`). Check the
   response for push eligibility AND funds-availability tier (immediate / next business day / few
   business days) before promising "instant" to the recipient. Use AVS/ANI fields to confirm the
   recipient is the cardholder (account-takeover prevention).
2. **Tokenize** — `accountCreate` (`POST /v1/clients/{ClientID}/accounts`) with the recipient card
   and owner, unique `referenceID`. Keep the returned `accountID`.
3. **(Cross-border only) price the FX** — `queryfx` (`POST /v4/clients/{ClientID}/fxrate`) returns
   the real-time rate and beneficiary-currency amount (currently sandbox-only).
4. **Pay out** — `transactionCreate` (`POST /v1/clients/{ClientID}/transactions`) with
   `type: "push"`, `amount`, `accounts.sourceAccountID` = your funding account,
   `accounts.destinationAccountID` = the recipient token, unique `referenceID`.
5. **Reconcile**:
   - `SC 207` or timeout: call `transactionRetrieve` or Retrieve-via-ReferenceID (24h window)
     before any retry — push funds may already be in flight.
   - `networkRC` per errors/tabapay-decline-codes.yml; respect "reattempt not permitted" codes.
   - Push transactions are generally irrevocable once completed — exceptions handling is manual
     (see the provider's exceptions-handling guide), so validate before sending.

Sandbox: RTP payouts simulate failures via magic account numbers (111111112 → P04 invalid account,
111111121 → P14 participant deceased, ...); card payouts use the published test cards.
