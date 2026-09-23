# typeship — agent context

This package contains the generated TypeScript SDK for **typeship** (API v1.0.0, package v0.14.0).

Resolve an OpenAPI or GraphQL Definition, diagnose it, and keep every
selected CLI, MCP, and SDK Target current.

Every operation but one requires a bearer credential: an organization
API key from the console, or an OAuth access token carrying the operation's
read, generate, or write capability and the organization selected during
consent. OAuth grants cannot switch organizations after consent. A browser
session is not a credential for this API. The exception is POST /generate,
which works anonymously with the free plan's limits.

## Ground rules
- For a linked repository Delivery, commit package customizations to the rolling Draft. Typeship three-way merges those commits with the next unmodified Generation, preserves exact bytes and file modes, and stops for explicit review when both sides touch the same region or file ownership is ambiguous.
- A preserved file participates in the combined package only when the package manifest, exports, build, and tests include it. Configure Target checks for every custom build or test requirement; do not assume a file is published merely because it survives regeneration.
- Application-only wrappers may still live outside this package. Code intentionally shipped from this package belongs on its rolling Draft and must pass the combined-package checks.
- Zero runtime dependencies; everything runs on platform `fetch` (Node 18+, browsers, edge).
- `api.md` is the native method reference; `api.json` is the machine-readable operation, schema, safety, and example contract. Read them before guessing.
- Start with the local build or installation instructions in `README.md`. Generation does not publish a registry package.

## Authentication
- TypeScript SDK: pass the `bearerToken` client option explicitly; the SDK does not read credential environment variables.

## Using the SDK
```ts
import { TypeshipClient } from "@typeship-ax/sdk";
const client = new TypeshipClient({ /* auth options above */ });
```
- Awaiting a call returns `ApiResult<T, E>`: check `result.ok`, or `unwrap(result)` to throw the typed error. Malformed successful JSON is `ResponseParseError`.
- Paginated methods return a `PagePromise`: awaiting it returns the first page's `ApiResult`; `for await (const item of client.x.list())` walks every page and throws the typed API error if a page fails.
- Every method takes a last `{ timeoutMs, maxRetries, headers, signal }` argument for per-call overrides; every result carries `response: { status, headers, requestId }`.
- Uploads take a `Blob` (a `File` for a filename).
- `debug: true` (or a function) on the client logs one redacted line per request.

## Documentation
- The reference for this exact package: `api.md` (offline, always current with the code).
- Conceptual guides live on the docs site. For questions about how the API's concepts fit together (flows, ordering, environments), fetch `https://typeship.dev/llms-full.txt` and read the relevant sections; `https://typeship.dev/llms.txt` is the page index. Relative links in the spec resolve against `https://typeship.dev`.
