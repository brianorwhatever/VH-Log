## Security Considerations

*This section is non-normative.*

This section follows the guidelines in [[spec:RFC3552]] and aligns with the
[[spec:DID-CORE]] requirements in [DID Core
7.3](https://www.w3.org/TR/did-core/#security-requirements). The security
considerations in [VH-Log's Security
Considerations](../next/index.html#security-considerations) section apply to
`did:vh` unchanged, except as noted below. The sections below cover only what
differs for `did:vh`. The requirements referred to here are defined in the
normative sections of this specification and of VH-Log.

### No Authoritative Location

A `did:webvh` [[ref: Resolver]] treats the DID's web location as the
authoritative place to find the current [[ref: DID Log]]. A `did:vh` DID has
no such location. The integrity of any copy of the log is fully protected by
the [[ref: SCID]], the hash chain and the proofs, so a [[ref: Resolver]] can
safely accept a log from any source. What a [[ref: Resolver]] cannot know from
the log alone is whether it is *complete* — whether newer entries exist.

The resulting threats, and their mitigations, are:

- **Withholding and truncation.** A source, or the party presenting a DID,
  supplies an older copy of the log, for example one from before a key
  rotation or deactivation. A [[ref: Resolver]] verifies the log it is given
  but cannot detect this. Mitigation: clients that need the current version
  source the log from a source expected to be current, such as a listed
  [[ref: watcher]], and never treat the absence of newer entries at one source
  as proof that none exist. See [Log Freshness and
  Duplicity](#log-freshness-and-duplicity).
- **Duplicity (split view).** A [[ref: DID Controller]], or an attacker with
  its update keys, creates two different extensions of one log and gives them
  to different parties. Mitigation: clients that obtain copies from several
  sources compare them and do not rely on versions past the point of
  divergence; peers apply updates so that non-extending entries are rejected;
  [[ref: witnesses]] are expected to refuse an entry that does not extend the
  log they hold, although others cannot verify that they do;
  [[ref: watchers]] — including ones run by relying parties — detect and can
  report conflicting logs.
- **Compromised keys and no location to reclaim.** In `did:webvh`, control of
  the web location is a second factor an attacker who steals the update keys
  also needs in order to publish. A `did:vh` log has no such location, so an
  attacker with the update keys can distribute a valid update through any
  source. Mitigation: [[ref: DID Controllers]] are expected to use
  [[ref: pre-rotation]] (see [Authorized Keys and
  Pre-Rotation](#authorized-keys-and-pre-rotation)), and to use
  [[ref: witnesses]] for DIDs relied on by many parties, so that an update
  also needs the approval of a threshold of [[ref: witnesses]] (see
  [Witnesses](#witnesses)).

### The `src` Option and Other Client-Supplied Values

The `src` resolution option, and a `src` DID URL query parameter, are supplied
by the client or by whoever gave it the DID URL, and so are untrusted input.
The same is true of a `didLog` value, which is why it is verified in full:

- **Server-side request forgery.** A web location `src` value can name any
  host. [[ref: Resolvers]] apply to web location retrievals the transport
  requirements in VH-Log's [Retrieving Log
  Resources](../next/index.html#retrieving-log-resources) section (see [Web
  Locations](#web-locations)), including rejecting IP literals and not
  following redirects automatically. A [[ref: Resolver]] can also decline to
  retrieve from `src` references at all (see [`did:vh` Resolution
  Options](#didvh-resolution-options)), or allow only locations permitted by
  its configuration. The same hardening applies to a client retrieving from a web location or
  a [[ref: watcher]] itself. The one exception is a loopback (`http`)
  reference, which is accepted only by a [[ref: Resolver]] local to its
  client and never from a DID URL; see [Web Locations](#web-locations). A
  [[ref: Resolver]] supporting [other schemes](#other-schemes) applies
  equivalent protections to those retrievals and to any gateway it uses.
- **Stale or wrong logs.** A web location may serve an old copy of the log,
  or a log for a different DID. The first is covered under [No Authoritative
  Location](#no-authoritative-location); the second is detected by the
  [[ref: SCID]] check that the [Resolution
  Algorithm](#resolution-algorithm) applies to every entry.
- **Local file and service access.** A local directory or loopback `src` reference is accepted only
  by a [[ref: Resolver]] local to its client; a [[ref: Resolver]] that accepts
  requests from other systems never reads local files or retrieves from
  loopback hosts, and a `src` DID URL parameter is never a local directory or
  loopback reference. A local [[ref: Resolver]]
  reads only `did.jsonl` and `did-witness.json` from the named directory. See
  [Local Directories](#local-directories).
- **Misleading association.** The domain in a `src` value says nothing about
  who controls the DID. A [[ref: DID Controller]] can publish a log at any
  location it can write to, and anyone can put any `src` value in a DID URL.
  For this reason, [[ref: Resolvers]] and their clients do not treat a web
  location as evidence of an association between the DID and the owner of
  the domain (see [The `src` Option](#the-src-option)).

### Endpoint Authentication

Retrievals from web locations and from [[ref: watchers]] over HTTP use HTTPS
with server authentication, as described in VH-Log's [Endpoint
Authentication](../next/index.html#endpoint-authentication) section. The only
exception is a retrieval from a loopback (`http`) reference by a
[[ref: Resolver]] local to its client. As with
all sources, a [[ref: DID Log]] is verified by its content, not by the
authentication of the server that provided it.

### Peer-to-Peer Use

When [[ref: DID Logs]] are exchanged peer-to-peer, each party holds the only
copies of the other's log, and an update may carry only the new entry. With no
other copy to compare against, the hash chain is what protects a party: it
applies an update as defined in [Applying Peer-to-Peer
Updates](#applying-peer-to-peer-updates), so that entries that do not continue
from the log it holds are rejected, and keeps a rejected update as evidence of
duplicity.

### Post Quantum Attacks

As defined in [VH-Log's Post Quantum
Attacks](../next/index.html#post-quantum-attacks) section.
[[ref: Pre-rotation]] is especially valuable for `did:vh`, for the reason
given under [No Authoritative Location](#no-authoritative-location).

### Resolver Validation Checklist

This checklist extends [VH-Log's Resolver Validation
Checklist](../next/index.html#resolver-validation-checklist) with
`did:vh`-specific points. It does not restate or replace the normative
requirements.

- **DID syntax:** the `did` input matches the `did:vh` ABNF.
- **Resolution options:** `didLog` is a string, `didWitness` (only with
  `didLog`) a JSON array; a `src` value is an absolute URL with no query or
  fragment that is either a valid web location — `https` with a domain name
  host (or `http` with a loopback host), an optional port, and valid path
  segments — a valid local directory — a `file:` URL with an absolute path
  and an empty or `localhost` host — or a URL with another scheme, which
  returns `featureNotSupported` if the [[ref: Resolver]] does not support
  it; at most one version option; malformed options return
  `invalidOptions`. A `didLog` value is verified in full, like any other copy.
  Local directory and loopback references are accepted only by a
  [[ref: Resolver]] local to its client, and never from a DID URL.
- **Parameters:** `method` is an acceptable `did:vh` value; `logVersion` and
  `portable` are absent.
- **SCID and identity:** every entry's `state.id` is exactly `did:vh:` plus the
  first entry's `parameters.scid`, which equals the [[ref: SCID]] in the DID
  being resolved.
- **Witnesses:** as defined in [Witnesses](#witnesses) — `did:key` witness
  identifiers, keys recovered only from the `did:key`, body and fragment
  multibase values equal.
- **Passing the log:** exactly one of `didLog` and `src`; `didWitness` only with
  `didLog`; a declined `src` returns `featureNotSupported`.

## Privacy Considerations

*This section is non-normative.*

This section addresses the privacy considerations in alignment with
[[spec:RFC6973]] Section 5, and aligns with the [[spec:DID-CORE]] requirements
in [DID Core 7.4](https://www.w3.org/TR/did-core/#privacy-requirements). The
privacy considerations in [VH-Log's Privacy
Considerations](../next/index.html#privacy-considerations) section apply to
`did:vh` unchanged, except as noted below.

### Surveillance

Retrieving a [[ref: DID Log]] reveals the [[ref: Resolver]]'s interest in the
DID to the source it retrieves from: the host of a web location, or a
[[ref: watcher]]. Because `did:vh` lets a [[ref: Resolver]] choose its
sources, a [[ref: Resolver]] can reduce this by using a cached copy, a
[[ref: watcher]] it trusts, or a copy supplied by the party presenting the
DID, and by not retrieving web locations chosen by others. A
[[ref: Resolver]] or client can also retrieve through privacy-enhancing
network tools, such as Tor or a VPN, so that the source does not learn the
network address of the party interested in the DID.

### Correlation and Identification

A `did:vh` DID contains no domain name, so it does not by itself link its
controller to a domain owner. `src` values and `alsoKnownAs` entries can
reintroduce such a link, so [[ref: DID Controllers]] that want to avoid it are
advised not to publish them.

For peer-to-peer use, a [[ref: DID Controller]] can create a separate `did:vh`
DID for each relationship, so that the relationships cannot be correlated by
identifier.

### Right to Erasure ([GDPR Art. 17](https://gdpr-info.eu/art-17-gdpr/))

A `did:vh` [[ref: DID Log]] is likely to be held in several places — by
[[ref: watchers]], at web locations, and by peers. A [[ref: DID Controller]]
cannot remove every copy itself. The [[ref: watcher]] deletion operation
defined in VH-Log can be used to request removal from [[ref: watchers]]; how
such requests are handled is best defined by ecosystem governance.
