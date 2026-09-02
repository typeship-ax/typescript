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

## Install

```sh
npm install @typeship-ax/sdk
```

## TypeScript SDK

```ts
import { TypeshipClient } from "@typeship-ax/sdk";

const client = new TypeshipClient({ bearerToken: process.env.SDK_TOKEN! });

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
import { BadRequestError } from "@typeship-ax/sdk";

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
