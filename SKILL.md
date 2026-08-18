---
name: typeship
description: Work with the typeship API: authentication, the generated TypeScript SDK, CLI, MCP server, and where its documentation lives.
---

This package is the generated TypeScript SDK + CLI + MCP server for **typeship** (v1.0.0).

Generate a zero-dependency SDK — TypeScript, Python, or Go — plus a CLI
and an MCP server, from an OpenAPI spec.

Every operation requires an API key, created in the console and sent as
`Authorization: Bearer tsk_live_...`. A browser session is not a
credential for this API.

## Ground rules
- Generated code: never edit files in this package by hand — changes are lost on regeneration. Wrap the client in your own code instead.
- Zero runtime dependencies; everything runs on platform `fetch` (Node 18+, browsers, edge).
- `api.md` in this package is the complete method reference: every operation, parameter, and error class. Read it before guessing.

## Authentication
- Bearer token: `TYPESHIP_TOKEN` env var, or the `bearerToken` client option.

## Using the SDK
```ts
import { TypeshipClient } from "typeship";
const client = new TypeshipClient({ /* auth options above */ });
```
- Every call returns `ApiResult<T, E>`: check `result.ok`, or `unwrap(result)` to throw the typed error. Nothing throws on HTTP errors by default.
- Paginated methods return a `PagePromise`: `for await (const item of client.x.list())` walks every page.
- Every method takes a last `{ timeoutMs, maxRetries, headers, signal }` argument for per-call overrides; every result carries `response: { status, headers, requestId }`.
- Uploads take a `Blob` (a `File` for a filename); GraphQL operations take a raw selection set as their second argument.
- `debug: true` (or a function) on the client logs one redacted line per request.

## Using the CLI
- `typeship <resource> <command>` mirrors the SDK exactly; `typeship docs search <term>` finds operations and guides; `typeship docs <resource> <command>` prints the full reference.
- Flags only, JSON to stdout, exit codes 0/1/2 — safe to run from scripts and agents.

## MCP server
- `typeship mcp --claude` wires the bundled MCP server into agent clients; it shares credentials with `typeship login`.

## Documentation
- The reference for this exact package: `api.md` (offline, always current with the code).
- Conceptual guides live on the docs site. For questions about how the API's concepts fit together (flows, ordering, environments), fetch `https://typeship.dev/llms-full.txt` and read the relevant sections; `https://typeship.dev/llms.txt` is the page index.
