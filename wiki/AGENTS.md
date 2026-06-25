# Wiki Maintenance Rules

The wiki is for durable agent knowledge: business logic, architecture routing,
investigation conclusions, query strategy, and repeatable pitfalls. It is not a
copy of source files, generated inventories, or CI logs.

## Page Style

Keep pages short, stable, and easy to revise. Prefer routing tables, entry
points, and evidence summaries over exhaustive lists. Link to source paths or
commands; do not paste large code blocks.

Use these evidence labels when useful:

* Verified facts: confirmed behavior that future agents may rely on
* Graph evidence: graph queries, symbols, routes, or architecture summaries used
* Source evidence: files or docs read to verify behavior
* Inference: reasoned conclusions that are not directly stated by source
* Open questions: known gaps or assumptions that still need validation

## Update Rules

Update `wiki/log.md` for meaningful wiki changes. When adding a new flow, query,
or QA page, start from the matching template in `wiki/templates/`.

Do not hand-maintain long module lists, caller/callee dumps, or generated
indexes. Use `codebase-memory-mcp` to regenerate those on demand.
