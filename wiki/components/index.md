# Component Routing

Use this page to choose the first files, modules, and tests to inspect. Confirm
symbol-level behavior with graph tools and source before changing code.

## Core Server

| Area | Start here | Notes |
| --- | --- | --- |
| Domain contracts and shared model types | `core`, `common`, `server-spi`, `server-spi-private`; see [Core Contracts](core-contracts.md) | Prefer SPI boundaries over reaching into private internals unless the existing code does so |
| Runtime services, auth, sessions, events, login, forms | `services`; see [Runtime Services](services-runtime.md) | Graph searches for authentication, events, forms, and token flows usually land here |
| Storage and cache behavior | `model`, `model/jpa`, `model/infinispan`, `model/storage*`; see [Persistence And Storage](persistence-storage.md) | Check both model contracts and provider-specific implementations |
| User federation | `federation/ldap`, `federation/kerberos`, `federation/sssd`, `federation/ipatuura`; see [Federation](federation.md) | LDAP/Kerberos often spans `services`, `model`, and admin UI configuration |
| Authorization services | `authz`; see [Authorization, SCIM, SSF, And AuthZEN](authorization-scim-ssf-authzen.md) | Keep authorization-service tests near the touched module |
| Protocol/domain extensions | `scim`, `ssf`, `authzen`, `saml-core`, `saml-core-api`; see [Authorization, SCIM, SSF, And AuthZEN](authorization-scim-ssf-authzen.md) | Each has feature-specific services and tests |

## Quarkus and Distribution

| Area | Start here | Notes |
| --- | --- | --- |
| Build-time augmentation | `quarkus/deployment`; see [Quarkus Distribution](quarkus-distribution.md) | `KeycloakProcessor` is a common graph entry for provider discovery and build steps |
| Runtime bootstrap and CLI wiring | `quarkus/runtime`; see [Quarkus Distribution](quarkus-distribution.md) | `KeycloakMain` and `KeycloakRecorder` are common entry points |
| Server app artifact | `quarkus/server`; see [Quarkus Distribution](quarkus-distribution.md) | Quarkus application wrapper for the server extension |
| Distribution packaging | `quarkus/dist`, `distribution`; see [Quarkus Distribution](quarkus-distribution.md) | Check generated distribution content before changing packaging |
| Container image | `quarkus/container`, `operator/scripts/Dockerfile-custom-image`; see [Quarkus Distribution](quarkus-distribution.md) | CI and operator tests may depend on image layout |

## REST, Admin, and UI

| Area | Start here | Notes |
| --- | --- | --- |
| Admin REST API v2 contract | `rest/admin-v2/api`; see [Admin REST](admin-rest.md) | API interfaces define route-level contracts |
| Admin REST API v2 implementation | `rest/admin-v2/services`; see [Admin REST](admin-rest.md) | Implementations often delegate to `services` and model APIs |
| Admin UI extension endpoints | `rest/admin-ui-ext`; see [Admin REST](admin-rest.md) | Used by UI-specific server endpoints |
| Admin console | `js/apps/admin-ui`; see [JavaScript UI And Themes](javascript-ui.md) | React, Vite, PatternFly, pnpm, wireit |
| Account console | `js/apps/account-ui`; see [JavaScript UI And Themes](javascript-ui.md) | React, Vite, PatternFly, Playwright tests |
| Shared JS libraries | `js/libs/keycloak-admin-client`, `js/libs/ui-shared`; see [JavaScript UI And Themes](javascript-ui.md) | Client API changes can affect UI packages |
| Themes | `themes`, `js/themes-vendor`, `quarkus/dist/src/main/content/themes`; see [JavaScript UI And Themes](javascript-ui.md) | Keep server-rendered and bundled theme behavior separate |

## Operator, Docs, and Automation

| Area | Start here | Notes |
| --- | --- | --- |
| Kubernetes Operator | `operator`; see [Operator And Tests](operator-tests.md) | Build with `-Poperator`; CI uses local API server, Minikube, remote, and OLM paths |
| Documentation | `docs`, `docs/guides`, `docs/documentation` | User-facing behavior changes usually need docs |
| CI workflows | `.github/workflows`; see [Docs, CI, And Build](../docs-ci-build.md) | `ci.yml`, `js-ci.yml`, and `operator-ci.yml` are the main workflow routers |
| Shared CI actions/scripts | `.github/actions`, `.github/scripts`; see [Docs, CI, And Build](../docs-ci-build.md) | Prefer existing actions over adding ad hoc workflow logic |
| Ownership hints | `.github/CODEOWNERS`; see [Docs, CI, And Build](../docs-ci-build.md) | Use as a broad maintainer map, not as source behavior evidence |

## Tests

| Test surface | Start here |
| --- | --- |
| Modern base tests | `tests`, `tests/base`, `test-framework`; see [Operator And Tests](operator-tests.md) |
| Legacy integration tests | `testsuite/integration-arquillian`; see [Operator And Tests](operator-tests.md) |
| Quarkus distribution tests | `quarkus/tests`; see [Operator And Tests](operator-tests.md) |
| Admin API v2 tests | `rest/admin-v2/tests`; see [Admin REST](admin-rest.md) and [Operator And Tests](operator-tests.md) |
| Feature-specific tests | `scim/tests`, `ssf/tests`, `authzen/tests`; see [Authorization, SCIM, SSF, And AuthZEN](authorization-scim-ssf-authzen.md) |
| Clustering and database support | `tests/clustering`, `test-framework/db-*`, `test-framework/clustering`; see [Operator And Tests](operator-tests.md) |

## Open Questions

* Some graph route nodes come from generated or test resources with empty
  `file_path`; verify route behavior in source before relying on those nodes.
* The repository carries both newer test infrastructure and legacy Arquillian
  tests. Choose tests based on the changed module rather than assuming one suite
  covers all behavior.
