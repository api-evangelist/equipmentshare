---
name: Install a T3OS app in a workspace (unattended API key)
description: >-
  Perform the EquipmentShare T3OS workspace-installed auth flow — a one-shot admin install that
  yields a workspace-scoped API key — and call the T3OS ERP GraphQL API unattended with the
  X-API-Key header. Use for syncs, exports, scheduled jobs, and event handlers with no user in
  the loop.
api: T3OS ERP GraphQL API (es-erp-api)
graphql_endpoint: https://api.equipmentshare.com/es-erp-api/graphql
operations: [installWorkspaceApp, getWorkspaceById]
source: https://github.com/EquipmentShare/t3os-examples/tree/main/apps/workspace-hello-world
generated: '2026-07-19'
method: generated
---

# Install a T3OS app in a workspace

Grounded in the MIT-licensed `t3os-examples/apps/workspace-hello-world` reference app.

## Prerequisites
- A registered workspace app: `registerApp(kind: WORKSPACE_INSTALLED)` returning a `T3OS_APP_ID`
  and an `installCallbackUrl` (its host becomes the `aud` claim on install tokens).

## Steps

1. **Send the admin to install.** Redirect the workspace admin to the T3OS install screen at
   `https://app.t3os.ai/oauth/install` for your `app_id`. The admin signs in, picks a workspace,
   and clicks Install. T3OS mints a principal, creates a workspace-scoped API key, and signs an
   install-token JWT.

2. **Receive the install token.** T3OS redirects to your registered `installCallbackUrl` with
   `?install_token=<jwt>`.

3. **Verify the JWT.** Verify the token against the es-erp-api JWKS at
   `https://api.equipmentshare.com/es-erp-api/.well-known/jwks.json`, checking
   `iss=https://api.equipmentshare.com/es-erp-api`, `aud` = your callback host, and your `app_id`.
   The verified token carries the workspace-scoped API key.

4. **Store the key securely.** Encrypt the API key (the reference app uses AES-256-GCM) and
   persist it keyed by `workspace_id`. Keys do not expire; they are invalidated only on uninstall.

5. **Call the API unattended.** POST GraphQL to
   `https://api.equipmentshare.com/es-erp-api/graphql` with header `X-API-Key: <key>`. Confirm
   with the `getWorkspaceById` query.

6. **Handle uninstall.** Subscribe to T3OS workspace/uninstall events and delete the stored key
   when the workspace uninstalls your app.

## Conventions & errors
- Errors come back in the GraphQL top-level `errors[]` array.
- See `conventions/equipmentshare-conventions.yml` and `authentication/equipmentshare-authentication.yml`.
