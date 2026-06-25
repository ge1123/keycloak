# Flow Entry Points

This page records stable starting points for cross-component flows. It is not a
full call graph; regenerate detailed paths with graph tools.

## Request And Server Lifecycle

Verified facts:

* Keycloak runs as a Quarkus application in this repository.
* `quarkus/deployment` contains build-time augmentation.
* `quarkus/runtime` contains runtime bootstrap and recorder logic.
* `quarkus/server` is the Quarkus application wrapper.
* `quarkus/dist` packages the distribution.

Graph evidence:

* Query `KeycloakProcessor KeycloakRecorder session factory provider discovery`
  finds `KeycloakProcessor.configureKeycloakSessionFactory` and
  `KeycloakRecorder.createSessionFactory`.

Source evidence:

* `quarkus/README.md` describes the deployment/runtime split and identifies
  `KeycloakMain`, `KeycloakProcessor`, and `KeycloakRecorder` as important
  bootstrap concepts.
* See [Quarkus Distribution](../components/quarkus-distribution.md) for the
  concise provider discovery, session factory, CLI startup, build command, and
  packaging flows.

## Storage And Session Flow

Start in [Persistence And Storage](../components/persistence-storage.md) for
datastore facade, JPA persistence, Infinispan session/cache, and storage manager
routing.

Verified facts:

* `DefaultDatastoreProvider` is the central datastore facade and prefers cache
  providers when present before falling back to storage managers or direct
  providers.
* `AbstractStorageManager` resolves enabled realm component storage providers,
  verifies capability interfaces, caches provider instances in the session, and
  enlists them for close.
* `InfinispanUserSessionProvider` handles online session cache behavior and
  consults `UserSessionPersisterProvider` for offline session persistence and
  recovery.

Pitfall:

* Graph traces around inherited storage-manager methods can point at sibling
  managers. Verify source before treating the trace as exact runtime flow.

## Authentication And Token Flow

Start in `services` for authentication flow implementation, token endpoints,
forms, sessions, and event emission. Use `server-spi` and `server-spi-private`
for extension contracts and internal SPI behavior.

Verified facts:

* `RealmsResource` routes `/{realm}/login-actions` to `LoginActionsService`
  after resolving the realm and creating an event builder.
* `LoginActionsService` validates session codes, resolves the browser flow, and
  delegates execution to `AuthenticationProcessor`.
* `AuthenticationManager` owns login/session cookie lifecycle and next-action
  progression after authentication.

See [Runtime Services](../components/services-runtime.md) for source-verified
login, required action, form, email, event, and account routing.

Graph evidence:

* Query `authentication flow login token endpoint` returns authentication flow
  classes in `services/src/main/java/org/keycloak/authentication` and UI flow
  components in `js/apps/admin-ui/src/authentication`.

Inference:

* UI flow editors and runtime authentication execution are separate surfaces.
  Route UI changes through `js/apps/admin-ui`; runtime behavior through
  `services` and SPI/model contracts.

## Admin API And Admin UI Flow

Start with API contracts in `rest/admin-v2/api`, implementation in
`rest/admin-v2/services`, UI-specific endpoints in `rest/admin-ui-ext`, and UI
consumers in `js/apps/admin-ui` plus `js/libs/keycloak-admin-client`.

Graph evidence:

* Query `admin REST realm clients users resources` finds `AdminApi`,
  `DefaultAdminApi`, `ClientsApi`, `DefaultClientsApi`, and the JS admin client
  `Clients` resource.

Open questions:

* For admin API behavior, verify whether the active path is legacy admin REST,
  Admin API v2, or an admin UI extension endpoint before changing code.

## Event Flow

Start in `server-spi-private` for event builder behavior, `services` for event
listeners, and `model/jpa` for event store persistence.

Graph evidence:

* Query `event listener provider event store` finds `EventBuilder`,
  `EmailEventListenerProvider`, `JBossLoggingEventListenerProvider`,
  `JpaEventStoreProvider`, and SSF transmitter listener classes.

Inference:

* Event emission can cross feature modules such as SSF and metrics. Confirm the
  provider factory and enabled listener configuration before assuming an event
  is persisted or transmitted.

## Federation And Data Sync Flow

Start in `federation/ldap` for LDAP storage providers, LDAP mappers, Kerberos
integration, and import/sync behavior. Check admin UI configuration in
`js/apps/admin-ui/src/user-federation` when UI behavior is involved.

Graph evidence:

* Query `user storage federation ldap kerberos` finds `LDAPStorageProvider`,
  `LDAPStorageProviderFactory`, `LDAPProviderKerberosConfig`, and LDAP mapper
  classes.

Open questions:

* Provider-specific sync behavior should be verified in source and tests because
  model contracts and federation implementations can diverge in important edge
  cases.
* SSSD and Ipatuura have external-system assumptions; confirm CI or local
  environment support before treating module compilation as behavior coverage.

## Job And Automation Flow

There is no single generic job framework identified during setup. Route
automation questions by surface:

* CI jobs: `.github/workflows`, `.github/actions`, `.github/scripts`
* Operator cluster jobs and scripts: `operator/scripts`
* Test utility servers: `testsuite/utils`
* Build-time generation: Maven plugins configured in root and module `pom.xml`

Open questions:

* If a future investigation identifies a durable background job or scheduler
  subsystem, add a dedicated flow page using `wiki/templates/flow.md`.
