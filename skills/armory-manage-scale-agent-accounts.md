---
name: Manage Kubernetes accounts across Clouddriver and the Armory Scale Agent
description: >-
  Inspect which Kubernetes accounts an Armory Scale Agent is handling, page through the credentials
  registry, and create, update, delete or audit an account definition.
api: openapi/armory-scale-agent-agent-accounts-openapi.yml
generated: '2026-08-06'
method: generated
source: >-
  openapi/_original/armory-scale-agent-swagger.json +
  https://docs.armory.io/plugins/scale-agent/reference/dynamic-accounts/
operations:
  - getAccountUsingGET
  - getNamespacesUsingGET
  - listAccountCredentialsUsingGET
  - listAccountsByTypeUsingGET
  - getAccountCredentialsDetailsUsingGET
  - createAccountUsingPOST
  - updateAccountUsingPUT
  - deleteAccountUsingDELETE
  - getAccountHistoryUsingGET
---

# Manage Kubernetes accounts

Two account surfaces sit side by side here and they are not the same thing. The Armory-specific
endpoints under `/agents/kubernetes/accounts` tell you what a Scale Agent is actually handling. The
Clouddriver `/credentials` endpoints are the credential registry those accounts are defined in.

## Read what the Agent is handling

1. `getAccountUsingGET` (`GET /agents/kubernetes/accounts/{accountName}`) returns the account if it
   is handled by an Armory Agent. The `KubesvcAccount` it returns carries the Agent-assignment state
   nothing else exposes: `zoneId` (the zone the account is pinned to, absent when it can float to any
   Agent), `status`, `certFingerprint`, `kubeHost`, `kubeVersion`, `dynamic`, `lastUpdated`,
   `maxWatchAgeSeconds` and `errorMessage`.
2. `getNamespacesUsingGET` (`GET /agents/kubernetes/accounts/{accountName}/namespaces`) returns the
   namespaces that account covers.
3. A `400` from either means the account name is not defined in Clouddriver at all — go look at the
   credentials registry before assuming an Agent problem.

## Page the credentials registry

4. `listAccountCredentialsUsingGET` (`GET /credentials`) lists account credentials. It pages with
   `limit` plus a `startingAccountName` continuation token rather than page numbers — carry the last
   name you saw forward. `expand` widens each record.
5. `listAccountsByTypeUsingGET` (`GET /credentials/type/{accountType}`) narrows to one provider type.
6. `getAccountCredentialsDetailsUsingGET` (`GET /credentials/{accountName}`) returns one account.

The Dynamic Accounts listing documented by Armory pages differently — `page` (default 1) and `limit`
(default 100), returning `{items, page, limit, total}` sorted by account name ascending. Do not
assume one paging contract across the whole API.

## Write

7. `createAccountUsingPOST` (`POST /credentials`) takes a `CredentialsDefinition` body. Armory
   documents `name` (matching the Clouddriver account name), an optional `zoneId` to pin the account
   to a specific Agent replicaset — defaulting to `deploymentName_namespace` — and an optional
   `kubeConfigFile` that supports `encrypted`/`encryptedFile` secret references.
8. `updateAccountUsingPUT` (`PUT /credentials`) replaces a definition.
9. `deleteAccountUsingDELETE` (`DELETE /credentials/{accountName}`) removes one.
10. `getAccountHistoryUsingGET` (`GET /credentials/{accountName}/history`) is the audit trail — read
    it before and after any write so you can prove what changed.

## Rules

- Never put a raw kubeconfig in `kubeConfigFile`. Armory supports encrypted secret references
  specifically so the credential does not land in an API payload.
- Account writes are not idempotent — `/credentials` accepts no `clientRequestId`. Read first with
  `getAccountCredentialsDetailsUsingGET`, then decide between POST and PUT. A blind retry of
  `createAccountUsingPOST` is not safe.
- `403` on these endpoints is usually Fiat account authorization or an Armory Policy Engine rule, not
  a broken credential.
