# Changelog

Protocol versions are tagged in this repository. Entries below record
normative text changes that implementers may care about.

## [Unreleased]

### Clarifications

- Merge the two Design sentences that both allowed trying alternative sources before streaming into the existing retry-policy bullet. No intentional wire/header/env change.
- Merge two Design bullets that both allowed anytime cache eviction into one rule (delete/evict for any reason, independent policies). No intentional wire/header/env change.
- Define BCP 14 requirement keywords and use **MAY** for optional behavior (was non-standard **CAN**).
- Prose clarity in `SPEC.md` (grammar, phrasing, Challenges aligned with protocol-only repo scope). No intentional wire/header/env change.
- README implementations table includes the Java SDK (`fetchurl/sdk-java`).
- Algorithm name normalization: lowercase then keep only `[a-z0-9]` (was incorrectly “discarding letters…”; would not strip `-` from `SHA-256`).
- Source “content size” means HTTP `Content-Length` on the source response; reject when absent (matches reference server and SDK expectations).
- Hash path segment is the full lowercase hex digest (`sha1` 40 / `sha256` 64 / `sha512` 128); servers SHOULD **400** on non-hex or wrong length (and still SHOULD **400** above 255 chars). Uppercase hex MAY be accepted via lowercase normalization. Error conditions list invalid digests under 400. “Downstream” in the health rule means a server probing an upstream fetchurl server.
- Empty-file (zero-length) digests for algorithms in scope are named explicitly (`sha1` / `sha256` / `sha512`); servers SHOULD serve them as a cache hit with a zero-byte body without contacting source or upstream.
- Missing source `Content-Length` is a failed source (not a silent stream): MAY try other sources before streaming; if none succeed, SHOULD **502**. Error conditions list that case under 502.
- Integrity failures mid-stream: if transferred bytes do not match source `Content-Length`, the server MUST abort the connection like a hash mismatch. Failed hash or size verification MUST NOT complete as a durable cache addition (only verified content is stored). Error conditions list size mismatch under unexpected aborts.
- Security: outbound source fetches MUST use `http`/`https` only; by default MUST NOT connect to non-public destinations (loopback, RFC 1918/ULA, link-local including cloud metadata, unspecified, multicast, RFC 6598 CGNAT), including on redirect hops; resolved IPs MUST be checked. Optional off-by-default non-public mode MAY exist for tests. SHOULD bound connect/TLS/response-header waits; MUST NOT rely only on a full-body client timeout. Operator-configured upstream bases MAY be private; client `X-Source-Urls` follow the public rules unless the non-public mode is on. Error conditions list total Security rejection under 502.
- Source responses usable for cache fill require final HTTP status **200** and a usable `Content-Length` (non-200 is a failed source like a missing length; MAY try alternatives before streaming; if none succeed, SHOULD **502**). Successful server responses to the client (cache hit, empty-file short-circuit, or completed fill) MUST be **200**. Error conditions list non-200 source status under 502.

## [0.1.0] — initial

Initial normative text extracted from the former monorepo (`lucasew/fetchurl` / `fetchurl/fetchurl`).
