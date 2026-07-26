---
name: Connect to T3OS on behalf of a user (OAuth2 + PKCE)
description: >-
  Authenticate a user against the EquipmentShare T3OS platform with the user-delegated OAuth 2.0
  Authorization Code + PKCE flow, then call the T3OS ERP GraphQL API with the resulting Bearer
  token. Use when your integration acts on behalf of a specific signed-in user.
api: T3OS ERP GraphQL API (es-erp-api)
graphql_endpoint: https://api.equipmentshare.com/es-erp-api/graphql
operations: [getWorkspaceById]
source: https://github.com/EquipmentShare/t3os-examples/tree/main/apps/oauth-hello-world
generated: '2026-07-19'
method: generated
---

# Connect to T3OS on behalf of a user

Grounded in the MIT-licensed `t3os-examples/apps/oauth-hello-world` reference app. T3OS is the
EquipmentShare developer platform; user-delegated auth is handled by the production Auth0 tenant
`equipmentshare-erp.us.auth0.com`.

## Prerequisites
- A registered T3OS app (`registerApp`) with a `client_id`/`client_secret` and a `redirectUri`
  that exactly matches what you send to `/authorize`.
- Audience: `https://api.equipmentshare.com/es-erp-api/delegated`.

## Steps

1. **Generate PKCE + state.** Create a PKCE `code_verifier`, its S256 `code_challenge`, and a
   random `state`. Store them in a signed session cookie.

2. **Redirect to `/authorize`.** Send the user to
   `https://equipmentshare-erp.us.auth0.com/authorize` with `response_type=code`,
   `client_id`, `redirect_uri`, `state`, `code_challenge`, `code_challenge_method=S256`,
   `audience=https://api.equipmentshare.com/es-erp-api/delegated`, and the `scope` you need
   (request the narrowest `<resource>_reader`, e.g. `contact_reader`; `all_resources_reader` is
   the broadest read-only scope). The user signs in and picks a workspace at the consent screen.

3. **Handle the callback.** At your `redirectUri`, verify `state`, then POST to
   `https://equipmentshare-erp.us.auth0.com/oauth/token` with `grant_type=authorization_code`,
   the `code`, your `code_verifier`, `client_id`/`client_secret`, and `redirect_uri`. You receive
   `access_token`, `refresh_token`, and `id_token`. The `workspace_id` claim tells you the active
   workspace; the `https://es-erp/uid` claim is the stable user id.

4. **Call the API.** POST GraphQL to `https://api.equipmentshare.com/es-erp-api/graphql` with
   `Authorization: Bearer <access_token>`. Prove the credential with the `getWorkspaceById` query
   using the `workspace_id` from the token.

5. **Refresh on demand.** When the access token is near expiry, POST `/oauth/token` with
   `grant_type=refresh_token`. For sign-out that also clears the Auth0 SSO cookie, redirect to
   `https://equipmentshare-erp.us.auth0.com/v2/logout`.

## Conventions & errors
- Errors come back in the GraphQL top-level `errors[]` array (no RFC 9457 problem+json).
- See `conventions/equipmentshare-conventions.yml` and `scopes/equipmentshare-scopes.yml`.
