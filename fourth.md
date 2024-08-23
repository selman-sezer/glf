# 1 - TokenBucket

This project implements a **Token Bucket** algorithm for rate-limiting. The token bucket is a simple mechanism to control how many actions (e.g., API requests) can be performed over a given time window. The bucket accumulates tokens at a steady rate which you can fine-tune and stop it for a duration, and each action consumes a token. If no tokens are available, the action can either fail or wait for tokens to refill.

## Features
- **Asynchronous Consumption**: Supports waiting for tokens asynchronously when they are unavailable.
- **Customizable Refill Rates**: Allows fine-tuning of bucket capacity, refill rate, and time windows.
- **Force Stop and Refill**: Can force a bucket to wait and stop refilling for a specified duration.
- **Verbose Mode**: Logs the state of the token bucket for debugging purposes.


## Getting Started

### Prerequisites
- **Node.js**: Ensure that you have Node.js installed on your machine.
- **TypeScript**: This project uses TypeScript for type safety. You should have `tsc` installed, or use the provided compiled JavaScript if applicable.

## Usage

### Get Started

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

### Usage with Verbose Logging

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

### Wait for enough Tokens to be in a Tokenbucket

```typescript
import { TokenBucket } from 'token-bucket';

const tokenBucket = new TokenBucket({
  capacity: 20,
  fillPerWindow: 5,
  windowInMs: 2000,
}, true);

// This while loop waits for tokenBucket to asynchronously consumes the specified number of tokens, 5 in this case.
let numberOfTokensToBeConsumed = 5;
while (!await tokenBucket.consumeAsync(numberOfTokensToBeConsumed));
```


## Configuration Options

### TokenBucketOptions

- **capacity**: The maximum number of tokens in the bucket.
- **fillPerWindow**: The number of tokens added to the bucket per window.
- **windowInMs**: The size of the window in milliseconds.
- **initialTokens**: The initial number of tokens in the bucket. Defaults to the capacity if not provided.
```typescript
/*
capacity is set to 20, so bucket can hold maximum 20 tokens in a given moment.
fillPerWindow is set to 2, so at each window 2 tokens will be put into the bucket.
windowInMs is set to 1500.
initialTokens is set to 40, so initial number of tokens in the bucket is no longer defaul value (i.e. capacity which is set to 20).
*/
import { TokenBucket } from 'token-bucket';

const tokenBucket = new TokenBucket({
  capacity: 20,
  fillPerWindow: 2,
  windowInMs: 1500,
  initialTokens: 40,
});

```

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
The intented use of this method is the when the an AsyncCaller object which is explained below receives an 429 Too Many Requests error. The related methods of the AsyncCaller class are implemented accordingly. We advice not to use it anywhere else unless it is absolutely necessary.


---
---
---



# 2 - AsyncCaller

AsyncCaller is a TypeScript library for making asynchronous calls with retry, concurrency, and rate limiting capabilities.

## Features

- **Rate Limiting**: Control the rate of requests using a token bucket algorithm.
- **Retry Mechanism**: Automatically retry failed requests with customizable retry options.
- **Automatic Check For 429 Erros**:If the function sent to the async caller is a function called with fetch(), it automatically checks the incoming headers for 429 and sets the tokenbucket accordingly.
- **Other Errors**: If the function sent to the async caller is fetch, it does not retry with error codes 400 and 500. (except 429)
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

### Get Started

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

### Set the retryOptions

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


### Sending All Requests At Once
```typescript
/*
With the help of async caller, you can completely transfer the rate limiting issue to the async caller function and
send all requests at once with await Promise.all.
async caller works in accordance with the given limits.
Thanks to the async caller, your site can be used as quickly as possible within the specified rate limits.
*/
// Without async caller
  for (const user of users) {
    const userData = await fetch(https://www.somewebsite.com/fetchuserinfo/${user.id});
    // do something with userData
  }
  // too slow, each request has to wait the previous one to complete.

  // Without async caller
  await Promise.all(users.map(async user => {
    const userData = await fetch(https://www.somewebsite.com/fetchuserinfo/${user.id});
    // do something with userData
  }));
  // sends all of them together, but will probably get rate limited.

  // with async caller
  const asyncCaller = new AsyncCaller({
    tokenBucketOptions: {
      capacity: 10,
      fillPerWindow: 10,
      windowInMs: 1000,
    },
  });
  await Promise.all(users.map(async user => {
    const userData = await asyncCaller.call(async () => fetch(https://www.somewebsite.com/fetchuserinfo/${user.id}));
    // do something with userData
  }));

```

## Configuration Options

### TokenBucketOptions

- **capacity**: The maximum number of requests allowed in a window.
- **fillPerWindow**: The number of requests to allow per window. This determines the rate at which requests are allowed.
- **windowInMs**: The size of the window in milliseconds.
- **initialTokens**: The initial number of allowed requests. If not provided, it defaults to the capacity.

### RetryOptions

- **maxRetries**: The maximum number of retries. Default is 3. The maxRetries is the number of the extra tries.
For example if maxRetries is set to 10, the total number of tries would be 11 with the first try.
- **minDelayInMs**: The minimum delay between retries in milliseconds. Default is 1000.
- **maxDelayInMs**: The maximum delay between retries in milliseconds. Default is 10000.
- **backoffFactor**: The factor by which the delay should be increased after each retry. Default is 2.
```typescript
/* In this example maxRetries is set to 1. It means that in total it will be tried two times (i.e. once for first try, once for retry.
  The minimum delay between retries (i.e. minDelayInMs) is set to 100 milliseconds.
  The maximum delay between retries (i.e. maxDelayInMs) is set to 10000 milliseconds.
  backoffFactor is set to 3, so the delay will be increased 3 times after each retry.
*/
import { AsyncCaller } from 'async-caller';

const asyncCaller = new AsyncCaller({
  retryOptions: {
    maxRetries: 1,
    minDelayInMs: 100,
    maxDelayInMs: 10000,
    backoffFactor: 3,
  },
});
```
### Concurrency

- **concurrency**: The maximum number of concurrent tasks allowed. Default is 5.
```typescript
/* concurrency is set to 10. So maximum 10 tasks can be handles concurrently.
*/
import { AsyncCaller } from 'async-caller';

const asyncCaller = new AsyncCaller({
    retryOptions: {
    maxRetries: 10,
    minDelayInMs: 300,
    maxDelayInMs: 15000,
    backoffFactor: 2,
  },
  concurrency: 10,
});
```
