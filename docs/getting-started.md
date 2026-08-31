# Getting started

## Define an archive type

The descriptor defines the type name, system, reference-id type, description, an optional
documentation link, an optional encryption key reference, and the schema versions:

```json
{
  "archiveType": "Decree",
  "system": "JME",
  "referenceIdType": "ch.admin.bit.jeap.audit.type.JmeDecreeArchive",
  "description": "archive type example for a decree",
  "documentationUrl": "https://foo/bar",
  "expirationDays": 90,
  "encryptionKey": { "keyId": "my-archive-data-key" },
  "versions": [
    { "version": 1, "schema": "Decree_v1.avdl" },
    { "version": 2, "schema": "Decree_v2.avdl", "compatibilityMode": "BACKWARD" },
    { "version": 3, "schema": "Decree_v3.avdl", "compatibilityMode": "BACKWARD", "compatibleVersion": 1 }
  ]
}
```

`compatibilityMode` is one of `BACKWARD`, `FORWARD`, `FULL` or `NONE`. By convention the root schema of
a version is an Avro protocol whose record name equals the archive-type name, and each version uses its
own namespace so generated classes for different versions do not collide:

```text
@namespace("ch.admin.bit.jeap.processarchive.test.decree.v3")
protocol DecreeProtocol {
    import idl "ch.admin.bit.jeap.processarchive.test.DecreeReference.avdl";
    record Decree {
        ch.admin.bit.jeap.processarchive.test.DecreeReference decreeReference;
        timestamp_ms createdAt;
        string title;
        string payload;
    }
}
```

See `archive-types/jeap/processsnapshot/ProcessSnapshot.json` in this repository for a complete,
working example.

## Validate locally

```bash
./mvnw verify
```

The `jeap-process-archive-type-registry-maven-plugin` (`registry` goal) validates, among other things,
that:

- schemas merged to the trunk branch have not been changed;
- the Avro protocol contains a record named like the archive type;
- referenced schemas parse correctly (including imports);
- the registry layout matches the expected structure;
- schemas are named `<archive-type>_v<version>.avdl`;
- versions are compatible according to their `compatibilityMode`.

## Evolve an archive type

Add a new entry to the descriptor's `versions` array with a new `.avdl` schema file and the
compatibility mode it keeps to its predecessor. **Never** change or delete an already published schema
version — published versions are immutable so archived data can still be read in the future.

## Publish Java bindings

The `jeap-process-archive-avro-maven-plugin` generates Java bindings from the Avro schemas and can
deploy them to a Maven repository, so PAS instances and source services can consume the generated
classes as normal dependencies. It offers the `compile-archive-types` and
`deploy-archive-type-artifacts` goals, using `archivetype.template.pom.xml` as the POM template for
each generated artifact.

## Related

- [Architecture](architecture.md)
- [Archive Type Registry concept (jeap-process-archive-service)](https://github.com/jeap-admin-ch/jeap-process-archive-service/blob/main/docs/archive-type-registry.md)
- [jeap-archive-type-registry README](../README.md)
