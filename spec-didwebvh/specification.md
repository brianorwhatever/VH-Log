## `did:webvh` DID Method Specification

### Conformance

As well as sections marked as non-normative, all examples, notes, and
informative checklists in this specification are non-normative. Everything
else in this specification is normative.

The key words **MAY**, **MUST**, **MUST NOT**, **NOT REQUIRED**,
**RECOMMENDED**, **REQUIRED**, **SHOULD**, and **SHOULD NOT** in this
specification are to be interpreted as described in BCP 14 [[spec:RFC2119]]
[[spec:RFC8174]] when, and only when, they appear in all capitals, as shown
here.

### Relationship to VH-Log

`did:webvh` is a [[ref: specialisation]] of the [[ref: VH-Log]] specification.
VH-Log defines the general-purpose log mechanism underlying
`did:webvh` — including the log entry structure, cryptographic chaining, [[ref: SCID]],
[[ref: parameters]], [[ref: witnesses]], [[ref: watchers]], and resolution algorithm.
`did:webvh` constrains VH-Log by:

- Specifying the [[ref: state]] object as a W3C DID Document ([[ref: DIDDoc]]).
- Naming the log file `did.jsonl` and the witness proofs file `did-witness.json`.
- Locating the log via the [DID-to-HTTPS Transformation](#the-did-to-https-transformation).
- Using `method` (e.g., `did:webvh:1.0`) as the did:webvh-specific version of the
  VH-Log `logVersion` parameter — each `method` value corresponds to a VH-Log version
  and defines the same cryptographic algorithm set as the corresponding `logVersion`.
- Adding the `portable` [[ref: parameter]] to enable optional [[ref: DID portability]].
- Defining the DID method identifier, CRUD operations, DID URL resolution, and DID URL
  path handling.

Sections in this specification that implement VH-Log mechanisms are annotated with
a reference to the corresponding section in [[ref: VH-Log]]. Implementers **SHOULD**
consult [[ref: VH-Log]] for the normative definition of the log mechanism; this
specification defines only the did:webvh-specific constraints and extensions.

### Target System

The target system of the `did:webvh` DID method is the host (or domain)
name when the domain specified by the DID is resolved through the Domain Name
System (DNS) and verified by processing a log of DID versions.

### Method Name

The namestring that identifies this DID method is: `webvh`. A DID that uses this
method MUST begin with the following prefix: `did:webvh`. Per the DID
specification, this string MUST be in lowercase. The remainder of the DID, after
the prefix, is the [method-specific identifier](#method-specific-identifier),
specified below.

### Method-Specific Identifier

The `did:webvh` method-specific identifier contains both the [[ref:
self-certifying identifier]] (SCID) for the DID, and a fully qualified domain
name (with an optional path) that is secured by a TLS/SSL certificate. Given the
DID, a [transformation to an HTTPS URL](#the-did-to-https-transformation) is
performed such that the [[ref: DID Log]] for the `did:webvh` DID can be retrieved (via
an `HTTP GET`) and processed to produce the [[ref: DIDDoc]] for the DID. As per
the Augmented Backus-Naur Form (ABNF) notation below, the [[ref: SCID]] **MUST**
be the first element of the method-specific identifier.
  
Formal rules describing valid domain name syntax are described in
[[spec:RFC1035]], [[spec:RFC1123]], and [[spec:RFC2181]]. Each `did:webvh` DID's
globally unique [[ref: SCID]] **MUST** be
[generated](#scid-generation-and-verification) during the creation of the DID
based on its initial content and placed into the DID identifier for publication
and use.

The domain name element of the method-specific identifier MUST match the name
found in the SSL/TLS certificate per [[spec:RFC6125]] and the its replacement
[[spec:RFC9525]], and it MUST NOT include IP addresses. A port MAY be included
and the colon MUST be percent encoded to prevent a conflict with paths.
Directories and subdirectories MAY optionally be included, delimited by colons
rather than slashes.

As specified in the following Augmented Backus-Naur Form (ABNF) notation
[[spec:rfc2234]] the [[ref: SCID]] **MUST** be present in the DID string. See
examples below. The `domain-segment` and `path-segment` elements refer to
[[spec:rfc3986]]'s ABNF for a Generic URL (page 49). Attempting to replicate
here the full ABNF of those elements from that RFC would inevitably be wrong.

```abnf
webvh-did = "did:webvh:" scid ":" domain-segment 1+( "." domain-segment ) [ percent-encoded-port ] *( ":" path-segment )
scid = 46(base58-alphabet) ; The characters in the base58-btc-alphabet are as defined in the referenced W3C "Controller Documents" specification 
domain-segment = ; A part of a domain name as defined in RFC3986, such as "example" and "com" in "example.com"
percent-encoded-port = "%3A" ( "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ) 1*4( DIGIT )
path-segment= ; A part of a URL path as defined in RFC3986, such as "path", "to", "folder" in "path/to/folder"
```

The ABNF for a `did:webvh` is almost identical to that of `did:web`, with changes
only to the DID Method (`webvh` instead of `web`), and the addition of the
`<scid>:` (defined in the [SCID](#scid-generation-and-verification)) section of
this specification) element in `did:webvh` that is not in `did:web`. As specified
in the [DID-to-HTTPS Transformation](#the-did-to-https-transformation) section
of this specification, `did:webvh` and `did:web` DIDs that have the same fully
qualified domain and path transform to the same HTTPS URL, with the exception of
the final file -- `did.json` for `did:web` and `did.jsonl` for `did:webvh`. For
`did:webvh` DIDs using [[ref: witnesses]], another file `did-witness.json` is also
found (logically) beside the `did.jsonl` web server file. See the
[witnesses](#did-witnesses) section of this specification for details.

### The DID to HTTPS Transformation

The `did:webvh` [method-specific identifier](#method-specific-identifier) is
defined to enable a transformation of the DID to an HTTPS URL for publishing
and retrieving the [[ref: DID Log]]. This section defines the transformation
from DID to HTTPS URL, including a number of examples.

Given a `did:webvh`, the HTTPS URL for the [[ref: DID Log]] is generated by
carrying out the following steps. The steps are carried out by the [[ref: DID Controller]]
to determine where to publish the [[ref: DID Log]], and by all resolvers to
retrieve the [[ref: DID Log]]. The process described here includes the appropriate handling of [international domain names](#international-domain-names).

 1. **Remove the 'did:webvh:' prefix** from the input identifier.
 2. **Remove the SCID segment**, which is the first segment after the prefix.
 3. **Transform the domain segment**, the first segment (up to the first `:` character) of the remaining string.
    - If the domain segment contains a port, decode percent-encoding and preserve the port.
    - Apply Unicode normalisation as defined in [[spec:rfc3491]] (see this explainer on [Unicode normalization](https://dencode.com/en/string/unicode-normalization)).
    - Apply IDNA (Punycode) encoding as per IDNA2008 [[spec:rfc9233]]. See the [FAQ on IDNA](https://corp.unicode.org/~asmus/proposed_faq/idn.html) for more details. For domains that do not contain international domain name elements, this should result in no change.
 4. **Transform the path**, the 0 or more segments after the first `:` character, delimited by `:` characters.
    - For each segment, validate that **after percent-decoding** it: is non-empty; is not `.` or `..`; does not contain `/`, `\`, or NUL; does not begin or end with whitespace. A segment that fails any check **MUST** cause the transformation (and resolution) to fail. The checks **MUST** be applied to the decoded form because percent-encoded variants (`%2E%2E`, `%2f`, `%5c`, `%00`) are otherwise opaque.
    - Percent-encode each (now-validated) segment per [[spec:rfc3986]] using uppercase hex digits.
    - Replace each `:` separator with `/` to create the encoded path.
 5. **Reconstruct the HTTPS URL**:
    - Format as `https://{domain}:{port}/{encoded_path}/did.jsonl` if a port is present.
    - Format as `https://{domain}/{encoded_path}/did.jsonl` if there are path segments and no port.
    - If no path segments exist, format as `https://{domain}:{port}/.well-known/did.jsonl` or `https://{domain}/.well-known/did.jsonl` as applicable.
 6. The content type for the `did.jsonl` file **SHOULD** be `text/jsonl`.

 If the DID is using [[ref: witnesses]], an extra JSON file containing the witness proofs for the [[ref: DID Log Entries]] must be published and retrieved during resolution. The URL for the extra file is defined by replacing the `/did.jsonl` at the end of the [[ref: DID Log]] URL with `/did-witness.json`.

 When this algorithm is used for resolving a DID path (such as `<did>/whois` or `<did>/path/to/file` as defined in the section [DID URL Handling](#did-url-resolution)) using the implicit `services`, update step **5.** to not include the `.well_known/` path segment, and to append the DID URL path instead of `did.jsonl`.

The following are some examples of various DID-to-HTTPS transformations based
on the processing steps specified above.

::: example

`did:webvh` DIDs and the corresponding web locations of their `did:webvh` log file.
In the examples,`{SCID}` is a placeholder for where the generated [[ref: SCID]] will be
placed in the actual DIDs and HTTPS URLs. Note that when the `{SCID}` follows
the literal `did:webvh:` as a separate element, the `{SCID}` is not part of the
HTTPS URL.

---

domain/`did:web`-compatible

`did:webvh:{SCID}:example.com` -->

`https://example.com/.well-known/did.jsonl`

subdomain

`did:webvh:{SCID}:issuer.example.com` -->

`https://issuer.example.com/.well-known/did.jsonl`

path

`did:webvh:{SCID}:example.com:dids:issuer` -->

`https://example.com/dids/issuer/did.jsonl`

path w/ port

`did:webvh:{SCID}:example.com%3A3000:dids:issuer` -->

`https://example.com:3000/dids/issuer/did.jsonl`

internationalized domain

 `did:webvh:{SCID}:jp納豆.例.jp:用户` -->

 `https://xn--jp-cd2fp15c.xn--fsq.jp/%E7%94%A8%E6%88%B7/did.jsonl`

:::

A client resolving a `did:webvh` DID **MAY** choose to use a [[ref: watcher]] as the source of data about a `did:webvh` DID, rather than resolving the DID's HTTPS location to retrieve the [[ref: DID Log]]. See the specification section on [Watchers](#did-watchers) for information about `did:webvh` and [[ref: watchers]].

The location of the `did:webvh` `did.jsonl` [[ref:DID Log]] file is the same as
where the comparable `did:web`'s `did.json` file is published. A [[ref: DID Controller]]
**MAY** publish both DIDs and so, both files. The process
to do so is described in the [publishing a parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification.

::: warning

While the transformation from a did:webvh identifier to an HTTPS resource relies on DNS resolution, clients should not assume  that a `did:webvh` identifier is inherently bound to or controlled by the entity associated with the corresponding DNS domain. In fact, a `did:webvh` [[ref: DID Log]] may be obtained from sources other than its corresponding HTTPS location (perhaps indexed by its [[ref: SCID]]), and in such cases, the same verification steps may be applied to determine its validity.

Verification of a did:webvh identifier using this specification ensures cryptographic validity, but that does not imbue "trust" in the identifier itself. Trust in a did:webvh DID should be derived from external sources, such as verifiable credentials issued by trusted parties (possibly discovered by resolving the DID's [/whois](#whois-linkedvp-service) URL) or via Trust Registries that maintain authoritative records of trusted DIDs in a given context. Implementers should exercise caution and avoid conflating technical verification with trustworthiness, ensuring that reliance on a `did:webvh` identifier is informed by independent verification mechanisms.

:::

### The DID Log File

The `did:webvh` DID Log file is an instance of a [[ref: VH-Log]] log file as defined in the
[[ref: VH-Log]] specification. The log entry structure, [[ref: JSON Lines]] serialisation,
cryptographic chaining, and verification algorithm are all inherited from [[ref: VH-Log]].

The did:webvh-specific constraints on the VH-Log format are:

- The log file is named `did.jsonl`.
- The `state` property of each entry **MUST** contain the [[ref: DIDDoc]] for this version of the DID.
- The `parameters` property follows the [`did:webvh` DID Method Parameters](#didwebvh-did-method-parameters)
  defined in this specification.

The [[spec: json-schema-core]] definition of the [[ref: DID log entry]] structure is in the [log_entry.json](https://raw.githubusercontent.com/decentralized-identity/didwebvh/refs/heads/main/schemas/v1.0/log_entry.json) file in this repository.

Examples of [[ref: DID Logs]] and [[ref: DID log entries]] can be found in the [Examples] section on the `did:webvh` information website.

[Examples]: https://didwebvh.info/latest/example/

::: example

**did:webvh-specific examples of log entries that fail verification** (in addition to the general examples in [[ref: VH-Log]]). Illustrative and non-exhaustive; normative rejection criteria are defined in the verification algorithm in this specification.

1. **Wrong cryptosuite on log-entry proof.** `{"type":"DataIntegrityProof","cryptosuite":"ecdsa-jcs-2019","proofPurpose":"assertionMethod"}` — rejected even if the signature is structurally valid, because `parameters.method` with value `did:webvh:1.0` requires `eddsa-jcs-2022`.

2. **`state.id` SCID in an entry does not match [[ref: SCID]] in DID and in `parameters.scid`.** DID `did:webvh:Qm111...` [[ref: DID Log]] first entry has `parameters.scid: "Qm111..."` but in any entry has `state.id: "did:webvh:Qm222...:example.com"` — the [[ref: SCID]] values are inconsistent.

3. **SCID changes under portability.** Entry N has `state.id: "did:webvh:Qm222...:new.example.com"` while entry N-1 carried `Qm111...` — the [[ref: SCID]] is immutable across all entries.

4. **Unknown `method` value.** First entry has `parameters.method: "did:webvh:99.0"`, `"did:webvh:1.0-rc1"`, or `"didwebvh:1.0"` — unrecognised values are rejected and never silently downgraded.

:::

### DID Method Operations

#### Create (Register)

Creating a `did:webvh` DID follows the Create algorithm defined in [[ref: VH-Log]], with the following did:webvh-specific constraints applied at each stage:

1. **Define the DID string** (did:webvh-specific, before generating the log entry)

   The DID **MUST** begin with the literal string "`did:webvh:{SCID}:`", where `{SCID}` is a placeholder for the [[ref: SCID]] calculated later. This is followed by a fully qualified domain name (with an optional path) secured by a TLS/SSL certificate, identifying the web location where the [[ref: DID Log]] (`did.jsonl`) will be published.

   The DID **MUST** be valid per the ABNF in the [Method-Specific Identifier](#method-specific-identifier) section.

   1. The [[ref: SCID]] is not by default part of the HTTPS URL. A [[ref: DID Controller]] **MAY** include the [[ref: SCID]] in the URL by inserting additional `{SCID}` placeholders in the domain or path components of the method-specific identifier. Such additional instances do not alter the [DID-to-HTTPS Transformation](#the-did-to-https-transformation).

2. **The initial [[ref: state]] object MUST be a [[ref: DIDDoc]]** (did:webvh specialisation of VH-Log Create step 3)

   The [[ref: DIDDoc]] **MUST** contain a top-level `id` property set to the DID string from step 1, using the `{SCID}` placeholder wherever the [[ref: SCID]] will appear. All other absolute references to the DID within the [[ref: DIDDoc]] **MUST** likewise use the `{SCID}` placeholder form (e.g., `did:webvh:{SCID}:example.com#key-1`). The [[ref: DIDDoc]] **MAY** contain any other content the [[ref: DID Controller]] requires.

3. **The log file is named `did.jsonl`** (did:webvh specialisation of VH-Log Create step 7)

   The log file **MUST** be published at the HTTPS location determined by the [DID-to-HTTPS Transformation](#the-did-to-https-transformation) from the DID string.

4. **The witness file is named `did-witness.json`** (did:webvh specialisation of VH-Log witness file)

   If [[ref: witnesses]] are used, the witness proofs **MUST** be collected and published in `did-witness.json` (at the same HTTPS location as `did.jsonl`, with `/did.jsonl` replaced by `/did-witness.json`) **BEFORE** the [[ref: DID Log]] is published. See [DID Witnesses](#did-witnesses).

A controller **MAY** generate an equivalent `did:web` [[ref: DIDDoc]] and publish it as defined in the [Publishing a Parallel `did:web` DID](#publishing-a-parallel-didweb-did) section. The `did:web` [[ref: DIDDoc]] can be used for backwards compatibility during a transition from `did:web` to `did:webvh`; verifiers using `did:web` lose the verifiable properties and history of `did:webvh`.

#### Read (Resolve)

Resolving a `did:webvh` DID follows the Read (Resolve) algorithm defined in [[ref: VH-Log]], with the following did:webvh-specific retrieval steps prepended and specialisations applied within the algorithm.

**Retrieval (before the VH-Log algorithm):**

1. Use the [DID-to-HTTPS Transformation](#the-did-to-https-transformation) to derive the HTTPS URL of the [[ref: DID Log]] file.
2. Perform an HTTPS `GET` request to the URL meeting the requirements of VH-Log's [Retrieving Log Resources](../next/index.html#retrieving-log-resources) section.
3. When performing DNS resolution during the HTTPS `GET`, the client **SHOULD** use [[spec:rfc8484]] to prevent tracking of the identity being resolved.
4. Apply the VH-Log Read algorithm to the retrieved file, incorporating the did:webvh-specific specialisations below.

**did:webvh-specific specialisations within the VH-Log algorithm:**

- **Parameters (VH-Log step 1):** `parameters` **MUST** adhere to [`did:webvh` DID Method Parameters](#didwebvh-did-method-parameters).
- **Witness verification (VH-Log step 2.1):** If witnesses are active, resolvers **MUST** retrieve and verify the `did-witness.json` file. See [DID Witnesses](#did-witnesses).
- **State verification (VH-Log step 6):** Initialise a counter `didIdMatchCount` to `0` before processing the first entry. For each entry, after retrieving the `state` (DIDDoc):
  1. Parse `state.id` as a `did:webvh` DID per the [Method-Specific Identifier](#method-specific-identifier) ABNF; if parsing fails, resolution **MUST** terminate.
  2. The SCID segment of `state.id` **MUST** be byte-for-byte identical to `parameters.scid` from the first entry **and** to the SCID segment of the DID being resolved. This check **MUST** apply to **every** entry. A mismatch **MUST** terminate resolution.
  3. If `state.id` matches the DID being resolved exactly, increment `didIdMatchCount` by 1.

**Post-loop check:**

After processing all entries, `didIdMatchCount` **MUST** be greater than `0`. If it is `0`, the log **MUST** be rejected — no entry’s `state.id` matched the DID being resolved.

[[ref: Resolvers]] **SHOULD NOT** cache a log that fails verification. This ensures the [[ref: DID Controller]] has the opportunity to recover a DID that may have been erroneously or maliciously invalidated.

A [[ref: Resolver]] **MAY** use a [[ref: watcher]] in addition to, or in place of, retrieving the log from the source. See [Watchers](#did-watchers).

As defined in the [[spec:DID-RESOLUTION]] specification, a did:webvh resolver should return the following DID Document Metadata when resolving a `did:webvh` DID Document:

```json
{
  "versionId": "1-QmRRaLXwc6BjBuBPosSupJwEQ8w9f3znP7yfbpGfwcnLr6",
  "versionTime": "2025-01-23T04:12:36Z",
  "created": "2025-01-23T04:12:36Z",
  "updated": "2025-01-23T04:12:36Z",
  "scid": "QmPEQVM1JPTyrvEgBcDXwjK4TeyLGSX1PxjgyeAisdWM1p",
  "portable": false,
  "deactivated": false,
  "ttl": "3600",
  "witness": { ...
  },
  "watchers: [ ...
  ]
}
```

where the items in the Metadata JSON object are:

- `versionId` — The `versionId` from the [[ref: Log Entry]] of the resolved DIDDoc version.
- `versionTime` — The `versionTime` from the [[ref: Log Entry]] of the resolved DIDDoc version, in [[ref: ISO8601]] timestamp format.
- `created` — The [[ref: ISO8601]] timestamp of the DID's first [[ref: log entry]], indicating when (according to the [[ref: DID Controller]]) the DID was created.
- `updated` — The [[ref: ISO8601]] timestamp of the DID's last valid [[ref: log entry]].
- `scid` — The [[ref: SCID]]] of the resolved DID.
- `portable` — A boolean value indicating whether the resolved DID has [[ref: portability]] active and so may be moved in the future, as defined in the [portability](#did-portability) section of this specification.
- `deactivated` — A boolean indicating whether the DID has been deactivated. When `true`, the DID is no longer active.
- `ttl` - A string containing the unsigned integer value of the DID's `ttl` [[ref; parameter]] (time-to-live) in seconds. The TTL is guidance from the [[ref: DID Controller]] for those resolving the DID about how long to cache the DID. The value is a string containing the integer value because the [[spec: DID-RESOLUTION]] specification requires that DID metadata not be integers. The value needs to be converted to an integer by the resolver client before use.
- `witness` — An object containing the current (active in the last valid [[ref: DID log entry]]) configuration of witnesses for the DID. The object is as defined as the same named object in the [witness list](#witness-lists) section of this specification.
  - The value of the `threshold` attribute of the `witness` object is a string containing the integer value of `threshold` because the [[spec: DID-RESOLUTION]] specification requires that DID metadata not include integers. The value needs to be converted to an integer by the resolver client before use.
- `watchers` — An array containing the current (active in the last valid [[ref: DID log entry]]) list of [[ref: watcher]] URLs that have agreed to monitor and cache the DID’s state.

The "last valid [[ref: log entry]]" for some of the items above references the case where a DID resolution request references a DIDDoc that was valid, but where the DID Log has later [[ref: log entries]] that fail verification, as noted in the DID resolution steps earlier in this section. If all [[ref: log entries]] pass verification, the last valid [[ref: log entry]] is the last [[ref: log entry]].

When a DID resolution error occurs, the `error` field **MUST** be included in the `didResolutionMetadata`, as defined by the [[spec:DID-RESOLUTION]] specification. In addition, resolvers **SHOULD** provide supplemental "Problem Details" metadata following [[spec:rfc9457]], using the following structure:

```json
"didResolutionMetadata": {
  "error": "invalidDid",
  "problemDetails": {
    "type": "https://w3id.org/security#INVALID_CONTROLLED_IDENTIFIER_DOCUMENT_ID",
    "title": "The resolved DID is invalid.",
    "detail": "Parse error of the resolved DID at character 3, expected ':'."
  }
}
```

As described in [[spec:DID-EXTENSION-RESOLUTION]], the following values **MUST** be used in the `error` field of the resolution metadata when resolving a `did:webvh` DID under the corresponding error conditions:

- `notFound` — The [[ref: DID Log]] or the resource referenced by a DID URL was not found. If the [[ref: DID Log]] does not exist at the DID's designated HTTPS location (according to the [DID-to-HTTPS Transformation](#the-did-to-https-transformation)), the resolver **MAY** attempt to retrieve it from alternative sources, such as [[ref: Watchers]], for verification and resolution.
- `invalidDid` — Any error that renders the `did:webvh` DID invalid during resolution.

Resolvers **SHOULD** populate the `problemDetails` field to aid in diagnosing and understanding resolution failures. The [did:webvh information site](https://didwebvh.info) may serve as a non-normative reference for common `did:webvh` resolution error types and explanations.

##### Reading did:webvh DID URLs

A `did:webvh` DID identifies a log by its [[ref: SCID]]; host/path locate where the log is hosted. When a resolver receives a request, the [[ref: SCID]] segment of the requested DID **MUST** equal the [[ref: SCID]] segment of `state.id` in all entries of the retrieved log **and** equal `parameters.scid` from the first entry. A log whose `state.id` [[ref: SCID]] does not match the requested DID — even if host/path matches — **MUST NOT** be returned; resolution **MUST** terminate. This rule applies independently of whether portability is enabled.

A `did:webvh` resolver **MUST** resolve the [[spec:DID-Core]] `versionId` and
`versionTime` DID URL query parameters. The `versionId` query argument value
**MUST** match the full `versionId` from a [[ref: DID Log entry]] for the
resolver to return that version of the DIDDoc. If a [[ref: DID Log entry]] with
that `versionId` is not found, a `NotFound` **MUST** be returned. A specified
time in [[ref: ISO8601]] format as the query argument for `versionTime` **MUST**
return the DIDDoc from the [[ref: DID Log entry]] that was active at that time,
if any. If the DID was not active at the specified time, a `NotFound` **MUST**
be returned.

A `did:webvh` resolver **SHOULD** resolve the DID URL query parameter
`versionNumber` with an integer value if there is a [[ref: DID Log entry]] with
a `versionId` with a matching integer prior to the literal `-` -- the
`versionNumber` for that [[ref: DID Log entry]] as defined in the process for
setting the `versionId` in the [creating the DID](#create-register) section of
this specification. The `versionNumber` query parameter is not in the
[[spec:DID-Core]] specification.

A `did:webvh` resolver **MAY** implement the resolution of the `/whois` and a DID
URL Path using the [whois LinkedVP Service](#whois-linkedvp-service) and 
[DID URL Path Resolution Service](#did-url-path-resolution-service) as defined in
this specification by processing the [[ref: DID Log]] and then dereferencing the
DID URL based on the contents of the [[ref: DIDDoc]]. The client of a resolver that does
not implement those capabilities must use the resolver to resolve the
appropriate [[ref: DIDDoc]], and then process the resulting DID URLs themselves. Since
the default DID-to-HTTPS URL transformation is trivial, `did:webvh`
[[ref: DID Controllers]] are strongly encouraged to use the default behavior
for DID URL Path resolution.

#### Update (Rotate)

Updating a `did:webvh` DID follows the [Update](#update) algorithm defined in [[ref: VH-Log]], with the following did:webvh-specific constraints:

- **State changes (VH-Log Update step 1):** Changes **MUST** be made to the [[ref: DIDDoc]]. The top-level `id` in the [[ref: DIDDoc]] **MUST** contain the value of the DID. If the DID is configured with `portable: true`, the `id` **MAY** be changed to reflect a different HTTPS location while retaining the [[ref: SCID]] and verifiable history. See [DID Portability](#did-portability).
- **Parameters:** `parameters` **MUST** be drawn from those defined in [`did:webvh` DID Method Parameters](#didwebvh-did-method-parameters).
- **Witness file:** If [[ref: witnesses]] are active, the [[ref: DID Controller]] **MUST** collect the [[ref: threshold]] of witness proofs and publish the updated `did-witness.json` file **BEFORE** publishing the updated [[ref: DID Log]]. See [DID Witnesses](#did-witnesses).
- **Log file:** The updated log file **MUST** be published as `did.jsonl` at the HTTPS location determined by the [DID-to-HTTPS Transformation](#the-did-to-https-transformation) from the DID. If [[ref: watchers]] are configured, trigger webhooks to notify them. See [Watchers](#did-watchers).

A controller **MAY** generate an updated `did:web` [[ref: DIDDoc]] and publish it as defined in the [Publishing a Parallel `did:web` DID](#publishing-a-parallel-didweb-did) section.

#### Deactivate (Revoke)

Deactivating a DID allows a [[ref: DID Controller]] to signal that the DID is no longer being maintained or updated. There is an explicit approach to deactivation that aligns with the [[spec: DID-CORE]] specification, and other approaches that a `did:webvh` [[ref: DID Controller]] can use if the [[spec: DID-CORE]] approach does not achieve their desired outcome. This section covers the various methods for signaling that a DID is retired.

To deactivate a `did:webvh` DID per the [[spec: DID-CORE]] specification, the [[ref: DID Controller]] **MUST** add to the [[ref: DID log entry]] [[ref: parameters]] the property name and value `"deactivated": true`. Once done, a resolver **MUST NOT** return the [[ref: DIDDoc]] and **MUST** include `"deactivated": true` in the DID Resolution Metadata. A [[ref: DID Controller]] deactivating a `did:webvh` DID **MAY** update some [[ref: parameters]] attributes to further indicate the deactivation of the DID, such as setting the `updateKeys` array to `[]`, preventing further versions of the DID. If the DID is using [[ref: pre-rotation]], two [[ref: DID log entries]] are required to accomplish that state: the first to stop the use of pre-rotation, and the second to set `updateKeys` to []. For additional details about turning off [[ref: pre-rotation]] see the [pre-rotation](#pre-rotation) section of this specification.

A concern with using the [[spec: DID-CORE]] approach to deactivation is the resolver requirement that the [[ref: DIDDoc]] not be returned for a deactivated DID. A [[ref: DID Controller]] might want the final [[ref: DIDDoc]] to continue to be resolved, while also signaling that the DID is no longer being updated. In the future, a DID Resolution query parameter (`returnDeactivatedDidDocument=true`) has been proposed to be added to the [[spec: DID-RESOLUTION]] specification to allow a client to request the [[ref: DIDDoc]] of a deactivated DID. However, even if that parameter is adopted, it places the burden on the resolver client to decide whether and how to use it. An alternative that a `did:webvh` [[ref: DID Controller]] could use is to signal that a DID is no longer being updated by setting the `updateKeys` array to empty (`[]`), as discussed above, and not setting the `deactivated` [[ref: parameter]] to `true`. The result is that the final [[ref: DIDDoc]] continues to be returned by default, and the DID Resolution Metadata indicates that the DID can no longer be updated.

To resolve a prior version of a deactivated `did:webvh` DID, a client can use the appropriate DID Resolution query [[ref: parameters]] `versionId`, `versionTime`, or the did:webvh-specific `versionNumber` (as described in the Read (Resolve) section of this specification). When such a DID is resolved in this way, the DID Resolution Metadata **MUST** include the property name and value `"deactivated": true`.

A [[ref: DID Controller]] can “deactivate” a DID by removing the published [[ref: DID Log]] and associated files and resources. Once removed, attempts to retrieve the [[ref: DID Log]] will result in an `Not Found` error status when resolving the DID. [[ref: Watchers]] monitoring a removed DID **SHOULD** continue to cache the last known valid state of the DID indefinitely so that their clients can still resolve and reference it, even after the [[ref: DID Log]] has been deleted.

### DID Method Processes

The [DID Method Operations](#did-method-operations) reference several processes
that are executed during [[ref: DIDDoc]] generation and DID resolution verification. Each
of those processes is specified in the following sections.

#### `did:webvh` DID Method Parameters

`did:webvh` uses the [[ref: VH-Log]] parameters mechanism. All `did:webvh` [[ref: Log entries]] contain the JSON object `parameters`. This object defines the DID processing [[ref: parameters]] used by the [[ref: DID Controller]] when publishing the current and subsequent [[ref: DID log entries]]. DID Resolvers **MUST** use the same [[ref: parameters]] to process the [[ref: DID Log]] to resolve the DID. The `parameters` object **MUST** only include properties defined in the version of the `did:webvh` DID Method specification being used.

The `method` parameter serves the same role as the `logVersion` parameter in [[ref: VH-Log]]; it uses the `did:webvh` namespace prefix (e.g., `did:webvh:1.0` instead of `vh-log:1.0`). Each `method` value corresponds to a [[ref: VH-Log]] version and defines the same cryptographic algorithm set as the corresponding `logVersion`. The `portable` parameter is a `did:webvh`-specific extension with no VH-Log equivalent.

**General Rules for Parameters:**

The general rules for parameters — default values, allowed values, deactivation, the prohibition on using JSON `null`, and the note on legacy `null`-using implementations — are as defined in [VH-Log's General Rules for Parameters](../next/index.html#general-rules-for-parameters) section. The only did:webvh-specific addition: the `method` parameter (see below) plays the role VH-Log's `logVersion` plays for the "Default Values" rule. `method` is required in the first [[ref: log entry]] and **MAY** appear in later entries to upgrade the DID to a newer [[spec:semver]] version of this specification.

::: example
An example of the `parameters` property in the first [[ref: DID Log]] entry:

```json
{
  "portable": true,
  "updateKeys": [
    "z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"
  ],
  "nextKeyHashes": [
    "enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"
  ],
  "method": "did:webvh:1.0",
  "scid": "{SCID}"
}
```
:::

The following lists the `did:webvh`-specific [[ref: parameters]]. The parameters
`scid`, `updateKeys`, `nextKeyHashes`, `witness`, `watchers`, `deactivated`, and `ttl`
are used in `did:webvh` exactly as defined in [[ref: VH-Log]], with one
did:webvh-specific constraint: `witness[].id` values **MUST** be `did:key` DIDs (see the
[DID Witnesses](#did-witnesses) section). Refer to [[ref: VH-Log]] for the normative
definitions of those parameters.

- `method`: The did:webvh counterpart to VH-Log's `logVersion` parameter. Specifies the
  `did:webvh` specification version to be used for processing the [[ref: DID Log]]. Each
  value defines the permitted cryptographic algorithms and corresponds to a VH-Log version.
  - **MUST** appear in the first [[ref: log entry]] and **MUST** be one of the enumerated
    acceptable values below.
  - Resolvers **MUST** reject any `method` value that is not **exactly** one of the
    acceptable values for the version(s) of this specification the resolver supports.
    Unknown values **MUST NOT** be silently downgraded, defaulted, or ignored —
    resolution **MUST** terminate.
  - If not present in later entries, the previous value continues to be active.
  - **MAY** appear in later entries to upgrade the spec version. A change to a *lower*
    version than currently active **MUST** be rejected.
  - Acceptable values:
    - `did:webvh:1.0` (corresponds to VH-Log `vh-log:1.0`)
      - Permitted hash algorithms: `SHA-256` [[spec:rfc6234]] (multihash code `0x12`)
        **only**. Any other algorithm **MUST** cause resolution to terminate.
      - Permitted [[ref: Data Integrity]] cryptosuites for **both** log-entry proofs
        **and** witness proofs: exactly `eddsa-jcs-2022` [[spec:di-eddsa-v1.0]].
        Resolvers **MUST** verify the proof's `cryptosuite` property; an absent,
        mismatched, or non-conformant `cryptosuite` **MUST** cause the entry to be
        rejected. Verifying only `proofPurpose` is **insufficient**.
        - [[ref: witness]] [[ref: did:key]] identifiers **MUST** use a key compliant
          with the `eddsa-jcs-2022` cryptosuite defined in [[spec:di-eddsa-v1.0]].

- `portable`: Boolean (JSON `true` / `false`) indicating if the DID is portable,
  allowing a [[ref: DID Controller]] to move the DID while retaining its [[ref: SCID]]
  and verifiable history. See the [DID Portability](#did-portability) section for
  details. This parameter has no VH-Log equivalent.
  - Setting `portable: true` is permitted **only** in the first entry. A later entry
    that sets `portable: true` **MUST** be rejected by resolvers, regardless of
    historical state.
  - Defaults to `false` if omitted in the first entry. Resolvers **SHOULD** warn if
    `portable` is omitted from the first entry.
  - Retains value if omitted in later entries.
  - Setting `portable: false` in any later entry permanently disables portability;
    later entries **MUST NOT** set it back to `true`.
  - Even when `portable: true`, the SCID segment of `state.id` (and of
    `parameters.scid` in the first entry) **MUST NOT** change for the life of the
    DID. Only host/path portions may change.

#### Cryptographic Agility

`did:webvh` inherits cryptographic agility from [[ref: VH-Log]] unchanged. See the
Cryptographic Agility section of [[ref: VH-Log]] for the full definition. In
`did:webvh`, the `method` parameter serves the role of VH-Log's `logVersion` for
version-specific algorithm policies, using `did:webvh`-namespaced values (e.g.,
`did:webvh:1.0` maps to `vh-log:1.0`).

#### SCID Generation and Verification

The SCID generation and verification algorithm for `did:webvh` is identical to the
algorithm defined in the [[ref: VH-Log]] specification. It is reproduced here for
implementer convenience.

The [[ref: self-certifying identifier]] or `SCID` is a required [[ref: parameter]] in the
first [[ref: DID log entry]] and is the hash of the DID's inception event.

##### Generate SCID

The SCID is generated using `base58btc(multihash(JCS(preliminary log entry with placeholders), <hash algorithm>))` as defined in [[ref: VH-Log]]. The preliminary log entry is constructed as described in [Create (Register)](#create-register) step 4 of this specification, with the literal string `{SCID}` used wherever the SCID will appear. The hash algorithm **MUST** be one permitted by the active `method` parameter.

##### Verify SCID

The SCID is verified using the algorithm defined in [[ref: VH-Log]]: extract `scid` from the first entry's `parameters`; reconstruct the preliminary log entry by removing the `proof`, replacing `versionId` with `{SCID}`, and replacing all occurrences of the `scid` value with `{SCID}`; apply the same hash function; confirm the output matches `parameters.scid`. If not, terminate resolution with an error.

#### Entry Hash Generation and Verification

The entry hash generation and verification algorithm for `did:webvh` is identical to
[VH-Log's Entry Hash Generation and Verification](../next/index.html#entry-hash-generation-and-verification)
algorithm, applied to [[ref: DID log entries]].

##### Generate Entry Hash

The entry hash is generated using `base58btc(multihash(JCS(entry), <hash algorithm>))` as defined in [[ref: VH-Log]]. The input `entry` has its `versionId` set to the previous entry's `versionId` (or the SCID for the first entry) and the `proof` property removed. The hash algorithm **MUST** be one permitted by the active `method` parameter.

##### Verify The Entry Hash

The entry hash is verified using the algorithm defined in [[ref: VH-Log]]: extract the `entryHash` from the `versionId`; determine the hash algorithm from the [[ref: multihash]] prefix; remove `proof` and set `versionId` to the predecessor value (or SCID for the first entry); compute `base58btc(multihash(JCS(entry), <hash algorithm>))`; confirm it matches the extracted `entryHash`. If not, terminate resolution with an error.

#### Authorized Keys

The authorized keys mechanism — the required proof properties, how the active
`updateKeys` list is determined, and the distinct rules for when
[[ref: Pre-Rotation]] is and is not active — is as defined in
[VH-Log's Authorized Keys](../next/index.html#authorized-keys) section, applied
here to [[ref: DID log entries]]. The only did:webvh-specific constraint is on
`cryptosuite`: it **MUST** be one listed in the
[parameters](#didwebvh-did-method-parameters) defined by the active `method`
[[ref: parameter]]. Resolvers **MUST NOT** accept a structurally-valid signature
over a different cryptosuite than the active `method` allows.

The `did:webvh` Implementation Guide contains further discussion on the management
of keys authorized to update the DID.

#### DID Portability

As noted in the [Update (rotate)](#update-rotate) section of the specification,
a `did:webvh` DID can be renamed by changing the `id` DID string in the
DIDDoc to one that resolves to a different HTTPS URL if the following conditions are met.

- The [[ref: DID Log]] of the renamed DID **MUST** contain all of the [[ref: log entries]]
  from the creation of the DID.
- The [[ref: log entry]] in which the DID is renamed **MUST** be a valid DID entry
  building on the prior [[ref: DID log entries]], per this specification.
- The [[ref: parameter]] `portable` **MUST** be set to `true` in the **first** [[ref: log entry]]. An entry that introduces `portable: true` after the first entry **MUST** be rejected.
- The [[ref: SCID]] **MUST** be the same in the original and renamed DID. Specifically, the SCID segment of `state.id` in **every** [[ref: log entry]] (including the renamed entry and all subsequent entries) **MUST** equal the `parameters.scid` from the first entry. Only the host/path portion of `state.id` may change under portability; the SCID segment is immutable for the life of the DID. A "portable rename" entry whose `state.id` carries a different SCID **MUST** be rejected.
- The [[ref: DIDDoc]] **MUST** contain the prior DID string as an `alsoKnownAs` entry.
- [[ref: DID Controllers]] **SHOULD** account for any DNS requirements in making domain changes that impact a `did:webvh` DID being moved, such as those outlined in [[spec:rfc1034]] (“Domain Names - Concepts and Facilities”), and [[spec:rfc1035]] (“Domain Names Implementation and Specification”).

Because the DNS portion of the DID is used to find the [[ref: DID Log]], a
domain or namespace that is reassigned while DID resources remain under it can
cause a DID to be misattributed. A [[ref: DID Controller]] relinquishing a DNS
name or namespace **SHOULD** first either move the DID to a new location under
its control using portability, or deactivate it.

**Security Note — Misleading Prior Domain Association**

When using portability, a `did:webvh` identifier may include a domain component that was never actually used to host its DID Log before being "moved" to a domain under the [[ref: DID Controller]]’s control. This creates a potential for misleading claims of association with the original domain. Resolvers and clients of resolvers **MUST** ignore any prior domain components when evaluating the history or trustworthiness of a `did:webvh` DID; only the current hosting location and its associated verifiable history are relevant. In addition, the [whois](#whois-linkedvp-service) DID URL capability can be used to obtain attestations about the DID and [[ref: DID Controller]] from relevant authorities.

#### Pre-Rotation Key Hash Generation and Verification

`did:webvh` implements the key pre-rotation mechanism — including its rationale,
the hash generation and verification algorithm, and the requirement to treat
spent pre-rotation keys as destroyed — exactly as defined in
[VH-Log's Pre-Rotation Key Hash Generation and Verification](../next/index.html#pre-rotation-key-hash-generation-and-verification)
section, applied here to [[ref: DID log entries]]. See the non-normative section
about [Using Pre-Rotation Keys](https://didwebvh.info/latest/implementers-guide/prerotation-keys/)
in the `did:webvh` Implementer's Guide for additional, did:webvh-specific guidance.

#### DID Witnesses

`did:webvh` implements the witness mechanism defined in the [[ref: VH-Log]] specification.
The witness proofs file for `did:webvh` is named `did-witness.json`; its URL is derived
from the [[ref: DID Log]] URL by replacing `did.jsonl` with `did-witness.json`. The data
model and verification rules are as defined in [[ref: VH-Log]] and are reproduced below
with `did:webvh`-specific details.

The [[ref: witness]] mechanism — including when witnesses activate, replacement witness lists, the [[ref: threshold]] algorithm, and the witnessing process — is as defined in [[ref: VH-Log]]. The `did:webvh`-specific constraints are that witness `id` values **MUST** be `did:key` DIDs, and witness proofs are stored in `did-witness.json`.

`did:webvh` witnesses prevent a [[ref: DID Controller]] from updating/removing versions of a DID without detection. They also mitigate against attackers who compromise both the [[ref: DID Controller]]'s authorization key(s) and their web server: without also compromising a [[ref: threshold]] of witnesses, such an attacker cannot rewrite the [[ref: DID Log]] undetected. As described in VH-Log's [Witnesses](../next/index.html#witnesses) section, this depends on witnesses behaving as required — approving only entries that extend their own copy of the [[ref: DID Log]] — which resolvers cannot verify; [[ref: watchers]] provide detection that does not depend on trusting the witnesses.

##### Witness Lists

The witness list mechanism — when the `witness` parameter takes effect, how replacement lists activate, and when witnessing is required — is as defined in [[ref: VH-Log]].

##### Witness DIDs and Reputation

Since `did:webvh` witness DIDs must be `did:key` DIDs, there is not an
explicitly published identifier for each witness. If there is a need in an ecosystem
to identify who the witnesses are, a mechanism should be defined by the
governance of the ecosystem, such as the entry of the DID in a trust registry.
Such mechanisms are outside the scope of this specification.

When a [[ref: did:key]] DID is used in any `did:webvh` context — as a witness `id`, as a `verificationMethod` controller, or as an `assertionMethod` reference in a [[ref: Data Integrity]] proof the [[ref: did:key]] specification **MUST** be followed. Notably, the multibase value in the method-specific identifier (the **body** of the DID) **MUST** equal the multibase value in any fragment identifier (if present) that references the lone verification method within the DID. For `did:key:z6MkABC...#z6MkABC...`, body and fragment **MUST** be byte-for-byte equal. Verifiers **MUST** reject any reference where they differ, because the body authoritatively defines the public key while the fragment is the Verification Method id; permitting divergence would let an attacker claim a proof was made by `did:key:A` while actually signing with `did:key:B`.

##### The `witness` Parameter

The `witness` element in a [[ref: parameters]] object of a [[ref: DID Log entry]]
has the following data structure:

```json

"witness" : {
  "threshold": n,
  "witnesses" : [
      {
         "id": "<did:key DID of witness>"
      }
   ]
}

```

where:

- `threshold` and the uniqueness/counting rules for `witnesses[].id` are as
  defined in [VH-Log's `witness` Parameter](../next/index.html#the-witness-parameter)
  section.
- `witnesses[].id`: the did:webvh-specific identifier format is a `did:key` DID
  (see [Witness DIDs and Reputation](#witness-dids-and-reputation)). The `did:key`
  **MUST** decode to a public key compatible with one of the cryptosuites listed
  in the [parameters](#didwebvh-did-method-parameters) for the active `method`. A
  witness `id` whose body decodes to a key of any other type **MUST** be rejected
  at parameter-validation time — not at signature-verification time — so an
  invalid witness configuration cannot ever take effect.

##### Witness Threshold Algorithm

The threshold algorithm is as defined in [[ref: VH-Log]]: verify each [[ref: Data Integrity]] proof independently; attribute each to a distinct witness `id` from the active list (unattributable proofs are discarded); the update is "[[ref: witnessed]]" when the count of distinct attributed `id` values meets or exceeds `threshold`, otherwise resolution **MUST** terminate with an error. Governance decisions about what "approve" means are outside the scope of this specification.

##### The Witness Proofs File

Proofs from [[ref: witnesses]] are placed into a separate file (`did-witness.json`)
from the [[ref: DID Log]], located via the same [DID to HTTPS
Transformation](#the-did-to-https-transformation) used for the [[ref: DID Log]],
with the last element changed from `did.jsonl` to `did-witness.json`. The media
type of the file **SHOULD** be `application/json`.

The data model, the meaning of its `versionId` and `proof` fields, the rule that
a witness proof implies approval of all prior entries, the requirement to
publish `did-witness.json` before the updated `did.jsonl` (to avoid a
publication race condition), and the guidance on pruning stale proofs are all as
defined in [VH-Log's Witness Proofs File](../next/index.html#the-witness-proofs-file)
section. The permitted [[ref: Data Integrity]] cryptosuite for witness proofs
**MUST** be one listed in the [parameters](#didwebvh-did-method-parameters) for
the active `method`.

Because a witness `id` is a `did:key` DID, the verification key is fully determined by decoding the `did:key` body — no DID resolution is required and no `verificationMethod` lookup is permitted. Resolvers verifying a witness proof **MUST**:

1. Parse the proof's `verificationMethod` as `did:key:<multibase>#<multibase>` and recover the public key by decoding the body multibase per the [[ref: did:key]] specification.
2. Verify the signature using **only** that key. Resolvers **MUST NOT** dereference the witness DID for a key, **MUST NOT** consult any other key store, and **MUST NOT** accept a proof whose body multibase decodes to a key structurally invalid for the cryptosuites mandated by the active `method`.

Since resolvers cannot verify an unpublished [[ref: DID log entry]], a witness
proof on an unpublished entry does not carry the implication of approval of
prior entries. As a result, `did-witness.json` may at times legitimately contain
two proofs for the same witness: the most recent proof for a published entry,
and an additional proof for an entry not yet published.

##### Witnessing a DID Version Update

The witnessing process follows [[ref: VH-Log]]: the [[ref: DID Controller]] shares the complete [[ref: DID Log Entry]] (including `proof`) with active [[ref: witnesses]]; each [[ref: witness]] independently verifies it using the full [Read (Resolve)](#read-resolve) algorithm and, if approved, returns a [[ref: Data Integrity]] proof signed with its `did:key` DID; the [[ref: DID Controller]] collects the [[ref: threshold]] of proofs into `did-witness.json` and publishes that file **before** publishing the updated `did.jsonl`.

##### Verifying Witness Proofs During Resolution

A `did:webvh` resolver **MUST** verify witness proofs following the algorithm defined in [[ref: VH-Log]] (complete all non-witness verifications first; retrieve `did-witness.json`; discard entries whose `versionId` is not in the log; verify enough proofs to meet [[ref: threshold]] for all entries requiring witnessing). For each witness proof, the resolver **MUST** additionally confirm the `did:key`-specific constraint — extract `verificationMethod` and confirm that:

1. It is a valid, [[ref: did:key]] specification-compliant DID URL of the form `did:key:<multibase>#<multibase>` where body and fragment multibases are byte-equal (see [`did:key` body/fragment check](#witness-dids-and-reputation)).
2. No previously verified proof was found in the witnesses array for the same `versionId` and `id` (one count per witness per entry).

A proof failing either requirement **MUST** be discarded from threshold counting.

A [[ref: DID Controller]] is expected to prune the `did-witness.json` file to include only the last proof for each witness for a published [[ref: DID log entry]]. However, if a [[ref: DID Controller]] does not prune the file, a resolver **MAY** do the pruning as part of the resolution process, verifying only the minimum number of proofs needed to meet the [[ref: threshold]] for each [[ref: DID log entry]]. While it is expected that a [[ref: DID Controller]] will exclude any proofs that fail verification, a resolver **MAY** ignore any proofs that fail verification and still resolve the DID if there are enough valid proofs to meet the [[ref: threshold]] requirements.

If you want to learn more about the practical application of witnesses, see the
Implementer's Guide section on
[Witnesses](https://didwebvh.info/latest/implementers_guide/#witnesses) on the
`did:webvh` information site for more discussion on the witness capability and
using it in production scenarios.

#### DID Watchers

`did:webvh` implements the watcher mechanism defined in the [[ref: VH-Log]] specification.
The `did:webvh`-specific additions are the HTTP API operations for watcher endpoints,
defined in the [Watcher HTTP API Operations](#watcher-http-api-operations) section below.

The watcher concept — purpose, roles, governance, and the `watchers` parameter mechanism — is as defined in [[ref: VH-Log]]. When resolving a `did:webvh` DID, resolvers **MUST** provide the active list of [[ref: watchers]] in the DID Resolution Metadata as noted in the [Read (Resolve)](#read-resolve) section. Watcher URIs **MAY** use schemes other than HTTP(S) (such as a DID per [[spec:DID-CORE]]); if a [[ref: watcher]] uses HTTP(S), it **MUST** support the API defined below.

##### Watcher HTTP API Operations

The following HTTP API operations define the interaction between [[ref: watchers]] and other components. Included with the specification is the [did:webvh v1.0 Watcher OpenAPI YML Definition](https://raw.githubusercontent.com/decentralized-identity/didwebvh/refs/heads/main/watcherOpenAPI/watcher-v1.0.0.yml) that includes the request and response data models and status codes for each of the endpoints.

- **GET `<WATCHER URL>/log?scid=<SCID>`**: Returns the latest [[ref: DID Log]] for the given [[ref: SCID]].
- **POST `<WATCHER URL>/log?did=<DID>`**: Notifies the [[ref: watcher]] of a log update, prompting retrieval of the latest [[ref: DID Log]] and [[ref: witness]] file. This endpoint uses the `did` as the query parameter instead of the [[ref: SCID]] to ensure that the [[ref: watcher]] is notified in the case of the DID moving to a new web location. The [[ref: watcher]] is expected to continue indexing the DID using its [[ref: SCID]].
- **POST `<WATCHER URL>/log/delete?scid=<SCID>`**: Notifies the [[ref: watcher]] that the given `<SCID>` should be deleted from the [[ref: watcher]]'s cache. If removed, subsequent requests for that `<SCID>` from clients should return a `404 Not Found` status. The body of the URL is a Data Integrity proof from the requester that may be used by the [[ref: Watcher]] to decide on the legitimacy of the request. The [[ref: Watcher]] will act (or not) on the request according to its governance, which is out of scope of this specification. For example, a [[ref: watcher]] might implement a workflow be completed to approve the deletion of a `<SCID>` from the [[ref: watcher]]'s cache. The endpoint could be used to carry out a "right to be forgotten" order, such as might be required under Europe's [General Data Protection Regulation (GDPR)](https://gdpr-info.eu/).
- **GET `<WATCHER URL>/witness?scid=<SCID>`**: Returns the latest `witness.json` file for the given [[ref: SCID]].
- **GET `<WATCHER URL>/resource?scid=<SCID>&path=<resourcePath>`**: Retrieves the requested resource.
- **POST `<WATCHER URL>/resource?scid=<SCID>&path=<resourcePath>`**: Notifies the [[ref: watcher]] of a new or updated resource.
- **POST `<WATCHER URL>/resource/delete?scid=<SCID>&path=<resourcePath>`**: Notifies the [[ref: watcher]] that the given `<resourcePath>` associated with the `<SCID>` should be deleted from the [[ref: watcher]]'s cache. If removed, subsequent requests for that `<SCID>` and `<resourcePath>` from clients should return a `404 Not Found` status. The body of the URL is a Data Integrity proof from the requester that may be used by the [[ref: Watcher]] to decide on the legitimacy of the request. The [[ref: Watcher]] will act (or not) on the request according to its governance, which is out of scope of this specification. For example, a [[ref: watcher]] might implement a workflow be completed to approve the deletion of a `<SCID>` from the [[ref: watcher]]'s cache. The endpoint could be used to carry out a "right to be forgotten" order, such as might be required under Europe's [General Data Protection Regulation (GDPR)](https://gdpr-info.eu/).

#### Publishing a Parallel `did:web` DID

Each time a `did:webvh` version is created, the [[ref: DID Controller]] **MAY**
generate a corresponding `did:web` to publish along with the `did:webvh`. If
this is being done, the `did:webvh` DIDDoc **SHOULD** have the corresponding
`did:web` in the `alsoKnownAs` array. To publish a parallel `did:web` DIDDoc, the
[[ref: DID Controller]] **MUST**:

1. Start with the resolved version of the [[ref: DIDDoc]] from `did:webvh`.
2. If the "implicit" `did:webvh` services (as defined in the [DID URL
   Resolution](#did-url-resolution) section) are not already present in the
   [[ref: DIDDoc]], they **MUST** be added. These services are the `relativeRef`
   service with `id: "#files"` or `id: "<did>#files"` and the `whois` service
   with `id: "#whois"` or `id: "<did>#whois"`, with the `serviceEndpoint` for
   both derived from the [DID-to-HTTPS
   transformation](#did-to-https-transformation).
3. Execute a text replacement across the [[ref: DIDDoc]] of `did:webvh:<SCID>:` to
   `did:web:`, where `<scid>` is the actual `did:webvh` [[ref: SCID]].
4. Add to the [[ref: DIDDoc]] `alsoKnownAs` array, the full `did:webvh` DID. If
   the `alsoKnownAs` array does not exist in the [[ref: DIDDoc]], it **MUST** be
   added.
5. Remove any duplicate entries in the `alsoKnownAs` array, including the
   `did:web` DID itself if it was duplicated in the earlier steps.
6. Publish the resulting [[ref: DIDDoc]] as the file `did.json` at the web location
   determined by the specified `did:web` DID-to-HTTPS transformation.

The benefit of doing this is that resolvers that have not been updated to
support `did:webvh` can continue to resolve the [[ref: DID Controller]]'s DIDs.
`did:web` resolvers that are aware of `did:webvh` features can use that knowledge,
and the existence of the `alsoKnownAs` `did:webvh` data in the [[ref: DIDDoc]] to get the
verifiable history of the DID.

The risk of publishing the `did:web` in parallel with the `did:webvh` is that the
added security and convenience of using `did:webvh` are lost.

### DID URL Resolution

The `did:webvh` DID method embraces the expressive power of DID URLs while
preserving the semantic simplicity of a web-based resolution model. In
particular, `did:webvh` implementations **MUST** support path-based DID URL
resolution in a manner consistent with the [DID Core
specification](https://www.w3.org/TR/did-core/#did-url-path).

Specifically, a `did:webvh` resolver **MUST**:

- Resolve any `did:webvh` DID URL with a path component using an implicit
  `relativeRef` service as defined in [[spec:DID-CORE]]. The path is appended
  directly to the HTTPS URL obtained from the [DID-to-HTTPS
  transformation](#the-did-to-https-transformation), excluding any `.well-known`
  prefix.  
  - For example, resolving
    `did:webvh:{SCID}:example.com/governance/issuers.json` retrieves the file
    located at `https://example.com/governance/issuers.json`.  
  - This behavior can be overridden by defining an explicit service in the DID
    Document.

- Resolve the special path `/whois` using an implicit [[spec:LINKED-VP]]
  service. This applies regardless of whether a `whois` service is explicitly
  defined in the [[ref: DIDDoc]]. The resolver **MUST** retrieve the Verifiable
  Presentation, if published by the [[ref: DID Controller]], from the web
  location corresponding to the DID-to-HTTPS transformation (excluding
  `.well-known`), using the path `/whois.vp` and media type `application/vp` as
  registered in the [IANA Media Types
  Registry](https://www.iana.org/assignments/media-types/application/vp).  
  - For example, resolving `did:webvh:{SCID}:example.com/whois` returns the
    content of `https://example.com/whois.vp` if available.

In both cases, a [[ref: DID Controller]] **MAY** override the implicit
resolution behavior by defining explicit services in the DID Document, which
take precedence over the defaults.

The sections below formalize the structure and resolution rules for each default
service and describe how they may be overridden by the [[ref: DID Controller]].

### DID URL Path Resolution

The automatic resolution of `did:webvh` DID URL paths follows the
[[spec:DID-CORE]] `relativeRef` mechanism, enabling path-based access to web
resources directly tied to the DID’s domain. The approach is derived from
Examples 2 and 8 in [Section 3.2 of DID
Core](https://www.w3.org/TR/did-core/#did-url-syntax):

- A DID URL such as `did:example:123456/resume.pdf` (see Example 2)  
  is semantically equivalent to:  
  `did:example:123456?service=files&relativeRef=/resume.pdf` (see Example 8).

- The `service=files` reference resolves against a DID Document service with
  `"id": "#files"` or `"id": "<did>#files"` and a `type` of `relativeRef`.

The `did:webvh` method implicitly defines this service, with a `serviceEndpoint`
derived from the [DID-to-HTTPS
transformation](#the-did-to-https-transformation). The final path segment
(`did.jsonl`) is replaced by the DID URL path. If the resulting HTTPS URL
contains `.well-known/`, that segment **MUST** be removed before dereferencing
the resource.

For example, the following is the implicit service definition for the DID
`did:<scid>:webvh:example.com`:

```json
{
  "id": "#files",
  "type": "relativeRef",
  "serviceEndpoint": "https://example.com/"
}
```

A [[ref: DID Controller]] **MAY** explicitly define a service entry with `"id":
"#files"` in the [[ref: DIDDoc]]. If present, this **MUST** override the
implicit service described above. `id` can be be an absolute reference that
includes the DID with the `#files` fragment (`<did>#files`), or a relative
reference as above.

To resolve a DID URL of the form `<did:webvh DID>/path/to/file`, a did:webvh
resolver MUST:

1. Resolve the base did:webvh DID by retrieving, verifying, and processing its
   [[ref: DID Log]], as defined in this specification.

2. Locate the service entry with `id` `"#files"` or `"<did>#files"` in the
   resulting [[ref: DIDDoc]], or fall back to the implicit service if none is
   defined.

3. Construct the URL by appending the DID URL path to the serviceEndpoint, and
   attempt to retrieve the resource from that location.
   - If the scheme of the serviceEndpoint is not supported by the resolver
     (e.g., non-HTTP(S) protocol), the resolver **MUST** return an `invalidDid`
     error.
   - If the resolution of the constructed URL fails with a “not found” condition
     (e.g., HTTP 404), the resolver **MUST** return the `notFound` error.

#### whois LinkedVP Service

### WHOIS Resolution

The `#whois` service enables recipients of a `did:webvh` DID to retrieve a
[[ref: Verifiable Presentation]]—optionally published by the [[ref: DID Controller]]—containing
one or more embedded [[ref: Verifiable Credentials]].
These credentials may help resolvers or relying parties make informed trust
decisions about the controller of the DID.

The intention is that resolving `<did:webvh DID>/whois` yields a [[ref:
Verifiable Presentation]] published by the [[ref: DID Controller]] that includes
credentials with the DID as the `credentialSubject`. The contents of the
presentation are determined solely by the [[ref: DID Controller]], who selects
which credentials to include. It is up to the resolver or relying party to
decide what assertions (and issuers) are relevant for establishing trust.

`did:webvh` DIDs **automatically** support a `/whois` service endpoint,
implicitly defined using the [[spec:LINKED-VP]] service type. The
`serviceEndpoint` is computed using the standard [DID-to-HTTPS
transformation](#the-did-to-https-transformation), replacing `did.jsonl` with
`whois.vp`, and omitting any `.well-known/` prefix.

The default `#whois` service is:

```json
{
  "@context": "https://identity.foundation/linked-vp/contexts/v1",
  "id": "#whois",
  "type": "LinkedVerifiablePresentation",
  "serviceEndpoint": "<did-to-https-translation>/whois.vp"
}

```

The file located at the `serviceEndpoint` **MUST** contain a [[ref: Verifiable Presentation]]
conforming to the [[ref: W3C VCDM]]. It **MUST** be signed by the
DID. `id` can be be an absolute reference that includes the DID with the
`#whois` fragment (`<did>#whois`), or a relative reference as above.

The presentation **MUST** include at least one [[ref: Verifiable Credential]]
where the `credentialSubject.id` is the DID. Such [[ref: Verifiable Credentials]]
might serve to bind the DID to other identifiers associated with
the [[ref: DID Controller]]. Additional credentials in the presentation **MAY**
be associated with those other identifiers, rather than the DID itself. For
example, the presentation could include a credential linking the DID for a
business to that business’s registration ID, and a second credential (perhaps an
ISO certification) where the `credentialSubject.id` is the registration ID.

A [[ref: DID Controller]] **MAY** explicitly define a `service` with `"id":
"#whois"` in the [[ref: DIDDoc]]. If present, this entry **MUST** override the
implicit service defined above. This is required if the controller wishes to:

- Publish the WHOIS Verifiable Presentation in a different format (i.e., not
  [[ref: W3C VCDM]])
- Serve the WHOIS presentation from a different location or using a non-default
  media type

To resolve the DID URL `<did:webvh DID>/whois`, a resolver MUST:

1. Resolve the base `did:webvh` DID by retrieving, verifying, and processing the
   [[ref: DID Log]]. The resolver will use either an explicitly defined service
   with `"id": "#whois"` or `"id": "<did>#whois"`, or the implicit service defined above.
2. Construct and attempt to retrieve the resource from the `serviceEndpoint`
   URL.
   - If the scheme of the `serviceEndpoint` is unsupported by the resolver
     (e.g., non-HTTP(S)), the resolver **MUST** return the `invalidDid` error.
   - If the request to the service endpoint results in a "not found" condition
     (e.g., HTTP 404), the resolver **MUST** return the `notFound` error.

The returned `whois.vp` **MUST** contain a [[ref: W3C VCDM]] [[ref: verifiable presentation]]
signed by the DID and containing [[ref: verifiable credentials]]
that **MUST** have the DID as the `credentialSubject`.

If a [[ref: DID Controller]] publishes a parallel `did:web` DID and a `whois.vp`
file, the `/whois` endpoint can be resolved using either DID, returning the same
content either way. The [[ref: verifiable presentation]] proof can reference
either DID or include two proofs, each referencing a verification method for one
of the DIDs. If only one DID is referenced, since both DIDs will have an
`alsoKnownAs` for one another and include the same verification methods, a
resolver using the DID not referenced in the proof can choose to verify the
proof with the already resolved DID, or resolve the referenced DID before
verifying the proof.

A [[ref: DID Controller]] **MAY** explicitly add to their [[ref: DIDDoc]] a
`did:webvh` service with the `"id": "#whois"` or `"id": "<did>#whois"`. Such an
entry **MUST** override the implicit `service` above. If the [[ref: DID Controller]]
wants to publish the `whois` [[ref: verifiable presentation]] in a
different format than the [[ref: W3C VCDM]] format, they **MUST** explicitly add
to their [[ref: DIDDoc]] a service with the `"id": "#whois"` or `"id":
"<did>#whois"` to specify the name and implied format of the
[[ref: verifiable presentation]].
