# @typeship-ax/sdk

TypeScript SDK for typeship. [API reference](./api.md)

Generated from the OpenAPI spec by [typeship](https://typeship.dev). Change the spec or generation settings, then regenerate; generated files are not hand-edited.

- **Zero runtime dependencies** — built on the platform `fetch` (Node 18+, browsers, edge runtimes)
- **Typed error unions** — every call returns `ApiResult<T, E>` where `E` lists each documented error for that exact operation
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

Requires Node.js 18+ or a modern browser/edge runtime with `fetch`, `AbortController`, and Web Streams. The package is ESM.

Save the quickstart example below in the package directory. The package import resolves to the local build.

## Install a published package

Generation does not publish a package. Before using the registry command below, confirm `name` and `version` in `package.json`, publish under a name you control, and verify that release is available on npm.

```sh
npm install @typeship-ax/sdk@0.17.1
```

## Quickstart

```ts
import { TypeshipClient } from "@typeship-ax/sdk";

const client = new TypeshipClient({ bearerToken: process.env.TYPESHIP_TOKEN! });

const result = await client.account.retrieve();
if (result.ok) {
  console.log(result.data);
}
```

## Authentication

- **Bearer token** — `bearerToken` (a string, or a callback for tokens that expire), sent as `Authorization: Bearer <token>`.

`defaultHeaders` adds headers to every request (API version headers, tenant ids); `onRequest` can rewrite any request before it is sent.

## Error handling

Awaiting a call returns a discriminated result instead of throwing on request errors.
The error side includes the documented HTTP errors for that operation plus `ResponseParseError`, validation, and transport failures:

```ts
import { ResponseParseError, UnauthorizedError } from "@typeship-ax/sdk";

const result = await client.account.retrieve();

if (!result.ok) {
  if (result.error instanceof ResponseParseError) {
    console.error(result.error.body); // malformed successful JSON, preserved as text
  }
  if (result.error instanceof UnauthorizedError) {
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

## Response metadata

Every successful result includes `response.status`, `response.headers`, `response.requestId`, and `response.rawBody`. HTTP and response-parse errors expose the same metadata on `error.response`. A transport failure can still include `result.response` when headers arrived before the body read failed.

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
