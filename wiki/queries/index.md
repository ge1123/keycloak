# Reusable Query Strategies

Use graph tools first for code structure. Fall back to `rg` for literal strings,
configuration, Markdown, scripts, generated files, localization, or incomplete
graph coverage.

## Find The Current Project

1. Run `list_projects`.
2. Select the project whose `root_path` is this repository.
3. Run `index_status`.
4. If missing or stale, run `index_repository` for this repo.

Current indexed project name observed during setup:
`Users-wuchuni-Desktop-system_design-keycloak`.

## Common Graph Searches

| Goal | Strategy |
| --- | --- |
| Find a Java class or method | `search_graph` with `name_pattern` or a targeted natural-language `query`, then `get_code_snippet` |
| Find REST endpoints | `search_graph(label="Route")`, then filter by `file_path` and verify annotations in source |
| Trace runtime behavior | Find the exact method, then use `trace_path` when available; verify framework lifecycle behavior in source |
| Understand a subsystem | `get_architecture(aspects=["structure","dependencies","clusters"])`, then narrow with `search_graph` |
| Investigate UI behavior | Search symbols in `js/apps/*` and `js/libs/*`; fall back to `rg` for route strings and i18n keys |
| Investigate CI behavior | Use `rg` and direct reads in `.github/workflows`, `.github/actions`, and `.github/scripts` |

## Graph Pitfalls Observed During Ingest

Verified facts:

* Broad `query_graph` predicates over `file_path` returned empty or unreliable
  results in multiple batches even when `search_graph` returned valid
  `file_path` values. Prefer `search_graph(file_pattern=...)` for module
  scoping unless a future investigation verifies the Cypher schema.
* `trace_path` on generic method names such as `getProvider`, `format`,
  `scope`, or `getProfile` is noisy. Search exact symbols first, then trace the
  exact qualified name.
* Inherited methods can be reported through a sibling implementation in traces.
  Verify inherited storage-manager behavior in source before recording call
  flow conclusions.

Recommended exact-symbol examples:

```text
search_graph(name_pattern=".*configureKeycloakSessionFactory.*")
search_graph(query="KeycloakSession ProviderFactory EventListenerTransaction")
search_graph(file_pattern="model/.*", query="datastore storage manager jpa infinispan")
search_graph(qn_pattern=".*federation\\.ldap.*LDAPStorageProvider.*")
search_graph(qn_pattern=".*rest\\.admin-v2.*")
search_graph(query="admin ui routes realm clients users")
search_graph(query="account ui routes applications account security signing")
search_graph(query="authorization policy provider permission evaluator resource server")
search_graph(query="SCIM resource type provider Users Groups Schemas ServiceProviderConfig")
search_graph(query="shared signals ssf transmitter stream verification outbox poll push")
search_graph(query="AuthZen evaluation evaluations well-known AuthorizationProvider evaluators")
search_graph(file_pattern="operator/.*", query="reconciler dependent resource keycloak realm import")
search_graph(query="KeycloakIntegrationTest Registry Supplier lifecycle")
search_graph(file_pattern="quarkus/tests/.*", query="CLITestExtension distribution raw launch")
```

## High-Value Starting Queries

Authentication flow:

```text
search_graph(query="authentication flow login token endpoint")
```

Events:

```text
search_graph(query="event listener provider event store")
```

LDAP and federation:

```text
search_graph(query="user storage federation ldap kerberos")
```

Admin API v2:

```text
search_graph(query="admin REST realm clients users resources")
```

Quarkus provider discovery and bootstrap:

```text
search_graph(query="KeycloakProcessor KeycloakRecorder session factory provider discovery")
```

## Verified Facts

* The graph index contains Java and TypeScript symbols, route nodes, test edges,
  imports, and call relationships.
* Route searches can include generated, test, or synthetic entries; use route
  nodes as pointers, not final proof.
* CI and build configuration are better inspected as text because workflow YAML,
  shell scripts, and Maven profile wiring are configuration-heavy.

## Source Evidence

* `README.md`
* `docs/building.md`
* `docs/tests.md`
* `quarkus/README.md`
* `js/package.json`
* `js/apps/admin-ui/package.json`
* `js/apps/account-ui/package.json`
* `.github/workflows/ci.yml`
* `.github/workflows/js-ci.yml`
* `.github/workflows/operator-ci.yml`
* `.github/CODEOWNERS`
