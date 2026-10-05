## VH-Log Version Changelog

The following lists the substantive changes in each version of the specification.

- Version 0.1
  - Initial pre-draft. Extracted and generalised the log mechanism from the
    [did:webvh v1.0 specification](https://identity.foundation/didwebvh/),
    removing DID-specific content and making the following generalisations:
    - `state` is defined as an arbitrary JSON object rather than a DIDDoc.
    - The log file resource name (`vh-log.jsonl`) and witness file resource name
      are generalised; specialisations may define their own names.
    - The `method` parameter (did:webvh-specific) is replaced by `logVersion`
      (`vh-log:1.0`), which defines the VH-Log spec version and permitted
      cryptographic algorithms.
    - Witness identity and key format are specialisation-defined rather than
      requiring `did:key` DIDs.
    - The DID-to-HTTPS transformation, DID URL resolution, `/whois`, and
      parallel `did:web` publishing are removed as DID-specific concerns.
    - Portability is removed as a DID-specific concern.
    - Specialisation-defined parameters are explicitly supported.
