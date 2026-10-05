## `did:vh` Version Changelog

- Version 0.1 (Pre-Draft)
  - Initial draft. Defines `did:vh` as a [[ref: specialisation]] of
    [[ref: VH-Log]], derived from `did:webvh` with the domain and path removed
    from the DID.
  - Defines resolution as the DID Resolution `resolve(did,
    resolutionOptions)` function, with `did:vh` resolution options for the two
    ways of passing the log: directly (`didLog`, `didWitness`) or by
    reference (`src`, aligned with `did:scid`, naming an `https` web
    location, a location using another supported URL scheme such as `ipns`,
    or, for a local [[ref: Resolver]], a loopback `http` location or a local
    directory), plus the version options.
  - Describes, non-normatively, how a client sources the [[ref: DID Log]]
    (the log itself or a reference to it) before resolution — known in context, from [[ref: watchers]],
    peer-to-peer exchange (with updates sent as the complete log or just the
    new entry), and from
    a web location — and how to manage freshness and duplicity.
  - Removes the `did:webvh` features that depend on a web location: the
    DID-to-HTTPS transformation, the `portable` parameter, the implicit
    `#files` and `#whois` services, and parallel `did:web` publishing.
