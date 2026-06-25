# Wiki Source And Trust

## Coverage

This wiki was initialized from:

* repository root docs and build files
* Maven module declarations in root `pom.xml`
* Quarkus module documentation
* JavaScript workspace package scripts
* GitHub Actions workflows and CODEOWNERS
* codebase-memory-mcp architecture and graph searches
* RabbitMQ repository agent/wiki structure as a format reference only

RabbitMQ-specific broker details were not copied.

## Trust Model

Source behavior has highest priority. Graph evidence is preferred for discovery
and relationship finding, but it is not the final authority for runtime
behavior. Documentation and CI configuration are authoritative for documented
workflow intent, but source and actual workflow files should be checked when
behavior matters.

## Known Limits

* The initial graph index was created in fast mode, so semantic coverage may be
  less rich than a moderate or full index.
* Route nodes can include generated, test, or synthetic entries and may have
  empty `file_path` values.
* CI workflows are configuration-heavy and should be inspected directly.
* This wiki is intentionally sparse; detailed symbol maps should be regenerated
  with graph tools.
