# Authorization, SCIM, SSF, And AuthZEN

## Purpose

Route feature-module investigations for authorization services, SCIM, Shared
Signals Framework, and AuthZEN while preserving feature gates and separating
public APIs, admin APIs, and delivery internals.

## Scope

`authz`, `scim`, `ssf`, and `authzen`.

## Verified Facts

* `authz` is split into `policy` and `client`.
* `scim` is split into `core`, `model`, `client`, `services`, and `tests`.
* `ssf` is split into `core`, `transmitter`, `services`, and `tests`.
* `authzen` is split into `services` and `tests`.
* These modules are feature layers on Keycloak SPI/service wiring, not
  standalone servers.
* SCIM requires profile feature `SCIM_API` plus realm `isScimApiEnabled`.
* SSF requires profile feature `SSF` plus realm attribute
  `ssf.transmitterEnabled`.
* AuthZEN requires profile feature `AUTHZEN`.

## Entry Points

* Authorization:
  `authz/policy/common` for built-in policy providers and permission provider
  behavior; `authz/client` for authorization client crypto/API behavior.
* Authorization policy providers:
  the service-loader file for `PolicyProviderFactory` registers aggregate, JS,
  resource, role, scope, time, user, client, group, UMA, client-scope, and
  regex policy providers.
* SCIM:
  `ScimRealmResourceFactory` is the realm resource provider factory with id
  `scim`; `/v2/{resourceType}` dispatches to `ScimResourceTypeProvider`
  implementations for User, Group, ServiceProviderConfig, ResourceTypes, and
  Schemas.
* SSF:
  `SsfRealmResourceProviderFactory` is the realm endpoint provider with id
  `ssf`; `SsfRealmResourceProvider` exposes `/ssf/transmitter`.
* SSF subareas:
  public receiver/transmitter APIs in `ssf/services` and
  `ssf/transmitter/resources`, admin APIs in `ssf/services/admin`, and delivery
  internals in `ssf/transmitter/delivery`, `outbox`, and `stream`.
* AuthZEN:
  `AuthZenRealmResourceProviderFactory` has provider id `authzen`; endpoints
  are POST `access/v1/evaluation` and `access/v1/evaluations`.

## Flows

* Authorization permissions:
  built-in permission providers delegate to associated policies and cache
  decisions in `DefaultEvaluation`.
* SCIM:
  `ScimRealmResourceFactory` checks feature/realm gates, requires a valid bearer
  token, rejects public clients, then routes `/v2/{resourceType}` to registered
  resource-type providers. Generic CRUD/search lives mostly in
  `ScimResourceTypeResource`; resource-specific behavior belongs in
  `scim/model`.
* SSF:
  realm resource creation returns null unless transmitter is enabled. The
  transmitter endpoint exposes streams, status, verify, subjects, and poll.
  SSF also registers event SPI, transmitter SPI, global event listener,
  well-known providers, realm admin extension, and outbox delivery components.
* AuthZEN:
  AuthZEN resolves client, resource server, subject identity, resource, and
  scope, then delegates decisions to existing Keycloak authorization evaluators.
  Treat it as an adapter over Keycloak authorization services, not a separate
  PDP engine.

## Investigation Strategy

Recommended graph searches:

```text
search_graph(query="authorization policy provider permission evaluator resource server")
search_graph(query="SCIM resource type provider Users Groups Schemas ServiceProviderConfig")
search_graph(query="shared signals ssf transmitter stream verification outbox poll push")
search_graph(query="AuthZen evaluation evaluations well-known AuthorizationProvider evaluators")
```

If `file_pattern` fails for these top-level modules, use natural-language
searches or qualified-name searches, then verify source.

## Pitfalls

* Provider factories may return null when realm feature state is disabled; route
  existence alone is not proof of runtime availability.
* Route-label graph searches were noisy for these modules and included
  unrelated or generated route nodes. Verify JAX-RS annotations in source.
* During ingest, `search_graph(file_pattern="authz/.*")` and similar top-level
  module patterns returned zero even though symbol searches found module files.
* MCP `get_code_snippet` and `trace_path` had transient transport failures
  during this batch; source reads were used as runtime authority.

## Related Wiki Pages

* [Runtime Services](services-runtime.md)
* [Admin REST](admin-rest.md)
* [Operator And Tests](operator-tests.md)

## Open Questions

* Whether the `file_pattern` failures are schema/index-specific or a general
  graph path issue.
* Whether SSF outbox admin operations deserve a dedicated flow note after a
  deeper trace pass.
