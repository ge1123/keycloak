# Quarkus Distribution

## Purpose

Route Keycloak Quarkus extension, bootstrap, CLI, distribution packaging, and
container image investigations without mixing build-time augmentation with
runtime behavior.

## Scope

`quarkus/deployment`, `quarkus/runtime`, `quarkus/server`,
`quarkus/dist`, and `quarkus/container`.

## Verified Facts

* `quarkus/deployment` owns build-time augmentation: SPI/provider discovery,
  build steps, Jandex/indexing, build-time configuration, and Quarkus subsystem
  customization.
* `quarkus/runtime` owns runtime code: CLI integration, recorders, runtime
  configuration, health/bootstrap services, Arc/CDI wiring, and the JAX-RS
  application bridge.
* `quarkus/server` is the mutable Quarkus application wrapper. Its packaging
  config sets mutable-jar behavior, output under `lib`, user providers under
  `../providers`, and main class `keycloak`.
* `quarkus/dist` packages the server app, deployment artifact, client CLI,
  scripts, config, themes, providers, data, license, and cache config into ZIP
  and tar.gz distributions.
* `quarkus/container` builds a UBI micro image from a Keycloak distribution
  tarball, sets `KC_RUN_IN_CONTAINER=true`, exposes `8080`, `8443`, and `9000`,
  and uses `/opt/keycloak/bin/kc.sh` as the entrypoint.

## Entry Points

* `KeycloakProcessor.configureKeycloakSessionFactory` in
  `quarkus/deployment` for build-time session-factory configuration.
* `KeycloakProcessor.loadFactories` for build-time SPI/provider loading.
* `KeycloakRecorder.createSessionFactory` in `quarkus/runtime` for recorded
  runtime session-factory creation.
* `QuarkusKeycloakSessionFactory` for provider factory initialization and
  runtime lookup.
* `QuarkusKeycloakApplication.createSessionFactory` for retrieving the factory
  from CDI/Arc.
* `KeycloakMain.main`, `KeycloakMain.start`, and `KeycloakMain.run` for CLI and
  Quarkus application startup.
* `Picocli.start`, `Picocli.build`, and `Build.runCommand` for command routing
  and build/reaugmentation.
* `quarkus/dist/pom.xml`, `quarkus/dist/assembly.xml`, and
  `quarkus/container/Dockerfile` for packaging and image layout.

## Flows

* Build-time provider flow:
  `KeycloakProcessor.configureKeycloakSessionFactory` loads factories, checks
  providers, optionally configures persistence units and theme resources, then
  records a synthetic singleton bean with `recorder.createSessionFactory(...)`.
* Runtime session factory flow:
  `KeycloakRecorder.createSessionFactory` creates
  `QuarkusKeycloakSessionFactory`; the constructor initializes provider
  factories, sets jar theme representations, calls `factory.init(scope)`, and
  fills the factory map.
* Runtime application flow:
  `QuarkusKeycloakApplication.createSessionFactory` retrieves the
  `QuarkusKeycloakSessionFactory` from Arc/CDI.
* CLI startup flow:
  `KeycloakMain.main` delegates arguments to Picocli, `Picocli.start` delegates
  to `KeycloakMain.start`, and `KeycloakMain.start` invokes `Quarkus.run`.
* Server command flow:
  `KeycloakMain.run` gets `QuarkusKeycloakApplication`; non-server commands run
  `COMMAND.onStart(application, sessionFactory)`, while server mode waits for
  exit after startup.
* Build command flow:
  `Build.runCommand` validates config with persisted config disabled, sets
  removed artifacts, calls `picocli.build`, and `Picocli.build` invokes the
  Quarkus entry point in rebuild mode.

## Investigation Strategy

* Use `search_graph(name_pattern=".*configureKeycloakSessionFactory.*")`, then
  `get_code_snippet` before tracing provider bootstrap.
* For CLI issues, start from `KeycloakMain`, `Picocli`, and the relevant command
  class instead of `quarkus/deployment`.
* For distribution or image layout issues, inspect `quarkus/dist` and
  `quarkus/container` source directly; graph evidence is secondary for archive
  and Dockerfile behavior.
* Use `trace_path` only on exact qualified names. Common method names such as
  `format`, `scope`, and `getProfile` produced noisy traces during ingest.

## Pitfalls

* `quarkus/deployment` code does not run during ordinary runtime startup; it
  runs during build or reaugmentation. Runtime-only bugs usually belong in
  `quarkus/runtime`, `services`, model, or SPI modules unless a recorded value
  is involved.
* `quarkus/README.md` describes the right concepts but uses stale method names
  for session-factory configuration; current source/graph use
  `configureKeycloakSessionFactory` and `createSessionFactory`.
* Cypher `file_path` predicates returned no rows during ingest while
  `search_graph(file_pattern=...)` worked.

## Related Wiki Pages

* [Core Contracts](core-contracts.md)
* [Docs, CI, And Build](../docs-ci-build.md)
* [Flow Entry Points](../flows/index.md)

## Open Questions

* Whether stale symbol names in `quarkus/README.md` should be corrected
  upstream.
* Which CI jobs specifically validate container image layout versus raw
  distribution layout.
