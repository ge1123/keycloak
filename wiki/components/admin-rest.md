# Admin REST

## Purpose

Route Admin REST API v2 and Admin UI extension REST work without conflating the
public v2 API contract with admin-console helper endpoints.

## Scope

`rest/admin-v2/api`, `rest/admin-v2/services`, `rest/admin-v2/tests`, and
`rest/admin-ui-ext`.

## Verified Facts

* Admin API v2 contract work starts in `rest/admin-v2/api`.
* Admin API v2 implementation work starts in `rest/admin-v2/services`.
* Admin API v2 tests live in `rest/admin-v2/tests` and use the newer
  `keycloak-test-framework-*` artifacts.
* Admin UI extension REST work starts in `rest/admin-ui-ext`; it is a separate
  console-helper surface, not the same as Admin API v2.
* Admin API v2 currently has a client-focused contract surface rooted at
  `admin/api/{realmName}/clients/v2`.
* Admin API v2 is gated by `Profile.Feature.CLIENT_ADMIN_API_V2`.
* Admin UI extension REST registers an `AdminRealmResourceProviderFactory` with
  provider id `ui-ext` and is gated by `Profile.Feature.ADMIN_V2`.

## Entry Points

* `rest/admin-v2/api/src/main/java/org/keycloak/admin/api/AdminRootV2.java`
  defines the v2 root path `admin/api` and realm dispatch.
* `AdminApi`, `ClientsApi`, and `ClientApi` define the current client-oriented
  v2 API contract.
* `rest/admin-v2/services/src/main/java/org/keycloak/rest/admin/api/DefaultAdminRootV2.java`
  implements the v2 root provider and feature gate.
* `DefaultAdminApi` authenticates with existing v1
  `AdminRoot.authenticateRealmAdminRequest`, resolves realm by name, sets realm
  context, and builds `AdminPermissions`.
* `DefaultClientsApi` and `DefaultClientApi` are thin adapters over
  `DefaultClientService`.
* `AdminEventV2Builder` marks v2 admin events with `apiVersion=v2`.
* `rest/admin-ui-ext/src/main/java/org/keycloak/admin/ui/rest/AdminExtProvider.java`
  registers the `ui-ext` provider.
* `AdminExtResource` exposes admin-console helper subresources such as
  authentication management, brute-force user, roles, role mappings, sessions,
  realms, and root realm info.
* `services/src/main/java/org/keycloak/services/resources/admin/RealmAdminResource.java`
  dispatches unresolved `{extension}` segments through
  `session.getProvider(AdminRealmResourceProvider.class, extension)`.

## Flows

* Admin API v2 root:
  `AdminRootV2` declares `admin/api`, `DefaultAdminRootV2` checks
  `CLIENT_ADMIN_API_V2`, dispatches realm paths, and rejects disabled requests
  with `NotFoundException`, including preflight handling.
* Realm authentication:
  `DefaultAdminApi` reuses existing v1 admin realm authentication, then sets
  realm context and constructs admin permissions before returning API resources.
* Client API:
  v2 REST classes delegate behavior to `DefaultClientService`; service-layer
  support includes model mappers, query parsing/evaluation, roles service,
  exception mapping, and patch handling.
* OpenAPI generation:
  `rest/admin-v2/services/pom.xml` runs SmallRye OpenAPI over v2
  representations, API interfaces, and validation constraints, then copies the
  generated OpenAPI artifact into the services JAR and
  `js/libs/keycloak-admin-client/openapi.json`.
* Admin UI extension:
  `RealmAdminResource` dispatches the `{extension}` path to provider id
  `ui-ext`, and `AdminExtResource` contributes console-specific relative
  subpaths.

## Investigation Strategy

* Verify final REST paths by inspecting the root resource, subresource
  interfaces, provider service files, and dispatch sites. Do not infer final
  paths from subresource classes alone.
* Use `search_graph(qn_pattern=".*rest\\.admin-v2.*")` when path filtering is
  unreliable.
* For Admin UI extension endpoints, inspect both `RealmAdminResource` and
  `AdminExtResource`.
* For admin-client generated API changes, check the Admin API v2 OpenAPI
  generation flow and `js/libs/keycloak-admin-client/openapi.json`.

## Pitfalls

* Admin API v2 is not a broad replacement for every legacy admin REST endpoint
  yet; current graph/source evidence shows a client-focused API surface.
* Admin UI extension REST serves console-specific helper endpoints and should
  not be documented as a general public Admin API v2 surface.
* Feature gates differ: `CLIENT_ADMIN_API_V2` for Admin API v2 and `ADMIN_V2`
  for Admin UI extension REST.
* Generated OpenAPI and Kiota client artifacts are API-contract surfaces; prefer
  changing server contracts upstream when appropriate.

## Related Wiki Pages

* [JavaScript UI And Themes](javascript-ui.md)
* [Runtime Services](services-runtime.md)
* [Docs, CI, And Build](../docs-ci-build.md)

## Open Questions

* Whether `CLIENT_ADMIN_API_V2` remains the long-term gate once Admin API v2
  grows beyond client APIs.
* Whether generated `openapi.json` is expected to be updated manually or only by
  Maven process-classes workflows.

