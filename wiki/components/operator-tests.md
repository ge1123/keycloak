# Operator And Tests

## Purpose

Route Kubernetes Operator and test-suite investigations across controller
entry points, image/OLM scripts, modern JUnit 5 test infrastructure, legacy
Arquillian, Quarkus distribution tests, and feature-specific test modules.

## Scope

`operator`, `operator/src`, `operator/scripts`, `operator/Dockerfile`,
`tests`, `test-framework`, `testsuite/integration-arquillian`,
`quarkus/tests`, and feature-specific tests under `scim/tests`, `ssf/tests`,
`authzen/tests`, and `rest/admin-v2/tests`.

## Verified Facts

* `operator` is a Quarkus Operator SDK module for Keycloak Kubernetes custom
  resources.
* `operator` depends on Fabric8, Quarkus Operator SDK, Kubernetes/OpenShift
  clients, Keycloak core/admin-client/admin-v2-api, and test dependencies for
  Quarkus, kube API test, Rest Assured, Mockito, and Fabric8 JUnit.
* `test-framework` is the newer JUnit 5 framework and owns Keycloak server,
  database, browser, and injected resource lifecycles.
* `tests/*` modules consume the modern test framework; `test-framework` is
  infrastructure.
* `testsuite/integration-arquillian` remains a separate legacy integration
  suite.
* `quarkus/tests` covers Quarkus CLI/distribution behavior and depends on
  distribution artifacts, Quarkus JUnit internals, Rest Assured, Testcontainers,
  DB containers, and JDBC drivers.
* `scim/tests`, `ssf/tests`, and `authzen/tests` are children of
  `keycloak-tests-parent` and import the test-framework BOM.

## Entry Points

* `operator/src/main/java/org/keycloak/operator/controllers/KeycloakController.java`
  is the main `Reconciler<Keycloak>` and uses Java Operator SDK workflow
  dependents for StatefulSet/deployment, admin secret, ingress, services,
  network policy, and ServiceMonitor.
* `KeycloakRealmImportController` reconciles `KeycloakRealmImport`, waits for
  the referenced Keycloak StatefulSet, reconciles import job/secret dependents
  when ready replicas exist, and reports started/done/error status.
* `KeycloakClientBaseController` is the base reconciler/cleaner for OIDC/SAML
  client CRs. It requires the referenced Keycloak CR, verifies
  `client-admin-api:v2`, converts CR specs to Admin API v2 representations,
  invokes the admin API, hashes desired state, and updates status.
* `operator/Dockerfile` builds the operator image from `target/quarkus-app`.
* `operator/scripts/Dockerfile-custom-image` builds a Keycloak image with
  postgres, health, metrics, and rolling-updates enabled for operator
  integration tests requiring custom images.
* `@KeycloakIntegrationTest` uses `KeycloakIntegrationTestExtension`, which
  manages registry lifecycle and parameter support.
* `CoreTestFrameworkExtension` registers suppliers for admin clients, realms,
  clients, users, server modes, URLs, databases, events, HTTP helpers,
  certificates, and crypto.
* `CLITestExtension` under `quarkus/tests/junit5` can run raw distributions,
  copy test providers, configure env vars, start DB/Infinispan containers, and
  invoke distribution hooks before startup.

## Flows

* Operator reconcile:
  `KeycloakController` handles pause annotation, default `instances`,
  OpenShift hostname defaults, update strategy decisions, managed workflow
  reconciliation, status aggregation, and polling/rescheduling.
* Operator build:
  `operator/pom.xml` runs Quarkus build, copies filtered Kubernetes resources
  into `target`, copies `ubi-null.sh` from `quarkus/container`, adds generated
  sources, runs surefire in `verify`, and assembles Quarkus and OLM artifacts.
* Operator CI:
  local API-server tests run `./mvnw install -Poperator -pl :keycloak-operator
  -am`; Minikube remote tests split by slow groups; OLM tests build/push bundle
  and catalog images, install OLM, deploy examples, verify CRDs, and test
  namespace modes.
* Modern tests:
  start in the feature's `tests/*` module for behavior coverage, then inspect
  `test-framework` suppliers only when lifecycle, injection, server mode, or
  resource setup matters.
* Legacy tests:
  use `testsuite/integration-arquillian` for areas that have not migrated or
  still depend on Arquillian/Selenium/Graphene/WildFly/container setup.

## Investigation Strategy

Recommended graph searches:

```text
search_graph(file_pattern="operator/.*", query="reconciler dependent resource keycloak realm import")
search_graph(query="KeycloakIntegrationTest Registry Supplier lifecycle")
search_graph(file_pattern="quarkus/tests/.*", query="CLITestExtension distribution raw launch")
```

Use source reads for operator scripts, Dockerfiles, workflow YAML, Maven
profiles, and test resource config. The graph is useful for Java symbols but
weak for shell, OLM, image, and workflow behavior.

## Pitfalls

* Operator tests are not a single mode. `local_apiserver` is the default, while
  tests requiring a real Keycloak pod or ingress usually need `remote`/Minikube
  or CI OLM mode.
* Broad graph regex searches for `Controller|Reconciler|DependentResource`
  returned no matches during ingest, while BM25/file-pattern searches worked.
  Verify operator class inventories from source layout.
* Do not assume modern `@KeycloakIntegrationTest` coverage replaces legacy
  Arquillian coverage. Choose by changed area and existing nearby tests.
* Scripts, Dockerfiles, workflow YAML, and Maven profile behavior should be read
  as source/config, not inferred from graph.

## Related Wiki Pages

* [Quarkus Distribution](quarkus-distribution.md)
* [Docs, CI, And Build](../docs-ci-build.md)
* [Authorization, SCIM, SSF, And AuthZEN](authorization-scim-ssf-authzen.md)
* [Admin REST](admin-rest.md)

## Open Questions

* Which operator tests are expected to be runnable locally without Minikube
  versus only in CI remote/OLM modes.
* Whether `operator/README.md` should be updated to prefer `./mvnw` from the
  repository root in examples.
* Whether legacy Arquillian areas have an active migration plan to the new test
  framework, and which suites are intentionally left legacy.

