## Security Considerations

This section follows the guidelines in [[spec:RFC3552]] and addresses the security requirements for all VH-Log operations defined in this specification.

### Threats and Attacks

Implementations of VH-Log **MUST** mitigate the following classes of attack for all log operations:

- **Eavesdropping** — All network communication (e.g., retrieval of log files, witness files, or other log-associated resources) **SHOULD** be performed over TLS (HTTPS). Plaintext HTTP **MUST NOT** be used except for testing or non-production deployments where confidentiality is not required. The verifiability of VH-Log ensures that tampering with the contents of individual log entries is detectable; TLS protects against passive observation and other network-based risks.

- **Replay attacks** — Implementations **MUST** verify the log (including monotonic progression and referenced hashes) and reject logs that do not verify as outlined in the [Read](#read-resolve) section of this specification.

- **Message insertion, deletion, and modification** — Each log entry and witness file is integrity-protected using cryptographic signatures. Implementations **MUST** verify the log and reject any log that fails verification as outlined in the [Read](#read-resolve) section of this specification.

- **Truncation or withholding of log entries** — An attacker (or misconfigured intermediary) could serve an older-but-valid prefix of the log, truncating newer entries and presenting a stale state.
  - **Mitigations (non-exhaustive):**
    - **Resolver cache-and-compare:** Resolvers **SHOULD** remember the latest `versionId` previously observed for a log and **SHOULD** warn or fail resolution when presented with a truncated log.
    - **Witness verification:** Where [[ref: Log Controllers]] utilise witnesses, Resolvers **MUST** verify that all entries are properly witnessed according to the [Witnesses](#witnesses) section of this specification. Notably, witness proofs of unpublished or truncated entries **MUST** be ignored.
    - **Multi-source resolution:** Resolvers **MAY** attempt retrieval from multiple [[ref: Watchers]] and detect divergence in latest entries.
    - **End-to-end TLS:** While signatures detect per-entry tampering, use of TLS **SHOULD** be enforced to reduce opportunities for active truncation in transit.

- **Denial of Service (DoS) and amplification** — A malicious or compromised log server could attempt to exhaust resolver resources by serving pathologically large log files, or by holding a connection open and streaming log entries indefinitely. This risk is most acute for resolvers that encounter previously-unseen logs on behalf of their clients.

  - The one-second `versionTime` monotonicity requirement provides a natural constraint on log length — a log file with implausibly rapid `versionTime` progression is likely malformed and resolvers are advised to reject it. However, this does not fully prevent the attack, as a patient attacker can pace log entry generation to match valid `versionTime` increments.

  - Resolvers are advised to require a `Content-Length` header in HTTP responses before processing begins, allowing them to determine file size upfront and reject oversized logs before expending processing resources. The presence of HTTP range request support (`Accept-Ranges: bytes`) is a useful positive signal that a server is serving a static resource rather than dynamically generated content. Regardless of server behavior, since a declared `Content-Length` cannot itself be relied upon as a security guarantee, resolvers are advised to apply the following defensive practices:

    - Apply a maximum byte limit on log file retrieval, terminating the connection if exceeded regardless of declared `Content-Length`; treat absence of `Content-Length` as grounds for a more conservative limit or outright rejection
    - Apply a hard timeout on the entire fetch-and-process operation for a single resolution request
    - Apply rate limiting on resolution requests, particularly for previously-unseen logs
    - Apply concurrency limits to bound the number of simultaneous resolution operations

  - The subjective elements of these limits (e.g., what constitutes "excessively large" or "repeated") are best established through community or ecosystem governance. These mitigations are standard HTTP service hardening practices and are not unique to VH-Log.

- **Man-in-the-middle (MitM)** — HTTPS and signature verification of log entries and witness proofs protect against undetected modification of individual entries. Risks specific to withholding or truncation are addressed in **Truncation or withholding of log entries** above. Use of TLS further ensures server authenticity and reduces opportunities for active interference.

- **Conflicting parallel updates / split view** — Multiple authorized updates from the same parent entry can produce divergent logs. Publication components **MUST** enforce monotonic extension of the current tip; witnesses **MUST NOT** sign more than one child of the same parent; consolidation of the witness file **MUST** fail on conflicting branches. Resolvers **SHOULD** cache the highest observed version/hash and **SHOULD** detect/warn on older branches; Watchers **SHOULD** detect and report divergence across sources. Because no other party can verify that a witness has followed these rules, detecting split views ultimately relies on [[ref: watchers]] — which any party may run — comparing copies of the log from multiple sources.

- **Other attacks** — The required VH-Log verification process mitigates downgrade attacks on cryptographic algorithms and prevents poisoning of log or witness files, since unauthorized changes fail signature verification. Verification does not, however, address availability risks; implementers **SHOULD** consider operational measures (e.g., [Watchers](#watchers) and well-known web techniques) to improve resilience.

### Residual Risks

Residual risks include:

- Compromise of the web hosting infrastructure serving the log resources.
  - While this can impact access to the log file and associated files, it does not compromise the integrity of the log entries themselves, nor the verifiability of the log.
- Compromise of [[ref: Log Controller]] private keys.
  - A [[ref: Log Controller]] can mitigate this risk through the use of [pre-rotation keys](#pre-rotation-key-hash-generation-and-verification).
  - In doing so, [[ref: Log Controllers]] **SHOULD** avoid reusing revealed pre-rotation keys. While not invalid per this specification, re-use of a pre-rotation key after disclosure reduces compromise containment. Mitigation: follow the one-time-use best practice and securely destroy revealed private keys (see [Pre-Rotation Key Hash Generation and Verification](#pre-rotation-key-hash-generation-and-verification)). Resolvers are **NOT REQUIRED** to enforce this, but **MAY** warn.
  - Additional good security practices **SHOULD** also be followed, such as using hardware security modules (HSMs) or secure enclaves for key storage, enforcing strong access controls, maintaining secure backups of critical keys, and performing regular key rotations.
- Weaknesses in underlying cryptographic algorithms after deployment.
- Misconfiguration of cache control or TTL values.
- Implementation errors in resolvers or [[ref: Log Controllers]].
- Resolvers operating in contexts where they may encounter large numbers of previously-unseen logs — such as open verification services — face elevated resource exhaustion risk and are advised to implement rate limiting and monitoring for abusive resolution patterns, with the ability to block offending sources.

### Integrity Protection and Update Authentication

All log operations (create, update, deactivate) are integrity-protected by the cryptographic verification of log entries. Update authentication is provided by verifying the [[ref: Log Controller]]'s proof(s) against the valid `updateKeys` in the log [[ref: parameters]].

Because a VH-Log log and its associated entries are self-certifying, they can be verified and trusted regardless of how they are retrieved — whether directly from the host, via a cache, through a CDN, from a [[ref: watcher]], or via a trusted resolver service. The verification process ensures authenticity and integrity independent of the transport channel.

### Authentication Characteristics

The authentication of log updates is based on possession of the private keys associated with the update and pre-rotation keys. The security of the log therefore depends on the strength of these keys, their secure storage, and the cryptographic algorithms used.

### Unique Assignment of Logs

In VH-Log, uniqueness of a log is based on the [[ref: self-certifying identifier]] (SCID) generated at the inception of the log. The SCID is cryptographically bound to the [[ref: Log Controller]]'s keys and ensures that no two independently created logs can have the same identifier.

The location of the log (as defined by the [[ref: specialisation]]) is used solely for discovery of the log file and associated files; it is not used for verification of log control. The [[ref: specialisation]] **SHOULD** document any assumptions about log location that may affect uniqueness or verifiability.

### Endpoint Authentication

Log resource retrieval endpoints **MUST** be authenticated using TLS server authentication. Self-signed certificates **SHOULD NOT** be used in production. While the verifiability of VH-Log ensures that any tampering with the contents of individual log entries is detectable, TLS provides additional protection against active network attacks (including truncation or withholding) and ensures the authenticity of the server providing the log resources.

### Resolver Transport Hardening (SSRF and Network Boundary)

A VH-Log resolver acts as an HTTP client on behalf of an untrusted log identifier — a Server-Side Request Forgery vector without explicit safeguards. Resolvers **MUST**:

1. **No automatic redirects.** Do not auto-follow HTTP 3xx responses when fetching log files or witness files. If redirect-following is an opt-in, re-apply every check below at each target.
2. **IP-literal rejection.** Reject IPv4/IPv6 literal hosts both (a) after percent-decoding the identifier's domain segment, and (b) after DNS resolution. Default deny: loopback (`127.0.0.0/8`, `::1`), private (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `fc00::/7`), link-local (`169.254.0.0/16`, `fe80::/10`). Dev opt-ins permitted but off by default.
3. **Case-insensitive percent-decoding.** Per [[spec:rfc3986]] §2.1, normalise hex case **before** any allow/deny decision. Rejecting `%3A` but accepting `%3a` is non-compliant.
4. **Re-validate after decoding.** All host/path checks apply to decoded values. Percent-encoded IP literals and traversal sequences (`%2E%2E`, `%2e%2e`) **MUST** be rejected after decoding.
5. **Response size cap.** Enforce a max body size for both log and witness files; check `Content-Length` first; treat absence as grounds for a stricter cap or rejection. Suggested default: 5 MiB.
6. **Operation timeout.** Enforce a wall-clock timeout on the complete resolution. Suggested default: 30 s.
7. **HTTPS only.** Reject any non-`https` scheme, including after a redirect.
8. **No localhost in production.** Do not issue requests to `localhost`, `127.0.0.0/8`, `::1`, or names resolving to them. Test opt-ins off by default.

These are standard SSRF defences; stated normatively here because common implementations have been found vulnerable to at least one of (1)–(4).

### Network Topology

VH-Log relies on web infrastructure and does not require peer-to-peer networking. However, implementations relying on CDN caching or load balancers **MUST** ensure these intermediaries do not serve stale or tampered log data.

### Cryptographic Protection

The following data is protected:

- **Log entries** — Signed by the [[ref: Log Controller]]'s keys.
- **Witness proofs** — Signed by witness keys.

These signatures provide integrity and update authentication but not confidentiality; log entries are public.

Secret data (e.g., [[ref: Log Controller]] private keys, witness private keys, random seeds) **MUST** be protected in secure storage and never exposed in the log.

### Signature Implementation

VH-Log uses standard [[ref: Data Integrity]] proof mechanisms for signing log entries and witness proofs, as defined in the cryptographic suite used. Implementations **MUST** follow the suite's signature generation and verification requirements.

### Cross-Origin Resource Sharing (CORS) Policy Considerations

To support scenarios where log resolution is performed by client applications running in a web browser, the file served for the log **MUST** be accessible by any origin. To enable this, the log file HTTP response **MUST** include the following header:

`Access-Control-Allow-Origin: *`

### Post Quantum Attacks

VH-Log's [[ref: Key Pre-Rotation]] approach provides enough flexibility for "post-quantum safety." For guidance on post-quantum attack mitigation, implementors **SHOULD** refer to relevant implementation guidance on pre-rotation keys and post-quantum cryptography.

### Resolver Validation Checklist (informative)

The following checklist maps normative resolver requirements to concrete validation points, and is intended to assist conformance test suite authors and implementers auditing their own resolver implementations. An entry appearing here does not restate or replace the normative requirements defined in the verification algorithm in this specification.

**Transport:** HTTPS only; no auto 3xx; reject IP literals before & after percent-decoding; reject private/loopback/link-local DNS resolutions in production; normalize percent-encoding case; validate path segments after decoding (`.`, `..`, `/`, `\`, NUL, leading/trailing whitespace); enforce max response size with `Content-Length` first; enforce a wall-clock timeout.

**Log structure:** unbroken `1, 2, 3, ...` version sequence; strictly increasing UTC ISO8601 `versionTime`; every `versionTime` ≤ now (bounded skew); `entryHash` chain verified for every entry; `logVersion` is an explicitly supported value (never silently downgraded).

**SCID & identity:** first entry's `parameters.scid` is the genesis self-hash; [[ref: SCID]] never changes.

**Keys & proofs:** every proof has `type: DataIntegrityProof`, the `cryptosuite` and `proofPurpose` required by the active `logVersion`; `verificationMethod` key in active `updateKeys`; under pre-rotation, `updateKeys` explicit in every entry and every key hashes to a value in previous `nextKeyHashes`.

**Witnesses:** `threshold` is positive integer ≤ count of distinct `witnesses[].id`; all `witnesses[].id` distinct; threshold met by counting verified proofs from distinct witness identifiers, not total proof count; each accepted proof's `versionId` corresponds to an entry in the log file being verified; proofs verified using the method defined by the [[ref: specialisation]].

**Failure modes:** unknown parameter values, malformed `witness`, hash algorithm mismatch, and cryptosuite mismatch all **MUST** fail resolution — never silently coerced.

## Privacy Considerations

This section addresses the privacy considerations in alignment with [[spec:RFC6973]] Section 5.

### Surveillance

VH-Log publishes logs to publicly accessible HTTPS endpoints. While the contents of the log are generally intended to be public, the timing, frequency, and correlation of updates can be observed and may reveal operational patterns or associations.

Resolution of a log also exposes the resolver's network activity to DNS providers and web servers, which could be used for tracking. Controllers and resolvers **MAY** use privacy-enhancing technologies such as VPNs, TOR, or trusted universal resolver services to reduce this risk.

### Stored Data Compromise

Log data is stored on web servers. A compromise of the hosting infrastructure could allow tampering with log resources. HTTPS and cryptographic signatures protect integrity, but confidentiality is not provided.

### Implementation Hygiene (informative)

The following practices are not unique to VH-Log but represent recurring failure points observed across implementations. Implementers should treat these as baseline hygiene.

- **Dependency currency.** Keep cryptographic and HTTP dependencies on supported, patched versions. Run a vulnerability scanner on every build.
- **Filesystem permissions.** Private keys and secret-bearing configuration files **SHOULD** be created with owner-only permissions (e.g., `0600` on POSIX). **SHOULD NOT** read secret material from the current working directory or other untrusted locations by default.
- **CLI tools.** **MUST NOT** print private keys, mnemonics, or other long-lived secrets to stdout/stderr or shell history. Where display is necessary, prompt before printing and offer a file-output alternative with restrictive permissions.
- **HTTPS certificate validation.** **MUST NOT** disable certificate validation by default.
- **Error messages.** Resolvers **SHOULD NOT** include internal stack traces, file paths, or library versions in error details returned to clients.

### Correlation

The use of a static log identifier and public log entries can enable correlation of activities over time. [[ref: Log Controllers]] **SHOULD** avoid embedding personal identifiers or unnecessary information in [[ref: state]] objects.

### Identification

Logs are public and can be linked to real-world identities through the hosting location. Entities that require anonymity **SHOULD** consider whether VH-Log is appropriate for their use case.

### Right to Erasure

While it is possible for a [[ref: Log Controller]] to delete published data as described in [Deactivate](#deactivate), it is **RECOMMENDED** for monitoring [[ref: watchers]] to cache the last known state indefinitely. This means that the ability and specific process of complete data erasure depends on [[ref: watcher]] behavior and **SHOULD** be defined by the governance of the ecosystem.

### Secondary Use

Information published in the log may be repurposed by third parties. [[ref: Log Controllers]] **SHOULD** minimise the publication of data that could be used for purposes beyond the intended use.

### Disclosure

All data in the log is publicly accessible. Sensitive data **MUST NOT** be included.

### Exclusion

VH-Log does not require controller ownership of any specific infrastructure. The [[ref: specialisation]] defines the hosting location, and controllers may publish logs on web-hosting platforms that serve static files over HTTPS. This reduces barriers to participation.

Residual exclusion risks remain: access to such platforms typically requires an account and compliance with provider terms of service; platforms might impose geoblocking, payment requirements, or content restrictions; and accounts can be suspended. [[ref: Log Controllers]] **SHOULD** maintain the ability to republish or mirror log resources under alternative hosts (including using [[ref: watchers]]) and **SHOULD** document a transition plan so that participants are not locked out if a hosting provider becomes unavailable. The verifiable history of the log ensures that it can be verified regardless of the source.
