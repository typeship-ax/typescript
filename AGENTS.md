# typeship — agent context

This package contains the generated TypeScript SDK + CLI + MCP server for **typeship** (v0.6.0).

Generate production SDKs, CLIs, and MCP servers from an OpenAPI or
GraphQL spec, and keep every selected output current.

Every operation but one requires an API key, created in the console and
sent as `Authorization: Bearer ak_...`. A browser session is not a
credential for this API. The exception is POST /generate, which works
anonymously with the free plan's limits.

## Ground rules
- Generated code: never edit files in this package by hand — changes are lost on regeneration. Wrap the client in your own code instead.
- Zero runtime dependencies; everything runs on platform `fetch` (Node 18+, browsers, edge).
- `api.md` is the native method reference; `api.json` is the machine-readable operation, schema, safety, and example contract. Read them before guessing.

## Authentication
- Bearer token: `TYPESHIP_TOKEN` env var, or the `bearerToken` client option.

## Using the SDK
```ts
import { TypeshipClient } from "typeship-ax";
const client = new TypeshipClient({ /* auth options above */ });
```
- Awaiting a call returns `ApiResult<T, E>`: check `result.ok`, or `unwrap(result)` to throw the typed error.
- Paginated methods return a `PagePromise`: awaiting it returns the first page's `ApiResult`; `for await (const item of client.x.list())` walks every page and throws the typed API error if a page fails.
- Every method takes a last `{ timeoutMs, maxRetries, headers, signal }` argument for per-call overrides; every result carries `response: { status, headers, requestId }`.
- Uploads take a `Blob` (a `File` for a filename).
- `debug: true` (or a function) on the client logs one redacted line per request.

## Using the CLI
- `typeship <resource> <command>` exposes the same API operations as the SDK; `typeship docs search <term>` finds operations and guides; `typeship docs <resource> <command>` gives a concise contract and example (add `--schema` for full schemas or `--json` for the machine contract).
- Path parameters are positional; other inputs are flags. JSON goes to stdout and exit codes are 0/1/2. Errors are one JSON envelope on stderr: `{status, issues: [{code, message}], next_steps, detail}`; branch on `issues[].code`. Every operation classified as destructive requires `--force` (or `--yes`).
- `typeship agent-guide --format json` explains the conventions; `typeship help --json` is the command surface as data; `typeship init --all` connects this machine (credential, MCP clients, an AGENTS.md block); `typeship doctor` checks it.

## MCP server
- `typeship mcp install --all` writes the MCP server into every agent client found on the machine; it shares credentials with `typeship login` (or references the auth env var for a hosted endpoint). `--read-only` registers a server that cannot write.
- Tool arguments are checked against the schema before any request (unknown or mistyped arguments are one `isError` result with per-argument issues); pass `fields` (dotted paths) to keep only the result keys you need; errors carry `code` and `next_steps`.

## Documentation
- The reference for this exact package: `api.md` (offline, always current with the code).
- Conceptual guides live on the docs site. For questions about how the API's concepts fit together (flows, ordering, environments), fetch `https://typeship.dev/llms-full.txt` and read the relevant sections; `https://typeship.dev/llms.txt` is the page index.
