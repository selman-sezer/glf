# 2 -  TokenBucket

TokenBucket is a TypeScript library implementing a token bucket algorithm for rate limiting.

## Features

- **Rate Limiting**: Control the rate of requests using a token bucket algorithm.
- **Verbose Logging**: Optional verbose logging for debugging and monitoring.

## Installation

To install the package, use npm or yarn:

```sh
npm install token-bucket
```

or

```sh
yarn add token-bucket
```

## Usage

### Basic Usage

```typescript
import { TokenBucket } from 'token-bucket';

const tokenBucket = new TokenBucket({
  capacity: 10,
  fillPerWindow: 1,
  windowInMs: 1000,
});

if (tokenBucket.consume()) {
  // Your logic when a token is consumed successfully
}
```

### Advanced Usage with Verbose Logging

```typescript
import { TokenBucket } from 'token-bucket';

const tokenBucket = new TokenBucket({
  capacity: 20,
  fillPerWindow: 5,
  windowInMs: 2000,
}, true);

async function makeRequest() {
  if (await tokenBucket.consumeAsync()) {
    // Your async logic when a token is consumed successfully
  } else {
    console.log("Rate limit exceeded. Waiting for tokens...");
  }
}

makeRequest();
```

### Waiting for enough Token to be in Tokenbucket

```typescript
import { TokenBucket } from 'token-bucket';

const tokenBucket = new TokenBucket({
  capacity: 20,
  fillPerWindow: 5,
  windowInMs: 2000,
}, true);

// This while loop waits for tokenBucket to asynchronously consumes the specified number of tokens.
let numberOfTokensToBeConsumed = 5;
while (!await tokenBucket.consumeAsync(numberOfTokensToBeConsumed));
```


## Configuration Options

### TokenBucketOptions

- **capacity**: The maximum number of tokens in the bucket.
- **fillPerWindow**: The number of tokens added to the bucket per window.
- **windowInMs**: The size of the window in milliseconds.
- **initialTokens**: The initial number of tokens in the bucket. Defaults to the capacity if not provided.

## Methods

### consume(amount: number = 1): boolean

Consumes the specified number of tokens from the bucket. Returns `true` if the tokens were successfully consumed, otherwise `false`.

### consumeAsync(amount: number = 1): Promise<boolean>

Asynchronously consumes the specified number of tokens from the bucket. Returns a promise that resolves to `true` if the tokens were successfully consumed, otherwise `false`.

### start(): TokenBucket

Starts the internal timer for refilling tokens. Returns the `TokenBucket` instance to allow chaining.

### stop(): void

Stops the internal timer for refilling tokens.

### forceWaitUntilMilisecondsPassed(delayInMs: number): void

Forces the bucket to wait until the specified amount of milliseconds have passed before refilling.

## License

This project is licensed under the MIT License.




---
---
---



# 1 - AsyncCaller

AsyncCaller is a TypeScript library for making asynchronous calls with retry, concurrency, and rate limiting capabilities.

## Features

- **Rate Limiting**: Control the rate of requests using a token bucket algorithm.
- **Retry Mechanism**: Automatically retry failed requests with customizable retry options.
- **Concurrency Control**: Limit the number of concurrent tasks.

## Installation

To install the package, use npm or yarn:

```sh
npm install async-caller
```

or

```sh
yarn add async-caller
```

## Usage

### Basic Usage

```typescript
import { AsyncCaller } from 'async-caller';

const asyncCaller = new AsyncCaller({
  tokenBucketOptions: {
    capacity: 10,
    fillPerWindow: 10,
    windowInMs: 1000,
  },
});

async function fetchData() {
  // Your async function logic
}

asyncCaller.call(fetchData)
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### Advanced Usage

```typescript
import { AsyncCaller } from 'async-caller';

const asyncCaller = new AsyncCaller({
  tokenBucketOptions: {
    capacity: 20,
    fillPerWindow: 100,
    windowInMs: 60000,
  },
  retryOptions: {
    maxRetries: 5,
    minDelayInMs: 500,
    maxDelayInMs: 20000,
    backoffFactor: 2,
  },
  concurrency: 10,
});

async function fetchData() {
  // Your async function logic
}

asyncCaller.call(fetchData)
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

## Configuration Options

### TokenBucketOptions

- **capacity**: The maximum number of requests allowed in a window.
- **fillPerWindow**: The number of requests to allow per window. This determines the rate at which requests are allowed.
- **windowInMs**: The size of the window in milliseconds.
- **initialTokens**: The initial number of allowed requests. If not provided, it defaults to the capacity.

### RetryOptions

- **maxRetries**: The maximum number of retries. Default is 3.
- **minDelayInMs**: The minimum delay between retries in milliseconds. Default is 1000.
- **maxDelayInMs**: The maximum delay between retries in milliseconds. Default is 10000.
- **backoffFactor**: The factor by which the delay should be increased after each retry. Default is 2.

### Concurrency

- **concurrency**: The maximum number of concurrent tasks allowed. Default is 5.


