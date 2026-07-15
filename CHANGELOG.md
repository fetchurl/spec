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

## [0.1.0] — initial

Initial normative text extracted from the former monorepo (`lucasew/fetchurl` / `fetchurl/fetchurl`).
