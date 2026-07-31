---
name: Manage agent skills
description: Search, read, create, and update the markdown Skills available to Method AI Agents.
api: openapi/method-security-openapi-original.yml
operations:
  - getTokenWithClientCredentials
  - searchSkills
  - getSkill
  - createSkill
  - updateSkill
---

# Manage agent skills

Method Skills are markdown documents (with labels) that agents can search and
apply. This flow curates them through the API.

## Auth
Get a bearer token with `getTokenWithClientCredentials` and send
`Authorization: Bearer <access_token>` on every call.

## Steps
1. **Search skills** with `searchSkills` (`POST /api/v1/skills/search`): pass an
   optional `nameLike`, plus `pageSize` and `pagingToken` for paging. Read the
   returned `skills[]` (`id`, `name`, `labels`, timestamps).
2. **Read a skill** with `getSkill` (`GET /api/v1/skills/{skillId}`) to load its
   full `content` (markdown) and `labels`.
3. **Create a skill** with `createSkill` (`POST /api/v1/skills/`): send `name`
   and `content` (both required), plus optional `description` and `labels`.
4. **Update a skill** with `updateSkill` (`PUT /api/v1/skills/{skillId}`): send
   `name` and `content` (required). `labels` REPLACES the existing set — omit it
   to keep current labels, or pass `[]` to clear them.

## Rules
- On `updateSkill`, an omitted `labels` leaves labels unchanged; an empty list
  clears them — be deliberate.
- `name` and `content` are required on both create and update.
- No idempotency key — a repeated create makes a second skill; search first to
  avoid duplicates.
