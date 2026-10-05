## Abstract

`did:webvh` (DID Web + Verifiable History) is a [[ref: DID Method]] defined as a
[[ref: specialisation]] of the [[ref: VH-Log]] specification.
VH-Log defines a general-purpose, append-only, cryptographically chained log structure
for recording the verifiable history of a versioned [[ref: state]] object. `did:webvh`
constrains VH-Log by specifying:

- The [[ref: state]] object is a W3C DID Document ([[ref: DIDDoc]]).
- The log file is named `did.jsonl`, located via a
  [DID-to-HTTPS transformation](#the-did-to-https-transformation) derived from the
  `did:webvh` identifier.
- The DID identifier incorporates the [[ref: SCID]] and the web location, using the
  same DID-to-HTTPS transformation as [[ref: did:web]].

Through VH-Log, `did:webvh` provides the following features:

- Ongoing publishing of the full history of the DID, including all
  [[ref: DIDDoc]] versions instead of, or alongside, an existing `did:web` DIDDoc.
- The ability to resolve the full history of the DID using a verifiable chain of
  updates to the [[ref: DIDDoc]] from genesis to deactivation.
- A [[ref: self-certifying identifier]] (SCID) for the DID, globally unique and
  embedded in the DID, derived from the initial [[ref: DID log entry]]. It ensures
  the integrity of the DID's history, mitigating the risk of attackers creating a
  new object with the same identifier.
- [[ref: DIDDoc]] updates containing a proof signed by a [[ref: DID Controller]]-authorized
  key.
- An optional mechanism for publishing pre-rotation key hashes to prevent loss of
  control of a DID when an active private key is compromised.
- An optional mechanism for having collaborating [[ref: witnesses]] approve updates to
  the DID before publication.
- Support for cryptographic agility through versioned specification upgrades,
  algorithm-identifying formats (e.g., [[ref: multihash]] and [[ref: Data Integrity]]
  proofs), and per-entry `method` parameters in the [[ref: DID log]] that enable DIDs
  to evolve cryptographically over time.
- An optional mechanism for publishing the location of `did:webvh` [[ref: watchers]] in
  the [[ref: DID log]] that resolvers can use as another source of DID data for
  long-term resolution or detection of malicious [[ref: DID Controllers]].

`did:webvh` also defines, as DID-specific capabilities:

- The same DID-to-HTTPS transformation as [[ref: did:web]], enabling easy deployment
  and backward compatibility.
- An optional mechanism for enabling [[ref: DID portability]] via the [[ref: SCID]],
  allowing the DID's web location to be moved and the DID string to be updated, while
  retaining a connection to the predecessor DID(s) and preserving verifiable history.
  This parameter has no VH-Log equivalent.
- DID URL path handling that defaults (but can be overridden) to automatically
  resolving `<did>/path/to/file` by a comparable DID-to-HTTPS translation as for the
  [[ref: DIDDoc]].
- A DID URL path `<did>/whois` that defaults to automatically returning (if published
  by the [[ref: DID Controller]]) a [[ref: Verifiable Presentation]] containing
  [[ref: Verifiable Credentials]] with the DID as the `credentialSubject`, signed by
  the DID. It draws inspiration from the traditional WHOIS protocol [[spec:rfc3912]],
  offering an easy-to-use, decentralized, trust registry.

Combined, the additional features enable greater trust, security, and verifiability
without compromising the simplicity of [[ref: did:web]].

For information beyond this specification about the `did:webvh` DID method and how (and
where) it is used in practice, please visit
[https://didwebvh.info/](https://didwebvh.info/)
