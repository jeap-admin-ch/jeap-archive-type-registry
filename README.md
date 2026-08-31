# jEAP Archive Type Registry

This repository is a **template/example instance** of a jEAP Archive Type Registry: a Git-based,
design-time schema repository for the Avro
[archive types](https://github.com/jeap-admin-ch/jeap-process-archive-service/blob/main/docs/archive-types.md)
that a [jeap-process-archive-service](https://github.com/jeap-admin-ch/jeap-process-archive-service)
(PAS) instance stores and later reads back. It is not application code — it contains Avro `.avdl`
schemas and JSON descriptors plus the Maven build that validates and publishes them.

A real registry for a business system is a separate Git repository, owned by the teams that own the
archived data; this repository shows the expected layout and tooling.

* One JSON descriptor per archive type (`archiveType`, `system`, `referenceIdType`, `description`,
  optional `documentationUrl` / `expirationDays` / `encryptionKey`, and a `versions` array)
* One or more Avro `.avdl` schemas per type, one version per file
* A validation build (`jeap-process-archive-type-registry-maven-plugin`) that keeps the registry
  consistent and forbids changing an already published schema version
* Publishing of generated Java bindings via the `jeap-process-archive-avro-maven-plugin`
  (`compile-archive-types` / `deploy-archive-type-artifacts`), so PAS instances and source services
  consume the classes as ordinary Maven dependencies
* Shared Avro definitions under `_common` for records reused across systems or within a system

## Documentation

| Topic | File |
|---|---|
| Architecture (what a registry is, its properties, layout) | [docs/architecture.md](docs/architecture.md) |
| Getting started (define, validate, evolve and publish an archive type) | [docs/getting-started.md](docs/getting-started.md) |

## Repository structure

Single-module Maven project (`packaging=pom`); the POM only drives the validation and publishing
build. The content lives under:

```
schema/                              JSON schema for the descriptor (IDE validation & completion)
archive-types/
  _common/                           global Avro definitions shared across systems
  <system>/                          archive types of one business system, e.g. jeap/
    _common/                         Avro definitions shared within the system
    <archive-type>/
      <archive-type>.json            descriptor (name, system, versions, ...)
      <archive-type>_v<n>.avdl       schema of version n
archivetype.template.pom.xml         POM template for each generated archive-type artifact
```

See `archive-types/jeap/processsnapshot` for a worked example (`ProcessSnapshot`, system `jeap`,
versions 1 and 2).

## Note

This repository is part the open source distribution of jEAP. See [github.com/jeap-admin-ch/jeap](https://github.com/jeap-admin-ch/jeap)
for more information.

## License

This repository is Open Source Software licensed under the [Apache License 2.0](./LICENSE).
