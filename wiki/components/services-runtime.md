# Runtime Services

## Purpose

Route runtime server behavior in `services`, especially browser login,
authentication flows, required actions, authentication sessions, events, forms,
email, and account server behavior.

## Scope

`services`, authentication flows, sessions, events, forms, email, login, and
account server behavior.

## Verified Facts

* `RealmsResource` routes realm-scoped runtime resources such as
  `/{realm}/login-actions` and `/{realm}/account`.
* `LoginActionsService` owns browser login action endpoints, session-code
  checks, locale handling, authentication flow dispatch, and required-action
  processing.
* `AuthenticationProcessor` executes authentication flows selected for the
  current authentication session.
* `AuthenticationSessionManager` creates and updates root/browser
  authentication sessions, including browser auth-session cookies.
* `AuthenticationManager` owns login/session cookie lifecycle and next-action
  progression after authentication.
* `FreeMarkerLoginFormsProvider` is the bundled login form renderer.
* `FreeMarkerEmailTemplateProvider` is the bundled email template renderer and
  delegates sending to `EmailSenderProvider`.
* `AccountLoader` selects account behavior by request shape: CORS preflight,
  JSON REST API, or account-console theme resource provider.

## Entry Points

* `services/src/main/java/org/keycloak/services/resources/RealmsResource.java`
  for realm-scoped JAX-RS routing.
* `services/src/main/java/org/keycloak/services/resources/LoginActionsService.java`
  for browser login, action posts, required actions, and login flow dispatch.
* `services/src/main/java/org/keycloak/authentication/AuthenticationProcessor.java`
  for flow execution.
* `services/src/main/java/org/keycloak/services/managers/AuthenticationSessionManager.java`
  for root authentication sessions and tab cleanup.
* `services/src/main/java/org/keycloak/services/managers/AuthenticationManager.java`
  for login cookies, identity cookies, and next actions.
* `services/src/main/java/org/keycloak/forms/login/freemarker/FreeMarkerLoginFormsProvider.java`
  for theme/locale/message/template rendering.
* `services/src/main/java/org/keycloak/email/freemarker/FreeMarkerEmailTemplateProvider.java`
  for email templates and event email.
* `services/src/main/java/org/keycloak/services/resources/account/AccountLoader.java`
  for account REST versus console routing.

## Flows

* Realm routing:
  `RealmsResource` resolves the realm and creates an `EventBuilder`, then
  routes `login-actions` to `LoginActionsService` and `account` to
  `AccountLoader`.
* Login authentication:
  `LoginActionsService.authenticate` initializes `EventType.LOGIN`, validates
  `SessionCodeChecks`, processes locale, resolves the browser flow with
  `AuthenticationFlowResolver.resolveBrowserFlow(authSession)`, then wires an
  `AuthenticationProcessor` with auth session, flow path, browser-flow flag,
  flow id, connection, event builder, realm, session, URI, and request.
* Action posts:
  `POST /authenticate` routes back through the same authenticate path, so action
  posts share validation and processor dispatch.
* Required actions:
  `processRequireAction` verifies the action code, initializes login event
  context, creates the required-action provider, and handles `CANCELLED`,
  `SUCCESS`, `CHALLENGE`, and `FAILURE` before saving browser history or
  redirecting.
* Auth sessions and cookies:
  `AuthenticationSessionManager.createAuthenticationSession` creates a root
  authentication session and can set browser auth-session cookies.
  `updateAuthenticationSessionAfterSuccessfulAuthentication` removes the
  completed tab and briefly keeps remaining tabs available for automatic
  completion.
* Event emission:
  `EventBuilder.send` stamps time/id, reads enabled event types from the realm,
  and sends in the current transaction or reopens a transaction for immediate
  sends.
* Account behavior:
  `AccountLoader` returns CORS preflight for `OPTIONS`, routes JSON
  accept/content to `AccountRestService`, otherwise uses the selected account
  theme resource provider. REST API requests require bearer authentication,
  account audience validation/transformation, service-account rejection, CORS,
  and API-version resolution.

## Investigation Strategy

* For browser login: search exact symbols around `LoginActionsService`,
  `AuthenticationProcessor`, and `AuthenticationManager`.
* For forms: search `LoginFormsProvider` in `server-spi-private` and
  `FreeMarkerLoginFormsProvider` in `services`.
* For email: search `EmailTemplateProvider`, `EmailSenderProvider`, and
  `FreeMarkerEmailTemplateProvider`.
* For events: start with `EventBuilder` in `server-spi-private`, bundled
  listeners in `services`, `JpaEventStoreProvider` in `model/jpa`, and feature
  consumers such as SSF.
* Verify JAX-RS annotations and request media-type logic in source. Route graph
  nodes can have blank `file_path` values.

## Pitfalls

* `services` runtime behavior often crosses SPI contracts, model storage,
  Quarkus bootstrap, feature modules, and tests. Keep the page as a routing
  layer, not a call graph.
* Route graph output is useful as discovery but not final evidence for runtime
  endpoints; source annotations and resource locators are authoritative.
* Legacy Arquillian tests still cover many form/login paths. Check newer
  `tests/base` coverage before assuming the legacy suite is the only route.

## Related Wiki Pages

* [Core Contracts](core-contracts.md)
* [Persistence And Storage](persistence-storage.md)
* [Federation](federation.md)
* [Flow Entry Points](../flows/index.md)

## Open Questions

* Which newer `tests/base` coverage replaces legacy Arquillian form/login tests
  for each login-action path.
* Whether account console resource-provider behavior is documented elsewhere or
  should receive a dedicated flow page.

