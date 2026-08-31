# AGENTS.md

Guidance for AI coding agents working **in this repository**. For how to *use* an Archive Type
Registry (defining/evolving archive types), read [README.md](README.md) and the [docs/](docs/) folder
instead.

## Project

This repository is an **Archive Type Registry** — a template/example instance of the Git-based schema
registry that a [jeap-process-archive-service](https://github.com/jeap-admin-ch/jeap-process-archive-service)
(PAS) reads archive-data schemas from. It is not application code: it contains Avro `.avdl` schemas and
JSON descriptors only, plus the Maven build that validates and publishes them.

## Repository layout

```
pom.xml                              # Single module (packaging=pom); drives validation + Java-binding publishing
archivetype.template.pom.xml         # POM template for each generated archive-type artifact
archive-types/
  _common/                           # global Avro definitions shared across systems
  <system>/                          # archive types of one business system, e.g. jeap/
    _common/                         # Avro definitions shared within the system
    <archive-type>/
      <archive-type>.json            # descriptor (name, system, versions, ...)
      <archive-type>_v<n>.avdl       # schema for version n
schema/                              # JSON schema for the descriptor (IDE validation/completion)
Jenkinsfile, publiccode.yml, LICENSE
```

There are **no CHANGELOG.md** and **no child modules** in this repository.

## Build & validate

```bash
./mvnw verify
```

- Parent: `ch.admin.bit.jeap:jeap-spring-boot-parent`. Java 21.
- The build runs the `jeap-process-archive-type-registry-maven-plugin` (`registry` goal), and can run
  the `jeap-process-archive-avro-maven-plugin` (`compile-archive-types`,
  `deploy-archive-type-artifacts`) to generate and publish Java bindings.
- Validation **must pass** before merging to `main`. It rejects edits to archive-type versions that are
  already published — never change or delete an existing `.avdl`; add a new version instead.

## Conventions (load-bearing)

- **Directory naming**: `archive-types/<system>/<archive-type>/`, archive-type names are unique only
  within a system — addressed by the coordinates `(system, archive-type name, version)`.
- **Descriptor fields**: `archiveType`, `system`, `referenceIdType`, `description`, optional
  `documentationUrl`, optional `expirationDays`, optional `encryptionKey`, and a `versions` array. Each
  version has `version`, `schema`, and optional `compatibilityMode`
  (`BACKWARD`/`FORWARD`/`FULL`/`NONE`) and `compatibleVersion`. See
  [docs/getting-started.md](docs/getting-started.md).
- **Avro convention**: the root schema of a version is a protocol whose record name equals the archive
  type name; each version uses its own namespace so generated classes for different versions do not
  collide.
- **Immutability**: an archive-type version checked into `main` is frozen. Evolve a type by adding a
  new entry to `versions` with a new `.avdl`, declaring its `compatibilityMode`.
- Reference the JSON schema under `schema/` from descriptors in your IDE for validation and completion.

## Docs

When adding or changing archive types, keep [docs/architecture.md](docs/architecture.md) and
[docs/getting-started.md](docs/getting-started.md) in sync with the actual layout/conventions used.

## Versioning

- Semantic Versioning. The project `<version>` lives directly in `pom.xml` (single module).
- On a feature branch keep the `-SNAPSHOT` suffix; CI removes it when releasing.
- This repo has **no CHANGELOG.md**. When bumping the version, also update `softwareVersion` and
  `releaseDate` in `publiccode.yml`.
- Use the JIRA ID from the branch name as the commit-message prefix (e.g. `JEAP-1234 Add ...`); do not
  use conventional commits.
