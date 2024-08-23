# TokenBucket

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

