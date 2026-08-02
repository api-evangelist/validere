---
name: Push JSON data into the Validere platform
description: Register a JSON payload against a pre-registered dataset schema on the
  Data Platform Source API, or obtain a presigned S3 URL for file upload.
api: openapi/validere-dataplatform-source-openapi-original.json
operations:
- register_json_data_data_json_post
- get_file_upload_url_data_file_upload_url_get
generated: '2026-07-21'
method: generated
---

## Authenticate first

1. Obtain a `client_id`/`client_secret` (and the `audience`) from Validere's technical support team — there is no self-serve signup.
2. `POST https://validere.auth0.com/oauth/token` with JSON body `{"grant_type": "client_credentials", "client_id": ..., "client_secret": ..., "audience": "https://validere360.com/api"}`.
3. Use the returned `access_token` as `Authorization: Bearer <token>` on every call. Cache it until `expires_in` (86400 s in the documented example) lapses.

## Steps

1. You need three Validere-issued values: `client_id`, `dataset_id`, and a dataset JSON Schema registered with Validere in advance (the platform validates every payload against it).
2. Push data with `register_json_data_data_json_post` (`POST /data/json` on `https://api.validere.io/platform/v1`). A schema-validation failure returns HTTP 400 synchronously.
3. For files, call `get_file_upload_url_data_file_upload_url_get` (`GET /data/file_upload_url`) with the file name and content type, then PUT the file to the returned presigned S3 URL.

## Rules

- No idempotency-key contract exists: duplicate submissions surface later as `ObjectAlreadyExistsError` at the sink step (see errors/validere-problem-types.yml).
- Asynchronous ingestion errors are NOT returned on the push call — track them via the Transactions API (see the track-data-transactions skill).
