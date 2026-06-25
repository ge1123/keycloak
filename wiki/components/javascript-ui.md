# JavaScript UI And Themes

## Purpose

Route Admin UI, Account UI, JS admin client, shared UI, theme vendor, and theme
packaging investigations while separating generated API artifacts from source
UI behavior.

## Scope

`js/apps/admin-ui`, `js/apps/account-ui`, `js/apps/keycloak-server`,
`js/libs/keycloak-admin-client`, `js/libs/ui-shared`, `js/themes-vendor`,
`themes`, and `quarkus/dist/src/main/content/themes`.

## Verified Facts

* CODEOWNERS routes `/js/`, `/themes/`, JS message bundles, and
  `/rest/admin-ui-ext/` to UI maintainers and maintainers.
* `js/pom.xml` is the Maven parent for account UI, admin UI, admin client,
  ui-shared, and themes-vendor.
* `js/package.json` uses `pnpm@11.1.1`; root `pnpm build` delegates through
  Wireit to account UI, admin UI, admin client, ui-shared, and themes-vendor.
* Maven bridges JS into the Java build with `frontend-maven-plugin`, installing
  Node and pnpm, running frozen pnpm install, then `pnpm build`.
* Admin UI creates a hash router rooted at `/`.
* Account UI route availability is partly server/content driven through
  `content.json` and environment feature flags.
* `themes` packages built-in themes and depends on admin UI, account UI, and
  themes-vendor artifacts.

## Entry Points

* `js/apps/admin-ui/src/main.tsx` creates the Admin UI router.
* `js/apps/admin-ui/src/routes.tsx` composes feature route arrays and requires
  route handle access metadata.
* `js/apps/admin-ui/src/App.tsx` wraps routes in admin client, server info,
  realm, whoami, recent realms, access, and subgroup contexts.
* Admin feature navigation usually starts at the owning `routes.ts` or
  `routes/*.tsx` file before rendered components.
* `js/apps/account-ui/src/root/Root.tsx` fetches account content, maps menu
  items to route elements, filters disabled items by environment features, and
  creates a browser router rooted at `environment.baseUrl`.
* `js/libs/keycloak-admin-client/src/client.ts` wires admin resource classes.
* `js/libs/ui-shared/src/main.ts` exports shared alerts, environment context,
  form controls, table controls, masthead, user-profile fields, storage hooks,
  error helpers, and error boundary utilities.
* `themes/src/main/resources/META-INF/keycloak-themes.json` declares built-in
  theme names and types.
* `quarkus/dist/src/main/content/themes/README.md` documents the mutable
  distribution theme directory for custom themes.

## Flows

* Admin UI routing:
  `main.tsx` creates the router, `routes.tsx` composes feature routes, and
  `App.tsx` installs contexts needed by route elements. For navigation changes,
  start at the route table and owning feature route file.
* Account UI routing:
  `Root.tsx` reads server-provided content and environment flags before mapping
  routes, so a local component route may not be reachable in a given server
  configuration.
* UI/server environment:
  `ui-shared` reads injected JSON from `<script id="environment">`.
  `KeycloakProvider` initializes `keycloak-js` with server base URL, realm,
  client id, `login-required`, PKCE S256, and query response mode.
* Admin client:
  Admin UI creates `KeycloakAdminClient`, sets realm/base URL from injected
  environment, and registers a token provider backed by `keycloak.updateToken`.
* Account API:
  account UI requests go to
  `{serverBaseUrl}/realms/{realm}/account/...` with bearer tokens from
  `keycloak-js`.
* Admin client generation:
  `js/libs/keycloak-admin-client` builds by running Kiota generation from
  `openapi.json`, generating doc examples, then running TypeScript compilation.
* Theme packaging:
  `themes` packages built-in themes and verifies them. `js/themes-vendor`
  bundles React, React DOM, and a web-crypto shim into theme vendor resources.
  The Quarkus distribution theme directory is for mutable custom themes; built
  in templates come from the packaged themes JAR.

## Investigation Strategy

Recommended graph queries:

```text
search_graph(query="admin ui routes realm clients users")
search_graph(query="account ui routes applications account security signing")
search_graph(query="keycloak admin client resources users clients realms")
search_graph(query="KeycloakProvider getInjectedEnvironment useEnvironment KeycloakContext")
trace_path(function_name="initAdminClient", direction="inbound", mode="calls")
```

Use text/source reads for `package.json`, `pom.xml`, `content.json`,
`messages_*.properties`, `theme.properties`, generated OpenAPI artifacts, and
FreeMarker/theme files.

## Pitfalls

* Maven build order depends on `keycloak-admin-v2-services` so OpenAPI is
  generated before JS build. Admin API v2 OpenAPI changes are not isolated JS
  changes.
* Account UI routes are content and feature-flag driven; verify
  `public/content.json`, server-provided content, and
  `AccountEnvironment.features`.
* Generated `src/generated`, OpenAPI, and Kiota paths should be treated as
  generated/API-contract surfaces.
* Built-in themes should be traced through the packaged `keycloak-themes` JAR;
  do not confuse them with mutable distribution custom-theme content.

## Related Wiki Pages

* [Admin REST](admin-rest.md)
* [Runtime Services](services-runtime.md)
* [Docs, CI, And Build](../docs-ci-build.md)

## Open Questions

* Whether `quarkus/dist/src/main/content/themes` receives generated or assembled
  theme content elsewhere in the distribution build.

