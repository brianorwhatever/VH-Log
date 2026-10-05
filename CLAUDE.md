# CLAUDE.md — vh-log-spec

## Purpose

This repository contains the **Verifiable History Log (VH Log)** specification. VH Log is a
general-purpose, append-only, cryptographically chained log structure for recording the history
of a versioned state object. It is a clean extraction and generalisation of the log mechanism
defined in the [did:webvh specification](https://identity.foundation/didwebvh/), with the goal
that did:webvh (and other specifications) can be defined as specialisations of VH Log.

The specification is not yet published. Work here is pre-standard and should be treated as a
working draft.

## Relationship to did:webvh

VH Log extracts the following from did:webvh and generalises them:

- The log entry structure (`versionId`, `versionTime`, `parameters`, `state`, `proof`)
- The cryptographic chaining mechanism (each entry's `versionId` is derived from a hash of
  the entry content, chaining entries together)
- The Self-Certifying Identifier (SCID) — derived from the genesis entry, embedded in the
  log's identifier, and used to verify log authenticity
- The witness mechanism (threshold of external proofs required per entry)
- The watcher role (parties that archive and re-serve logs)
- The resolution algorithm (deriving the current or historical `state` from log entries)
- The `parameters` mechanism (per-entry configuration such as `updateKeys`, `nextKeyHashes`,
  `witnesses`, `deactivated`, `ttl`)

Things that remain DID-specific and are NOT in VH Log:

- The resource naming convention `did.jsonl` — VH Log uses generic `vh-log.jsonl` or
  allows the specialisation to specify the resource name
- The `did-witness.json` resource name — specialisation-specific
- DID URL resolution (`versionTime`, `versionId` query parameters) — the resolution algorithm
  is defined in VH Log but DID URL syntax is defined in did:webvh
- The DIDDoc as the `state` type — in VH Log, `state` is an arbitrary JSON object

## Specification Tooling

This spec uses [Spec-Up](https://github.com/decentralized-identity/spec-up) v0.11.6 (npm,
official package — NOT the old `github:brianorwhatever/spec-up` fork) for rendering.

To render the spec locally:
```
npm install
npm run render
```

To watch for changes:
```
npm run edit
```

The render scripts are `render.mjs` and `edit.mjs` in the repo root (ESM, required by spec-up
v0.11.6). The `.npmrc` in the repo root sets `node-options=--dns-result-order=ipv4first` —
this is required on machines where IPv6 is broken; it is safe to leave in place on machines
where IPv6 works fine.

External references not in spec-up's bundled specref data are defined per spec in a
`spec_refs` array in `specs.json` (same entry format as the old fork's `external_specs`:
`{ "name": { href, title, rawDate, authors, status } }`). The local plugin `spec-refs.mjs`
merges them into the corpus used by `[[spec:NAME]]`. Do NOT use `external_specs` — in 0.11.6
it only fetches other Spec-Up pages for `[[xref:]]` terms, and fetching non-Spec-Up pages
produces huge jsdom CSS error dumps.

Rendered output for VH-Log goes to `next/`, for did:vh to `didvh-next/`, and for did:webvh to
`didwebvh-next/`. Specialisations link to VH-Log (and did:vh links to did:webvh) with relative
URLs such as `../next/index.html#<anchor>`.

## Repository Structure

```
spec/               # VH-Log specification source (Spec-Up Markdown)
  header.md
  abstract.md
  overview.md
  specification.md
  security_and_privacy.md
  definitions.md
  references.md
  version.md
spec-didwebvh/      # did:webvh specification source, reworked to reference VH-Log as a
                    # specialisation (de-duplication pass still pending)
  (same files)
spec-didvh/         # did:vh specification source (current first target)
  (same files)
next/               # Rendered HTML output for VH-Log (do not edit directly)
didvh-next/         # Rendered HTML output for did:vh (do not edit directly)
didwebvh-next/      # Rendered HTML output for did:webvh (do not edit directly)
spec-refs.mjs       # Spec-Up plugin adding each spec's `spec_refs` to [[spec:]] lookups
render.mjs          # ESM render script (node render.mjs)
edit.mjs            # ESM watch script (node edit.mjs)
specs.json          # Spec-Up configuration for all specs in this repo

# Planned, not yet created (see Planned updates below):
spec-eddsa-jcs-prerotation/  # eddsa-jcs-prerotation-2026 cryptosuite spec
```

## Key Concepts

### Log Entry Structure

Each entry in a VH Log contains:

```json
{
  "versionId": "<entry-hash>",
  "versionTime": "<ISO 8601 timestamp>",
  "parameters": { ... },
  "state": { ... },
  "proof": [ ... ]
}
```

- `versionId`: A hash of the entry content (excluding `proof`), base64url-encoded. For the
  genesis entry, this value is used to derive the SCID.
- `versionTime`: The claimed time of this entry. Monotonically increasing.
- `parameters`: Configuration that takes effect from this entry onward. See Parameters section.
- `state`: The versioned object at this point in the log. Type is defined by the specialisation.
- `proof`: One or more Data Integrity proofs from authorised update keys (and optionally
  witnesses).

### SCID

The Self-Certifying Identifier is derived from the `versionId` of the genesis entry (entry 0).
It is embedded in the log's canonical identifier and allows a verifier to confirm that a given
log is the authentic log for that identifier — not a substituted or forged alternative.

### Parameters

Parameters accumulate across entries (later entries override earlier ones). Key parameters:

- `updateKeys`: The set of keys authorised to sign subsequent entries
- `nextKeyHashes`: Pre-committed hashes of future update keys (enables key pre-rotation)
- `witnesses`: The set of witness DIDs and the required threshold
- `deactivated`: Boolean — if true, the log is deactivated and no further entries are valid
- `ttl`: Suggested cache duration for resolvers

### Resolution Algorithm

Given a log and a target version (by `versionTime` or `versionId`), the resolver:

1. Verifies the SCID against the genesis entry
2. Processes entries in order, verifying each entry's proof and hash chain
3. Accumulates parameters
4. Returns the `state` from the latest entry at or before the target version

### Witnesses and Watchers

- **Witnesses**: External parties that sign log entries, providing additional tamper-evidence.
  A threshold of witness proofs may be required per entry (governance-defined).
- **Watchers**: Parties that archive log copies and can serve them independently of the
  original publisher. Watcher copies are independently verifiable due to the cryptographic
  chain.

## Authoring Guidelines

- Write normative requirements using RFC 2119 terms (MUST, SHOULD, MAY).
- Security and Privacy Considerations sections are **non-normative** in all three specs
  (following DID Core §9/§10; changed 2026-10-05). Put every RFC 2119 requirement in the
  normative body and have the considerations sections describe threats and link to it.
  VH-Log's transport/TLS/CORS/SSRF rules live in "Publishing and Retrieving Log Resources".
- Each normative statement should be individually testable.
- Keep DID-specific content out of this spec — if something is DID-specific, note it as
  specialisation-defined behaviour.
- Test vectors must be provided for all cryptographic operations (SCID derivation, entry
  hashing, chain verification).
- Cross-reference did:webvh where alignment is intentional.

## Relationship to Other Specs in This Work

- **vp-vh-spec**: A specialisation of VH Log where `state` is a W3C Verifiable Presentation.
  Depends on this spec.
- **whois-vh-spec**: A specialisation of VP-VH where discovery is via `/.well-known/whois-vh`
  or a DIDDoc service entry. Depends on vp-vh-spec.
- **whois-vh** (implementation): TypeScript implementation of whois-vh resolution and
  management. Depends on all three specs.
- **eddsa-jcs-prerotation-2026** (planned, hosted in this repo): Data Integrity cryptosuite
  extending eddsa-jcs-2022 to automate VH-Log's mandatory key pre-rotation. See Planned updates.
- **did:vh** (hosted in this repo, `spec-didvh/`): Specialisation of VH-Log similar to
  did:webvh but with no domain/path component — the DID is `did:vh:<SCID>`. Comparable to the
  `did:scid:vh` format of the proposed ToIP `did:scid` metamethod.

## Current Work and Next Steps

### Work completed (session 2026-06-06)

- Migrated Spec-Up from the old GitHub fork to npm v0.11.6 (see Specification Tooling above)
- Fixed `specs.json`: removed `katex` (not used; packaging bug in spec-up 0.11.6 means fonts
  are missing), simplified `external_specs` to the new `{ "handle": "url" }` format
- Both specs now render cleanly via `npm run render`
- Renamed `spec-didwevh/` → `spec-didwebvh/` (corrected typo)

### Work completed (VH-Log generalisation)

- `spec/` has been reworked into the generalised VH-Log spec: DID-specific content stripped,
  `did.jsonl` replaced with `vh-log.jsonl`, DIDDoc references replaced with the generic `state`
  object, header status set to "Pre-Draft — Editors Draft v0.1"
- `spec-didwebvh/` has been reworked to frame did:webvh as a specialisation of VH-Log (see its
  `abstract.md`), though a full de-duplication pass against VH-Log is still needed (see Planned
  updates below)
- `README.md` is still stale — it describes the repo as did:webvh-only and does not mention
  VH-Log or the dual-spec structure. Needs updating.

### did:vh — current first target (started 2026-09-25)

did:vh replaced did:webvh as the first specialisation to build out. `spec-didvh/` v0.1 draft
exists and renders cleanly. Key design decisions in that draft (open to revision):

- DID is `did:vh:<SCID>` only; spec version lives in the `method` parameter (`did:vh:1.0` ↔
  `vh-log:1.0`), not in the DID string (unlike `did:scid:vh:1:<SCID>`).
- Log/witness file names `did.jsonl` / `did-witness.json`, same as did:webvh.
- **Resolve vs. source are separated.** Read (Resolve) is normative: a did:vh resolver
  implements DID Resolution's `resolve(did, resolutionOptions)` — `did` is the bare
  `did:vh:<SCID>`, and exactly ONE DID Log is passed per call in one of two ways: directly (`didLog` JSONL string + optional `didWitness` JSON array — content only) or by
  reference (`src`, naming a directory holding `did.jsonl`/`did-witness.json`: a web location
  (https) or a local directory (`file:`)). Plus `versionId`/`versionTime`/
  `versionNumber`. Options are did:vh-specific, to be registered in the DID Extensions
  resolution registry. Malformed options → `invalidOptions`; resolver must support logs passed
  directly and may decline `src` by policy → `featureNotSupported`. The resolver does NOT
  discover logs, use its own configured sources, or compare multiple copies (see TODO).
- "Sourcing the DID Log" (renamed from "Obtaining", 2026-10-02: "source" = get the log OR a
  reference to it; "obtain"/"retrieve" = get the files themselves) is a separate,
  explicitly non-normative section (no RFC 2119
  keywords): known in context, from watchers (client GETs `/log?scid=` and `/witness?scid=`
  and passes them directly), peer-to-peer exchange, from a web location, advertising
  sources (`watchers` param, `alsoKnownAs` DID URLs with `src`), and freshness/duplicity
  guidance for clients comparing copies.
- Peer-to-peer updates send only the new log entries (plus the full `did-witness.json` if any
  new entry needs witnessing, optional otherwise). The receiver skips already-held entries,
  appends the rest to a copy of its log, and resolves with `versionId` = last new entry; the
  entry-hash chaining means a non-extending update fails verification, so no second copy of the
  log is needed. The normative MUST for this lives in Update (Rotate) → Applying Peer-to-Peer
  Updates (moved 2026-10-05 from Security Considerations, which is now non-normative).
- Local directory `src` references (`file:` URL, RFC 8089, a directory — not a file — from
  which `did.jsonl`/`did-witness.json` are read; changed 2026-10-04 from `file:` URLs in
  `didLog`/`didWitness`) only for a resolver local to its client (library/CLI);
  network-facing resolvers must reject them (`invalidOptions`); a `src` DID URL parameter must
  be a web location — the dereferencer returns `invalidDidUrl` for `file:` (untrusted input).
- `src`/version options can also be DID URL query parameters; `didLog`/`didWitness` cannot.
  The dereferencer validates `src` (→ `invalidDidUrl`) and may apply policy — pass it on,
  ignore it, or fetch the files itself and pass them directly. (The DID Resolution "MUST
  pass all DID parameters as resolution options" rule is being removed from that spec, per
  Stephen.)
- Error names follow did:webvh's camelCase style (`invalidDid`, `notFound`, `invalidOptions`,
  `featureNotSupported`); current DID Resolution uses `INVALID_DID` / `NOT_FOUND` /
  `INVALID_OPTIONS` / `FEATURE_NOT_SUPPORTED` — needs aligning (across did:webvh too).
