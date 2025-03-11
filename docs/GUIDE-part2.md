## Environment Setup

Never commit your real API keys or environment variables to version control. Always use `.env.local` for local development and secure environment variable management for production.

Create a `.env.local` file:

```
ANVIL_API_URL=https://preprod.api.ada-anvil.app/v2/services
ANVIL_API_KEY=your_api_key_here
REDIS_URL=redis://localhost:6379
TREASURY_ADDRESS=addr_test1...
```

Create a `.env.example` file with the same keys but without values for documentation purposes:

```
ANVIL_API_URL=
ANVIL_API_KEY=
REDIS_URL=
TREASURY_ADDRESS=
```

## Setting Up Redis for Transaction Storage

Redis will serve as a temporary storage for transaction data during the minting process. Let's create a utility file for Redis operations:

```typescript
// src/lib/redis.ts
import { createClient } from 'redis';

let redisClient: ReturnType<typeof createClient> | null = null;

export async function getRedisClient() {
  if (!redisClient) {
    redisClient = createClient({
      url: process.env.REDIS_URL
    });
    
    redisClient.on('error', (err) => console.error('Redis Client Error', err));
    
    if (!redisClient.isOpen) {
      await redisClient.connect();
    }
  }
  
  return redisClient;
}

export async function storeTransaction(txHash: string, txData: any, expirySeconds = 300) {
  try {
    const client = await getRedisClient();
    await client.set(txHash, JSON.stringify(txData), {
      EX: expirySeconds,
      NX: true,
    });
    return true;
  } catch (error) {
    console.error('Error storing transaction:', error);
    return false;
  }
}

export async function getTransaction(txHash: string) {
  try {
    const client = await getRedisClient();
    const data = await client.get(txHash);
    return data ? JSON.parse(data) : null;
  } catch (error) {
    console.error('Error retrieving transaction:', error);
    return null;
  }
}
```

## Anvil API Integration

Next, let's create a utility file for interacting with the Anvil API:

```typescript
// src/lib/anvil.ts
import { randomBytes } from 'crypto';

const ANVIL_API_URL = process.env.ANVIL_API_URL;
const ANVIL_API_KEY = process.env.ANVIL_API_KEY;

// Ensure headers are set properly for all API calls
const getHeaders = () => ({
  'Content-Type': 'application/json',
  'X-Api-Key': ANVIL_API_KEY || '',
});

type Character = {
  attributes: Record<string, string>;
  name: string;
};

/**
 * Generate a unique NFT with random attributes
 */
export function generateUniqueNFT(): Character {
  return {
    attributes: {
      part1: randomBytes(8).toString('hex'),
      part2: randomBytes(8).toString('hex'),
      part3: randomBytes(8).toString('hex'),
    },
    name: `0x${new Date().getTime()}`,
  };
}

/**
 * Convert a date to a Cardano slot number
 */
export function dateToSlot(date: Date): number {
  return Math.floor(date.getTime() / 1000) - 1596491091 + 4924800;
}
```
