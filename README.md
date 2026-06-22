# fetchurl protocol specification

Normative specification for **fetchurl**: a simple content-addressable URL cache protocol for CI and package managers.

This repository holds **only** the protocol. Implementations live elsewhere.

## Specification

See [SPEC.md](./SPEC.md). Tagged releases (e.g. `v0.1.0`) mark immutable protocol versions.

## Implementations

| Component | Repository |
|-----------|------------|
| Reference server (Go) | [fetchurl/fetchurl](https://github.com/fetchurl/fetchurl) |
| JavaScript SDK | [fetchurl/sdk-js](https://github.com/fetchurl/sdk-js) |
| Python SDK | [fetchurl/sdk-python](https://github.com/fetchurl/sdk-python) |
| Rust SDK | [fetchurl/sdk-rust](https://github.com/fetchurl/sdk-rust) |

## Versioning

- Protocol changes happen in this repo and are tagged (`v0.1.0`, `v0.2.0`, …).
- Servers and SDKs declare which protocol version(s) they implement; their semver is independent of protocol tags.
- Breaking wire/header/environment semantics require a new protocol minor or major; document the delta in `CHANGELOG.md`.

## Contributing

Open issues here for **protocol** proposals and clarifications. Implementation bugs belong in the relevant server/SDK repository.
