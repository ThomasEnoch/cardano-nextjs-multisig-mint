## UI Components

These components handle the client-side interactions in our application. They are designed following React best practices and accessibility standards.

### Wallet Connector

This component handles connecting to the Eternl wallet using the Weld library:

```typescript
// src/components/WalletConnector.tsx
'use client';

import { useState, useEffect } from 'react';
import { AnvilWallet, Eternl } from '@ada-anvil/weld';

export default function WalletConnector() {
  const [wallet, setWallet] = useState<AnvilWallet | null>(null);
  const [address, setAddress] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<string>('');

  // Check if wallet is already connected
  useEffect(() => {
    async function checkWalletConnection() {
      try {
        // Initialize Eternl wallet
        const eternl = new Eternl();
        
        // Check if wallet is already enabled
        if (await eternl.isEnabled()) {
          setWallet(eternl);
          
          // Get first address
          const addresses = await eternl.getUsedAddresses();
          if (addresses.length > 0) {
            setAddress(addresses[0]);
          }
        }
      } catch (err) {
        console.error('Error checking wallet connection:', err);
      }
    }
    
    checkWalletConnection();
  }, []);

  const connectWallet = async () => {
    setLoading(true);
    setError('');
    
    try {
      // Initialize Eternl wallet
      const eternl = new Eternl();
      
      // Enable wallet
      await eternl.enable();
      setWallet(eternl);
      
      // Get first address
      const addresses = await eternl.getUsedAddresses();
      if (addresses.length > 0) {
        setAddress(addresses[0]);
      } else {
        throw new Error('No addresses found in wallet');
      }
    } catch (err) {
      console.error('Error connecting wallet:', err);
      setError(err instanceof Error ? err.message : 'Failed to connect wallet');
    } finally {
      setLoading(false);
    }
  };

  const disconnectWallet = async () => {
    if (wallet) {
      try {
        setWallet(null);
        setAddress('');
      } catch (err) {
        console.error('Error disconnecting wallet:', err);
        setError(err instanceof Error ? err.message : 'Failed to disconnect wallet');
      }
    }
  };

  return (
    <div className="p-4 bg-white shadow-md rounded-lg mb-6">
      <h2 className="text-xl font-semibold mb-4">Wallet Connection</h2>
      
      {!address ? (
        <button 
          onClick={connectWallet}
          disabled={loading}
          className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 disabled:bg-blue-300"
          aria-busy={loading}
        >
          {loading ? 'Connecting...' : 'Connect Eternl Wallet'}
        </button>
      ) : (
        <div>
          <p className="mb-2">
            <span className="font-semibold">Connected Address:</span>
            <br />
            <span id="changeAddressBech32" className="text-sm break-all">{address}</span>
          </p>
          <button 
            onClick={disconnectWallet}
            className="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700"
          >
            Disconnect
          </button>
        </div>
      )}
      
      {error && (
        <div className="text-red-600 mt-2" role="alert">
          {error}
        </div>
      )}
    </div>
  );
}
```

This wallet connector follows accessibility standards by using proper ARIA attributes and providing clear error messages, ensuring a better user experience. The component has three main states: disconnected, connecting, and connected, with appropriate UI for each.
