# Persistence And Storage

## Purpose

Route storage, persistence, cache, and datastore investigations across model
contracts, private managers, JPA providers, Infinispan providers, and storage
admin services.

## Scope

`model`, `model/storage`, `model/storage-private`, `model/jpa`,
`model/infinispan`, and `model/storage-services`.

## Verified Facts

* `model/storage` provides shared storage abstractions, provider models,
  adapters, and `AbstractStorageManager`; it is not the main implementation
  layer.
* `model/storage-private` owns private manager/facade behavior such as
  `UserStorageManager`, datastore provider factory code, import/export,
  migration, scheduled sync, and private persister SPIs.
* `model/jpa` provides persistent local implementations, JPA connection
  providers, local realm/user/client/group/role providers, user-session and
  revoked-token persisters, Liquibase/updater logic, and DB compatibility
  verification.
* `model/infinispan` provides cache, session, cluster, embedded/remote
  Infinispan, and ProtoStream compatibility behavior.
* `model/storage-services` contributes admin/resource-layer storage management
  and import/export service providers.
* POM evidence shows `model/infinispan` depends on JPA and storage-private,
  JPA depends on storage and storage-private, and storage-private depends on
  storage.

## Entry Points

* `org.keycloak.storage.datastore.DefaultDatastoreProvider` is the central
  datastore facade.
* `org.keycloak.storage.datastore.DefaultDatastoreProviderFactory` creates the
  datastore provider and participates in provider wiring.
* `org.keycloak.storage.AbstractStorageManager` resolves enabled realm
  component storage providers, sorts provider models by priority, checks
  capability interfaces, caches provider instances in the `KeycloakSession`,
  and enlists them for close.
* `org.keycloak.storage.UserStorageManager` routes local and external user
  lookups, queries, validation, and federated storage interactions.
* `org.keycloak.connections.jpa.JpaConnectionProvider` and factory classes are
  the JPA connection entry points.
* `JpaRealmProviderFactory`, `JpaUserProviderFactory`,
  `JpaUserSessionPersisterProviderFactory`, and
  `JpaRevokedTokensPersisterProviderFactory` are common JPA provider anchors.
* `InfinispanUserSessionProvider`, `InfinispanUserSessionProviderFactory`,
  `RemoteUserSessionProviderFactory`, and
  `DefaultInfinispanTransactionProviderFactory` are common cache/session
  anchors.
* `model/storage-services/.../UserStorageProviderResource` is an admin
  operations anchor for sync, remove, and unlink imported users.

## Flows

* Datastore facade: `DatastoreProvider.users()` prefers a cache user provider
  when present, otherwise falls back to `UserStorageManager`. Similar
  cache-first routing exists for other realm-backed domains.
* External user lookup: `UserStorageManager.getUserById` checks `StorageId`;
  local IDs go to local storage plus validation, while provider IDs route to
  the matching storage provider capability.
* Provider instantiation: `AbstractStorageManager.getStorageProviderInstance`
  resolves the realm component model, fetches the provider factory, checks the
  capability interface, creates the provider with the session and component
  model, then enlists it for close.
* User query: `UserStorageManager` merges local, enabled external providers,
  and federated storage with special pagination/count handling and graceful
  degradation when providers throw.
* Offline sessions: `InfinispanUserSessionProvider` writes offline sessions to
  the persister and imports/reimports offline session data into Infinispan when
  needed.
* Infinispan transactions: `DefaultInfinispanTransactionProviderFactory`
  enlists after-completion transactions and conditionally enables prepare-phase
  DB locking when JDBC isolation is `READ_COMMITTED`.

## Investigation Strategy

* Use `search_graph(file_pattern="model/.*", query="storage provider manager
  datastore jpa infinispan")` to locate storage symbols.
* Use exact qualified names before `trace_path`; shorthand names failed during
  ingest.
* Verify provider discovery through `META-INF/services` and module POMs. The
  graph can find provider classes but not always active service-loader wiring.
* For user federation storage behavior, cross-check this page with the
  federation page and the owning provider implementation.

## Pitfalls

* A graph trace for `UserStorageManager.getUserById` reported
  `RoleStorageManager.getStorageProviderInstance` as a callee, but source shows
  inherited `AbstractStorageManager` logic is the relevant implementation.
* Two-hop traces from `DefaultDatastoreProvider.users` included unrelated
  callers; treat them as discovery only.
* Broad Cypher file path queries returned empty or malformed results during
  ingest. `search_graph(file_pattern="model/.*")` worked better.
* JPA and storage-private tests can update DB compatibility snapshots only
  intentionally; do not use snapshot-update profiles as routine verification.

## Related Wiki Pages

* [Core Contracts](core-contracts.md)
* [Federation](federation.md)
* [Component Routing](index.md)

## Open Questions

* Which full integration suite best validates `model/storage-services` admin
  endpoints beyond module compilation.
* Whether inherited-method resolution in the graph can be improved for storage
  manager traces.

