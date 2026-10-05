## `did:vh` DID Method Specification

### Conformance

As well as sections marked as non-normative, all examples, notes, and
informative checklists in this specification are non-normative. Everything
else in this specification is normative.

The key words **MAY**, **MUST**, **MUST NOT**, **NOT REQUIRED**,
**RECOMMENDED**, **REQUIRED**, **SHOULD**, and **SHOULD NOT** in this
specification are to be interpreted as described in BCP 14 [[spec:RFC2119]]
[[spec:RFC8174]] when, and only when, they appear in all capitals, as shown
here.

### Relationship to VH-Log and `did:webvh`

`did:vh` is a [[ref: specialisation]] of the [[ref: VH-Log]] specification.
VH-Log defines the log mechanism underlying `did:vh` — the log entry structure,
cryptographic chaining, [[ref: SCID]], [[ref: parameters]], [[ref: witnesses]],
[[ref: watchers]], and the resolution algorithm. This specification defines
only what is specific to `did:vh`, and implementers **MUST** consult
[[ref: VH-Log]] for the normative definition of the log mechanism.

`did:vh` constrains VH-Log by:

- Specifying the [[ref: state]] object as a W3C DID Document ([[ref: DIDDoc]]).
- Naming the log file `did.jsonl` and the witness proofs file
  `did-witness.json`.
