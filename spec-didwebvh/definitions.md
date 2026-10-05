## Definitions

[[def: VH-Log, Verifiable History Log, VH Log]]

~ The Verifiable History Log specification defines a general-purpose, append-only,
cryptographically chained log structure for recording the verifiable history of a
versioned [[ref: state]] object. `did:webvh` is a [[ref: specialisation]] of VH-Log.
See the [VH-Log specification](../next/index.html).

[[def: specialisation, specialisations]]

~ As defined in [VH-Log](../next/index.html#term:specialisation): a specification
that uses [[ref: VH-Log]] as its foundation. `did:webvh` is a [[ref: specialisation]]
of [[ref: VH-Log]] in which the [[ref: state]] is a W3C DID Document ([[ref: DIDDoc]])
and the log is located via a DID-to-HTTPS transformation.

[[def: state]]

~ As defined in [VH-Log](../next/index.html#term:state): the versioned object
recorded in each [[ref: log entry]]. In `did:webvh`, the [[ref: state]] is the
[[ref: DIDDoc]] for the current version of the DID.

[[def: base58btc]]

~ As defined in [VH-Log](../next/index.html#term:base58btc): applies
[[spec:draft-msporny-base58-03]] to convert data to a `base58` encoding, used
for encoding hashes for [[ref: SCIDs]] and [[ref: entry hashes]].

[[def: Data Integrity]]

~ As defined in [VH-Log](../next/index.html#term:data-integrity) — see
[W3C Data Integrity](https://www.w3.org/community/reports/credentials/CG-FINAL-data-integrity-20220722/),
a specification of mechanisms for ensuring the authenticity and integrity of
structured digital documents using digital signatures and other cryptographic
proofs.

[[def: Decentralized Identifier, Decentralized Identifiers, DID, DIDs]]

~ Decentralized Identifiers (DIDs) [[spec:did-core]] are a type of identifier that enable
verifiable, decentralized digital identities. A DID refers to any subject (e.g.,
a person, organization, thing, data model, abstract entity, etc.) as determined
by the controller of the DID.

[[def: DID Controller, DID Controller's, DID Controllers]]

~ The entity that controls (create, updates, deletes) a given DID, as defined
in the [[spec:DID-CORE]].

[[def: DIDDoc]]

~ A DID Document as defined by the [[spec: DID-Core]] -- the document returned when a DID is resolved.

[[def: DID Log, DID Logs]]

~ A DID Log is a list of [[ref: Entries]], with an entry added for each update of the DID,
including new versions of the [[ref: DIDDoc]] or changed information necessary to generate or validate the DID.

[[def: DID Log Entry, DID Log Entries, Entries, Log Entries, Log Entry]]

~ A DID Log Entry is a JSON object that defines the authorized
transformation of a [[ref: DIDDoc]] from one version to the next. The initial entry
establishes the DID and version 1 of the [[ref: DIDDoc]]. All entries are stored
in the [[ref: DID Log]].

[[def: DID Method, DID Methods]]

~ DID methods are the mechanism by which a particular type of DID and its
associated DID document are created, resolved, updated, and deactivated. DID
methods are defined using separate DID method specifications. This document is
the DID Method Specification for `did:webvh`.

[[def: DID Portability, did:webvh portability, `did:webvh` portability, portability]]

~ `did:webvh` portability is the capability to change the DID string for the
DID while retaining the [[ref: SCID]] and the history of the DID. This is useful
when forced to change (such as when an organization is acquired by another,
resulting in a change of domain names) and when changing DID hosting service
providers.

[[def: DID Resources, DID Resource]]

~ A DID Resource is an object (often a file) that is referenced by a DID URL, with a path to the resource. The DID URL allows resolvers to locate and retrieve specific content associated with a DID. Examples include configuration files, schemas, credential definitions, or other structured data linked to the DID.

[[def: did:web]]

~ `did:web` as described in the [W3C specification](https://w3c-ccg.github.io/did-method-web/)
is a DID method that leverages the Domain Name System (DNS) to perform the DID operations.
It is valued for its simplicity and ease of deployment compared to [[ref: DID methods]] that are
based on distributed ledgers or blockchain technology, but also comes with increased
challenges related to trust, security and verifiability. `did:web` provides a starting point for `did:webvh`,
which complements `did:web` with specific features to address its challenges
while still providing ease of deployment.

[[def: did:key]]

~ `did:key` as described in the [W3C
specification](https://w3c-ccg.github.io/did-key-spec/) is a DID method that
derives a DID document directly from a single, encoded cryptographic public key,
requiring no registry, ledger, or network interaction to resolve. It is valued
for its simplicity and self-contained nature, making it well-suited for
ephemeral, offline, or peer-to-peer use cases. However, `did:key` DIDs are
inherently static — they cannot be updated or rotated — which limits their use
in long-lived or high-assurance contexts. `did:key` is commonly used in
`did:webvh` implementations for keys such as the [[ref: Pre-Rotation]] key and
update key, where its simplicity and cryptographic self-sufficiency are
advantageous.

[[def: Entry Hash, entryHash, entry hashes]]

~ As defined in [VH-Log](../next/index.html#term:entry-hash), applied to
`did:webvh`: a hash generated over the input data of a [[ref: DID log entry]]
(excluding the [[ref: Data Integrity]] proof), chaining each version to its
predecessor. The generated entry hash is included in the entry's `versionId`
and **MUST** be verified by a resolver.

[[def: ISO8601, ISO8601 String]]

~ As defined in [VH-Log](../next/index.html#term:iso8601) — a date/time
expressed using the [ISO8601 Standard](https://en.wikipedia.org/wiki/ISO_8601).

[[def: JSON Lines, JSON Line]]

~ As defined in [VH-Log](../next/index.html#term:json-lines): a file of JSON
Lines, as described on the site [https://jsonlines.org/](https://jsonlines.org/) —
lines of JSON with whitespace removed and separated by a newline, convenient
for handling streaming JSON data or log files.

[[def: Pre-Rotation, Key Pre-Rotation]]

~ As defined in [VH-Log](../next/index.html#term:pre-rotation): a technique for
a controller of a cryptographic key to commit to the public key it will rotate
to next, without exposing that actual public key. It protects against an
attacker that gains knowledge of the current private key from being able to
rotate to a new key known only to the attacker.

[[def: Linked-VP, Linked Verifiable Presentation]]

~ A [[spec:DID-CORE]] `service` entry that specifies where a [[ref: verifiable presentation]]
about the DID subject can be found. The [Decentralized Identity
Foundation](https://identity.foundation/) hosts the [Linked VP
Specification](https://identity.foundation/linked-vp/).

[[def: multihash]]

~ As defined in [VH-Log](../next/index.html#term:multihash): per
[[spec:multiformats]], a specification for differentiating instances of hashes
via an algorithm-and-length prefix. For interoperability, [[ref: DID Controllers]]
**MUST** only use the hash algorithms permitted by the active
`method` parameter.

[[def: parameters, parameter]]

~ As defined in [VH-Log](../next/index.html#term:parameters), applied to
`did:webvh` DID Log entries: a defined set of configurations that control how
the DID Controller generates entries and how resolvers process the DID Log.
This enables support for very long-lasting identifiers — decades.

[[def: Resolver, Resolvers]]

~ As defined in [VH-Log](../next/index.html#term:resolver): a party that
retrieves and processes a [[ref: DID Log]] to produce the current or
historical [[ref: DIDDoc]]. The resolver verifies the cryptographic chain,
[[ref: Data Integrity]] proofs, and [[ref: SCID]] of the DID Log.

[[def: self-certifying identifier, self-certifying identifiers, SCID, SCIDs]]

~ As defined in [VH-Log](../next/index.html#term:self-certifying-identifier),
applied to `did:webvh`: the SCID input is the initial [[ref: DIDDoc]] with the
placeholder `{SCID}` wherever the SCID is to be placed.

[[def: Verifiable Credential, Verifiable Credentials]]

~ A verifiable credential can represent all of the same information that a physical credential represents, adding technologies such as digital signatures, to make the credentials more tamper-evident and so more trustworthy than their physical counterparts. The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) is a W3C Standard.

[[def: Verifiable Presentation, Verifiable Presentations]]

~ A [[ref: verifiable presentation]] data model is part W3C's [Verifiable Credential Data
Model](https://www.w3.org/TR/vc-data-model/) that contains a set of [[ref:
verifiable credentials]] about a `credentialSubject`, and a signature across the
verifiable credentials generated by that subject. In this specification, the use
case of primary interest is where the DID is the `credentialSubject` and the DID
signs the [[ref: verifiable presentation]].

[[def: watcher, watchers]]

~ As defined in [VH-Log](../next/index.html#term:watcher), applied to DIDs:
watchers monitor DID Logs for changes on behalf of their clients, maintaining
a historical cache and verifying that the [[ref: DID Controller]] is
consistently following the prescribed evolution process. `did:webvh` watchers
provide endpoints for retrieving DID information and receiving [[ref:
webhooks]] notifying the watcher about updates to the DID and deletion
requests.

[[def: webhook, webhooks]]

~ As defined in [VH-Log](../next/index.html#term:webhook): an HTTP callback
mechanism (typically POST requests) for real-time, event-driven notification
between systems. Best practices are documented in [[spec:rfc8030]].

[[def: witness, witnesses, witnessed]]

~ As defined in [VH-Log](../next/index.html#term:witness), applied to
`did:webvh`: a witness receives from the [[ref: DID Controller]] a [[ref: DID Log]]
entry ready for publication, verifies it, and — per ecosystem governance —
returns a [[ref: Data Integrity]] proof attesting to approval. The witness
identity format for `did:webvh` is `did:key` (see [Witness DIDs and
Reputation](#witness-dids-and-reputation)).

[[def: threshold, witness threshold]]

~ As defined in [VH-Log](../next/index.html#term:threshold): an algorithm that
defines when a sufficient number of [[ref: witnesses]] have submitted valid
[[ref: Data Integrity]] proofs for a [[ref: DID Log entry]] to be approved and
published. Details are in the [Witness Threshold
Algorithm](#witness-threshold-algorithm) section.

[[def: W3C VCDM]]

~ A Verifiable Credential that uses the Data Model defined by the W3C [[spec:vc-data-model]] specification.
