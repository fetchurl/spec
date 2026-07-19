# Changelog

Protocol versions are tagged in this repository. Entries below record
normative text changes that implementers may care about.

## [Unreleased]

### Clarifications

- Define BCP 14 requirement keywords and use **MAY** for optional behavior (was non-standard **CAN**).
- Prose clarity in `SPEC.md` (grammar, phrasing, Challenges aligned with protocol-only repo scope). No intentional wire/header/env change.
- README implementations table includes the Java SDK (`fetchurl/sdk-java`).
- Algorithm name normalization: lowercase then keep only `[a-z0-9]` (was incorrectly “discarding letters…”; would not strip `-` from `SHA-256`).
- Source “content size” means HTTP `Content-Length` on the source response; reject when absent (matches reference server and SDK expectations).
- Hash path segment is the full lowercase hex digest (`sha1` 40 / `sha256` 64 / `sha512` 128); servers SHOULD **400** on non-hex or wrong length (and still SHOULD **400** above 255 chars). Uppercase hex MAY be accepted via lowercase normalization. Error conditions list invalid digests under 400. “Downstream” in the health rule means a server probing an upstream fetchurl server.

## [0.1.0] — initial

Initial normative text extracted from the former monorepo (`lucasew/fetchurl` / `fetchurl/fetchurl`).
