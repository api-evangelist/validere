---
name: Track data ingestion transactions
description: Follow the lifecycle of a data submission through the Validere pipeline
  and retry failed transactions.
api: openapi/validere-dataplatform-source-openapi-original.json
operations:
- list_all_transactions_transactions_get
- get_transaction_by_id_transactions__transaction_id__get
- get_transaction_details_by_id_transactions__transaction_id__details_get
- retry_transaction_by_id_transactions__transaction_id__retry_post
- list_all_transaction_errors_type_metadata_transaction_errors_get
generated: '2026-07-21'
method: generated
---

## Authenticate first

1. Obtain a `client_id`/`client_secret` (and the `audience`) from Validere's technical support team — there is no self-serve signup.
2. `POST https://validere.auth0.com/oauth/token` with JSON body `{"grant_type": "client_credentials", "client_id": ..., "client_secret": ..., "audience": "https://validere360.com/api"}`.
3. Use the returned `access_token` as `Authorization: Bearer <token>` on every call. Cache it until `expires_in` (86400 s in the documented example) lapses.

## Steps

1. List recent submissions with `list_all_transactions_transactions_get` (`GET /transactions`).
2. Fetch one with `get_transaction_by_id_transactions__transaction_id__get`; drill into per-step state with `get_transaction_details_by_id_transactions__transaction_id__details_get`.
3. Enumerate the possible error types (`UnsafeFileError`, `InvalidSchemaError`, `OperationError`, `ObjectAlreadyExistsError`, `MissingLookupError`, `LookupCollisionError`, `RequestExceptionError`) with `list_all_transaction_errors_type_metadata_transaction_errors_get`.
4. After fixing the cause (bad schema, missing lookup), re-run with `retry_transaction_by_id_transactions__transaction_id__retry_post`.

## Rules

- Pipeline steps are Validation → Transformation → Sink; each step can carry its own error type.
- `InvalidSchemaError` means the payload failed the registered JSON Schema for the dataset — fix the payload or re-register the schema with Validere before retrying.
