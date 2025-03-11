## API Routes

Next.js API routes with the App Router provide a secure way to interact with external services without exposing sensitive information to clients. The following sections detail the API routes needed for NFT minting.

### Minting API

This endpoint creates a new NFT minting transaction:

```typescript
// src/app/api/mint/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createTransaction, generateUniqueNFT } from '@/lib/anvil';
import { createOrLoadPolicy } from '@/lib/policy';
import { storeTransaction } from '@/lib/redis';

export async function POST(req: NextRequest) {
  try {
    // Get change address from request
    const { changeAddress } = await req.json();

    if (!changeAddress) {
      return NextResponse.json(
        { error: 'Missing change address' },
        { status: 400 }
      );
    }

    // Create policy and metadata
    const { keyHash, slot, policyId } = createOrLoadPolicy();
    const metadata = generateUniqueNFT();

    // Create asset object
    const asset = {
      version: 'cip25',
      assetName: { name: metadata.name, format: 'utf8' },
      metadata: {
        name: metadata.name,
        image: [
          'https://ada-anvil.s3.ca-central-1.amazonaws.com/',
          'logo_pres_V2_3.png',
        ],
        mediaType: 'image/png',
        description: 'Minting Platform Example using Anvil API',
        attributes: metadata.attributes,
      },
      policyId,
      quantity: 1,
    };

    // Create transaction
    const transaction = await createTransaction(
      changeAddress,
      asset,
      keyHash,
      slot,
      policyId,
    );

    // Store transaction in Redis
    await storeTransaction(transaction.hash, transaction);

    return NextResponse.json({
      success: true,
      transaction: {
        hash: transaction.hash,
        cbor: transaction.cbor,
      },
    });
  } catch (error) {
    console.error('Mint API error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Failed to create mint transaction' },
      { status: 500 }
    );
  }
}
```

### Transaction Submission API

This endpoint submits a signed transaction to the blockchain:

```typescript
// src/app/api/submit/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { submitTransaction } from '@/lib/anvil';
import { getTransaction } from '@/lib/redis';

export async function POST(req: NextRequest) {
  try {
    const { signedTx, txHash } = await req.json();

    if (!signedTx || !txHash) {
      return NextResponse.json(
        { error: 'Missing signed transaction or hash' },
        { status: 400 }
      );
    }

    // Verify that this is a transaction we created
    const storedTx = await getTransaction(txHash);
    if (!storedTx) {
      return NextResponse.json(
        { error: 'Transaction not found or expired' },
        { status: 404 }
      );
    }

    // Submit the signed transaction
    const result = await submitTransaction(signedTx);

    return NextResponse.json({
      success: true,
      result,
    });
  } catch (error) {
    console.error('Submit API error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Failed to submit transaction' },
      { status: 500 }
    );
  }
}
```

### Transaction Status API

This endpoint checks the status of a transaction:

```typescript
// src/app/api/transaction/[txId]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getTransactionStatus } from '@/lib/anvil';

export async function GET(
  req: NextRequest,
  { params }: { params: { txId: string } }
) {
  try {
    const txId = params.txId;
    
    if (!txId) {
      return NextResponse.json(
        { error: 'Missing transaction ID' },
        { status: 400 }
      );
    }
    
    const status = await getTransactionStatus(txId);
    
    return NextResponse.json(status);
  } catch (error) {
    console.error('Transaction status API error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Failed to get transaction status' },
      { status: 500 }
    );
  }
}
```
