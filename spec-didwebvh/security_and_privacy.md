## Security Considerations

This section follows the guidelines in [[spec:RFC3552]] and extends the security
considerations defined in [VH-Log's Security Considerations](../next/index.html#security-considerations)
section, which apply to `did:webvh` unchanged unless noted otherwise below. This
section also aligns with the [[spec:DID-CORE]] requirements in
[DID Core 7.3](https://www.w3.org/TR/did-core/#security-requirements).

### Threats and Attacks

The threat classes VH-Log implementations **MUST** mitigate — eavesdropping,
replay attacks, message insertion/deletion/modification, truncation or
withholding of log entries, denial of service and amplification,
man-in-the-middle, and conflicting parallel updates / split view — are as
defined in [VH-Log's Threats and Attacks](../next/index.html#threats-and-attacks)
section, applied here to the [[ref: DID Log]], `did.jsonl`, and `did-witness.json`.

`did:webvh` additionally **MUST** mitigate:

- **Misleading prior-domain association** — A DID may be ported from a domain it never actually used, creating a false impression of association with that domain. Mitigation: resolvers and clients **MUST** ignore prior domain components when evaluating the DID, as described in [Unique Assignment of DIDs](#unique-assignment-of-dids).

The use of DNSSEC [[spec:RFC4033]], [[spec:RFC4034]], [[spec:RFC4035]] is
essential to prevent spoofing and ensure authenticity of DNS records, since
`did:webvh` (unlike VH-Log in general) locates its log via DNS.

### Residual Risks

Residual risks are as defined in [VH-Log's Residual Risks](../next/index.html#residual-risks)
section, applied here to the [[ref: DID Controller]] and [[ref: DID Log]].

### Integrity Protection and Update Authentication

As defined in [VH-Log's Integrity Protection and Update Authentication](../next/index.html#integrity-protection-and-update-authentication)
section: all DID operations (create, update, deactivate) are integrity-protected
by cryptographic verification of [[ref: DID Log]] entries, with update
authentication provided by verifying the [[ref: DID Controller]]'s proof(s)
against the valid `updateKeys`. Because a `did:webvh` DID document and its log
entries are self-certifying, they can be verified and trusted regardless of
retrieval path — directly from the host, via a cache or CDN, from a
[DID Watcher](#did-watchers), or via a trusted resolver service.

### Authentication Characteristics

As defined in [VH-Log's Authentication Characteristics](../next/index.html#authentication-characteristics)
section: DID update authentication is based on possession of the private keys
associated with the update and pre-rotation keys, and DID security depends on
the strength of these keys, their secure storage, and the cryptographic
algorithms used.

### Unique Assignment of DIDs

In `did:webvh`, uniqueness of a DID is based on the [[ref: self-certifying identifier]]
(SCID) generated at the inception of the DID, as described in
[VH-Log's Unique Assignment of Logs](../next/index.html#unique-assignment-of-logs)
section. The SCID is cryptographically bound to the DID Controller's keys and
ensures that no two independently created DIDs can have the same identifier.

The DNS portion of the DID is used solely for discovery of the [[ref: DID Log]]
and associated files; it is not used for verification of DID control, and does
not need to be owned or directly controlled by the DID Controller. For example,
a DID can be published within a namespace provided by a hosting platform (e.g.,
a GitHub repository or pages site) that serves static files over HTTPS. In such
cases, platform policies and HTTPS server authentication are relied upon for
access and integrity at the transport layer, while DID verification is provided
entirely by the SCID and verifiable history of the DID.

A `did:webvh` identifier may include a domain component that was never actually
used to host its [[ref: DID Log]], before being moved — via the
[did:webvh portability](#did-portability) capability — to a different domain
under the [[ref: DID Controller]]'s control. This creates a potential for
misleading claims of association with the original domain. To prevent this,
resolvers and clients of resolvers **MUST** ignore any prior domain components
when evaluating the history or trustworthiness of a `did:webvh` DID; only the
current hosting location and its associated verifiable history are relevant. In
addition, the [whois](#whois-linkedvp-service) DID URL capability **SHOULD** be
used to obtain attestations about the DID and [[ref: DID Controller]] from
relevant authorities.

### Endpoint Authentication

As defined in [VH-Log's Endpoint Authentication](../next/index.html#endpoint-authentication)
section: DID resource retrieval endpoints **MUST** be authenticated using TLS
server authentication, and self-signed certificates **SHOULD NOT** be used in
production.

### Resolver Transport Hardening (SSRF and Network Boundary)

The SSRF and network-boundary hardening requirements for a `did:webvh`
resolver — no automatic redirects, IP-literal rejection, case-insensitive
percent-decoding, re-validation after decoding, response size caps, operation
timeouts, HTTPS-only, and no localhost in production — are as defined in
[VH-Log's Resolver Transport Hardening](../next/index.html#resolver-transport-hardening-ssrf-and-network-boundary)
section, applied here to fetches of `did.jsonl` and `did-witness.json`.

### Network Topology

As defined in [VH-Log's Network Topology](../next/index.html#network-topology)
section: `did:webvh` relies on web infrastructure and does not require
peer-to-peer networking; implementations relying on CDN caching or load
balancers **MUST** ensure these intermediaries do not serve stale or tampered
DID data.

### Cryptographic Protection

As defined in [VH-Log's Cryptographic Protection](../next/index.html#cryptographic-protection)
section: [[ref: DID Log]] entries and witness proofs are signed and
integrity-protected but not confidential, and secret data (controller private
keys, witness private keys, random seeds) **MUST** be protected in secure
storage and never exposed in the DID Log.

### Signature Implementation

As defined in [VH-Log's Signature Implementation](../next/index.html#signature-implementation)
section: `did:webvh` uses standard [[ref: Data Integrity]] proof mechanisms for
signing DID Log entries and witness proofs, and implementations **MUST** follow
the suite's signature generation and verification requirements.

### International Domain Names

`did:webvh` implementers **MAY** publish [[ref: DID Logs]] on domains that use
international domains. The [DID-to-HTTPS Transformation](#the-did-to-https-transformation)
section of this specification **MUST** be followed by [[ref: DID Controllers]]
and DID resolvers to ensure the proper handling of international domains.

### Cross-Origin Resource Sharing (CORS) Policy Considerations

As defined in [VH-Log's CORS Policy Considerations](../next/index.html#cross-origin-resource-sharing-cors-policy-considerations)
section, applied here to the [[ref: DID Log]] file: the HTTP response **MUST**
include `Access-Control-Allow-Origin: *` to support browser-based resolution.

### Publishing parallel `did:web`

`did:webvh` implementers that consider [publishing parallel `did:web` DID](#publishing-a-parallel-didweb-did) **SHOULD** evaluate
security impact from losing added security properties of `did:webvh`
and refer to [did:web Security and Privacy Considerations](https://w3c-ccg.github.io/did-method-web/#dns-considerations) for additional guidance.

### Post Quantum Attacks

As defined in [VH-Log's Post Quantum Attacks](../next/index.html#post-quantum-attacks)
section: `did:webvh` [[ref: Key Pre-Rotation]] provides enough flexibility for
"post-quantum safety." For did:webvh-specific guidance, implementors **SHOULD**
refer to the [corresponding Implementation Guide section](https://didwebvh.info/latest/implementers-guide/prerotation-keys/#post-quantum-attacks).

### Resolver Validation Checklist (informative)

The following checklist maps normative resolver requirements to concrete
validation points, and is intended to assist conformance test suite authors and
implementers auditing their own resolver implementations. It extends
[VH-Log's Resolver Validation Checklist](../next/index.html#resolver-validation-checklist-informative)
with `did:webvh`-specific points; entries appearing here do not restate or
replace the normative requirements defined in the verification algorithm in
this specification.

**Transport, log structure, and failure modes:** as defined in VH-Log's
checklist (linked above), substituting `method` for `logVersion`.

**SCID & identity:** as defined in VH-Log's checklist (genesis self-hash; SCID
never changes), plus: every entry's `state.id` parses as a `did:webvh` DID;
every entry's `state.id` [[ref: SCID]] equals first entry's `parameters.scid`
and the SCID segment of the DID being resolved; requested DID matches at least
one entry's `state.id`; `portable: true` only in first entry.

**Keys & proofs:** as defined in VH-Log's checklist, substituting `method` for
`logVersion`.

**Witnesses:** as defined in VH-Log's checklist, plus: proofs verified with the
key recovered from the `did:key` body (no DID resolution or
`verificationMethod` lookup); `did:key` DID URL in proofs must reference the
same key material in body and fragment (multibase values byte-equal).

## Privacy Considerations

This section addresses the privacy considerations in alignment with
[[spec:RFC6973]] Section 5, extends
[VH-Log's Privacy Considerations](../next/index.html#privacy-considerations),
and aligns with the [[spec:DID-CORE]] requirements in
[DID Core 7.4](https://www.w3.org/TR/did-core/#privacy-requirements).

### Surveillance

As defined in [VH-Log's Surveillance](../next/index.html#surveillance) section,
applied here to publicly accessible `did.jsonl` endpoints. Resolution of a
`did:webvh` identifier also exposes the resolver's network activity to DNS
providers and web servers, which could be used for tracking. Controllers and
resolvers **MAY** use privacy-enhancing technologies such as VPNs, TOR, or
trusted universal resolver services, and **MAY** adopt emerging approaches such
as [Oblivious DNS over HTTPS (ODoH)](https://datatracker.ietf.org/doc/html/draft-pauly-dprive-oblivious-doh-03)
to reduce this risk.

### Stored Data Compromise

As defined in [VH-Log's Stored Data Compromise](../next/index.html#stored-data-compromise)
section: DID data is stored on web servers, and a compromise of the hosting
infrastructure could allow tampering with DID resources; HTTPS and
cryptographic signatures protect integrity, but confidentiality is not
provided.

### Implementation Hygiene (informative)

As defined in [VH-Log's Implementation Hygiene](../next/index.html#implementation-hygiene-informative)
section — dependency currency, filesystem permissions, CLI secret handling,
HTTPS certificate validation, and error message hygiene — applied here to
`did:webvh` implementations.

### Unsolicited Traffic

Publishing a DID Log does not inherently solicit inbound traffic beyond normal
DID resolution. However, public exposure of service endpoints in the DID
Document may increase unsolicited interactions. [[ref: DID Controllers]]
**SHOULD** avoid publishing unnecessary endpoints.

### Misattribution

Because the DNS portion of the DID is used for discovery, a misattribution risk
arises if that DNS name is reassigned without the associated DID resources
being updated or removed. Controllers **SHOULD** ensure DID deactivation before
relinquishing a DNS name or namespace.

Where possible, Controllers **SHOULD** use the [DID Portability](#did-portability)
mechanism defined in this specification to move the DID to a new location
under their control. When portability is used, an HTTP redirect from the old
location to the new one is the preferred approach, even in cases where DID
ownership is transferred, as it enables seamless resolution while preserving
the DID's verifiable history.

### Correlation

As defined in [VH-Log's Correlation](../next/index.html#correlation) section:
[[ref: DID Controllers]] **SHOULD** avoid embedding personal identifiers or
unnecessary service endpoints in DID documents.

### Identification

As defined in [VH-Log's Identification](../next/index.html#identification)
section: DIDs are public identifiers and can be linked to real-world
identities through their domain ownership. Entities that require anonymity
**SHOULD** consider DID methods designed for pseudonymity.

### Right to Erasure ([GDPR Art. 17](https://gdpr-info.eu/art-17-gdpr/))

As defined in [VH-Log's Right to Erasure](../next/index.html#right-to-erasure)
section: while a [[ref: DID Controller]] can delete published data as
described in [Deactivate (Revoke)](#deactivate-revoke), it is **RECOMMENDED**
for monitoring [[ref: watchers]] to cache the last known state indefinitely, so
the ability and process for complete data erasure depends on [[ref: watchers]]
behavior and **SHOULD** be defined by ecosystem governance.

### Secondary Use

As defined in [VH-Log's Secondary Use](../next/index.html#secondary-use)
section: [[ref: DID Controllers]] **SHOULD** minimize publication of DID Log
data that could be used for purposes beyond the intended use.

### Disclosure

As defined in [VH-Log's Disclosure](../next/index.html#disclosure) section: all
data in the [[ref: DID Log]] is publicly accessible, and sensitive data
**MUST NOT** be included.

### Exclusion

As defined in [VH-Log's Exclusion](../next/index.html#exclusion) section:
`did:webvh` uses DNS for discovery but does not require controller ownership
of a DNS domain — Controllers **MAY** publish under a namespace they control
on a web-hosting platform that serves static files over HTTPS (for example, a
GitHub repository or pages space), reducing barriers to participation.

Residual exclusion risks remain: access to such platforms typically requires
an account and compliance with provider terms of service; platforms might
impose geoblocking, payment requirements, or content restrictions; and
accounts can be suspended. Controllers **SHOULD** maintain the ability to
republish or mirror DID resources under alternative hosts (including using
`did:webvh` [Watchers](#did-watchers)) and **SHOULD** document a transition
plan so that participants are not locked out if a hosting provider becomes
unavailable.

### No Phone Home Mitigations

A privacy concern in decentralized identity ecosystems is the possibility of an issuer of identity information (such as Verifiable Credentials) being notified when and where individuals present those credentials. This "phone home" surveillance problem (such as described by [nophonehome.com](https://nophonehome.com)) can occur if the presentation of a credential requires contacting the issuer's infrastructure, either directly or indirectly, in a way that can be linked to the credential holder. As `did:webvh` issuer DIDs may be self-hosted, this is particularly relevant.

While this concern is generally associated with the use of verifiable credentials rather than about the resolution of DIDs, a `did:webvh` server operated by an issuer might host related resources that are retrieved at credential presentation time — for example, revocation registries or status lists. If these resources are implemented in a way that enables linking access patterns to individual credential holders, the [[ref: DID Controller]] could use that information for surveillance.

Privacy-respecting credential issuers, credential holders, and verifiers all have a role in preventing "phone home" surveillance. The following practices can help:

- **Privacy-respecting Issuers (including DID Controllers hosting VC-related resources)**
  - **SHOULD NOT** design or deploy credential-related resources (such as revocation registries) in a way that enables the identification of individual holders at presentation time.
  - Use privacy-preserving designs — such as compact status lists, batching, and/or large revocation registries that provide "lost in a crowd" anonymity — to prevent correlation of access patterns to specific credential holders.
  - Use HTTP caching headers (e.g., `Cache-Control`, `ETag`) to enable CDNs, browsers, and resolvers to cache DID resources efficiently, reducing repeated origin requests that could enable tracking and improving performance.

- **Holders**
  - Use privacy-preserving techniques such as using [DID Watchers](#did-watchers), trusted intermediaries, or privacy-enhancing network tools (e.g., TOR, VPN) to retrieve revocation status data without revealing the holder's location or identity to the issuer.
  - Separate in time interactions with the issuer (e.g., retrieving revocation status) from credential presentations to verifiers, and structure retrievals to avoid creating identifiable access patterns that could enable correlation or surveillance.

- **Verifiers**
  - Be flexible in the timeliness of credential status checks, and consider omitting them entirely when risk is low, reducing or eliminating the need for the retrieval of status data.
  - Separate the retrieval of status or issuer data from the verification process by caching issuer-provided information where possible, so holders are not required to contact the issuer in real time.
  - Use privacy-enhancing network tools (e.g., TOR, VPN) or trusted intermediary resolvers to retrieve DID resources in a way that avoids revealing verifier identity or network location to the issuer or hosting provider.
  - Where possible, support privacy-preserving resolution protocols or intermediaries offered by the hosting party.

A related risk is that an issuer may deliberately or inadvertently create **holder-specific identifiers** for data elements that are expected to be common across all holders — for example, by issuing personalized revocation list URLs or unique resource paths. This enables tracking of specific holders even if the underlying credential is otherwise privacy-preserving. Preventing this requires shared responsibility: issuers **MUST NOT** generate such holder-specific identifiers; holders and verifiers **SHOULD** reject credentials or status mechanisms that contain them; and independent third parties, including [DID Watchers](#did-watchers), **SHOULD** monitor issuer implementations to detect and report violations of this principle.
