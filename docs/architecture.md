# Architecture

This repository is an **Archive Type Registry**: a design-time schema repository for the Avro
[archive types](https://github.com/jeap-admin-ch/jeap-process-archive-service/blob/main/docs/archive-types.md)
that a [jeap-process-archive-service](https://github.com/jeap-admin-ch/jeap-process-archive-service)
(PAS) instance stores and can later read back.

The PAS supports arbitrary transport formats for archive data, but Avro is recommended (and mandatory
for programmes such as DaziT). Using Avro brings several benefits:

- Java bindings can be generated from the schemas.
- Schemas are managed in a structured, validated way.
- The PAS can validate data against the schema already when storing it, ensuring it can be read later.

This repository is a template/example instance of such a registry, showing the expected layout and
tooling. A real registry for a business system is typically a separate Git repository, maintained
directly by the business teams that own the archived data.

## Properties

- a **separate Git repository**, maintained directly by the business teams;
- **immutable once published** — a published archive type version can no longer be changed;
- **validated on every build** by the `jeap-process-archive-type-registry-maven-plugin`, and only
  successfully validated builds may be merged to the trunk branch (`main`/`master`).

## Layout

```text
schema/                         JSON schema for the descriptor (IDE validation & completion)
archive-types/
  _common/                      global Avro definitions shared across systems
  <system>/                     archive types of one business system
    _common/                    Avro definitions shared within the system
    <archive-type>/
      <archive-type>.json       descriptor (name, system, versions, ...)
      <archive-type>_v1.avdl    schema of version 1
      <archive-type>_v2.avdl    schema of version 2
```

Archive-type names are only unique within a system, so a schema is addressed by the coordinates
**(system, archive-type name, version)**. See `archive-types/jeap/processsnapshot` in this repository
for a worked example (`ProcessSnapshot`, system `jeap`, versions 1 and 2).

## Related

- [Getting started](getting-started.md)
- [Archive types (jeap-process-archive-service)](https://github.com/jeap-admin-ch/jeap-process-archive-service/blob/main/docs/archive-types.md)
- [Archive Type Registry concept (jeap-process-archive-service)](https://github.com/jeap-admin-ch/jeap-process-archive-service/blob/main/docs/archive-type-registry.md)
