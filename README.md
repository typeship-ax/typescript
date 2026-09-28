# @typeship-ax/sdk

TypeScript SDK for the Typeship API. [API reference](./api.md)

Resolve an OpenAPI or GraphQL Spec, diagnose it, and keep every selected CLI, MCP, and SDK Target current.

## Installation

```sh
npm install @typeship-ax/sdk@0.26.0
```

Requires Node.js 20+ or a modern browser or edge runtime with `fetch`, `AbortController`, and Web Streams. The package is ESM.

## Quickstart

```ts
import { TypeshipClient } from "@typeship-ax/sdk";

const client = new TypeshipClient({ bearerToken: process.env.TYPESHIP_TOKEN! });

const result = await client.organization.get();
console.log(result);
```

## Authentication

- **Bearer token**: `bearerToken` (a string, or a callback for tokens that expire), sent as `Authorization: Bearer <token>`.

`defaultHeaders` adds headers to every request (API version headers, tenant ids); `onRequest` can rewrite any request before it is sent.

## Error handling

Awaiting a call returns the response data. Failures throw typed errors.
Every HTTP error is an `ApiError`. Each status family has one class, raised whether or not the operation documents the status: `BadRequestError` (400), `UnauthorizedError` (401), `ForbiddenError` (403), `NotFoundError` (404), `ConflictError` (409), `UnprocessableEntityError` (422), `RateLimitError` (429), and `ServerError` (5xx). Parse, validation, and transport failures have distinct classes:

```ts
import { ResponseParseError, NotFoundError } from "@typeship-ax/sdk";

try {
  const result = await client.organization.get();

  console.log(result); // typed success payload
} catch (error) {
  if (error instanceof ResponseParseError) {
    console.error(error.body); // malformed successful JSON, preserved as text
  }
  if (error instanceof NotFoundError) {
    console.error(error.message, error.code); // the API's own message and code
  }
  throw error;
}

```

Every error exposes `code`, `status`, `requestId`, `body`, and an actionable `message`. Every failure throws; no method returns an error as a value.

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

## Configuration

```ts
new TypeshipClient({
  baseUrl: "https://typeship.dev/api/v1", // default
  timeoutMs: 60_000, // per attempt
  maxRetries: 2,     // retryable failures only
  fetch: globalThis.fetch, // or your own: proxies, tests, instrumentation
});
```

Per-call overrides ride on the last argument: `{ timeoutMs, maxRetries, headers, signal }`.

`validate: true` checks request and response bodies against the spec's schemas at runtime, with no added dependencies.

Timeouts apply to each attempt. By default, the client makes up to two retries for `408`, `429`, `500`, `502`, `503`, and `504`; non-idempotent calls retry only on `429`, when the operation declares an idempotency key, or when explicitly enabled. `Retry-After` takes precedence over exponential backoff.

Use `onRequest`, `onResponse`, and `onError` for instrumentation. `debug` receives one structured event per attempt and never includes headers or bodies.

Generated from the OpenAPI spec by [Typeship](https://typeship.dev).
