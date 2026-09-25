# @typeship-ax/sdk

TypeScript SDK for typeship. [API reference](./api.md)

Generated from the OpenAPI spec by [typeship](https://typeship.dev).

- **Zero runtime dependencies** — built on the platform `fetch` (Node 20+, browsers, edge runtimes)
- **Typed errors** — calls resolve to response data and throw per-status API errors or transport, parse, and validation errors
- **Auto-pagination** — `for await` any list call to stream every item across every page
- **Retries built in** — idempotent requests retry with exponential backoff and `Retry-After` support
- **Forward-compatible responses** — request enums stay closed, while response enums and discriminator unions preserve values the server added after this package was generated
- **Optional runtime validation** — `validate: true` schema-checks request and response bodies against the spec, still zero dependencies
- **Tree-shakeable SDK** — per-resource modules, `sideEffects: false`

## Build from source

Run these commands in the downloaded or cloned package directory:

```sh
npm install
npm run build
```

Requires Node.js 20+ or a modern browser/edge runtime with `fetch`, `AbortController`, and Web Streams. The package is ESM.

Save the quickstart example below in the package directory. The package import resolves to the local build.

## Install a published package

Generation does not publish a package. Before using the registry command below, confirm `name` and `version` in `package.json`, publish under a name you control, and verify that release is available on npm.

```sh
npm install @typeship-ax/sdk@0.26.0
```

## Quickstart

```ts
import { TypeshipClient } from "@typeship-ax/sdk";

const client = new TypeshipClient({ bearerToken: process.env.TYPESHIP_TOKEN! });

const result = await client.organization.get();
console.log(result);
```

## Authentication

- **Bearer token** — `bearerToken` (a string, or a callback for tokens that expire), sent as `Authorization: Bearer <token>`.

`defaultHeaders` adds headers to every request (API version headers, tenant ids); `onRequest` can rewrite any request before it is sent.

## Error handling

Awaiting a call returns the response data. Failures throw typed errors.
Documented HTTP errors have per-status classes. Parse, validation, and transport failures have distinct classes:

```ts
import { ResponseParseError, UnauthorizedError } from "@typeship-ax/sdk";

try {
  const result = await client.organization.get();

  console.log(result); // typed success payload
} catch (error) {
  if (error instanceof ResponseParseError) {
    console.error(error.body); // malformed successful JSON, preserved as text
  }
  if (error instanceof UnauthorizedError) {
    // error.body is fully typed for this status
  }
  throw error;
}

```

Every error exposes `code`, `status`, `requestId`, `body`, and an actionable `message`. No non-throwing SDK variant is generated.

## Pagination

Await a list call for its first `Page`, or iterate it for lazy pagination. Both paths throw the typed error when a page fails:

```ts
for await (const item of client.projects.list()) {
  // every item from every page, fetched lazily
}

// or page manually:
const page = await client.projects.list();
page.items;
await page.getNextPage();
```

## Response metadata

Paginated pages expose `page.response` with status, headers, request id, and raw body. For other successful calls, use `onResponse` to observe HTTP metadata. HTTP and response-parse errors expose response metadata on `error.response`.

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

Timeouts apply to each attempt. By default, the client makes up to two retries for `408`, `429`, `500`, `502`, `503`, and `504`; non-idempotent calls retry only on `429`, when the operation declares an idempotency key, or when explicitly enabled. `Retry-After` takes precedence over exponential backoff.

Use `onRequest`, `onResponse`, and `onError` for instrumentation. `debug` receives one structured event per attempt and never includes headers or bodies.
