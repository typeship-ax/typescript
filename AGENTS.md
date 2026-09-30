# Typeship: agent guide

Instructions for coding agents that call the Typeship API through this TypeScript SDK (API version 1.0.0, package version 0.27.0).

Resolve an OpenAPI or GraphQL Spec, diagnose it, and keep every
selected CLI, MCP, and SDK Target current.

Every operation but one requires a bearer credential: an organization
API key from the console, or an OAuth access token carrying the operation's
read, generate, or write capability and the organization selected during
consent. OAuth grants cannot switch organizations after consent. A browser
session is not a credential for this API. The exception is POST /generate,
which works anonymously with the free plan's limits.

Examples use Parcel, a fictional delivery service. Replace its domains,
repository names, and resource identifiers with your own. The hosted
petstore Spec is a runnable sample.

## Before writing code
- `api.md` is the method reference; `api.json` is the machine-readable contract: every operation's inputs, outputs, errors, `safety` (`read`, `write`, or `destructive`), and an example. Look up exact names there instead of guessing.
- `README.md` covers installation and setup.
- Zero runtime dependencies; everything runs on platform `fetch` (Node 20+, browsers, edge).

## Authentication
- TypeScript SDK: pass the `bearerToken` client option explicitly; the SDK does not read credential environment variables.

## Using the SDK
```ts
import { TypeshipClient } from "@typeship-ax/sdk";
const client = new TypeshipClient({ /* auth options above */ });
```
- Awaiting a call returns the response data or throws a typed error. Catch `ApiError` for HTTP failures, `ResponseParseError` for malformed successful JSON, or `TransportError` for connection failures.
- Paginated methods return a `PagePromise`: awaiting it returns the first `Page`; `for await (const item of client.x.list())` walks every page and throws the typed error if a page fails.
- Every method takes a last `{ timeoutMs, maxRetries, headers, signal }` argument for per-call overrides. Errors carry `code`, `status`, `requestId`, `body`, and an actionable message; pages carry response metadata.
- Uploads take a `Blob` (a `File` for a filename).
- `debug: true` (or a function) on the client logs one redacted line per request.

## Safety
- Read credentials from the environment or a secret store. Never hard-code them, print them, or put them in URLs or command arguments.
- Check an operation's `safety` in `api.json` before calling it. Confirm with the user before running a `write` or `destructive` operation they did not ask for.
- The client already retries transient failures, honoring `Retry-After`, and retries a write only when that is safe. Do not wrap calls in another retry loop: a repeated write can apply twice.

## Documentation
- The reference for this exact package: `api.md` (offline, always current with the code).
- Conceptual guides live on the docs site. For questions about how the API's concepts fit together (flows, ordering, environments), fetch `https://typeship.dev/llms-full.txt` and read the relevant sections; `https://typeship.dev/llms.txt` is the page index. Relative links in the spec resolve against `https://typeship.dev/docs`.
