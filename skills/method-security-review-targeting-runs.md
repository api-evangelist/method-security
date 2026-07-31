---
name: Search and review targeting runs
description: Search Method targeting runs with a filter, load a target, and pull its linked reports.
api: openapi/method-security-openapi-original.yml
operations:
  - getTokenWithClientCredentials
  - searchTargets
  - getTarget
  - getTargetReports
---

# Search and review targeting runs

Targeting runs move through stages (TARGETED → VALIDATED → EXPLOITABLE →
EXPLOITED → REMEDIATED, plus INVALIDATED/DEFERRED/BLOCKED) and status
(AWAITING_APPROVAL, IN_PROGRESS, FINISHED).

## Auth
Get a bearer token with `getTokenWithClientCredentials` and send
`Authorization: Bearer <access_token>` on every call.

## Steps
1. **Search targets** with `searchTargets` (`POST /api/v1/targets/search`).
   Build a `TargetFilter` — combine leaves with `andFilter`/`orFilter`/`notFilter`.
   Useful leaves: `stage`, `status`, `environment` (by `environmentId`),
   `nameLike`, and `createdAtAfter`/`createdAtBefore`. Page with `pageSize` and
   the returned `pagingToken`.
2. **Load a target** with `getTarget` (`GET /api/v1/targets/{targetId}`) to read
   `stage`, `status`, `targetedAssetName`, `relatedLinks[]`, and `reports[]`.
3. **Pull reports** with `getTargetReports`
   (`GET /api/v1/targets/{targetId}/reports`); each `TargetReportSummary` gives a
   `reportId` you can fetch full markdown for via `getReport`.

## Rules
- Pagination is cursor-based: loop while a `pagingToken` is returned.
- `relatedLinks[]` point to platform resources (ISSUE, OBJECT, OBJECT_SET,
  OPERATOR_SESSION, OVERWATCH_SESSION, AGENT_SESSION) — they are URLs, not API ids.
- Read-only flow; 401/403 indicate token/permission problems.
