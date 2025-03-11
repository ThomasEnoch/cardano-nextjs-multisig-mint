### Minting Form

This component handles the minting process, guiding users through transaction creation, signing, and submission:

```typescript
// src/components/MintForm.tsx
'use client';

import { useState } from 'react';
import { AnvilWallet, Eternl } from '@ada-anvil/weld';

type MintStatus = 'idle' | 'creating' | 'created' | 'signing' | 'submitting' | 'completed';

export default function MintForm() {
  const [status, setStatus] = useState<MintStatus>('idle');
  const [error, setError] = useState<string>('');
  const [txHash, setTxHash] = useState<string>('');
  const [txCbor, setTxCbor] = useState<string>('');
  const [signedTx, setSignedTx] = useState<string>('');
  const [resultTxId, setResultTxId] = useState<string>('');
  
  const mintNFT = async () => {
    // Reset states
    setError('');
    setTxHash('');
    setTxCbor('');
    setSignedTx('');
    setResultTxId('');
    
    try {
      // Get wallet address
      const addressElement = document.getElementById('changeAddressBech32');
      if (!addressElement || !addressElement.textContent) {
        throw new Error('Please connect your wallet first');
      }
      
      const changeAddress = addressElement.textContent.trim();
      
      // Create transaction
      setStatus('creating');
      const response = await fetch('/api/mint', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ changeAddress }),
      });
      
      const data = await response.json();
      
      if (!response.ok) {
        throw new Error(data.error || 'Failed to create mint transaction');
      }
      
      setTxHash(data.transaction.hash);
      setTxCbor(data.transaction.cbor);
      setStatus('created');
      
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An unknown error occurred');
      setStatus('idle');
    }
  };
  
  const signTransaction = async () => {
    if (!txCbor) {
      setError('No transaction to sign');
      return;
    }
    
    setStatus('signing');
    setError('');
    
    try {
      // Initialize Eternl wallet
      const eternl = new Eternl();
      
      if (!(await eternl.isEnabled())) {
        await eternl.enable();
      }
      
      // Sign the transaction
      const signedTxCbor = await eternl.signTx(txCbor, true);
      setSignedTx(signedTxCbor);
      
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An unknown error occurred while signing');
      setStatus('created'); // Revert to created state
    }
  };
  
  const submitSignedTx = async () => {
    if (!signedTx || !txHash) {
      setError('No signed transaction to submit');
      return;
    }
    
    setStatus('submitting');
    setError('');
    
    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          signedTx,
          txHash,
        }),
      });
      
      const data = await response.json();
      
      if (!response.ok) {
        throw new Error(data.error || 'Failed to submit transaction');
      }
      
      setResultTxId(data.result.txId || 'Transaction submitted');
      setStatus('completed');
      
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An unknown error occurred while submitting');
      setStatus('created'); // Revert to created state
    }
  };
  
  return (
    <div className="p-6 bg-white shadow-md rounded-lg">
      <h2 className="text-2xl font-semibold mb-6">Mint NFT</h2>
      
      <div className="space-y-6">
        {/* Step 1: Create Transaction */}
        <div>
          <h3 className="text-lg font-medium mb-2">Step 1: Create Transaction</h3>
          <button 
            onClick={mintNFT}
            disabled={status !== 'idle' && status !== 'completed'}
            className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 disabled:bg-blue-300"
            aria-busy={status === 'creating'}
          >
            {status === 'creating' ? 'Creating...' : 'Create Mint Transaction'}
          </button>
        </div>
        
        {/* Step 2: Sign Transaction */}
        {(status === 'created' || status === 'signing') && (
          <div>
            <h3 className="text-lg font-medium mb-2">Step 2: Sign Transaction</h3>
            <p className="text-sm text-gray-600 mb-2">Transaction Hash: {txHash}</p>
            <button 
              onClick={signTransaction}
              disabled={status === 'signing'}
              className="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700 disabled:bg-green-300"
              aria-busy={status === 'signing'}
            >
              {status === 'signing' ? 'Signing...' : 'Sign with Eternl'}
            </button>
          </div>
        )}
        
        {/* Step 3: Submit Transaction */}
        {signedTx && (status === 'created' || status === 'submitting') && (
          <div>
            <h3 className="text-lg font-medium mb-2">Step 3: Submit Transaction</h3>
            <button 
              onClick={submitSignedTx}
              disabled={status === 'submitting'}
              className="bg-purple-600 text-white px-4 py-2 rounded hover:bg-purple-700 disabled:bg-purple-300"
              aria-busy={status === 'submitting'}
            >
              {status === 'submitting' ? 'Submitting...' : 'Submit Signed Transaction'}
            </button>
          </div>
        )}
        
        {/* Success Message */}
        {status === 'completed' && (
          <div className="bg-green-100 p-4 rounded-md">
            <h3 className="text-lg font-medium text-green-800 mb-2">NFT Minted Successfully!</h3>
            <p className="text-green-700">Transaction ID: {resultTxId}</p>
          </div>
        )}
        
        {/* Error Message */}
        {error && (
          <div className="bg-red-100 p-4 rounded-md" role="alert">
            <h3 className="text-lg font-medium text-red-800 mb-2">Error</h3>
            <p className="text-red-700">{error}</p>
          </div>
        )}
      </div>
    </div>
  );
}
```

The minting form implements a step-by-step process that guides users through the entire NFT minting flow, with clear visual indicators at each stage. Following the "Fail Fast, Fail Loud" principle, it provides comprehensive error handling and user-friendly error messages.
