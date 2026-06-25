# Keycloak Agent Wiki

This wiki is the persistent routing and investigation layer for Codex agents
working on this Keycloak repository. It should help an agent decide where to
start, which graph queries to run, and what prior conclusions are stable enough
to reuse.

## Read Order

1. `AGENTS.md` at the repository root for workflow and command policy.
2. `wiki/ingest-plan.md` when coordinating multi-batch wiki ingest.
3. `wiki/components/index.md` to route by feature, module, service, or test
   surface.
4. `wiki/queries/index.md` for reusable graph and search strategies.
5. `wiki/flows/index.md` for cross-component runtime flows.
6. `wiki/qa/index.md` for answered investigation questions and known pitfalls.
7. `wiki/source.md` to understand evidence quality and current coverage.

## When To Update

Update this wiki when an investigation produces:

* reusable architecture routing
* a graph query or search strategy worth repeating
* a stable explanation of a cross-component flow
* a source-vs-graph conflict and the corrected understanding
* a pitfall that would save future debugging time
* a concise answer to a question likely to recur

Do not update it for one-off implementation notes, temporary failing test logs,
large generated inventories, or caller/callee lists that graph tools can
recreate.

## Evidence Contract

Prefer concise sections named:

* Verified facts
* Graph evidence
* Source evidence
* Inference
* Open questions

If source and graph disagree, source wins. Record the conflict only when the
corrected understanding is reusable.
