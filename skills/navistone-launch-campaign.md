---
name: Launch a NaviStone direct-mail campaign
description: Onboard a client, create a campaign linked to a Modern Postcard creative, add ZIP/state geo-targeting, and read back output for the campaign.
api: openapi/_original/navistone-openapi-original.json
operations:
- ClientsController_create
- CampaignsController_create
- GeoTargetingController_create
- GeoTargetingController_addZip
- OutputController_findByCampaign
---

# Launch a NaviStone direct-mail campaign

Operating instructions for an agent using the NaviStone (Zazmic) Platform API to stand up a direct-mail campaign end to end.

## Authentication
Every call (except health checks) requires an `X-API-Key` request header. There is no OAuth and no idempotency-key contract, so treat writes as non-idempotent: do not blindly retry a `POST` on a network error without first checking whether the resource was created.

## Steps

1. **Create (or reuse) the client.** `POST /api/clients` (`ClientsController_create`) with `{ name, email, lookupKey }`. Only `name` is required. Capture the returned client `id` (a UUID). To find an existing client instead, `GET /api/clients` (`ClientsController_findAll`) with `name`/`active` filters and `page`/`limit` paging (limit max 100).

2. **Create the campaign.** `POST /api/campaigns` (`CampaignsController_create`) with `{ name, clientId, creativeId }`. `creativeId` is a Modern Postcard Creative ID (e.g. `MP-12345`) — the API supports **no file uploads**, only the creative reference. Capture the campaign `id`.

3. **Add geo-targeting.** Use `POST /api/geo-targeting/campaigns/{campaignId}` (`GeoTargetingController_create`) with **either** `zip` (optionally `radius` in miles) **or** `stateCode` — never both, or you get a 400. For a single ZIP without a radius, the shortcut `POST /api/geo-targeting/campaigns/{campaignId}/zip/{zipCode}` (`GeoTargetingController_addZip`) is simpler. Set `suppress: true` to exclude an area instead of including it.

4. **Read back output.** `GET /api/output/campaign/{campaignId}` (`OutputController_findByCampaign`) returns the production/output records (totals, monthly, daily average) for the campaign.

## Error handling
- `401` — missing/invalid `X-API-Key`.
- `400` — validation failed (common cause: both `zip` and `stateCode` supplied to geo-targeting).
- `404` — the referenced client/campaign ID does not exist.
See `errors/navistone-problem-types.yml` and `conventions/navistone-conventions.yml` for the full envelope.
