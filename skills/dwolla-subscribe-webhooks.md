---
generated: '2026-07-23'
name: Subscribe to webhooks and consume events
method: generated
description: Register a webhook subscription with an HMAC secret, verify inbound event signatures, and reconcile against the Events resource with retry handling.
api: openapi/dwolla-webhook-subscriptions-openapi.yml
operations: [createWebhookSubscription, getWebhookSubscription, listWebhooks, retryWebhook, listEvents, getEvent]
source: >-
  Grounded in asyncapi/dwolla-webhooks.yml, conventions/dwolla-conventions.yml, and
  errors/dwolla-problem-types.yml; operationIds verified in
  openapi/dwolla-webhook-subscriptions-openapi.yml, openapi/dwolla-webhooks-openapi.yml, and
  openapi/dwolla-events-openapi.yml.
---

# Subscribe to webhooks and consume events

Get real-time notifications of every state change (customer verified, transfer completed/failed, funding source added, etc.).

## Auth
- OAuth 2.0 client-credentials bearer token; `Accept: application/vnd.dwolla.v1.hal+json`; `Idempotency-Key` (UUID) on the create call. See `authentication/dwolla-authentication.yml`.

## Steps
1. **Create the subscription** — `createWebhookSubscription` (`POST /webhook-subscriptions`) with your HTTPS `url` and a `secret` you generate. Dwolla HMAC-SHA256 signs each payload with that secret in the `X-Request-Signature-SHA-256` header.
2. **Verify inbound signatures** — on each POST to your endpoint, recompute HMAC-SHA256 over the raw body with your secret and constant-time compare to the header. Reject on mismatch. Return `2xx` quickly; work asynchronously.
3. **Reconcile against Events** — webhooks are at-least-once; dedupe on event `id`. Backfill or replay by polling `listEvents` (`GET /events`) and `getEvent` (`GET /events/{id}`). Events are retained 30 days.
4. **Inspect delivery** — `listWebhooks` (`GET /webhook-subscriptions/{id}/webhooks`) shows delivery attempts and response codes; `retryWebhook` (`POST /webhooks/{id}/retries`) re-sends a failed delivery.

## Errors
- Dwolla retries failed deliveries automatically with backoff; a subscription is auto-paused after too many consecutive failures — un-pause via `updateWebhookSubscription`. HAL error envelope on the API side. See `errors/dwolla-problem-types.yml`.

## Event surface
- The full event/topic catalog is captured in `asyncapi/dwolla-webhooks.yml` (`type: Webhooks`).
