# Core Contracts

## Purpose

Help future agents route changes across Keycloak's low-level shared contracts
without confusing public SPI, internal SPI, and serialized representation
surfaces.

## Scope

`common`, `core`, `server-spi`, and `server-spi-private`.

## Verified Facts

* `common` is the lowest shared utility layer. It includes feature profile
  behavior in `org.keycloak.common.Profile`, shared constants, enums, crypto
  helpers, and utility classes.
* `core` contains serialized contract types used by tokens, admin/client
  representations, OIDC, DPoP, SD-JWT, user profile, workflow, and provider
  representations. Treat these as wire/API contracts.
* `server-spi` is the public server-side extension and model contract layer.
  Changes here can affect external providers and many repository modules.
* `server-spi-private` contains internal server contracts and helpers used by
  `services`, model implementations, Quarkus runtime/deployment, and tests.
* Module dependency direction observed from POMs: `core` depends on `common`,
  `server-spi` depends on `core`, and `server-spi-private` depends on
  `server-spi` and `core`.

## Entry Points

* `common/src/main/java/org/keycloak/common/Profile.java` for feature gates,
  feature versions, dependencies, and profile validation.
* `core/src/main/java/org/keycloak/representations/` for stable serialized
  representation contracts.
* `core/src/main/java/org/keycloak/representations/AccessToken.java` for token
  JSON fields layered on `IDToken`.
* `server-spi/src/main/java/org/keycloak/models/KeycloakSession.java` for the
  session-scoped provider facade, context, transactions, invalidation, and
  lifecycle close state.
* `server-spi/src/main/java/org/keycloak/models/KeycloakSessionFactory.java`
  for factory-level provider/session lifecycle.
* `server-spi/src/main/java/org/keycloak/provider/ProviderFactory.java` for
  provider factory lifecycle: `create`, `init`, `postInit`, `close`, `getId`,
  ordering, config metadata, and dependencies.
* `server-spi/src/main/java/org/keycloak/models/*Model.java` for public realm,
  client, user, group, role, session, and authentication model contracts.
* `server-spi-private/src/main/java/org/keycloak/events/EventBuilder.java`,
  `EventListenerProvider.java`, `EventStoreProvider.java`, and
  `EventListenerTransaction.java` for internal event creation, listener,
  storage, and transaction buffering.

## Flows

* Provider access: runtime code calls `KeycloakSession.getProvider(...)`; the
  session contract creates missing session-scoped providers through the selected
  `ProviderFactory.create(KeycloakSession)`.
* Provider bootstrap: graph traces from
  `KeycloakProcessor.configureKeycloakSessionFactory` point to factory loading,
  provider checks, theme/resource handling, and
  `KeycloakRecorder.createSessionFactory`. Use this as discovery and verify the
  exact Quarkus source before changing bootstrap behavior.
* Event listener buffering: `EventListenerTransaction` buffers user/admin
  events, sends them on commit, and clears them on rollback. Email, logging,
  metrics, and feature modules can consume this path.
* Feature profile: `Profile.configure(...)` selects enabled feature versions,
  validates versioned and unversioned config consistency, checks dependencies,
  and prevents disabling essential features.

## Investigation Strategy

* Start with `search_graph` for exact classes such as `KeycloakSession`,
  `ProviderFactory`, `Profile`, `AccessToken`, `EventBuilder`, and
  `EventListenerTransaction`.
* Use `get_code_snippet` after `search_graph` returns the exact qualified name.
* Prefer exact qualified names or `qn_pattern` for broad model searches; broad
  `*Model` searches mix `server-spi` and `server-spi-private`.
* Use `trace_path` on exact symbols only. Generic names such as `getProvider`
  are noisy.

## Pitfalls

* Broad `query_graph` file path predicates returned unreliable results during
  ingest even though `search_graph` exposed valid `file_path` values. Prefer
  `search_graph(file_pattern=...)` for module scoping until this is clarified.
* `server-spi-private` is internal but widely used. Do not infer public API
  stability from usage volume.
* `core` and `common` have module-level Java release settings that differ from
  the repository-wide Java 17 baseline; verify POMs before using newer APIs in
  these modules.

## Related Wiki Pages

* [Component Routing](index.md)
* [Persistence And Storage](persistence-storage.md)
* [Quarkus Distribution](quarkus-distribution.md)
* [Flow Entry Points](../flows/index.md)

## Open Questions

* Whether `server-spi-private` has a documented internal API stability policy
  beyond naming and module placement.
* Whether the `core` and `common` Java release settings are intentional for
  compatibility or legacy carryover.

