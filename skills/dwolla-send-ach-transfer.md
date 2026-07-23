---
generated: '2026-07-23'
method: generated
name: Send an ACH transfer between funding sources
description: Authenticate, initiate a bank-to-bank transfer over ACH (or Same-Day ACH / RTP / FedNow), then poll its status and read the failure reason if it fails.
api: openapi/dwolla-transfers-openapi.yml
operations: [createApplicationAccessToken, initiateTransfer, getTransfer, getTransferFailureReason]
source: >-
  Grounded in conventions/dwolla-conventions.yml, authentication/dwolla-authentication.yml, and
  errors/dwolla-problem-types.yml; operationIds verified in openapi/dwolla-tokens-openapi.yml and
  openapi/dwolla-transfers-openapi.yml.
---

# Send an ACH transfer between funding sources

Move money from a source funding source to a destination funding source over the U.S. banking rails.

## Auth
- OAuth 2.0 client-credentials (2-legged). Exchange your `client_id`/`client_secret` for an application access token via `createApplicationAccessToken` (`POST /token`, HTTP Basic on the token request). Tokens are short-lived (1 hour, no refresh — re-exchange on expiry). See `authentication/dwolla-authentication.yml`.
- Send `Authorization: Bearer {access_token}` and `Accept: application/vnd.dwolla.v1.hal+json` on every request.
- Use the sandbox host `https://api-sandbox.dwolla.com` while developing; production is `https://api.dwolla.com`.

## Idempotency (required for the write step)
- Send an `Idempotency-Key` header (a client-generated UUID) on `initiateTransfer` so a retried POST does not create a duplicate transfer. Keys are retained 24h; replaying returns the original `201 Created`. A `409 Conflict` means the original is still processing. See `conventions/dwolla-conventions.yml`.

## Steps
1. **Get a token** — `createApplicationAccessToken` (`POST /token`). Capture the `access_token`.
2. **Initiate the transfer** — `initiateTransfer` (`POST /transfers`). Body carries `_links.source` and `_links.destination` (funding source hrefs) and an `amount` object `{ currency: "USD", value: "10.00" }`. Add `clearing` for Same-Day ACH, or a processing-channel link for RTP/FedNow instant. On success the transfer URL is returned in the `Location` header.
3. **Poll status** — `getTransfer` (`GET /transfers/{id}`). Status moves `pending` -> `processed` (or `failed`/`cancelled`).
4. **Read failure detail** — if status is `failed`, call `getTransferFailureReason` (`GET /transfers/{id}/failure`) to get the ACH return code (R01 insufficient funds, R03 no account, etc.). See `errors/dwolla-decline-codes.yml`.

## Errors
- API-level problems use Dwolla's HAL error envelope `{ code, message, _embedded.errors[] }` (not RFC 9457). See `errors/dwolla-problem-types.yml`.
- `429 Too Many Requests` — back off exponentially; volume limits persist ~5 minutes. Concurrency limits apply to rapid same-wallet transfers. See `rate-limits/dwolla-rate-limits.yml`.

## Test values
- Advance sandbox transfers through processing with `simulateBankTransferProcessing` (`POST /sandbox/...`) instead of waiting for real bank timing. See `sandbox/dwolla-sandbox.yml`.