- No `portable` parameter, no implicit `#files`/`#whois` services (explicit services only), no
  parallel did:web.
- Watcher notification for did:vh without `src` carries the log in the POST body, plus a new
  POST `/witness?id=` for the witness file — an extension of the VH-Log/did:webvh watcher API.
- Follows VH-Log as it stands today (`updateKeys`/`nextKeyHashes`); will pick up the mandatory
  pre-rotation change (item 1 below) when VH-Log makes it.

did:vh TODOs:

- **Decide who handles multiple sources/watchers.** Should the resolver retrieve from (possibly
  multiple) watchers or other configured sources and compare copies itself (freshness,
  duplicity → error), or should the client retrieve from each source and make multiple
  `resolve()` calls, comparing the results? The spec currently avoids the question: one DID
  Log per `resolve()` call, no `watchers` resolution option, no resolver-configured sources,
  and freshness/duplicity handling is non-normative client guidance. (Earlier drafts had a
  `watchers` option, a `src` array, and normative resolver-side comparison rules — removed
  2026-09-25 pending this decision.)

- **Explore `src` naming a DID method (from did:scid).** did:scid lets `src` be a DID method
  that stores the verification data (e.g. `?src=did:cheqd:testnet`, using cheqd DID-Linked
  Resources), not just a URL. did:vh v0.1 supports `https` and `file:` URLs only (see the note in
  `spec-didvh/specification.md` under "The `src` Option"). The broader question to
  explore: how a DID Log (`did.jsonl`) and witness file (`did-witness.json`) can be stored and
  retrieved other than on a web server — ledgers/DID-Linked Resources, content-addressed
  storage, registries/databases — and what a storing DID method would need to specify (how
  the controller writes the log, how a resolver looks it up by SCID, how updates and the
  witness file are kept in sync).

