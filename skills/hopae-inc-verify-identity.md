---
name: Verify a user's identity with a global eID
description: >-
  Run a Hopae hConnect identity verification end-to-end — pick a provider, create a
  verification session, poll for completion, and read back the normalized user claims
  and provenance. Uses App Basic auth.
api: openapi/hopae-inc-hconnect-openapi-original.json
operations: [getProviders, createVerification, getVerification, VerificationController_getVerificationUserinfo, VerificationController_getVerificationEvidence]
---

# Verify a user's identity with a global eID (hConnect)

Use this skill to verify a user against one of Hopae Connect's 65+ government/bank eID providers.

## Auth
App-scoped endpoints use **HTTP Basic** auth with the App's `clientId:clientSecret`:
`Authorization: Basic base64(clientId:clientSecret)`. (These come from creating an App — see the onboarding skill.) Base URL sandbox: `https://sandbox.api.hopae.com/connect`.

## Steps
1. **(optional) List providers** — `getProviders` (`GET /v1/providers`) to discover valid `providerId` values (e.g. `mitid`, `frejaid`, `passport`).
2. **Create the verification** — `createVerification` (`POST /v1/verifications`) with a body like `{"providerId": "mitid"}`. Save the returned `verificationId` and any redirect URL the end-user must visit.
3. **Poll status** — `getVerification` (`GET /v1/verifications/{verificationId}`) until `status` reaches a terminal state (`completed` / `failed`). No personal attributes are returned here. Prefer webhooks (`verification.completed`) over tight polling when available.
4. **Read claims** — on `completed`, call `VerificationController_getVerificationUserinfo` (`GET /v1/verifications/{verificationId}/userinfo`) for normalized OIDC claims + provenance and level-of-assurance metadata.
5. **(optional) Read evidence** — `VerificationController_getVerificationEvidence` (`GET /v1/verifications/{verificationId}/evidence`) for the raw provider credential evidence object.

## Rules
- Errors are RFC 7807 (`title`, `status`, `detail`). Do **not** retry `AUTH_*`, `VALIDATION_*`, `SESSION_INVALID_STATUS_TRANSITION`, or `PROVIDER_NOT_ENABLED`. Retry `PROVIDER_UNAVAILABLE` / `PROVIDER_ERROR` / `SYSTEM_INTERNAL_ERROR` with exponential backoff.
- Mutating calls are idempotency-protected (`IDEMPOTENCY_KEY_CONFLICT` on payload mismatch) — reuse the same idempotency key only for identical retries.
- Sandbox rejects real eIDs; use test credentials.
