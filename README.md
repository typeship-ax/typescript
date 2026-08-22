# typeship-ax

TypeScript SDK, CLI, and MCP server for typeship. [API reference](./api.md)

Generated from the OpenAPI spec by [typeship](https://typeship.dev). Change the spec or generation settings, then regenerate; generated files are not hand-edited.

- **Zero runtime dependencies** — built on the platform `fetch` (Node 18+, browsers, edge runtimes)
- **Typed error unions** — every call returns `ApiResult<T, E>` where `E` lists each documented error for that exact operation
- **Auto-pagination** — `for await` any list call to stream every item across every page
- **Retries built in** — idempotent requests retry with exponential backoff and `Retry-After` support
- **Optional runtime validation** — `validate: true` schema-checks request and response bodies against the spec, still zero dependencies
- **Tree-shakeable SDK** — per-resource modules, `sideEffects: false`
- **Spec-native CLI** — every operation is a command with typed flags, stable JSON, help, and exit codes
- **Agent-ready MCP** — schema-derived tools, argument validation, read-only mode, and bounded results

## Install

```sh
npm install typeship-ax
```

## TypeScript SDK

```ts
import { TypeshipClient } from "typeship-ax";

const client = new TypeshipClient({ bearerToken: process.env.TYPESHIP_TOKEN! });

for await (const item of client.projects.list()) {
  console.log(item);
}
```

## Authentication

- **Bearer token** — `bearerToken` (a string, or a callback for tokens that expire), sent as `Authorization: Bearer <token>`.

`defaultHeaders` adds headers to every request (API version headers, tenant ids); `onRequest` can rewrite any request before it is sent.

## Error handling

Awaiting a call returns a discriminated result instead of throwing on request errors.
The error side is a union of the documented error classes for that operation:

```ts
import { BadRequestError } from "typeship-ax";

const result = await client.projects.list();

if (!result.ok) {
  if (result.error instanceof BadRequestError) {
    // result.error.body is fully typed for this status
  }
  throw result.error; // every branch is an Error subclass
}

result.data; // typed success payload
```

Prefer exceptions? `unwrap(result)` returns the data or throws the typed error.

## Pagination

Await a list call to inspect its `ApiResult`, or iterate it for concise lazy pagination. Async iteration throws the same typed API error if any page fails:

```ts
for await (const item of client.projects.list()) {
  // every item from every page, fetched lazily
}

// or page manually:
const page = await client.projects.list();
if (page.ok) {
  page.data.items;
  await page.data.getNextPage();
}
```

## CLI

The package ships `typeship`, a command for every API operation. It writes JSON to stdout and uses exit codes 0/1/2 for success, request failure, and invalid usage.

```sh
npm install -g typeship-ax
typeship login                      # stores a credential (or set TYPESHIP_TOKEN)
typeship projects list
typeship projects list --all | jq -r '.id'   # every page, one item per line
typeship <resource> <command> --help     # flags, types, an example
```

CLI conventions:

- Path parameters are positional; query and body fields are flags named after their wire fields.
- Arrays accept a comma list or repeated flags. Objects accept JSON; `--data @file` and `--data -` read a full body.
- `--fields id,name` projects results. `--all` streams paginated results as NDJSON.
- Destructive commands require confirmation or `--force`. Piped errors are stable JSON on stderr.

Auth: `typeship login` stores a credential under `~/.config/typeship/`; the environment (`TYPESHIP_TOKEN`) and flags (`--token`) win over it. `TYPESHIP_BASE_URL` / `--base-url` pick the endpoint.

Useful commands:

- `typeship init` configures credentials, MCP clients, and repository agent instructions.
- `typeship mcp install --all` registers the MCP server with supported agent clients.
- `typeship docs search <term>`, `typeship doctor`, and `typeship help --json` cover discovery and diagnostics.

## MCP server

A zero-dependency stdio server exposing every operation as a tool. The fastest setup is `typeship mcp install --all`. For a manual MCP client configuration:

```json
{
  "mcpServers": {
    "typeship": {
      "command": "npx",
      "args": [
        "-y",
        "--package",
        "typeship-ax",
        "typeship-mcp"
      ]
    }
  }
}
```

Tool input schemas are derived from the OpenAPI spec, so agents see real parameter types and required fields. Arguments are checked before anything reaches the API (unknown or mistyped ones come back as one `isError` result, nothing is dropped), every tool takes `fields` to keep only the result keys it needs, and errors carry a stable `code` and `next_steps`.

Add `--read-only` to `args` (or set `TYPESHIP_MCP_READ_ONLY=1`) for a server that cannot write, `--tools accounts,reports` (or `TYPESHIP_MCP_TOOLS`) to expose a subset, and `TYPESHIP_MCP_MAX_RESULT_CHARS` to change the result size cap (64,000). `typeship mcp install --all --read-only` writes read-only entries for every detected client.

## SDK configuration

```ts
new TypeshipClient({
  baseUrl: "https://typeship.dev/api/v1", // default
  timeoutMs: 30_000, // per attempt
  maxRetries: 2,     // retryable failures only
  fetch: globalThis.fetch, // or your own: proxies, tests, instrumentation
});
```

Per-call overrides ride on the last argument: `{ timeoutMs, maxRetries, headers, signal }`.
