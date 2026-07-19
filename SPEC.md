# Motivation

- Repeated downloads in CI environment
- Lack of standard in content addressable caching (we have pnpm and stuff but nothing standard that can be used by everyone)
- CI caching is mostly push based: build happens, artifacts are pushed based on a key
- Caching codes defined by intention instead of content (nix: hash of derivation, npm: cache of lockfile)

# Alternatives
- Backup the downloaded assets as workflow cache: looks very wasteful and loses value when anything changes (like the cache key is the hash of package-lock.json)
- Caching proxy: too much hassle to setup a MITM proxy with a custom cert and force traffic through it
- Let it redownload assets every time

# Parts
- Client: the one that wants a file (npm, uv, hex, Nix)
- Server: the server proposed by this document, intermediates downloads, stores confirmed files until eviction
- Upstream: a daisy-chained server
- Source: the one that would directly serve the file to the client otherwise

# Scope
- Algorithms: sha1, sha256, sha512
- Only public data (no auth)
- Focus on caching package manager dependencies (ex: npm packages)

# Requirements language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.html), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174.html)) when, and only when, they appear in all capitals, as shown here.

# Design

```
GET /api/fetchurl/sha256/e3b0... HTTP/1.1
X-Source-Urls: "https://cdn1.com/file.tar.gz", "https://backup.org/archive.tgz"
```
- This design doesn't cover how a client may get the source URLs and hashes of the content
- The server MUST only work with public, or well hidden, data
- `X-Source-Urls` MUST be a list of source URLs following [RFC 8941](https://www.rfc-editor.org/rfc/rfc8941.html#name-lists)
- `X-Source-Urls` SHOULD be no longer than 8192 characters. The server MAY truncate it and load all complete URLs, dropping a trailing truncated fragment.
- A source URL SHOULD be chosen randomly at fetch time, like a mirror list
- The server MUST NOT retry or fall back to alternative sources once response streaming has begun. Before the server begins streaming data to the client, it MAY try alternative sources if the initial source fails.
- The `FETCHURL_SERVER` environment variable MUST be a list of servers following [RFC 8941](https://www.rfc-editor.org/rfc/rfc8941.html#name-lists) if the first character is a `"`, otherwise the whole variable is interpreted as one single value
  - Example: `FETCHURL_SERVER="http://cache.local:8080/api/fetchurl"`
    - The client appends `/:algo/:hash` to produce `http://cache.local:8080/api/fetchurl/sha256/e3b0...`.
- The `FETCHURL_SERVER` environment variable MUST have the full URLs ready to append `/:algo/:hash`
- The `FETCHURL_SERVER` environment variable MAY be absent or empty, which MUST disable server support
- Clients are instructed by environment to use servers by using the `FETCHURL_SERVER` environment variable
- Clients MUST check for `FETCHURL_SERVER`
- Clients MAY fall back to direct download if something goes wrong
- Clients MUST know the hash and source URLs before connecting to a server
- Clients MUST check the hash of the files being downloaded on the server and assume that the Server is untrusted, no matter the provider or if it uses TLS
- Hashes MUST be represented as lowercase hexadecimal of the full digest for the algorithm. Expected path-segment lengths for algorithms in scope: `sha1` = 40, `sha256` = 64, `sha512` = 128 characters.
- Servers SHOULD reject a request with **400** when the hash path segment is not hexadecimal or is not the expected length for the (normalized) algorithm. Servers MAY accept uppercase hex digits by normalizing them to lowercase before lookup, storage, and comparison.
- Servers SHOULD reject hashes longer than 255 ASCII characters (also **400**).
- The server MAY delete any item at any moment for any reason
- The process of deletion and addition of a cache item MUST be atomic
- The source HTTP response MUST include a `Content-Length` header giving the content size. If it is absent (for example chunked encoding without a known length), the server MUST reject the request
- The server MAY start serving the data while it's checking for the hash to optimize time to first byte
- If the hash doesn't match at the end of the stream the server MUST abruptly close the connection
- The client MUST only accept the file if the connection ended gracefully, anything that resembles a failure MUST be considered as a rejection
- Daisy-chained servers SHOULD send the list of URLs via `X-Source-Urls` to their upstreams so the upstream can fall back to a source download.
- Servers MAY evict any data at any time and have their own independent eviction policies to balance cache hit rate against resource usage
- When saving data on disk, the data directory SHOULD follow `/:algo/:shard/:hash` where shard is the first n hexadecimal characters of the hash (n defaults to 2).
- Hashing algorithm names MUST be normalized by converting to lowercase and discarding every character that does not match `[a-z0-9]`. Examples: `SHA-256` → `sha256`, `sha1`, `sha512`
- Clients SHOULD prefer the sha256 algorithm if available
- Servers SHOULD deduplicate concurrent requests for the same `algo:hash` pair to avoid redundant upstream fetches
- Servers SHOULD treat the empty-file hash as a cache hit without downloading anything
- Servers SHOULD set the header `Cache-Control: public, max-age=31536000, immutable` on a real cache hit, where the server has proven that it has the valid file in its store.
- Servers MUST set the header `Content-Type: application/octet-stream` regardless of the upstream MIME type
- Servers MUST implement a health route at `/health` under the router referenced by `FETCHURL_SERVER`. Example: `/api/fetchurl/health`.
- Downstream servers (a fetchurl server calling another fetchurl server as upstream) MUST decide whether that upstream is healthy by checking the status code of the health route. 200 = OK. Anything else = not OK.

# Challenges
- Implementation: this repository holds only the protocol; specialized implementations can run on Cloudflare Workers or be written in Rust or Elixir for scalability and performance
- Adoption: implement logic on different clients to use a server
- Adoption: CI providers exposing servers in build environments with out-of-the-box support
- Performance: efficient automatic eviction by policy (deletion can be one atomic `rm`; tracking what to evict is the harder problem)

# Error conditions
- 400 - Bad request
  - Unsupported hash algorithm
  - Invalid hash path segment (not hex, wrong length for the algorithm, or longer than 255 characters)
- 404 - Not found
  - Cache miss, no sources
- 502 - Bad gateway
  - Upstream/source failed to respond.
- Unexpected aborts
  - Hash mismatch
  - Source/upstream unexpected abort
