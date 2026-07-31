---
name: Create an environment and run a blueprint
description: Create a Method environment, discover available blueprints, and run one to populate the Ontology.
api: openapi/method-security-openapi-original.yml
operations:
  - getTokenWithClientCredentials
  - createEnvironment
  - listBlueprints
  - runBlueprint
---

# Create an environment and run a blueprint

Blueprints are parameterized Method workflows that create Ontology Objects in an
Environment.

## Auth
Acquire a bearer token with `getTokenWithClientCredentials`
(`POST /method-api-gateway/api/auth/getToken`, `grant_type=client_credentials`)
and send `Authorization: Bearer <access_token>` on every call.

## Steps
1. **Create an Environment** with `createEnvironment`
   (`POST /api/v1/environments/`) passing a human-readable `title`. Keep the
   returned `environmentId`.
2. **(Optional) Attach intel** with `uploadEnvironmentIntel`
   (`POST /api/v1/environments/{environmentId}/intel`): send `filename`,
   optional `mimeType`, and base64 `contentBase64`; keep the `attachmentId`.
3. **List blueprints** with `listBlueprints` (`GET /api/v1/blueprints/`). For the
   blueprint you want, read its `parameters` map — each `ParameterDefinition`
   gives `key`, `displayName`, `type`, and `requirements`.
4. **Run the blueprint** with `runBlueprint`
   (`POST /api/v1/blueprints/{blueprintId}/run`) passing `environmentId` and a
   `parameters` map of typed `ParameterValue`s (`{type, value}`) keyed by the
   parameter keys from step 3.

## Rules
- `runBlueprint` returns 200 with `objectIds[]` (created Objects) AND `errors[]`
  (`BlueprintError`: `key`, `type`, `message`) — always check `errors[]` even on
  a 200; a non-empty list means some parameters failed validation.
- Supply every parameter marked in `requirements`; missing/invalid values surface
  in `errors[]`, not as an HTTP error.
- No idempotency key exists — a repeated run creates Objects again.