- Defining the DID as `did:vh:<SCID>`, with no location component.
- Defining how the [[ref: DID Log]] is [sourced](#sourcing-the-did-log) and
  passed to a [[ref: Resolver]] as `did:vh` resolution options, including
  the `src` option.
- Using `method` (e.g., `did:vh:1.0`) as the `did:vh`-specific form of the
  VH-Log `logVersion` parameter.
- Defining the DID method identifier, CRUD operations and DID URL handling.

`did:vh` is deliberately as close to
[`did:webvh`](https://identity.foundation/didwebvh/) as its lack of a location allows.
The differences are:

| | `did:webvh` | `did:vh` |
|---|---|---|
| DID | `did:webvh:<SCID>:<domain>[:<path>]` | `did:vh:<SCID>` |
| Locating the [[ref: DID Log]] | DID-to-HTTPS transformation | Passed to the [[ref: Resolver]] in [resolution options](#didvh-resolution-options), after the client [sources it](#sourcing-the-did-log) |
| `method` values | `did:webvh:1.0` | `did:vh:1.0` |
| `portable` parameter | Optional | Not used — the DID never changes |
| Implicit `#files` and `#whois` services | Derived from the HTTPS location | None |
| Parallel `did:web` | Optional | Not applicable |

The log format, [[ref: SCID]] and [[ref: entry hash]] algorithms, key
authorisation, pre-rotation, witness identifiers (`did:key`), witness file
format, and the `versionId`, `versionTime` and `versionNumber` DID URL query
parameters are the same in both methods.

### Target System

`did:vh` does not have a single target system. A `did:vh` [[ref: DID Log]] can
be held by any system that can store and return a file: a web server, a
[[ref: watcher]], a database or registry, a distributed ledger, or the wallets
of the parties that use the DID. The [[ref: SCID]] and the verifiable history
make the [[ref: DID Log]] verifiable no matter which of those systems it was
obtained from.

### Method Name

The namestring that identifies this DID method is: `vh`. A DID that uses this
method **MUST** begin with the prefix `did:vh:`. Per the DID specification, the
prefix **MUST** be in lowercase. The remainder of the DID, after the prefix, is
the [method-specific identifier](#method-specific-identifier).

### Method-Specific Identifier

The `did:vh` method-specific identifier is the [[ref: SCID]] of the DID, and
nothing else. The [[ref: SCID]] **MUST** be
[generated](#scid-generation-and-verification) during the creation of the DID
from its initial content.

A `did:vh` DID **MUST** conform to the following ABNF [[spec:RFC5234]]:

```abnf
vh-did = "did:vh:" scid
scid   = 46base58-char
base58-char = %x31-39 / %x41-48 / %x4A-4E / %x50-5A / %x61-6B / %x6D-7A
              ; the base58btc alphabet: 1-9, A-Z except I and O,
              ; a-z except l
```

A `did:vh` DID URL follows the generic DID URL syntax of [[spec:DID-CORE]],
with the addition of the optional [`src`](#the-src-option) query
parameter.

::: example
A `did:vh` DID, and DID URLs using the `src` and `versionNumber` query
parameters and a fragment:

```text
did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ
did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ?src=https://example.com/dids/issuer
did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ?versionNumber=3
did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ#key-1
```
:::

### The DID Log File

The `did:vh` [[ref: DID Log]] is a [[ref: VH-Log]] log file. The log entry
structure, [[ref: JSON Lines]] serialisation, cryptographic chaining, and
verification algorithm are all as defined in [[ref: VH-Log]].

The `did:vh`-specific constraints are:

- When the log is stored as a file, it **SHOULD** be named `did.jsonl`, and
  **MUST** be named `did.jsonl` when stored at a location named by a [`src`
  reference](#the-src-option). Its media type **SHOULD** be
  `text/jsonl`.
- The witness proofs file, when stored as a file, **SHOULD** be named
  `did-witness.json`, and **MUST** be named `did-witness.json` when stored at
  a location named by a `src` reference.
- The `state` property of each entry **MUST** contain the [[ref: DIDDoc]] for
  that version of the DID.
- The `parameters` property **MUST** follow [`did:vh` DID Method
  Parameters](#didvh-did-method-parameters).

::: example

**`did:vh`-specific examples of log entries that fail verification**, in
addition to the general examples in [[ref: VH-Log]]. Illustrative and
non-exhaustive.

1. **`state.id` is not the DID.** The first entry has `parameters.scid:
   "Qm111..."`, and an entry has `state.id:
   "did:vh:Qm111...?src=https://example.com"` or `"did:webvh:Qm111...:example.com"`.
   Every entry's `state.id` must be exactly `did:vh:Qm111...`.

2. **`portable` parameter present.** An entry includes `"portable": true`.
   `portable` is not a `did:vh` [[ref: parameter]].

3. **`did:webvh` method value.** The first entry has `parameters.method:
   "did:webvh:1.0"`. A `did:vh` log must use a `did:vh` `method` value.

:::

### DID Method Operations

#### Create (Register)

Creating a `did:vh` DID follows the Create algorithm defined in
[[ref: VH-Log]], with the following `did:vh`-specific constraints:

1. **The DID string** (VH-Log Create step 1) is the literal
   `did:vh:{SCID}`, where `{SCID}` is the placeholder for the [[ref: SCID]]
   calculated later.

2. **The initial [[ref: state]] object MUST be a [[ref: DIDDoc]]** (VH-Log
   Create step 3). Its top-level `id` **MUST** be `did:vh:{SCID}`. All other
   absolute references to the DID within the [[ref: DIDDoc]] **MUST** likewise
   use the placeholder form (e.g., `did:vh:{SCID}#key-1`). The
   [[ref: DIDDoc]] **MAY** contain any other content the
   [[ref: DID Controller]] requires.

3. **The [[ref: parameters]]** (VH-Log Create step 4) **MUST** follow
   [`did:vh` DID Method Parameters](#didvh-did-method-parameters), including a
   `did:vh` `method` value.

4. **Witnesses.** If [[ref: witnesses]] are used, the witness proofs **MUST**
   be collected into the witness proofs file before the [[ref: DID Log]] is
   distributed.

5. **Distribution** (VH-Log Create step 7). Instead of publishing the log at a
   location derived from the DID, the [[ref: DID Controller]] makes the
   [[ref: DID Log]] (and witness proofs file) available through the
   [[ref: DID Log sources]] (see [Sourcing the DID
   Log](#sourcing-the-did-log)) that the parties who will
   resolve the DID will use: giving it to a peer, sending it to
   [[ref: watchers]], publishing it at one or more web locations, or storing
   it where the ecosystem expects to find it. A [[ref: DID Controller]]
   **MUST** make the log available through at least one source that those
   parties can use.

#### Read (Resolve)

A `did:vh` [[ref: Resolver]] implements the `resolve` function defined in
[[spec:DID-RESOLUTION]]:

```text
resolve(did, resolutionOptions) →
   « didResolutionMetadata, didDocument, didDocumentMetadata »
```

The `did` input is the DID itself, `did:vh:<SCID>`, which says nothing about
where the [[ref: DID Log]] is. The client of the [[ref: Resolver]] therefore
passes the [[ref: DID Log]] — and, if [[ref: witnesses]] are in use, the
witness proofs — in `resolutionOptions`, in one of the [two
ways](#didvh-resolution-options) defined below: directly, or by reference, in
which case the [[ref: Resolver]] first retrieves the files. The
[[ref: Resolver]] then verifies the [[ref: DID Log]] and returns the requested
version of the [[ref: DIDDoc]].

The [[ref: Resolver]] does not discover the [[ref: DID Log]]. How a client
sources the [[ref: DID Log]] — gets the log itself, or a reference to it —
before calling `resolve` is described in the non-normative [Sourcing the DID
Log](#sourcing-the-did-log) section.

##### `did:vh` Resolution Options

In addition to the resolution options defined in [[spec:DID-RESOLUTION]], a
`did:vh` [[ref: Resolver]] **MUST** recognise the following options, and
support them as defined in this section. They are
`did:vh`-specific, and are to be registered in
[[spec:DID-EXTENSION-RESOLUTION]].

| Option | Value | Purpose |
|---|---|---|
| `didLog` | String | The complete [[ref: DID Log]], in [[ref: JSON Lines]] format. |
| `didWitness` | JSON array | The witness proofs file content, as defined in VH-Log's [Witness Proofs File](../next/index.html#the-witness-proofs-file) section. |
| `src` | String | A reference to the location where `did.jsonl` (and `did-witness.json`) are stored. See [The `src` Option](#the-src-option). |
| `versionId` | String | As defined in [[spec:DID-RESOLUTION]]: resolve the version with this `versionId`, as defined in VH-Log's [Selecting a Version](../next/index.html#selecting-a-version) section. |
| `versionTime` | String | As defined in [[spec:DID-RESOLUTION]]: resolve the version active at this [[ref: ISO8601]] time, as defined in VH-Log's [Selecting a Version](../next/index.html#selecting-a-version) section. |
| `versionNumber` | Integer | Resolve the version with this version number, as defined in VH-Log's [Selecting a Version](../next/index.html#selecting-a-version) section. Less safe than `versionId`; see [Log Freshness and Duplicity](#log-freshness-and-duplicity). |

The options support two ways of giving the [[ref: Resolver]] the
[[ref: DID Log]]:

- **Passing the log directly.** The client passes the content of the
  [[ref: DID Log]] in `didLog`, and of the witness proofs file (if any) in
  `didWitness`.
- **Passing the log by reference.** The client passes, in `src`, a reference
  to the location from which the [[ref: Resolver]] retrieves `did.jsonl` and
  `did-witness.json`, as defined in [The `src` Option](#the-src-option). In
  this version of `did:vh`, a reference is a web location, a location using
  another URL scheme the [[ref: Resolver]] supports, or, for a
  [[ref: Resolver]] local to its client, a local directory or loopback web
  location.

The following requirements apply to these options:

- Exactly one of `didLog` and `src` **MUST** be given.
- `didWitness` **MAY** be given only with `didLog`.
- At most one of `versionId`, `versionTime` and `versionNumber` **MAY** be
  given.
- A value that does not conform to the table above, or to the rules for
  `src`, **MUST** cause the [[ref: Resolver]] to return the `invalidOptions`
  error.
- A [[ref: Resolver]] **MUST** support logs passed directly. It **MAY** decline to
  retrieve from a `src` reference, based on its policy (for example, a
  [[ref: Resolver]] that never retrieves from locations chosen by its
  callers), and then **MUST** return the `featureNotSupported` error, except
  as defined in [Web Locations](#web-locations) and [Local
  Directories](#local-directories).
- A [[ref: Resolver]] **MUST** support `versionId` and `versionTime`, and
  **SHOULD** support `versionNumber`, as defined in VH-Log's [Selecting a
  Version](../next/index.html#selecting-a-version) section. A
  [[ref: Resolver]] that does not support `versionNumber` **MUST** return the
  `featureNotSupported` error when it is given.

::: example
Passing the log directly, resolving version 2:

```json
{
  "did": "did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ",
  "resolutionOptions": {
    "didLog": "{\"versionId\":\"1-Qm...\",...}\n{\"versionId\":\"2-Qm...\",...}\n",
    "didWitness": [ { "versionId": "2-Qm...", "proof": [ ... ] } ],
    "versionNumber": 2
  }
}
```

Passing the log by reference to a web location:

```json
{
  "did": "did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ",
  "resolutionOptions": {
    "src": "https://example.com/dids/issuer"
  }
}
```

Passing the log by reference to a local directory, using a local
[[ref: Resolver]]:

```json
{
  "did": "did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ",
  "resolutionOptions": {
    "src": "file:///var/lib/wallet/dids/QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ"
  }
}
```
:::

###### The `src` Option

The `src` ("source") option is a reference to the location where
`did.jsonl` and, if [[ref: witnesses]] are active, `did-witness.json` are
stored. A `src` value **MUST** be an absolute URL [[spec:RFC3986]], and its
scheme determines the kind of reference:

- `https`, or `http` with a loopback host: a [web
  location](#web-locations), such as `https://example.com/dids/issuer` or
  `http://localhost:8000/dids/issuer`.
- `file`: a [local directory](#local-directories) [[spec:RFC8089]], such as
  `file:///var/lib/wallet/dids/QmfGEU...`.
- Any other scheme: an [other scheme](#other-schemes) reference, such as
  `ipns://k51qzi5uqu5d.../dids/issuer`.

A value that is not an absolute URL, or that is not valid for its kind,
**MUST** cause the `invalidOptions` error.

Whatever the kind of reference, the [[ref: Resolver]] **MUST** retrieve the
files the same way: `did.jsonl` from the location, and, if the active
[[ref: parameters]] require [[ref: witnesses]], `did-witness.json` from the
same location (for a web location, as defined in [Web
Locations](#web-locations)). If `did.jsonl` is not found, the [[ref: Resolver]] **MUST**
return the `notFound` error.

Because a location can hold only one `did.jsonl`, a [[ref: DID Controller]]
or client storing several `did:vh` [[ref: DID Logs]] in one place **SHOULD**
give each its own location (a URL path or a directory). Using the
[[ref: SCID]] as the last path segment is a simple way to do so.

A `src` reference is not authoritative: a [[ref: DID Controller]] **MAY** stop
publishing at a web location at any time, and the [[ref: DID Log]] found
through any `src` reference may not be the latest. Because `src` is not part
of the DID, the DID does not change when the log is stored elsewhere.

A `src` reference also says nothing about who controls the DID: a
[[ref: DID Controller]] can publish a log at any location it can write to,
and anyone can put any `src` value in a DID URL. [[ref: Resolvers]] and their
clients **MUST NOT** treat a `src` reference as evidence of an association
between the DID and the owner of the location.

::: note
The `did:scid` `src` parameter can also name a DID method that stores the
verification data (for example, a DID-Linked Resource on a ledger). This
version of `did:vh` supports only URL references. Other kinds of reference,
such as DID values or references to other ways of storing a
[[ref: DID Log]], may be added in a later version.
:::

###### Web Locations

A web location reference is the base URL under which the files are
published on a web server, such as `https://example.com/dids/issuer`.

The following requirements apply to a web location reference:

- The value **MUST** be an absolute URL with the `https` or `http` scheme, and
  **MUST NOT** include a query or fragment component.
- The value **MAY** include a port, such as `https://example.com:3000/dids`.
- For `https`, the host **MUST** be a domain name, not an IP address. A
  host containing non-ASCII characters (an internationalised domain name)
  **MUST** be converted to its ASCII form (A-labels) as defined in IDNA2008
  [[spec:RFC5891]] before retrieval; a host that cannot be converted **MUST**
  cause the `invalidOptions` error.
- For `http`, the host **MUST** be a loopback host: `localhost`, an IPv4
  address in `127.0.0.0/8`, or `[::1]`. An `http` reference with any other
  host **MUST** cause the `invalidOptions` error. An `http` reference is a
  *loopback reference*, intended for development and testing.
- After any trailing `/` is removed, each path segment, once
  percent-decoded, **MUST** be non-empty, **MUST NOT** be `.` or `..`,
  **MUST NOT** contain `/`, `\` or NUL, and **MUST NOT** begin or end with
  whitespace. The checks apply to the decoded form because percent-encoded
  variants such as `%2E%2E`, `%2F`, `%5C` and `%00` would otherwise pass
  unnoticed. A segment that fails a check **MUST** cause the
  `invalidOptions` error.
- A loopback reference is subject to the same restrictions as a [local
  directory](#local-directories) reference: a [[ref: Resolver]] that accepts
  requests from other systems **MUST NOT** retrieve from it and **MUST**
  return the `invalidOptions` error, and it **MUST NOT** be taken from a DID
  URL. A loopback reference would otherwise let a caller make a
  [[ref: Resolver]] send requests to services on its own host.

To retrieve the [[ref: DID Log]] from a web location, the [[ref: Resolver]]
**MUST**:

1. Take the `src` value, removing any trailing `/`.
2. If the URL has no path, append `/.well-known/did.jsonl`. Otherwise, append
   `/did.jsonl`.
3. Retrieve the [[ref: DID Log]] with an HTTP `GET` (HTTPS for an `https`
   reference), following the transport requirements below.
4. If the active [[ref: parameters]] require [[ref: witnesses]], retrieve the
   witness proofs file from the same URL with the final `did.jsonl` replaced by
   `did-witness.json`.

When retrieving from a web location, a [[ref: Resolver]] **MUST** meet the
requirements of VH-Log's [Retrieving Log
Resources](../next/index.html#retrieving-log-resources) section, which
include HTTPS with server authentication, no automatic redirects, and
rejection of IP literals and private addresses. The only exception is a
loopback reference, for which the requirements to use HTTPS and to reject
loopback hosts do not apply.

This is the same layout as `did:webvh`, so a web location reference names the
same location as the domain and path of a `did:webvh` DID:

::: example

Web location `src` values and the corresponding [[ref: DID Log]] URLs:

`https://example.com` --> `https://example.com/.well-known/did.jsonl`

`https://example.com/dids/issuer` --> `https://example.com/dids/issuer/did.jsonl`

`https://example.com:3000/dids/issuer` --> `https://example.com:3000/dids/issuer/did.jsonl`

`https://example.com/users/{SCID}` --> `https://example.com/users/{SCID}/did.jsonl`

`http://localhost:8000/dids/issuer` --> `http://localhost:8000/dids/issuer/did.jsonl`

:::

###### Local Directories

A local directory reference identifies a directory on the system where the
[[ref: Resolver]] runs. It is for a [[ref: Resolver]] used by a client on the
same system — for example, a [[ref: Resolver]] used as a library or
command-line tool — such as a wallet that keeps the [[ref: DID Logs]] it holds
in local storage.

- A local directory reference **MUST** be a `file:` URL [[spec:RFC8089]] with
  an absolute path, an empty host or the host `localhost`, and no query or
  fragment component.
- To retrieve the files, the [[ref: Resolver]] **MUST** remove any trailing
  `/` from the path, and append `/did.jsonl` for the [[ref: DID Log]] and
  `/did-witness.json` for the witness proofs file. A file that does not exist
  or cannot be read is treated as not found.
- A [[ref: Resolver]] **MUST** treat the content of the files exactly as if it
  had been passed directly.
- A [[ref: Resolver]] that accepts requests from other systems — for example,
  over HTTP — **MUST NOT** read local files, and **MUST** return the
  `invalidOptions` error for a local directory reference.
- A local directory reference **MUST NOT** be taken from a DID URL, as
  defined in [Reading `did:vh` DID URLs](#reading-didvh-did-urls). A DID URL
  can come from an untrusted party; allowing it to name local directories
  would let such a party make a [[ref: Resolver]] read files on its host.

::: example

Local directory `src` values and the corresponding [[ref: DID Log]] files:

`file:///var/lib/wallet/dids/QmfGEU...` --> `/var/lib/wallet/dids/QmfGEU.../did.jsonl`

`file://localhost/home/alice/dids/QmfGEU.../` --> `/home/alice/dids/QmfGEU.../did.jsonl`

:::

###### Other Schemes

A [[ref: Resolver]] **MAY** support `src` references using other URL schemes,
provided the scheme has hierarchical paths through which a base URL names a
location holding `did.jsonl` and `did-witness.json` — for example, `ipns`
and `ipfs` URLs naming an IPFS directory. For such a reference:

- The value **MUST NOT** include a query or fragment component.
- The [[ref: Resolver]] **MUST** remove any trailing `/`, and append
  `/did.jsonl` for the [[ref: DID Log]] and `/did-witness.json` for the
  witness proofs file. The `/.well-known/` form used by web locations does
  not apply.
- A [[ref: Resolver]] that does not support the scheme **MUST** return the
  `featureNotSupported` error.
- A [[ref: Resolver]] **MUST** apply to the retrieval protections equivalent
  to those required for [web locations](#web-locations), including to any
  gateway it uses to reach the scheme's network.

::: example

Other scheme `src` values and the corresponding [[ref: DID Log]] locations:

`ipns://k51qzi5uqu5d.../dids/issuer` --> `ipns://k51qzi5uqu5d.../dids/issuer/did.jsonl`

`ipfs://bafybei.../` --> `ipfs://bafybei.../did.jsonl`

:::

::: note
Some schemes name immutable content. An `ipfs` URL names a directory by the
hash of its content, so any change to `did.jsonl` or `did-witness.json` —
including witness proofs added after an entry — gives the directory a new
identifier. An `ipfs` reference is therefore a snapshot of the
[[ref: DID Log]], useful for archiving but never updated; an `ipns` name,
which the [[ref: DID Controller]] repoints to each new directory, can serve
the latest log. See [Log Freshness and
Duplicity](#log-freshness-and-duplicity).
:::

##### Resolution Algorithm

A `did:vh` [[ref: Resolver]] **MUST** carry out the following steps, which
apply the Read (Resolve) algorithm defined in [[ref: VH-Log]] with
`did:vh`-specific additions.

1. The `did` input **MUST** conform to the ABNF in [Method-Specific
   Identifier](#method-specific-identifier); if it does not, return the
   `invalidDid` error.
2. Validate the `resolutionOptions` as defined in [`did:vh` Resolution
   Options](#didvh-resolution-options). If they are invalid, return the
   `invalidOptions` error.
3. Get the [[ref: DID Log]] according to how it was passed:
   - **Directly:** the `didLog` value.
   - **By reference:** retrieve it as defined in [The `src`
     Option](#the-src-option). If the [[ref: DID Log]] is not found, return
     the `notFound` error.

   The witness proofs file is needed only if the active [[ref: parameters]]
   require [[ref: witnesses]]. When it is, use the `didWitness` value if the
   log was passed directly, or retrieve the file from the same `src`
   reference as defined in [The `src` Option](#the-src-option).
4. Process the [[ref: DID Log]] with the VH-Log Read (Resolve) algorithm,
   with the following `did:vh`-specific steps:
   - **Parameters (VH-Log step 1):** `parameters` **MUST** adhere to [`did:vh`
     DID Method Parameters](#didvh-did-method-parameters).
   - **Witness verification (VH-Log step 2.1):** If [[ref: witnesses]] are
     active, verify the witness proofs as defined in [Witnesses](#witnesses).
   - **State verification (VH-Log step 6):** For every entry, `state.id`
     **MUST** be exactly `did:vh:` followed by the `scid` [[ref: parameter]]
     of the first entry, and that [[ref: SCID]] **MUST** equal the
     [[ref: SCID]] of the `did` input.

   As defined in [[ref: VH-Log]], an entry that fails verification makes it
   and all later entries invalid, while earlier entries remain valid. If the
   first entry is invalid, return the `invalidDid` error.
5. Select the version to return: the one identified by the `versionId`,
   `versionTime` or `versionNumber` option if given, or otherwise the latest
   valid version. If the requested version does not exist or is invalid,
   return the `notFound` or `invalidDid` error respectively.
6. Return the [[ref: DIDDoc]] for that version (unless the DID is
   deactivated, as defined in [Deactivate (Revoke)](#deactivate-revoke)) and
   the metadata defined below.

[[ref: Resolvers]] **SHOULD NOT** cache a log that fails verification.

##### DID Resolution Metadata

As defined in [[spec:DID-RESOLUTION]], a `did:vh` [[ref: Resolver]]
**SHOULD** return the following DID Document Metadata:

```json
{
  "versionId": "1-QmRRaLXwc6BjBuBPosSupJwEQ8w9f3znP7yfbpGfwcnLr6",
  "versionTime": "2025-01-23T04:12:36Z",
  "created": "2025-01-23T04:12:36Z",
  "updated": "2025-01-23T04:12:36Z",
  "scid": "QmPEQVM1JPTyrvEgBcDXwjK4TeyLGSX1PxjgyeAisdWM1p",
  "deactivated": false,
  "ttl": "3600",
  "witness": { ... },
  "watchers": [ ... ]
}
```

The items are taken from the VH-Log [Resolution
Result](../next/index.html#resolution-result), as follows:

- `versionId` — The `versionId` of the [[ref: DID log entry]] of the resolved
  [[ref: DIDDoc]] version.
- `versionTime` — The `versionTime` of the [[ref: DID log entry]] of the
  resolved [[ref: DIDDoc]] version, as an [[ref: ISO8601]] timestamp.
- `created` — The `versionTime` of the first [[ref: DID log entry]], as an
  [[ref: ISO8601]] timestamp: when, according to the
  [[ref: DID Controller]], the DID was created.
- `updated` — The `versionTime` of the last valid [[ref: DID log entry]], as
  an [[ref: ISO8601]] timestamp.
- `scid` — The [[ref: SCID]] of the DID.
- `deactivated` — A boolean that is `true` if the DID has been deactivated,
  as defined in [Deactivate (Revoke)](#deactivate-revoke).
- `ttl` — A string containing the unsigned integer value, in seconds, of the
  active `ttl` [[ref: parameter]]: guidance from the [[ref: DID Controller]]
  on how long to cache the resolution result.
- `witness` — The active `witness` [[ref: parameter]] object, as defined in
  VH-Log's [The `witness` Parameter](../next/index.html#the-witness-parameter) section, if
  [[ref: witnesses]] are active. Its `threshold` value is a string containing
  the integer value.
- `watchers` — The active list of [[ref: watcher]] URLs from the `watchers`
  [[ref: parameter]].

"Active" means as set by the last valid [[ref: DID log entry]]. That is the
last entry in the [[ref: DID Log]] unless later entries fail verification, in
which case it is the last entry before the first failure.

`ttl` and the `witness` `threshold` are strings, not integers, because
[[spec:DID-RESOLUTION]] does not permit integers in DID metadata; a client
converts them to integers before use. `updated`, and the values marked
active, reflect the [[ref: DID Log]] passed to the [[ref: Resolver]], which
may not be the latest (see [Log Freshness and
Duplicity](#log-freshness-and-duplicity)).

When a DID resolution error occurs, the `error` field **MUST** be included in
the `didResolutionMetadata`, as defined in [[spec:DID-RESOLUTION]], and
[[ref: Resolvers]] **SHOULD** include `problemDetails` following
[[spec:rfc9457]]. The following `error` values, from
[[spec:DID-EXTENSION-RESOLUTION]], **MUST** be used:

- `invalidDid` — The DID is malformed, or the [[ref: DID Log]] passed to the [[ref: Resolver]] (or
  the requested version) fails verification.
- `invalidOptions` — A resolution option is malformed or not permitted,
  including a local directory or loopback reference passed to a
  [[ref: Resolver]] that accepts requests from other systems.
- `notFound` — The [[ref: DID Log]] was not found through the `src`
  reference, the requested version does not exist, or the resource referenced
  by a DID URL was not found.
- `featureNotSupported` — The [[ref: Resolver]] declined to retrieve from a
  `src` reference, does not support the scheme of a `src` reference, or does
  not support `versionNumber`.

##### Reading `did:vh` DID URLs

The `src`, `versionId`, `versionTime` and `versionNumber` options can also be
given as DID URL query parameters, such as
`did:vh:<SCID>?src=https://example.com/dids/issuer&versionNumber=2`. `didLog` and
`didWitness` are resolution options only, and **MUST NOT** be used as DID URL
query parameters.

When a `did:vh` DID URL is dereferenced, the DID URL dereferencer (the client
of the [[ref: Resolver]]) decides which of these parameters to pass to
`resolve` as resolution options:

- It **SHOULD** pass a `versionId`, `versionTime` or `versionNumber` DID URL
  parameter as the resolution option of the same name. These select a version
  as defined in VH-Log's [Selecting a
  Version](../next/index.html#selecting-a-version) section.
- A `src` DID URL parameter **MUST** be percent-encoded as required for a
  query parameter value by [[spec:RFC3986]] ([Section
  3.4](https://www.rfc-editor.org/rfc/rfc3986#section-3.4) and, for
  characters that conflict with delimiters such as `&` and `=`, [Section
  2.2](https://www.rfc-editor.org/rfc/rfc3986#section-2.2)), and a DID URL **MUST NOT**
  include more than one. The dereferencer **MUST** percent-decode it once and
  validate it as defined in [The `src` Option](#the-src-option), and **MUST**
  return the `invalidDidUrl` error if it is not valid.
- A `src` DID URL parameter **MUST NOT** be a [local
  directory](#local-directories) or [loopback](#web-locations) reference. The
  dereferencer **MUST** return the `invalidDidUrl` error for such a
  reference, and **MUST NOT** pass it to the [[ref: Resolver]].
- The dereferencer **MAY** apply its own policy to a valid `src` value — for
  example, using it only if it names a location the dereferencer trusts, or
  ignoring it in favour of a copy of the [[ref: DID Log]] it already has. If it
  uses the `src` value, it passes it as the `src` resolution option, or
  retrieves the files itself and passes them directly in `didLog` and
  `didWitness`.

The `src` parameter never changes which version is returned. As a DID URL
query parameter, `src` is equivalent to the `src` parameter of
[`did:scid`](https://lf-toip.atlassian.net/wiki/spaces/HOME/pages/88572360/DID+SCID+Method+Specification)
when its value is an `https` URL.

#### Update (Rotate)

Updating a `did:vh` DID follows the Update algorithm defined in
[[ref: VH-Log]], with the following `did:vh`-specific constraints:

- **State changes (VH-Log Update step 1):** Changes are made to the
  [[ref: DIDDoc]]. The top-level `id` **MUST** remain `did:vh:<SCID>`.
- **Parameters:** `parameters` **MUST** follow [`did:vh` DID Method
  Parameters](#didvh-did-method-parameters).
- **Witnesses:** If [[ref: witnesses]] are active, the [[ref: DID Controller]]
  **MUST** collect the threshold of witness proofs, and make the updated
  witness proofs file available, **before** distributing the updated
  [[ref: DID Log]].
- **Distribution:** The [[ref: DID Controller]] **SHOULD** make the updated
  [[ref: DID Log]] available through every [[ref: DID Log source]] where it
  previously made the log available: updating each web location, notifying
  [[ref: watchers]] (see [Watchers](#watchers)), and sending it to peers.

##### Applying Peer-to-Peer Updates

When [[ref: DID Logs]] are exchanged [peer-to-peer](#peer-to-peer-exchange),
each party holds the only copies of the other's log, and an update may carry
only the new entries. A party that receives new [[ref: DID log entries]] for a
[[ref: DID Log]] it holds **MUST** apply them as follows:

1. Skip any received entry whose `versionId` is identical to that of an entry
   it already holds. If a received entry has the same version number as a held
   entry but a different `versionId`, the update is duplicitous and **MUST**
   be rejected.
2. Append the remaining entries, in order, to a copy of the held
   [[ref: DID Log]].
3. Resolve the DID with that copy as `didLog`, the received witness proofs
   file (or, if none was received, the held one) as `didWitness`, and the
   `versionId` of the last appended entry as `versionId`.
4. If resolution succeeds, replace the held [[ref: DID Log]] and witness proofs
   file with the ones used in step 3. Otherwise, reject the update and keep the
   held files unchanged.

A party **SHOULD** keep a rejected update as evidence of duplicity.

No second copy of the log is needed to detect an update that does not extend
the held log. The [[ref: entry hash]] of each entry is calculated with the
`versionId` of the previous entry as input, so an appended entry verifies only
if it continues from the last held entry. Requesting the `versionId` of the
last appended entry in step 3 ensures that resolution fails, rather than
returning an earlier version, if any appended entry is invalid.

#### Deactivate (Revoke)

A `did:vh` DID is deactivated as defined in [[ref: VH-Log]]'s
[Deactivate](../next/index.html#deactivate) section, by adding
`"deactivated": true` to the [[ref: parameters]] of a new
[[ref: DID log entry]]. Once a DID is deactivated, a [[ref: Resolver]]
**MUST NOT** return the [[ref: DIDDoc]] and **MUST** include
`"deactivated": true` in the DID Document Metadata, as defined in
[[spec:DID-CORE]]. Prior versions **MAY** be resolved with the version query
parameters, and the metadata for them **MUST** include `"deactivated": true`.

As defined in VH-Log's [Deactivate](../next/index.html#deactivate) section, a
[[ref: DID Controller]] can instead set `updateKeys` to `[]` without setting
`deactivated`, so that the final [[ref: DIDDoc]] is still returned but cannot
be updated.

Unlike `did:webvh`, removing a published [[ref: DID Log]] does not effectively
deactivate a `did:vh` DID, because copies may be held by [[ref: watchers]],
peers, and other sources. A [[ref: DID Controller]] that wants to retire a DID
**SHOULD** deactivate it explicitly and distribute the final log, so that all
sources report the deactivation. See [Log Freshness and
Duplicity](#log-freshness-and-duplicity) for why a [[ref: Resolver]] may not
see the deactivation immediately.

### Sourcing the DID Log

*This section is non-normative.*

A `did:vh` [[ref: Resolver]] resolves the [[ref: DID Log]] it is given. Before
calling `resolve`, the client must therefore *source* the [[ref: DID Log]]: it
either obtains the [[ref: DID Log]] and witness proofs themselves, which it
passes directly in `didLog` and `didWitness`, or learns a reference to where
they are stored, which it passes in `src`. In this specification, "source"
covers both outcomes, while "obtain" and "retrieve" mean getting the files
themselves. How the client
sources the log depends on how it received the DID. Each place it can source
the [[ref: DID Log]] from is a [[ref: DID Log source]]; this section describes
the common ones. They can be combined, and a [[ref: DID Log]] sourced from any
of them is verified by the [[ref: Resolver]] in exactly the same way.

#### Known in Context

In many uses, a client already has the [[ref: DID Log]], or knows where to get
it, because of the context in which it received the DID. For example:

- **A log sent with the DID.** A protocol that carries a DID — such as a
  credential or presentation exchange — also carries the [[ref: DID Log]] and
  witness proofs, or a web location for them. The client passes them
  directly, or passes the web location by reference in `src`.
- **An ecosystem DID host.** An ecosystem requires its participants to publish
  their `did:vh` [[ref: DID Logs]] to a registry or database it operates. The
  client retrieves the files from the host and passes them directly, or
  passes the host's web location by reference in `src`.
- **A previously resolved log.** The client kept the files from an earlier
  resolution, and passes them directly, or by reference to the local
  directory where it keeps them — perhaps after checking another source for
  newer entries.
- **A DID URL with a `src` parameter.** The client received a DID URL such as
  `did:vh:<SCID>?src=https://example.com/dids/issuer`, and uses the location as
  described in [Reading `did:vh` DID URLs](#reading-didvh-did-urls).

#### From Watchers

A [[ref: watcher]] retrieves, verifies, archives and re-serves
[[ref: DID Logs]], indexed by [[ref: SCID]]. Because a `did:vh` DID is its
[[ref: SCID]], a client that knows of a [[ref: watcher]] holding the DID —
for example, one its ecosystem uses — can retrieve:

- the [[ref: DID Log]] with **GET `<WATCHER URL>/log?scid=<SCID>`**, and
- if [[ref: witnesses]] are active, the witness proofs file with
  **GET `<WATCHER URL>/witness?scid=<SCID>`**.

These are the operations defined in [VH-Log's Watcher HTTP API
Operations](../next/index.html#watcher-http-api-operations). The client then
passes the files directly, or stores them in a local directory and passes
that by reference in `src`.

Once a client has resolved the DID, the `watchers` metadata lists the
[[ref: watchers]] the [[ref: DID Controller]] has chosen. A client can use them
to check for newer entries, for example when the copy it holds may be out of
date. A client is not limited to those [[ref: watchers]]: any party can run
its own [[ref: watcher]], or use one its ecosystem operates, and send it the
[[ref: DID Logs]] it relies on. See [Log Freshness and
Duplicity](#log-freshness-and-duplicity).

#### Peer-to-Peer Exchange

Two or more parties can use `did:vh` DIDs with one another by exchanging
[[ref: DID Logs]] directly, in the manner of `did:peer` [[spec:DID-PEER]], without publishing
them anywhere. Unlike `did:peer`, the DIDs keep all the VH-Log features,
including key rotation and pre-rotation. Each party keeps the other's
[[ref: DID Log]] (and witness proofs file, if [[ref: witnesses]] are active),
and its client passes them directly, or by reference to the [local
directory](#local-directories) where it keeps them.

**Initial exchange.** The [[ref: DID Controller]] gives the other parties the
complete [[ref: DID Log]] from the first entry, and the witness proofs file if
[[ref: witnesses]] are active.

**Updates.** When the DID is updated, the [[ref: DID Controller]] sends each
party either the complete [[ref: DID Log]] or just the new
[[ref: DID log entry]]. Sending just the new entry keeps updates small, and
works when each party receives every update in turn; a party that has missed
an update cannot apply it, and needs the complete [[ref: DID Log]]. If the new
entry must be [[ref: witnessed]], the [[ref: DID Controller]] also sends the
complete current witness proofs file; otherwise that is optional. The
protocol used to send updates is outside the scope of this specification.

**Applying an update.** A party receiving new entries applies them as defined
in [Applying Peer-to-Peer Updates](#applying-peer-to-peer-updates). Because
each [[ref: entry hash]] chains to the previous entry, this rejects any entry
that does not continue from the log the party holds, without needing a second
copy of the log.

A `did:vh` DID used only peer-to-peer can be created for each relationship, so
that no identifier is shared between relationships.

#### From a Web Location

A client that learns a web location where the [[ref: DID Controller]]
publishes the [[ref: DID Log]] can pass it by reference in the `src` option, or retrieve
`did.jsonl` and `did-witness.json` itself and pass them directly.
A client can learn such a location from a `src` DID URL query parameter, from
an `alsoKnownAs` entry in a previously resolved [[ref: DIDDoc]] (see
[Advertising DID Log Sources](#advertising-did-log-sources)), or from the
context in which it received the DID.

#### Advertising DID Log Sources

A [[ref: DID Controller]] can tell others where current copies of the
[[ref: DID Log]] are kept:

- The `watchers` [[ref: parameter]] lists [[ref: watchers]] that archive the
  log.
- The [[ref: DIDDoc]] `alsoKnownAs` property can include `did:vh` DID URLs for
  the same DID with a `src` parameter, one for each location where the
  [[ref: DID Log]] is published (a web location, or another scheme such as
  `ipns`), as in `did:scid`. Each location listed this
  way is expected to hold the same [[ref: DID Log]], although some may briefly
  lag others while an update is distributed.

```json
"alsoKnownAs": [
  "did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ?src=https://example.com/dids/issuer",
  "did:vh:QmfGEUAcMpzo25kF2Rhn8L5FAXysfGnkzjwdKoNPi615XQ?src=https://backup.example.org/{SCID}"
]
```

Both are available only after the DID has been resolved at least once, so
they serve to check for newer entries rather than to source the log in the
first place.

#### Log Freshness and Duplicity

A `did:vh` [[ref: DID Log]] has no single authoritative location, so a client
faces two risks that `did:webvh` handles by trusting the DID's web location:

- **Staleness.** A source may hold an older, shorter copy of the log, and so
  miss updates such as a key rotation or deactivation. A [[ref: Resolver]]
  verifies the log it is given, but cannot know whether newer entries exist
  elsewhere.
- **Duplicity.** A [[ref: DID Controller]], or an attacker holding its keys,
  could create two different extensions of the same log and show different
  ones to different parties.

For two verified copies of a [[ref: DID Log]] for the same DID, log B
**extends** log A if B contains every entry of A, in the same positions, with
the same `versionId`s. Since each `versionId` includes a hash chained to the
previous entry, it is enough to check that B contains an entry with the
`versionId` of A's last entry, at the same position.

A client that needs the current version of a DID can reduce the risk of
staleness by sourcing the log from a source expected to be current — such as
a [[ref: watcher]] listed in the log — rather than relying only on a cached
copy or a copy supplied by the party presenting the DID. The `ttl` metadata
indicates how long the [[ref: DID Controller]] expects a copy to stay current.

A client that obtains copies from more than one source can compare them. If
one extends the other, the longer is the more current. If neither extends the
other, the log is **duplicitous**: the client should not rely on any version
after the last entry the copies have in common, and may keep the conflicting
entries as evidence and report them, for example to [[ref: watchers]].

[[ref: Witnesses]] and [[ref: watchers]] address duplicity in different ways:

- **[[ref: Witnesses]]** are expected to approve only an entry that extends
  their own copy of the log, as defined in VH-Log's [Witnessing a Log Entry
  Update](../next/index.html#witnessing-a-log-entry-update). If they do, two
  conflicting entries can both be [[ref: witnessed]] only if a threshold of
  [[ref: witnesses]] colludes or is compromised. However, a witness proof
  shows only that the [[ref: witness]] approved the entry: no one else can
  confirm that the [[ref: witness]] kept its own copy of the log or checked
  the entry against it. How far [[ref: witnesses]] reduce duplicity therefore
  depends on how far they are trusted, which is a matter for ecosystem
  governance and outside the scope of this specification.
- **[[ref: Watchers]]** detect duplicity directly. A [[ref: watcher]] that
  retrieves the [[ref: DID Log]] from each source it knows of, and checks
  that each copy extends the one it holds, finds conflicting copies without
  relying on the [[ref: DID Controller]] or the [[ref: witnesses]]. The cost
  is retrieving and verifying the complete log on each check, and a
  [[ref: watcher]] can detect only the copies it sees: a conflicting copy
  given privately to one party is found only when that party shares it, for
  example by sending it to a [[ref: watcher]]. Any party can run its own
  [[ref: watcher]]; it need not be one of the [[ref: watchers]] the
  [[ref: DID Controller]] lists in the log. A relying party that runs its own,
  or uses one its ecosystem operates, gets a check that is independent of the
  [[ref: DID Controller]].

[[ref: DID Controllers]] of `did:vh` DIDs that are relied on by many parties
are encouraged to use [[ref: witnesses]] and [[ref: watchers]], and parties
relying on such DIDs are encouraged to use [[ref: watchers]] they trust.

Selecting a version with `versionNumber` is less safe than selecting it with
`versionId`. A `versionId` includes the [[ref: entry hash]], so it identifies
exactly one [[ref: DID log entry]], whichever copy of the log is resolved. A
`versionNumber` identifies an entry only within the copy being resolved: if
the log is duplicitous, `did:vh:<SCID>?versionNumber=3` can resolve to
a different [[ref: DIDDoc]] for different parties, while
`did:vh:<SCID>?versionId=3-Qm...` resolves to one [[ref: DIDDoc]] or fails. A
client recording which version of a DID it relied on — for example, the key
that verified a signature — is better served by `versionId`.

### DID Method Processes

#### `did:vh` DID Method Parameters

`did:vh` uses the [[ref: VH-Log]]
[parameters](../next/index.html#vh-log-parameters) mechanism, including its
general rules. The parameters `scid`, `updateKeys`, `nextKeyHashes`, `witness`,
`watchers`, `deactivated`, and `ttl` are used exactly as defined in
[[ref: VH-Log]], with one `did:vh`-specific constraint: `witness` `id` values
**MUST** be `did:key` DIDs (see [Witnesses](#witnesses)). The `parameters`
object **MUST NOT** include any other properties, except those listed below.

::: example
The `parameters` property in the first [[ref: DID log entry]]:

```json
{
  "method": "did:vh:1.0",
  "scid": "{SCID}",
  "updateKeys": [
    "z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"
  ],
  "nextKeyHashes": [
    "enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"
  ]
}
```
:::

- `method`: The `did:vh` form of VH-Log's `logVersion` parameter, with the
  same rules: it **MUST** appear in the first entry, **MAY** appear in later
  entries to upgrade to a later version, and a value that is not exactly one of
  the acceptable values **MUST** cause resolution to terminate. `logVersion`
  itself **MUST NOT** be used. Acceptable values:
  - `did:vh:1.0` — corresponds to VH-Log `vh-log:1.0`, and permits the same
    hash algorithm (`SHA-256` only) and [[ref: Data Integrity]] cryptosuite
    (`eddsa-jcs-2022` [[spec:di-eddsa-v1.0]] only) for both log-entry and
    witness proofs.

`did:vh` does not use the `did:webvh` `portable` [[ref: parameter]]. An entry
that includes it **MUST** be rejected.

#### Cryptographic Agility

`did:vh` inherits cryptographic agility from [[ref: VH-Log]] unchanged, with
the `method` [[ref: parameter]] in the role of `logVersion`.

#### SCID Generation and Verification

The [[ref: SCID]] is generated and verified exactly as defined in VH-Log's
[SCID Generation and
Verification](../next/index.html#scid-generation-and-verification) section.
The preliminary log entry is the one described in [Create
(Register)](#create-register), in which the `{SCID}` placeholder appears in
the DID string `did:vh:{SCID}` wherever it is used, and in `parameters.scid`.

#### Entry Hash Generation and Verification

The [[ref: entry hash]] is generated and verified exactly as defined in
VH-Log's [Entry Hash Generation and
Verification](../next/index.html#entry-hash-generation-and-verification)
section.

#### Authorized Keys and Pre-Rotation

The authorized keys mechanism is as defined in VH-Log's [Authorized
Keys](../next/index.html#authorized-keys) section, and key pre-rotation as
defined in its [Pre-Rotation Key Hash Generation and
Verification](../next/index.html#pre-rotation-key-hash-generation-and-verification)
section. The proof `cryptosuite` **MUST** be one permitted by the active
`method` [[ref: parameter]]; [[ref: Resolvers]] **MUST NOT** accept a
structurally valid signature using any other cryptosuite.

A `did:vh` log has no location whose control an attacker would also need in
order to publish an update (see [No Authoritative
Location](#no-authoritative-location)), so [[ref: DID Controllers]] **SHOULD**
use [[ref: pre-rotation]].

#### Witnesses

`did:vh` uses the [[ref: witness]] mechanism defined in VH-Log's
[Witnesses](../next/index.html#witnesses) section, with the same
identifier and key rules as `did:webvh`:

- Each `witness` `id` **MUST** be a [[ref: did:key]] DID whose key is
  compatible with a cryptosuite permitted by the active `method`. A `witness`
  [[ref: parameter]] containing any other `id` **MUST** be rejected when the
  parameter is validated.
- A witness proof's `verificationMethod` **MUST** be a `did:key` DID URL of the
  form `did:key:<multibase>#<multibase>`, with the two multibase values
  byte-for-byte equal. The verification key **MUST** be recovered only by
  decoding the `did:key`; [[ref: Resolvers]] **MUST NOT** resolve the witness
  DID or consult any other key store.
- The witness proofs file is `did-witness.json`, with the data model defined in
  VH-Log's [Witness Proofs File](../next/index.html#the-witness-proofs-file)
  section.

The witness proofs file reaches the [[ref: Resolver]] in the same way as the
[[ref: DID Log]]: passed directly in `didWitness`, or retrieved from the
location the `src` reference names. When passing the files directly, a client can obtain the
witness proofs file from a different [[ref: DID Log source]] than the
[[ref: DID Log]] — for example, from a [[ref: watcher]]'s `/witness`
operation.

Witnesses matter more for `did:vh` than for `did:webvh`. With no single
location for the log, an attacker holding the update keys can distribute an
update through any source; with [[ref: witnesses]] active, the update also
needs a threshold of witness proofs. [[ref: DID Controllers]] **SHOULD** use
[[ref: witnesses]] for DIDs relied on by many parties. Their value against
[duplicity](#log-freshness-and-duplicity) depends on how far the
[[ref: witnesses]] are trusted, as described there.

#### Watchers

`did:vh` uses the [[ref: watcher]] mechanism defined in VH-Log's
[Watchers](../next/index.html#watchers) section, including its HTTP API, which
the `did:webvh` [Watcher OpenAPI
definition](https://raw.githubusercontent.com/decentralized-identity/didwebvh/refs/heads/main/watcherOpenAPI/watcher-v1.0.0.yml)
describes. For `did:vh`, [[ref: watchers]] are also a primary [[ref: DID Log
source]], as described in [From Watchers](#from-watchers). Any party can run a
[[ref: watcher]] for a `did:vh` DID, whether or not it is listed in the
`watchers` [[ref: parameter]]. When resolving a `did:vh` DID,
[[ref: Resolvers]] **MUST** return the active list of [[ref: watchers]] in the
DID Document Metadata.

VH-Log's notification operation, **POST `<WATCHER URL>/log?id=<identifier>`**,
asks the [[ref: watcher]] to retrieve the latest log from its location. A
`did:vh` DID has no location, so for `did:vh`:

- **POST `<WATCHER URL>/log?id=<DID URL>`**, where the DID URL includes a
  `src` query parameter, asks the [[ref: watcher]] to retrieve the log (and witness
  proofs file) from that location. As for any DID URL, the `src` value
  cannot be a local directory or loopback reference.
- **POST `<WATCHER URL>/log?id=<DID>`**, where there is no `src` parameter,
  **MUST** carry the complete [[ref: DID Log]] as the request body, with media
  type `text/jsonl`. If [[ref: witnesses]] are active, the
  [[ref: DID Controller]] **MUST** first send the witness proofs file with
  **POST `<WATCHER URL>/witness?id=<DID>`**, with media type
  `application/json`.

In both cases the [[ref: watcher]] **MUST** fully verify the log, **MUST**
confirm that it extends any copy the [[ref: watcher]] already holds (see [Log
Freshness and Duplicity](#log-freshness-and-duplicity)), and indexes it by
[[ref: SCID]].
