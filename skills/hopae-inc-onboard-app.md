---
name: Create an App, activate providers, and wire webhooks
description: >-
  Provision a Hopae hConnect App with the Workspace API — create the App, activate the
  eID providers you need, attach a verification workflow, and enable HMAC-signed webhooks.
  Uses Workspace Bearer auth.
api: openapi/hopae-inc-hconnect-openapi-original.json
operations: [WorkspaceAppsController_createApp, WorkspaceAppsController_listProviders, WorkspaceActivationController_enableProvider, WorkspaceWorkflowController_createWorkflow, WorkspaceWorkflowController_setDefault, WorkspaceAppsController_updateWebhookConfig, WorkspaceAppsController_rotateWebhookSecret]
---

# Create an App, activate providers, and wire webhooks (hConnect Workspace API)

Use this skill to stand up a new hConnect integration programmatically.

## Auth
Workspace endpoints use **HTTP Bearer** with a workspace key:
`Authorization: Bearer sk_workspace_test_...` (sandbox) / `sk_workspace_prod_...` (production). Base URL sandbox: `https://sandbox.api.hopae.com/connect`.

## Steps
1. **Create the App** — `WorkspaceAppsController_createApp` (`POST /v1/apps`) with `{"name": "My App"}`. **Persist the returned `clientId` and `clientSecret` immediately** — the secret is shown only once and there is no retrieval endpoint.
2. **Discover providers** — `WorkspaceAppsController_listProviders` (`GET /v1/apps/{id}/providers?include=activation`) to see each provider's activation requirements.
3. **Activate a provider** — `WorkspaceActivationController_enableProvider` (`PATCH /v1/apps/{id}/providers/{providerId}/activation`). Direct providers accept an empty requirements object.
4. **Create a workflow** — `WorkspaceWorkflowController_createWorkflow` (`POST /v1/apps/{id}/workflows`), then `WorkspaceWorkflowController_setDefault` (`POST /v1/apps/{id}/workflows/{workflowId}/set-default`) to make it the App default.
5. **Configure webhooks** — `WorkspaceAppsController_updateWebhookConfig` (`PATCH /v1/apps/{id}/webhook-config`) with your endpoint URL and the `events` you want; then `WorkspaceAppsController_rotateWebhookSecret` (`POST /v1/apps/{id}/webhook-config/rotate-secret`) once to enable HMAC signing. Store the signing secret securely.

## Rules
- Verify every webhook with the `X-Hopae-Signature` header (HMAC-SHA256 over `<t>.<raw-body>`); reject when `|now - t| > 300s`. Apps that never rotate a secret receive **unsigned** deliveries.
- Workspace calls sent with Basic auth (or App calls sent with Bearer) return `401` — keep the two auth surfaces separate.
- Going to production requires a solutioning call with Hopae; only the API key changes, not the integration.
