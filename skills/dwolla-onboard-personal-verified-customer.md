---
generated: '2026-07-23'
name: Onboard a personal Verified Customer with a bank funding source
method: generated
description: Create a Personal Verified Customer, attach a bank funding source, and verify it (micro-deposits or Open Banking), handling KBA step-up when identity is not auto-verified.
api: openapi/dwolla-customers-openapi.yml
operations: [createCustomer, getCustomer, createCustomerFundingSource, initiateOrVerifyMicroDeposits, initiateKbaForCustomer, getKbaQuestions, verifyKbaQuestions]
source: >-
  Grounded in conventions/dwolla-conventions.yml and errors/dwolla-problem-types.yml; operationIds
  verified in openapi/dwolla-customers-openapi.yml, openapi/dwolla-funding-sources-openapi.yml, and
  openapi/dwolla-kba-openapi.yml.
---

# Onboard a personal Verified Customer with a bank funding source

Bring a real end user onto the platform so they can send and receive money.

## Auth
- OAuth 2.0 client-credentials bearer token; `Accept: application/vnd.dwolla.v1.hal+json`. See `authentication/dwolla-authentication.yml`.
- Send an `Idempotency-Key` (UUID) on each POST. See `conventions/dwolla-conventions.yml`.

## Steps
1. **Create the customer** — `createCustomer` (`POST /customers`) with `type: "personal"`, full legal name, email, address, date of birth, and `ssn` (last 4 or full). The `Location` header returns the customer URL.
2. **Check verification status** — `getCustomer` (`GET /customers/{id}`). If `status` is `verified`, continue. If `document`, upload an ID (see the documents flow). If `retry`, correct data and re-submit.
3. **Handle KBA step-up (only when required)** — when identity cannot be auto-verified, `initiateKbaForCustomer` (`POST /customers/{id}/kba`), fetch the dynamically generated questions with `getKbaQuestions` (`GET /kba/{id}`), then submit answers with `verifyKbaQuestions` (`POST /kba/{id}`).
4. **Attach a bank funding source** — `createCustomerFundingSource` (`POST /customers/{id}/funding-sources`) with routing/account number, or an Open Banking exchange link (Plaid/MX) for instant verification.
5. **Verify via micro-deposits (if not Open-Banking-verified)** — Dwolla sends two small deposits; the user reports them and you confirm with `initiateOrVerifyMicroDeposits` (`POST /funding-sources/{id}/micro-deposits`).

## Errors
- Validation failures return the HAL error envelope with `_embedded.errors[]` each carrying a JSON-pointer `path`. See `errors/dwolla-problem-types.yml`.
- Duplicate customer email/SSN returns a `DuplicateResource` error with a link to the existing resource.

## Test values
- Use sandbox names/SSNs from `sandbox/dwolla-sandbox.yml` to force `verified`, `retry`, `kba`, or `document` outcomes deterministically.
