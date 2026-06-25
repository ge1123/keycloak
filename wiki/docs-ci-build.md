# Docs, CI, And Build

## Purpose

Route documentation, CI, ownership, Maven, JavaScript, and workflow questions
without treating generated config or workflow files as code graph facts.

## Scope

`docs`, `docs/guides`, `.github/workflows`, `.github/actions`,
`.github/scripts`, `.github/CODEOWNERS`, root and module `pom.xml` files,
`js/package.json`, package-level `package.json`, pnpm, and wireit config.

## Verified Facts

* The root Maven parent is the repository-level orchestrator. Default modules
  include `common`, `core`, `server-spi`, `services`, `model`, `rest`, `authz`,
  `js`, `test-framework`, `tests`, `quarkus`, `scim`, `ssf`, and `authzen`.
* Root Maven profiles add large optional surfaces such as `testsuite`,
  `adapters`, `docs`, `distribution`, and `operator`.
* The JS workspace root uses `pnpm@11.1.1` and Wireit. Root `pnpm build` fans
  out to account UI, admin UI, admin client, ui-shared, and themes-vendor.
* `js/pom.xml` bridges JS into Maven with `frontend-maven-plugin`, installing
  Node and pnpm and running pnpm install/build.
* `docs/guides` and `docs/documentation` are distinct documentation surfaces:
  guide content/generation and assembled documentation distribution/tests.
* `.github/CODEOWNERS` defaults to `@keycloak/maintainers`, with routing
  overrides for cloud-native, UI, and SRE surfaces.

## Entry Points

* `docs/building.md` and `docs/tests.md` for build/test intent.
* `docs/guides/GENERATE-DOCS.md` for guide generation.
* `docs/documentation/README.md` for assembled documentation.
* `.github/workflows/ci.yml` for broad Java/server CI.
* `.github/workflows/js-ci.yml` for JS workspace CI and E2E routing.
* `.github/workflows/operator-ci.yml` for operator local API server, Minikube,
  remote, and OLM flows.
* `.github/actions/conditional/conditions` for PR path-to-job routing.
* `.github/actions/build-keycloak/action.yml` for the shared distribution build
  action.
* `.github/scripts/find-modules-with-unit-tests.sh` for CI unit-test module
  selection.

## Flows

* General CI uses `.github/actions/conditional` to decide whether broad Java,
  Quarkus, store, SSSD, WebAuthn, Aurora, Azure, additional DB, and admin-v2
  jobs run.
* The shared build action builds the distribution license processor, then runs
  Maven install/dependency resolution with tests and examples skipped.
* CI base unit tests run:
  `./mvnw test -am -pl "$(.github/scripts/find-modules-with-unit-tests.sh)"`.
* JS CI builds a Keycloak distribution artifact first, then runs
  workspace-specific lint/build/test and Playwright E2E against `kc.sh
  start-dev`.
* Operator CI builds and uploads a distribution, runs local API server tests
  with `./mvnw install -Poperator -pl :keycloak-operator -am`, then covers
  Minikube remote and OLM paths.
* Documentation CI runs the documentation profile against documentation tests
  and distribution, with external-link checks separated.

## Investigation Strategy

* Use graph tools for code symbols and Maven/plugin symbols, but inspect
  workflow YAML, shell scripts, CODEOWNERS, package metadata, and docs as source
  text.
* Use `.github/actions/conditional/conditions` before assuming which workflow
  runs for a PR path.
* Use CODEOWNERS only as maintainer-routing evidence, not runtime behavior
  evidence.
* Prefer the Maven wrapper from the repository root for local agent commands
  unless a module-specific doc says otherwise.

## Pitfalls

* Workflow and package behavior are configuration-heavy; graph evidence is low
  value except as a pointer to Maven/plugin symbols.
* CI Java setup may default to a newer JDK than the minimum supported by docs;
  do not infer the only supported local JDK from CI defaults.
* `conditional.sh` marks all jobs changed for non-PR refs. PR behavior depends
  on the regex patterns in `.github/actions/conditional/conditions`.
* Some guide docs show `mvn`, while repo policy and CI usually use `./mvnw` from
  the repository root.

## Related Wiki Pages

* [Component Routing](components/index.md)
* [Quarkus Distribution](components/quarkus-distribution.md)
* [Source And Trust](source.md)

## Open Questions

* Whether docs/guides and docs/documentation are both active for all
  user-facing docs, or whether one is preferred for new content by current
  project policy.
* Whether routine local verification in this checkout should standardize on JDK
  17, 21, or 25.

