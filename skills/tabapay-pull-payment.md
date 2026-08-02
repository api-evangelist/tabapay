---
name: Accept a pull payment with TabaPay
description: Collect an instant payment from a customer card — verify the card, tokenize it as an Account, create a pull Transaction, and reconcile the result.
api: openapi/tabapay-openapi.yml
operations: [cardQuery, accountCreate, transactionCreate, transactionRetrieve, transactionDelete]
generated: '2026-07-21'
method: generated
---

# Accept a pull payment (card acceptance)

Authentication: every request uses `Authorization: Bearer <token>` against your client-specific
`https://{FQDN}` base URL (issued by TabaPay at boarding). Request bodies MUST be compact JSON —
no whitespace or newlines, or the API returns `EM: JSON NOT PACKED`.

1. **Check the card** — `cardQuery` (`POST /v1/clients/{ClientID}/cards`) with the card number
   (optionally AVS fields `owner.address.*` and `owner.name.*` for ANI). Confirm `pull` eligibility
   in the response networks before charging; a card that cannot pull will fail later.
2. **Tokenize** — `accountCreate` (`POST /v1/clients/{ClientID}/accounts`) with card + owner and a
   unique `referenceID` (1-15 chars). Store the returned 22-character `accountID`; add
   `?RejectDuplicateCard` to refuse cards already on file (409 on duplicate).
3. **Charge** — `transactionCreate` (`POST /v1/clients/{ClientID}/transactions`) with
   `type: "pull"`, `amount`, `accounts.sourceAccountID` = the customer token,
   `accounts.destinationAccountID` = your settlement account, and a fresh unique `referenceID`.
4. **Reconcile errors, never blind-retry**:
   - `SC 207` (Multi-Status) or a timeout: the transaction may exist — call `transactionRetrieve`
     or Retrieve-via-ReferenceID (within 24h) before retrying. Reusing a `referenceID` returns 409.
   - Check `networkRC` against errors/tabapay-decline-codes.yml: `00` approved, `51` NSF, `05` do
     not honor; codes marked "reattempt not permitted" (04/07/12/14/41/43/46/57/78/R0/R1/R3) must
     not be retried with the same PAN.
   - `SC 429/503`: back off exponentially; target ~1 TPS, stay within 3-5 TPS.
5. **Reverse if needed** — `transactionDelete` (`DELETE /v1/clients/{ClientID}/transactions/{TransactionID}`)
   attempts a reversal of a pull transaction (not available for ACH).

Sandbox: use published test cards only (sandbox/tabapay-sandbox.yml); trigger errors with magic
amounts (0.01 error, 0.02 → 207, 0.09 partial approval).
