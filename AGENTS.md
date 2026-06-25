# Instructions for Codex Agents

## Role

Act as a maintainer-grade agent for this Keycloak repository. Keep changes
small, evidence-based, and consistent with the existing Java 17, Maven,
Quarkus, React/TypeScript, pnpm, Kubernetes Operator, and test-suite patterns.
Treat the repository wiki as the durable agent memory for architecture routing,
investigation results, and repeatable query strategies.

## Default Workflow

1. Read `wiki/index.md` and classify the task before broad exploration.
2. Use the smallest relevant wiki pages for routing, then use structured code
   tools to find the exact symbols, routes, and call relationships.
3. Verify behavior in source when graph evidence is ambiguous, generated, stale,
   or when runtime behavior depends on framework wiring.
4. Make narrowly scoped changes in the owning module and run the most targeted
   build, test, lint, or typecheck command that gives useful signal.
5. Update `wiki/` when the work produces reusable architecture understanding,
   query strategy, investigation pitfalls, source-vs-graph conflict notes, or a
   stable answer likely to be needed again.

## Code Discovery

Prefer `codebase-memory-mcp` and other available structured tools for code
discovery.

Use this priority order:

1. `list_projects`, then choose the project whose `root_path` matches this repo.
2. `index_status`; if the repo is not indexed or the index is stale for the task,
   run `index_repository`.
3. `search_graph` for classes, methods, functions, routes, and variables.
4. `trace_path` when available for callers, callees, and cross-component paths.
5. `get_code_snippet` after `search_graph` identifies an exact qualified name.
6. `query_graph` or `get_architecture` for broader structural questions.

Use `rg` or ordinary text search only for string literals, configuration,
Markdown, scripts, workflow files, generated files, localization bundles, or
when graph output is missing or insufficient. When graph and source disagree,
trust the source behavior and record the conflict in `wiki/` if it is reusable.

## Repository Routing

Start with these owners and entry points:

* Core domain contracts and shared models: `core`, `common`, `server-spi`, and
  `server-spi-private`
* Runtime services, authentication flows, sessions, events, forms, email, login,
  account behavior, and many REST-backed services: `services`
* Persistence and storage behavior: `model`, especially `model/jpa`,
  `model/infinispan`, `model/storage`, `model/storage-private`, and
  `model/storage-services`
* User federation: `federation/ldap`, `federation/kerberos`,
  `federation/sssd`, and `federation/ipatuura`
* Quarkus distribution, CLI bootstrap, build-time augmentation, runtime wiring,
  and packaging: `quarkus/deployment`, `quarkus/runtime`, `quarkus/server`,
  `quarkus/dist`, and `quarkus/container`
* Admin REST API v2: `rest/admin-v2/api`, `rest/admin-v2/services`, and
  `rest/admin-v2/tests`
* Admin UI extension endpoints: `rest/admin-ui-ext`
* Authorization services: `authz`
* SCIM, SSF, and AuthZEN features: `scim`, `ssf`, and `authzen`
* JavaScript UI and client packages: `js/apps/admin-ui`,
  `js/apps/account-ui`, `js/apps/keycloak-server`,
  `js/libs/keycloak-admin-client`, `js/libs/ui-shared`, and
  `js/themes-vendor`
* Themes and server-rendered UI resources: `themes` and Quarkus distribution
  theme content under `quarkus/dist/src/main/content/themes`
* Kubernetes Operator: `operator`, `operator/src`, `operator/scripts`, and
  `operator/Dockerfile`
* Tests: newer test infrastructure in `tests` and `test-framework`, legacy
  Arquillian tests in `testsuite/integration-arquillian`, Quarkus distribution
  tests in `quarkus/tests`, feature-specific tests under `scim/tests`,
  `ssf/tests`, `authzen/tests`, and `rest/admin-v2/tests`
* Documentation: `docs`, with server/operator guides under `docs/guides`
* CI, automation, and ownership: `.github/workflows`, `.github/actions`,
  `.github/scripts`, and `.github/CODEOWNERS`
* Build configuration: root `pom.xml`, module `pom.xml` files, `js/package.json`,
  package-level `package.json` files, and pnpm/wireit configuration

Use `.github/CODEOWNERS` to cross-check broad ownership when a change touches
cloud-native, UI, SRE, or testsuite areas.

## Build, Test, Lint, and Typecheck

Use the Maven wrapper from the repository root unless a module document says
otherwise.

Common Java and distribution commands:

* Full build without tests: `./mvnw clean install -DskipTests`
* Full build with tests: `./mvnw clean install`
* Server-only distribution path: `./mvnw -pl quarkus/deployment,quarkus/dist -am -DskipTests clean install`
* Quarkus dev mode: from `quarkus`, `../mvnw -f server/pom.xml compile quarkus:dev -Dkc.config.built=true -Dquarkus.args="start-dev"`
* Operator build/test slice: `./mvnw install -Poperator -pl :keycloak-operator -am`
* Targeted Maven module test: `./mvnw test -pl <module> -Dtest=<TestClass>`
* Unit-test modules used by CI: `./mvnw test -am -pl "$(.github/scripts/find-modules-with-unit-tests.sh)"`
* Formatting check: `./mvnw spotless:check`
* Formatting apply: `./mvnw spotless:apply`

Common JavaScript commands from `js`:

* Install/use the configured package manager: `pnpm install`
* Build all JS workspaces wired at the root: `pnpm build`
* Lint one workspace: `pnpm --fail-if-no-match --filter <workspace> lint`
* Build one workspace: `pnpm --fail-if-no-match --filter <workspace> build`
* Admin UI unit tests: `pnpm --fail-if-no-match --filter @keycloak/keycloak-admin-ui test`
* Admin UI integration tests: `pnpm --fail-if-no-match --filter @keycloak/keycloak-admin-ui test:integration`
* Account UI tests: `pnpm --fail-if-no-match --filter @keycloak/keycloak-account-ui test`

Do not run broad suites by default when a targeted command covers the changed
surface. Explain any important test gap when local verification is too expensive
or requires unavailable services.

## Documentation and Comments

Follow existing project style. Java source uses the repository license header
where present, Java 17 language level, and existing Keycloak naming patterns.
Do not introduce new dependencies without strong justification and project
discussion.

Add comments only for non-obvious behavior, framework lifecycle constraints, or
security-sensitive reasoning. Keep Markdown concise and link to canonical
project docs instead of duplicating them. Documentation changes should update
the relevant `docs/` area when user-facing behavior changes, and `wiki/` when
agent-facing architecture knowledge changes.

## Wiki Maintenance

The `wiki/` directory is an agent knowledge base, not a source mirror. Update it
after meaningful investigations that produce stable routing rules, reusable
graph queries, cross-component flow understanding, pitfalls, or resolved
questions. Do not paste large caller/callee lists, generated inventories, or
module dumps; regenerate those with graph tools.

Every durable wiki note should distinguish verified facts, graph evidence,
source evidence, inference, and open questions when those categories matter.
