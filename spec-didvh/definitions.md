## Definitions

[[def: VH-Log, Verifiable History Log, VH Log]]

~ The Verifiable History Log specification defines a general-purpose,
append-only, cryptographically chained log structure for recording the
verifiable history of a versioned [[ref: state]] object. `did:vh` is a
[[ref: specialisation]] of VH-Log. See the [VH-Log
specification](../next/index.html).

[[def: specialisation, specialisations]]

~ As defined in [VH-Log](../next/index.html#term:specialisation): a
specification that uses [[ref: VH-Log]] as its foundation. `did:vh` is a
[[ref: specialisation]] of [[ref: VH-Log]] in which the [[ref: state]] is a
W3C DID Document ([[ref: DIDDoc]]) and the DID contains only the
[[ref: SCID]].

[[def: state]]

~ As defined in [VH-Log](../next/index.html#term:state): the versioned object
recorded in each [[ref: log entry]]. In `did:vh`, the [[ref: state]] is the
[[ref: DIDDoc]] for that version of the DID.

[[def: Data Integrity]]

~ As defined in [VH-Log](../next/index.html#term:data-integrity): the W3C
specification of mechanisms for ensuring the authenticity and integrity of
structured digital documents using digital signatures and other cryptographic
proofs.

[[def: Decentralized Identifier, Decentralized Identifiers, DID, DIDs]]

~ Decentralized Identifiers (DIDs) [[spec:did-core]] are a type of identifier
that enable verifiable, decentralized digital identities. A DID refers to any
subject (e.g., a person, organization, thing, data model, abstract entity,
etc.) as determined by the controller of the DID.

[[def: DID Controller, DID Controller's, DID Controllers]]

~ The entity that controls (creates, updates, deactivates) a given DID, as
defined in [[spec:DID-CORE]].

[[def: DIDDoc]]

~ A DID Document as defined by [[spec:DID-CORE]] — the document returned when a
DID is resolved.

[[def: DID Log, DID Logs]]

~ The [[ref: VH-Log]] log for a `did:vh` DID: a list of
[[ref: DID log entries]], one added for each update of the DID.

[[def: DID Log Entry, DID Log Entries, Log Entries, Log Entry]]

~ A JSON object in a [[ref: DID Log]] that defines an authorized version of
the [[ref: DIDDoc]] and the [[ref: parameters]] in effect. The first entry
establishes the DID and version 1 of the [[ref: DIDDoc]].

[[def: DID Log source, DID Log sources]]

~ A place from which a client sources the [[ref: DID Log]] of a `did:vh` DID —
that is, gets either the log itself or a reference to it — before
passing it to a [[ref: Resolver]]: the context in which the DID was
received, a [[ref: watcher]], a peer, or a web location. See [Sourcing the
DID Log](#sourcing-the-did-log).

[[def: DID Method, DID Methods]]

~ The mechanism by which a particular type of DID and its associated DID
document are created, resolved, updated, and deactivated, defined in a DID
method specification. This document is the DID method specification for
`did:vh`.

[[def: did:key]]

~ `did:key`, as described in the [W3C CCG
specification](https://w3c-ccg.github.io/did-key-spec/), is a DID method that
derives a DID document directly from a single encoded public key, requiring no
registry or network interaction to resolve. `did:vh` uses `did:key` DIDs to
identify [[ref: witnesses]].

[[def: Entry Hash, entryHash, entry hashes]]

~ As defined in [VH-Log](../next/index.html#term:entry-hash): a hash over a
[[ref: DID log entry]] (excluding its proof) that chains the entry to its
predecessor, and is included in the entry's `versionId`.

[[def: ISO8601, ISO8601 String]]

~ As defined in [VH-Log](../next/index.html#term:iso8601): a date/time
expressed using the [ISO8601 Standard](https://en.wikipedia.org/wiki/ISO_8601).

[[def: JSON Lines, JSON Line]]

~ As defined in [VH-Log](../next/index.html#term:json-lines): lines of JSON
with whitespace removed, separated by newlines, as described at
[https://jsonlines.org/](https://jsonlines.org/).

[[def: parameters, parameter]]

~ As defined in [VH-Log](../next/index.html#term:parameters): the
configuration in each [[ref: DID log entry]] that controls how the
[[ref: DID Controller]] generates entries and how [[ref: Resolvers]] process
the [[ref: DID Log]].

[[def: Pre-Rotation, Key Pre-Rotation]]

~ As defined in [VH-Log](../next/index.html#term:pre-rotation): a technique
by which the controller of a key commits to the key it will rotate to next,
without revealing it, protecting against an attacker who learns the current
private key.

[[def: Resolver, Resolvers]]

~ As defined in [VH-Log](../next/index.html#term:resolver): a party that
obtains and verifies a [[ref: DID Log]] to produce the current or a
historical [[ref: DIDDoc]].

[[def: self-certifying identifier, self-certifying identifiers, SCID, SCIDs]]

~ As defined in [VH-Log](../next/index.html#term:self-certifying-identifier):
an identifier derived from the hash of the first [[ref: DID log entry]],
generated with the placeholder `{SCID}` wherever the SCID is to appear. A
`did:vh` DID is `did:vh:` followed by its SCID.

[[def: watcher, watchers]]

~ As defined in [VH-Log](../next/index.html#term:watcher): a party that
retrieves, verifies, archives and re-serves [[ref: DID Logs]], indexed by
[[ref: SCID]]. For `did:vh`, watchers are also a primary
[[ref: DID Log source]].

[[def: witness, witnesses, witnessed]]

~ As defined in [VH-Log](../next/index.html#term:witness): a party that
verifies a [[ref: DID log entry]] before it is distributed and, if it
approves, returns a [[ref: Data Integrity]] proof. `did:vh` witnesses are
identified by [[ref: did:key]] DIDs.
