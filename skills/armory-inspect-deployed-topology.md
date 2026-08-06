---
name: Inspect what is actually deployed - applications, clusters, server groups, instances
description: >-
  Walk the read surface from application down to a single instance's console output, and use search
  and cache introspection when you do not know where to start.
api: openapi/armory-scale-agent-applications-openapi.yml
generated: '2026-08-06'
method: generated
source: openapi/_original/armory-scale-agent-swagger.json
operations:
  - listUsingGET
  - getUsingGET
  - listByAccountUsingGET
  - getForAccountUsingGET
  - getForAccountAndNameUsingGET
  - getServerGroupsUsingGET
  - getServerGroupUsingGET
  - getTargetServerGroupUsingGET
  - getServerGroupSummaryUsingGET
  - getForApplicationUsingGET
  - getInstanceUsingGET
  - getConsoleOutputUsingGET
  - listUsingGET_7
  - searchUsingGET
  - getAgentIntrospectionsUsingGET
---

# Inspect deployed topology

The read surface is a strict hierarchy: **Application → Cluster → ServerGroup → Instance**. Walk it
down; do not try to jump.

## Walk down

1. `listUsingGET` (`GET /applications`) lists applications. `expand` inlines clusters;
   `restricted` limits to what the caller is authorized for. Leave `expand` off on large estates —
   the expanded projection is `ApplicationViewModel` and it gets heavy.
2. `getUsingGET` (`GET /applications/{name}`) returns one application with `attributes` and
   `clusterNames`.
3. `listByAccountUsingGET` (`GET /applications/{application}/clusters`) returns clusters grouped by
   account. `getForAccountUsingGET` narrows to one account;
   `getForAccountAndNameUsingGET` and `getForAccountAndNameAndTypeUsingGET` narrow further.
4. `getServerGroupsUsingGET`
   (`GET /applications/{application}/clusters/{account}/{clusterName}/{type}/serverGroups`) lists
   server groups; `getServerGroupUsingGET` returns one by name.
5. When you want "whatever is current" rather than a named version, use
   `getTargetServerGroupUsingGET` with a `target` (the newest/oldest/current selector Spinnaker
   supports) and `getServerGroupSummaryUsingGET` for a `summaryType` projection instead of the full
   object. Set `validateOldest` deliberately — it changes whether a stale target errors or resolves.
6. `getForApplicationUsingGET` (`GET /applications/{application}/serverGroupManagers`) gives you the
   controllers (e.g. Kubernetes Deployments) that own those server groups.
7. `getInstanceUsingGET` (`GET /instances/{account}/{region}/{id}`) returns one instance with its
   `health[]` and `healthState`. `getConsoleOutputUsingGET` (`.../console`) returns its console
   output — this is the one endpoint likely to return operator-visible secrets in a log line, so do
   not persist its output into anything shared.
8. `listUsingGET_7` (`GET /applications/{application}/rawResources`) returns Kubernetes resources
   that do not map onto the server-group model, carrying `kind`, `apiVersion`, `namespace` and
   `moniker`.

## When you do not know where to start

- `searchUsingGET` (`GET /search`) takes `q`, `type`, `platform`, `pageSize` and `page`, and returns
  a `SearchResultSet` with `totalMatches`. Use it to resolve a name to an entity before walking.
- `getAgentIntrospectionsUsingGET` (`GET /cache/introspection`) reports caching-agent health:
  `totalAdditions`, `totalEvictions`, `lastExecutionStartDate`, `lastExecutionDurationMs`,
  `lastError`. If the topology you are reading looks stale, check here before concluding the
  infrastructure is wrong — you may be reading a cold or failing cache.

## Rules

- Everything on this path is a **cache read**, not a live cloud query. Clouddriver serves from its
  cache; `getAgentIntrospectionsUsingGET` is how you judge whether that cache is current.
- `Moniker` (`app`, `cluster`, `stack`, `detail`, `sequence`) is the real cross-entity naming key.
  Join on it rather than on parsed name strings.
- These are all `GET`s and safe to retry. Do not send `clientRequestId` here — it exists only on the
  `/ops` submission endpoints.
