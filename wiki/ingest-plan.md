# Wiki Ingest Plan

This page is the coordinator queue for maintainer-grade wiki ingest. Keep it
small and update it after each completed batch.

## Current Status

Verified facts:

* The selected codebase-memory project is
  `Users-wuchuni-Desktop-system_design-keycloak`.
* `index_status` reports the project as `ready` with 177713 nodes and 870598
  edges on 2026-06-24.
* Existing wiki pages before this queue: `wiki/index.md`,
  `wiki/components/index.md`, `wiki/queries/index.md`,
  `wiki/flows/index.md`, `wiki/qa/index.md`, `wiki/source.md`,
  `wiki/log.md`, and templates under `wiki/templates/`.

Graph evidence:

* `get_architecture` reports dominant edge types including `CALLS`, `USAGE`,
  `DEFINES`, `DEFINES_METHOD`, `TESTS`, `IMPORTS`, `DECORATES`, `CONFIGURES`,
  `HANDLES`, and `HTTP_CALLS`.
* High-count graph packages include `src`, `integration-arquillian`, `base`,
  `jpa`, `infinispan`, `apps`, `core`, `tests`, `runtime`, `admin-v2`,
  `storage-private`, `ldap`, and `model`.

Source evidence:

* `wiki/index.md` defines the evidence contract and read order.
* `wiki/components/index.md` already routes top-level server, Quarkus, REST,
  UI, operator, docs, CI, and test surfaces.
* `wiki/queries/index.md` records the current project lookup strategy and
  reusable graph query examples.

Inference:

* The current graph is sufficient for the first ingest batches. Re-index only
  if source changes during ingest or a batch finds graph/source disagreement
  that looks like stale indexing.
* Existing index pages should remain the navigation layer. New batch files
  should hold durable findings and link back into `wiki/index.md`.

Open questions:

* The graph status output does not include a timestamp or index mode. Preserve
  `wiki/source.md`'s warning that initial coverage may be fast-mode quality
  until a future index run confirms otherwise.

## Execution Rules

* Start with 2-3 high-value batches.
* Inspect each diff before starting another batch.
* Normalize terminology into the existing read order: components, queries,
  flows, QA, source.
* Avoid overlapping agents on the same target files.
* Prefer sequential execution when target files include `wiki/index.md`.
* Parallelize only when target files are disjoint.

## Queue

This queue follows the A-O batch order requested for the 2026-06-25 deep
ingest. Keep status brief; durable facts belong in focused pages, not here.

| Status | Batch id | Scope | Durable output |
| --- | --- | --- | --- |
| Completed | A Core Contracts | `core`, `common`, `server-spi`, `server-spi-private` | [Core Contracts](components/core-contracts.md) |
| Completed | B Runtime Services | `services`, auth flows, sessions, events, forms, email, login/account server behavior | [Runtime Services](components/services-runtime.md) |
| Completed | C Persistence And Storage | `model`, `model/jpa`, `model/infinispan`, `model/storage*` | [Persistence And Storage](components/persistence-storage.md) |
| Completed | D Federation | `federation/ldap`, `federation/kerberos`, `federation/sssd`, `federation/ipatuura` | [Federation](components/federation.md) |
| Completed | E Quarkus Distribution | `quarkus/deployment`, `quarkus/runtime`, `quarkus/server`, `quarkus/dist`, `quarkus/container` | [Quarkus Distribution](components/quarkus-distribution.md) |
| Completed | F Admin REST API v2 | `rest/admin-v2/api`, `rest/admin-v2/services`, `rest/admin-v2/tests` | [Admin REST](components/admin-rest.md) |
| Completed | G Admin UI Extension REST | `rest/admin-ui-ext` | [Admin REST](components/admin-rest.md) |
| Completed | H Authorization | `authz` | [Authorization, SCIM, SSF, And AuthZEN](components/authorization-scim-ssf-authzen.md) |
| Completed | I SCIM / SSF / AuthZEN | `scim`, `ssf`, `authzen` | [Authorization, SCIM, SSF, And AuthZEN](components/authorization-scim-ssf-authzen.md) |
| Completed | J JavaScript UI And Client Packages | `js/apps/*`, `js/libs/*`, `js/themes-vendor` | [JavaScript UI And Themes](components/javascript-ui.md) |
| Completed | K Themes | `themes`, `quarkus/dist/src/main/content/themes` | [JavaScript UI And Themes](components/javascript-ui.md) |
| Completed | L Kubernetes Operator | `operator` | [Operator And Tests](components/operator-tests.md) |
| Completed | M Tests | `tests`, `test-framework`, `testsuite/integration-arquillian`, `quarkus/tests`, feature tests | [Operator And Tests](components/operator-tests.md) |
| Completed | N Docs And CI | `docs`, `docs/guides`, `.github/*`, CODEOWNERS | [Docs, CI, And Build](docs-ci-build.md) |
| Completed | O Build System | root/module POMs, JS package metadata, pnpm/wireit | [Docs, CI, And Build](docs-ci-build.md) |

## Sub-Agent Prompt Template

```text
You are an ingest worker for the Keycloak wiki.

Batch:
<batch id>

Scope:
<specific directories/modules/features>

Target wiki files:
<paths under wiki/>

Instructions:
1. Read `wiki/index.md` and any directly relevant existing wiki page.
2. Use codebase-memory-mcp first for code discovery.
3. Use source reads only to verify ambiguous graph evidence or framework/runtime wiring.
4. Produce concise durable wiki knowledge, not source summaries.
5. Record:
   - ownership / module responsibility
   - key entry points
   - important classes/functions/routes
   - common graph queries
   - runtime/build/test notes
   - pitfalls
   - verified facts vs inference
   - open questions
6. Update the target wiki file(s).
7. Update `wiki/index.md` if new pages are added or routing changed.
8. Do not modify production code.

Done criteria:
- Wiki page is updated.
- Claims are backed by graph/source evidence.
- No large generated dumps.
- Any uncertainty is explicitly labeled.
```

## Immediate Next Batches

Run these first, sequentially unless separate agents can guarantee no target
file overlap:

1. `architecture-routing`
2. `build-test-workflow`
3. `services-runtime`

## Conflict Log

Open questions:

* No source-vs-graph conflicts have been found in this coordinator setup pass.
