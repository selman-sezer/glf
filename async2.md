# AsyncCaller

> Asynchronous Call Manager with Retry, Concurrency, and Rate Limiting

## Features

- **Rate Limiting**: Control the rate of requests with a token bucket algorithm.
- **Retry Mechanism**: Automatically retry failed requests with configurable backoff strategy.
- **Concurrency Control**: Manage the number of concurrent tasks.
- **Verbose Logging**: Optional detailed logging for debugging.

## Installation

Using npm:

```bash
npm install async-caller
```

Using yarn:

```bash
yarn add async-caller
```

Using pnpm:

```bash
pnpm add async-caller
```

## Getting Started

### Basic Usage

```js
import { AsyncCaller } from "async-caller";

const asyncCaller = new AsyncCaller({
  tokenBucketOptions: {
    capacity: 10,
    fillPerWindow: 10,
    windowInMs: 1000,
  },
});

asyncCaller.call(async () => {
  // Your async function here
});
```

### Advanced Usage

```js
import { AsyncCaller } from "async-caller";

const asyncCaller = new AsyncCaller({
  tokenBucketOptions: {
    capacity: 20,
    fillPerWindow: 100,
    windowInMs: 60000,
  },
  retryOptions: {
    maxRetries: 5,
    minDelayInMs: 1000,
    maxDelayInMs: 20000,
    backoffFactor: 2,
  },
  concurrency: 10,
}, true); // Enable verbose logging

asyncCaller.call(async () => {
  // Your async function here
});
```

## Configuration Options

### TokenBucketOptions

- `capacity`: The maximum number of requests allowed in a window.
- `fillPerWindow`: The number of requests to allow per window. This determines the rate at which requests are allowed.
- `windowInMs`: The size of the window in milliseconds.
- `initialTokens`: The initial number of allowed requests. If not provided, it defaults to the capacity. Setting it to a lower value can be useful for gradually ramping up the rate.

### RetryOptions

- `maxRetries`: The maximum number of retry attempts.
- `minDelayInMs`: The minimum delay between retries.
- `maxDelayInMs`: The maximum delay between retries.
- `backoffFactor`: The factor by which the delay increases after each retry.

### Concurrency

- `concurrency`: The maximum number of concurrent tasks allowed.

## Methods

### `call(fn)`

Executes the provided asynchronous function with rate limiting, retry, and concurrency control.

```js
asyncCaller.call(async () => {
  // Your async function here
});
```

### `isClientSideError(errOrResponse)`

Checks if the error or response is a client-side error (status code 400-499).

### `isRateLimitedError(errOrResponse)`

Checks if the error or response is a rate-limited error (status code 429).

### `extractStatusCodeProperties(err)`

Extracts status code properties from an error or response object.

## Example

```js
import { AsyncCaller } from "async-caller";

const asyncCaller = new AsyncCaller({
  tokenBucketOptions: {
    capacity: 10,
    fillPerWindow: 10,
    windowInMs: 1000,
  },
  retryOptions: {
    maxRetries: 3,
    minDelayInMs: 1000,
    maxDelayInMs: 10000,
    backoffFactor: 2,
  },
  concurrency: 5,
}, true); // Enable verbose logging

asyncCaller.call(async () => {
  const response = await fetch("https://api.example.com/data");
  const data = await response.json();
  console.log(data);
});
```

## License

MIT License
