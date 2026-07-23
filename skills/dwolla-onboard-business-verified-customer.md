---
generated: '2026-07-23'
name: Onboard a Business Verified Customer and certify beneficial owners
method: generated
description: Create a Business Verified Customer, add its beneficial owners with KYC data, and certify ownership to satisfy U.S. CDD/beneficial-ownership requirements.
api: openapi/dwolla-beneficial-owners-openapi.yml
operations: [createCustomer, listBusinessClassifications, createBeneficialOwnerForCustomer, getBeneficialOwnershipStatusForCustomer, certifyBeneficialOwnershipForCustomer]
source: >-
  Grounded in conventions/dwolla-conventions.yml and errors/dwolla-problem-types.yml; operationIds
  verified in openapi/dwolla-customers-openapi.yml and openapi/dwolla-beneficial-owners-openapi.yml.
---

# Onboard a Business Verified Customer and certify beneficial owners

Onboard a business entity that can move money, meeting Customer Due Diligence (CDD) rules.

## Auth
- OAuth 2.0 client-credentials bearer token; `Accept: application/vnd.dwolla.v1.hal+json`; `Idempotency-Key` (UUID) on every POST. See `authentication/dwolla-authentication.yml`.

## Steps
1. **Look up the business classification** — `listBusinessClassifications` (`GET /business-classifications`) and pick the `industryClassification` id that matches the business.
2. **Create the business customer** — `createCustomer` (`POST /customers`) with `type: "business"`, `businessType`, `businessClassification`, EIN, controller person details, and the business address.
3. **Add beneficial owners** — for each individual owning ≥25% (or controlling), `createBeneficialOwnerForCustomer` (`POST /customers/{id}/beneficial-owners`) with name, DOB, address, and SSN/passport.
4. **Check ownership status** — `getBeneficialOwnershipStatusForCustomer` (`GET /customers/{id}/beneficial-ownership`). Resolve any owner in `document`/`incomplete` before certifying.
5. **Certify** — `certifyBeneficialOwnershipForCustomer` (`POST /customers/{id}/beneficial-ownership`) with `status: "certified"` to attest the ownership information is accurate. The customer cannot transact until this is done.

## Errors
- Owner verification issues surface per-owner; a `document` status means an ID upload is required for that owner. HAL error envelope with `_embedded.errors[]`. See `errors/dwolla-problem-types.yml`.

## Test values
- Sandbox EINs and controller/owner identities that drive each verification outcome are in `sandbox/dwolla-sandbox.yml`.