- **Add a comparison with did:cel** (<https://w3c-ccg.github.io/did-cel-spec/>). did:vh is used
  in a way comparable to did:cel — both are location-independent DIDs built on a log — but
  the relationship differs from `did:scid:vh`, which is built on the did:webvh log format
  itself. Today the spec only compares did:vh to did:scid (abstract.md closing paragraph;
  overview.md "Relationship to `did:scid`"). Where to put it is undecided — see the
  suggestion recorded 2026-10-04: broaden the overview section to "Related DID Methods" with
  did:scid and did:cel subsections, and widen the abstract's closing sentence to name both.
  Keep it out of the normative specification's "Relationship to VH-Log and `did:webvh`" table.
  Before drafting, read the did:cel spec so the differences are stated accurately (log
  structure and verification, where the log is kept, resolution, witness/timestamping model).

### Planned updates (agreed 2026-09-16; item 5 superseded by the did:vh work above)

1. **Mandatory key pre-rotation.** Remove `updateKeys` entirely. Rename `nextKeyHashes` →
   `prerotationHashes` (list of hashes), required on every entry. The key(s) used to produce an
   entry's proof(s) MUST correspond to a hash present in the *previous* entry's
   `prerotationHashes`. Exception: the genesis entry, whose signing key is self-asserted (same
   bootstrap trust model used today for `updateKeys` in entry 0 — no change to SCID derivation).

2. **New cryptosuite spec: `eddsa-jcs-prerotation-2026`**, hosted in this repo (new `specs.json`
   entry, e.g. `spec-eddsa-jcs-prerotation/`). Extends
   [`eddsa-jcs-2022`](https://w3c.github.io/vc-di-eddsa/), adding to each `proof`: a sequential
   proof number (incrementing once per proof across the whole log's history, not per entry) and
   a `prerotationHashes` array. Using this cryptosuite with VH-Log makes pre-rotation
   self-describing in the proof, so the log-level `prerotationHashes` parameter from item 1
   becomes unnecessary when it's in use. Rule: a key MUST NOT be reused anywhere in the proof
   sequence — the source of the quantum-resistance claim (a public key is revealed only at the
   moment of use, once). Enforceable directly as "never reuse a proof number."

3. **Complete witness/watcher specification.** `spec/specification.md` already has
   `#### Witnesses` and `#### Watchers` sections — this is an audit/completeness pass on
   existing content, not net-new material.

4. **De-duplicate `spec-didwebvh/` against VH-Log.** Strip text that VH-Log already covers
   normatively, replace with references, keep only genuinely DID-specific content.
   `spec-didwebvh/specification.md` (73KB) is currently larger than the pre-split combined spec
   (54KB), which is the concrete symptom.
   - Version selection (added to VH-Log 2026-10-04 as "Selecting a Version": `versionId`,
     `versionTime`, version number; resolvers MUST support the first two and SHOULD support
     version number, matching did:webvh): point did:webvh's `versionId`/`versionTime`/
     `versionNumber` text in Read (Resolve) at it. did:vh already points there.
   - General direction: material shared by did:vh and did:webvh belongs in VH-Log, so
     did:vh references VH-Log rather than its sibling. did:vh still references did:webvh
     for resolution metadata, the DID-to-HTTPS transformation (`src` web locations),
     witness `did:key` rules, and `#files`/`#whois` dereferencing — candidates to move.

5. ~~**New spec: `did:vh`**~~ — started; see "did:vh — current first target" above. Note the
   draft makes log discovery normative (four sources incl. `src`) rather than leaving it out of
   scope with only non-normative examples, as originally planned.

## Status

Pre-draft. Do not implement against this spec until it reaches Draft status.
