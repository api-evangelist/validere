---
name: Query assets and measurements in CarbonHub
description: Search facilities, equipment, and devices, then pull measurement series
  and aggregated measurements from the App API.
api: openapi/validere-carbonhub-openapi-original.json
operations:
- get_facilities_filter
- get_facility
- search_equipment
- get_equipment
- search_device
- list_measurement_series
- get_aggregated_measurements
- get_measurements
generated: '2026-07-21'
method: generated
---

## Authenticate first

1. Obtain a `client_id`/`client_secret` (and the `audience`) from Validere's technical support team — there is no self-serve signup.
2. `POST https://validere.auth0.com/oauth/token` with JSON body `{"grant_type": "client_credentials", "client_id": ..., "client_secret": ..., "audience": "https://validere360.com/api"}`.
3. Use the returned `access_token` as `Authorization: Bearer <token>` on every call. Cache it until `expires_in` (86400 s in the documented example) lapses.

## Steps

1. All calls run against `https://api.validere.io/app` (paths are `/app/v1/...`).
2. Discover filterable facility attributes with `get_facilities_filter`, then fetch a facility with `get_facility`.
3. Find equipment with `search_equipment` (`POST /app/v1/equipment/search`) and devices with `search_device`. Search endpoints accept `page`, `page_size`, `sort_by`, `sort_direction` and a Mongo-like `filter` object (`{"status": {"$in": ["active", "unknown"]}}`).
4. List series with `list_measurement_series`, then pull values with `get_measurements` (per series) or `get_aggregated_measurements` (across series; `device_id` filters to one source device).

## Rules

- Data model: equipment always belongs to a facility; measurements hang off measurement series (see data-model/validere-data-model.yml).
- Expect plain-JSON errors with standard HTTP codes (400/401/403/404/429-as-conflict); only a few operations declare 4xx in the spec — treat the docs error table as authoritative.
