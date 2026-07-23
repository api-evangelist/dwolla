---
generated: '2026-07-23'
name: Run a mass payment payout
method: generated
description: Disburse funds to many recipients in one batch of up to 5,000 items from a single source funding source, then reconcile each item's status.
api: openapi/dwolla-mass-payments-openapi.yml
operations: [initiateMassPayment, getMassPayment, listMassPaymentItems, getMassPaymentItem, updateMassPayment]
source: >-
  Grounded in conventions/dwolla-conventions.yml and errors/dwolla-problem-types.yml; operationIds
  verified in openapi/dwolla-mass-payments-openapi.yml.
---

# Run a mass payment payout

Pay a payroll or marketplace payout run without issuing thousands of individual transfers.

## Auth
- OAuth 2.0 client-credentials bearer token; `Accept: application/vnd.dwolla.v1.hal+json`; `Idempotency-Key` (UUID) on the create call. See `authentication/dwolla-authentication.yml`.

## Steps
1. **Initiate the mass payment** — `initiateMassPayment` (`POST /mass-payments`) with `_links.source` (the funding source to pull from) and an `items[]` array (up to 5,000), each item carrying its own `_links.destination` and `amount`. Optionally set `status: "deferred"` to stage without processing.
2. **Poll the batch** — `getMassPayment` (`GET /mass-payments/{id}`). Status moves `pending` -> `processing` -> `complete`.
3. **List item results** — `listMassPaymentItems` (`GET /mass-payments/{id}/items`); page through with `limit`/`offset`. Each item has a `status` (`success`/`failed`) and, on success, a `_links.transfer`.
4. **Inspect a single item** — `getMassPaymentItem` (`GET /mass-payments/{id}/items/{itemId}`) for one recipient's outcome.
5. **Release a deferred batch (optional)** — `updateMassPayment` (`POST /mass-payments/{id}`) with `status: "processing"` to start a staged batch, or `status: "cancelled"` to cancel it.

## Errors
- Per-item failures do not fail the whole batch; each item carries its own error. Batch-level validation uses the HAL error envelope. See `errors/dwolla-problem-types.yml`.
- Pagination is offset/limit; the batch item list can be large — use `offset`. See `conventions/dwolla-conventions.yml`.

## Test values
- Advance sandbox items to terminal states with `simulateBankTransferProcessing`. See `sandbox/dwolla-sandbox.yml`.
