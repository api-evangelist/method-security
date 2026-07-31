---
name: Triage and close a Method Issue
description: Fetch a Method Issue, review its evidence report, and update or close it with a reason.
api: openapi/method-security-openapi-original.yml
operations:
  - getTokenWithClientCredentials
  - getIssue
  - updateIssue
  - getReport
---

# Triage and close a Method Issue

Use the Method Platform API to investigate an Issue and drive it through its
workflow (OPEN → CLOSED/ARCHIVED). All calls go through the `method-api-gateway`
on your tenant host (`https://<instance>.method.delivery`).

## Auth
1. Get a bearer token with `getTokenWithClientCredentials`:
   `POST /method-api-gateway/api/auth/getToken` with `client_id`, `client_secret`,
   `grant_type=client_credentials`. Read `access_token` from the response.
2. Send `Authorization: Bearer <access_token>` on every subsequent call. Tokens
   are short-lived — re-request when `expires_in` elapses.

## Steps
1. **Load the Issue** with `getIssue` (`GET /api/v1/issues/{id}`). Inspect
   `severity` (CRITICAL…INFO), `status`, `description`, `remediation`, and
   `affectedObjectId` / `affectedObjectTitle`. A 404 means the id is unknown.
2. **Read supporting evidence** (optional): if you have a related report id, call
   `getReport` (`GET /api/v1/reports/{reportId}`) and read `markdown`.
3. **Update the Issue** with `updateIssue` (`POST /api/v1/issues/{id}/update`).
   Send only the fields you want to change (omitted fields are unchanged):
   - Re-severity: set `severity`.
   - Close it: set `status: CLOSED` and a required `closeReason` (one of
     RESOLVED, CLOSED_BY_AGENT, ACCEPTED_RISK, FALSE_POSITIVE, NOT_APPLICABLE).
   - Optionally add a `comment`.

## Rules
- `closeReason` is REQUIRED whenever `status` is CLOSED — the call fails otherwise.
- Updates are atomic across provided fields; there is no idempotency key, so do
  not blindly retry a write without re-reading the Issue first.
- Errors are plain JSON with an HTTP status (401 = bad/expired token, 403 =
  policy/permission, 404 = unknown id). See errors/method-security-problem-types.yml.
