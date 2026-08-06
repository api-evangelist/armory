---
name: Submit a cloud operation and poll its task to completion
description: >-
  Run a write against the Armory Scale Agent / Clouddriver surface correctly - submit to the
  cloud-provider-scoped /ops endpoint with an idempotency key, then poll the resulting Task until it
  reports a terminal status, and resume it if it stalls.
api: openapi/armory-scale-agent-operations-openapi.yml
generated: '2026-08-06'
method: generated
source: openapi/_original/armory-scale-agent-swagger.json + conventions/armory-conventions.yml
operations:
  - cloudProviderOperationsUsingPOST
  - cloudProviderOperationUsingPOST
  - getUsingGET_1
  - listUsingGET_6
  - resumeTaskUsingPOST
---

# Submit a cloud operation and poll its task

Every write on this API is asynchronous. You submit, you get a handle, you poll. Do not treat a 200
on the submit as "the work happened".

## Before you start

- This is self-hosted software. There is no vendor base URL. The endpoint is the operator's own
  Clouddriver instance, reached as `https://<clouddriver-loadbalancer-url>:<clouddriver-port>` per
  the Armory docs. Ask for it; never guess it.
- The spec declares no `securitySchemes`. Real deployments front this with mutual TLS and Fiat
  account/application authorization, and may add Armory Policy Engine rules on inbound HTTP
  (`spinnaker.http.authz`). A 403 here is usually policy, not a bad credential.

## Steps

1. **Submit the operation.** Call `cloudProviderOperationsUsingPOST`
   (`POST /{cloudProvider}/ops`) with the cloud provider in the path and the operation list as the
   JSON body. Use `cloudProviderOperationUsingPOST` (`POST /{cloudProvider}/ops/{name}`) when you are
   submitting a single named operation.

2. **Always set `clientRequestId`.** It is an optional query parameter, and you should treat it as
   mandatory. Clouddriver keys the created Task on it, so replaying the same submission with the same
   `clientRequestId` returns the existing Task instead of starting a second cloud operation. Generate
   it once per logical intent and reuse it on every retry of that intent — never per HTTP attempt.

3. **Do not use `/ops` or `/ops/{name}`.** `operationsUsingPOST` and `operationUsingPOST` are marked
   `deprecated: true` in the published spec. Use the cloud-provider-scoped forms above.

4. **Take the handle.** The response is a `StartOperationResult` with `id` and `resourceUri`. The
   `id` is the Task id.

5. **Poll the task.** Call `getUsingGET_1` (`GET /task/{id}`). Read `status` (a `Status` object with
   `phase`, `status`, `completed`, `failed`, `retryable`). Stop when `completed` or `failed` is true.
   `history[]` carries the phase-by-phase trail — surface it on failure rather than just the final
   status. Confirm `requestId` on the returned Task matches the `clientRequestId` you sent; that is
   how you know you are looking at your own work and not a replay of someone else's.

6. **Resume if it stalled.** If a Task is interrupted and `status.retryable` is true, call
   `resumeTaskUsingPOST` (`POST /task/{id}:resume`) rather than re-submitting the operation.

7. **Enumerate when you have lost the handle.** `listUsingGET_6` (`GET /task`) lists tasks. Prefer
   recovering the id this way over re-submitting.

## Error handling

- `400` — documented only for the Dynamic Accounts endpoints ("the account name is not defined in
  Clouddriver"). Elsewhere the spec is silent.
- `401` / `403` / `404` — declared on every operation with no schema and no body contract. Armory
  publishes no error-code registry, so do not pattern-match on error text; branch on status only.
- No `5xx` is declared anywhere in the spec. Treat any 5xx as retryable transport failure and reuse
  the same `clientRequestId`.
