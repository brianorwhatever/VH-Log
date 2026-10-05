# CLAUDE.md — vh-log-spec

## Purpose

This repository contains the **Verifiable History Log (VH Log)** specification. VH Log is a
general-purpose, append-only, cryptographically chained log structure for recording the history
of a versioned state object. It is a clean extraction and generalisation of the log mechanism
defined in the [did:webvh specification](https://identity.foundation/didwebvh/), with the goal
that did:webvh (and other specifications) can be defined as specialisations of VH Log.

Work here is pre-standard and should be treated as a working draft. The Editors Draft is
published at <https://swcurran.github.io/VH-Log/>.

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

```sh
npm install
npm run render   # render once
npm run edit     # watch
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

Rendered output goes to `next/`, which is git-ignored (do not edit or commit it). The root
`index.html` redirects to `next/`. On each push to `main`, the `render-specs` workflow renders
the spec and publishes the whole working tree (minus `node_modules`) to the `gh-pages`
branch, which GitHub Pages serves. Specialisations in other repos (e.g. did:vh) link to VH-Log with absolute
URLs such as `https://swcurran.github.io/VH-Log/next/index.html#<anchor>` — renaming a heading
here breaks those links.

## Repository Structure

```text
spec/               # VH-Log specification source (Spec-Up Markdown)
  header.md
  abstract.md
  overview.md
  specification.md
  security_and_privacy.md
  definitions.md
  references.md
  version.md
next/               # Rendered HTML output (git-ignored; do not edit directly)
index.html          # Redirect to next/
spec-refs.mjs       # Spec-Up plugin adding each spec's `spec_refs` to [[spec:]] lookups
render.mjs          # ESM render script (node render.mjs)
edit.mjs            # ESM watch script (node edit.mjs)
specs.json          # Spec-Up configuration

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
- Security and Privacy Considerations sections are **non-normative** (following DID Core
  §9/§10; changed 2026-10-05, also in did:vh). Put every RFC 2119 requirement in the
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
- **did:vh** (repo <https://github.com/swcurran/didvh>, local clone `/d2/repos/didvh`,
  published at <https://swcurran.github.io/didvh/>): the first specialisation of VH-Log —
  similar to did:webvh but the DID is just `did:vh:<SCID>`. Split out of this repo on
  2026-10-05; its design decisions and TODOs live in that repo's CLAUDE.md.
- **did:webvh** (<https://identity.foundation/didwebvh/>): VH-Log was extracted from it; a
  future did:webvh version is expected to be redefined as a VH-Log specialisation. A reworked
  copy was briefly kept here as `spec-didwebvh/` and removed 2026-10-05.

## Current Work and Next Steps

### History

- 2026-06-06: migrated Spec-Up from the old GitHub fork to npm v0.11.6; removed `katex`
  from `specs.json` (unused; spec-up 0.11.6 packaging bug leaves its fonts missing).
- `spec/` reworked into the generalised VH-Log spec: DID-specific content stripped,
  `did.jsonl` replaced with `vh-log.jsonl`, DIDDoc replaced with the generic `state` object,
  header status "Pre-Draft — Editors Draft v0.1".
- 2026-10-05: did:vh moved to its own repo; the did:webvh copy removed; repo cleaned up for
  GitHub Pages publishing.

### Open inconsistency

- `spec/version.md` says the did:webvh `method` parameter is replaced in VH-Log by
  `logVersion` (`vh-log:1.0`), while did:vh describes its `method` parameter
  (`did:vh:1.0`) as implying the VH-Log base version. Check the spec body and make the two
  agree.

### Planned updates (agreed 2026-09-16)

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

4. **Move shared did:vh / did:webvh material into VH-Log.** Material shared by the DID
   specialisations belongs here so they reference VH-Log rather than each other. did:vh
   still references did:webvh for resolution metadata, the DID-to-HTTPS transformation (`src`
   web locations), witness `did:key` rules, and `#files`/`#whois` dereferencing — candidates
   to move. (Version selection already moved here 2026-10-04 as "Selecting a Version".)

## Status

Pre-draft. Do not implement against this spec until it reaches Draft status.
