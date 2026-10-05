# The Verifiable History Log (VH-Log) Specification

The spec repository for the **Verifiable History Log (VH-Log)**: a
general-purpose, append-only, cryptographically chained log for recording the
history of a versioned state object.

Read the spec (Editors Draft):
[https://swcurran.github.io/VH-Log](https://swcurran.github.io/VH-Log/)

## Current Status of the Specification

**Pre-Draft — Editors Draft v0.1.** This is pre-standard work. Do not implement
against this specification until it reaches Draft status.

## Abstract

VH-Log defines an append-only log of entries, each capturing a new version of an
arbitrary JSON `state` object together with the cryptographic proofs that
authenticate it. The chain of entries forms a tamper-evident history that any
party can independently verify. VH-Log features include the following.

- Cryptographic chaining: each entry's `versionId` is derived from a hash of the
  entry's content, which includes the previous entry's `versionId`, linking all
  entries together.
- A Self-Certifying Identifier (SCID) derived from the genesis entry and
  embedded in the log's identifier, so a verifier can confirm that a given log
  is the authentic log for that identifier.
- A `parameters` mechanism for per-entry configuration (such as update keys,
  pre-rotation key hashes and witnesses) that evolves over the life of the log.
- An optional witness mechanism, in which a threshold of external parties sign
  each entry before publication.
- An optional watcher mechanism, in which parties archive copies of the log and
  serve them independently of the original publisher.
- A resolution algorithm that verifies the full chain and returns the `state`
  at the latest version, or at a selected earlier version.

The type of the `state` object is left to the *specialisation* that builds on
VH-Log, making VH-Log a reusable building block for any system that needs a
verifiable history of a versioned object.

## Relationship to Other Specifications

- **[did:webvh](https://identity.foundation/didwebvh/)** — VH-Log was extracted
  from the log mechanism of the did:webvh v1.0 specification. A future version
  of did:webvh is expected to be defined as a specialisation of VH-Log.
- **[did:vh](https://swcurran.github.io/didvh/)** — the first specialisation of
  VH-Log. The `state` object is a W3C DID Document and the DID is just
  `did:vh:<SCID>`, with no location component, so the DID's log can be sourced
  from anywhere — the context in which the DID was received, a watcher, a peer,
  or a web location. Source: [swcurran/didvh](https://github.com/swcurran/didvh).

## Contributing to the Specification

Pull requests (PRs) to this repository may be accepted. Each commit of a PR must
have a DCO (Developer Certificate of Origin -
[https://github.com/apps/dco](https://github.com/apps/dco)) sign-off. This can
be done from the command line by adding the `-s` (lower case) option on the `git
commit` command (e.g., `git commit -s -m "Comment about the commit"`).

Rendering and reviewing the spec locally requires `npm` and `node`. Follow these
steps:

- Fork and locally clone the repository.
- Run `npm install` from the root of your local repository.
- Edit the spec documents (in the `/spec` folder).
- Run `npm run render` to render the spec into the `/next` folder.
  - Use `npm run edit` to interactively edit, render and review the spec.
- Load the root `index.html` file in a browser to be redirected to the rendered
  specification.

The `.npmrc` file sets `node-options=--dns-result-order=ipv4first`, which is
needed on machines where IPv6 is broken and is harmless elsewhere.

The specification is in [Spec-Up] format (v0.11.6 from npm). See the
[Spec-Up Documentation] for a list of Spec-Up features and functionality.

[Spec-Up]: https://github.com/decentralized-identity/spec-up
[Spec-Up Documentation]: https://identity.foundation/spec-up/

## Publishing

On each push to `main`, the `spec-up-render` GitHub Action renders the spec and
publishes the result to the `gh-pages` branch, which GitHub Pages serves. The
root `index.html` redirects to the Editors Draft in `next/`.
