# Investigation QA

Use this page for stable answers that are likely to recur. Keep entries concise
and evidence-backed.

## Where should I start for a server runtime issue?

Verified facts:

* Runtime services usually start in `services`.
* Quarkus bootstrapping and runtime wiring start in `quarkus/runtime`.
* Build-time augmentation starts in `quarkus/deployment`.

Source evidence:

* `quarkus/README.md`
* `docs/building.md`

## Where should I start for Admin Console behavior?

Verified facts:

* React Admin Console code lives in `js/apps/admin-ui`.
* Shared UI components live in `js/libs/ui-shared`.
* Admin client bindings live in `js/libs/keycloak-admin-client`.
* Server endpoints may live in `rest/admin-v2` or `rest/admin-ui-ext`.

Graph evidence:

* Admin API query results connect `rest/admin-v2` classes with JS admin client
  resources.

## Which tests should I run first?

Inference:

* Prefer the owning module's tests before broad suites.
* Use `./mvnw test -pl <module> -Dtest=<TestClass>` for targeted Java changes.
* Use pnpm workspace filters for JS changes.
* Use `-Poperator -pl :keycloak-operator -am` for operator changes.

Source evidence:

* `docs/tests.md`
* `.github/workflows/ci.yml`
* `.github/workflows/js-ci.yml`
* `.github/workflows/operator-ci.yml`

## Are there non-Codex agent instruction files to merge?

Verified facts:

* During setup, no `CLAUDE.md`, `.claude/`, `GEMINI.md`, `CURSOR.md`,
  `.cursor/`, `.cursorrules`, or existing `AGENTS.md` files were found within
  depth 4 of this repository.

Open questions:

* A deeper future scan can be run if new agent files are introduced outside the
  usual root/tooling locations.
