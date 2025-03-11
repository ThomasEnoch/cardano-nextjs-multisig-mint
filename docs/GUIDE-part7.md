### Navigation

This component provides navigation for the application:

```typescript
// src/components/Navigation.tsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav className="bg-blue-600 text-white p-4" aria-label="Main Navigation">
      <div className="container mx-auto flex justify-between items-center">
        <Link href="/" className="text-xl font-bold">
          Cardano Minting
        </Link>
        
        <div className="space-x-4">
          <Link href="/" className="hover:underline" aria-current="page">
            Home
          </Link>
          <Link href="/info" className="hover:underline">
            Info
          </Link>
        </div>
      </div>
    </nav>
  );
}
```

## Pages

These pages combine server and client components to create the complete user interface.

### Main Page

The main page combines our wallet connector and minting form:

```typescript
// src/app/page.tsx
import WalletConnector from '@/components/WalletConnector';
import MintForm from '@/components/MintForm';

export default function Home() {
  return (
    <main className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold text-center mb-8">
        Cardano NFT Minting Platform
      </h1>
      
      <div className="max-w-2xl mx-auto">
        <WalletConnector />
        <MintForm />
      </div>
    </main>
  );
}
```

### Information Page

The information page provides details about NFTs on Cardano:

```typescript
// src/app/info/page.tsx
export default function InfoPage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">NFT Minting Information</h1>
      
      <div className="prose max-w-none">
        <h2>Cardano NFTs and Minting Process</h2>
        
        <p>
          NFTs on Cardano are native tokens with a quantity of 1. They're minted
          under a policy that determines rules for creating, modifying, and burning tokens.
        </p>
        
        <h3>Key Components</h3>
        
        <ul>
          <li>
            <strong>Policy ID</strong>: A unique identifier for your NFT collection, 
            derived from the policy script hash.
          </li>
          <li>
            <strong>Asset Name</strong>: A name for your specific NFT, which combined 
            with the Policy ID creates a unique Asset ID.
          </li>
          <li>
            <strong>Metadata</strong>: JSON data following the CIP-25 standard that 
            describes your NFT's properties.
          </li>
        </ul>
        
        <h3>Minting Process Steps</h3>
        
        <ol>
          <li>Connect your Cardano wallet (we support Eternl)</li>
          <li>Create a minting transaction</li>
          <li>Sign the transaction with your wallet</li>
          <li>Submit the signed transaction to the blockchain</li>
          <li>Wait for confirmation</li>
        </ol>
        
        <h3>CIP-25 Metadata Format</h3>
        
        <pre className="bg-gray-100 p-4 rounded-md overflow-auto">
{`{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "name": "Asset Name",
        "description": "Description of the asset",
        "image": "ipfs://QmRzicpReutwCkM6aotuKjErFCUD213DpwPq6ByuzMJaua",
        "mediaType": "image/png",
        "files": [
          {
            "name": "My NFT",
            "mediaType": "image/png",
            "src": "ipfs://QmRzicpReutwCkM6aotuKjErFCUD213DpwPq6ByuzMJaua"
          }
        ]
      }
    }
  }
}`}
        </pre>
      </div>
    </div>
  );
}
```

### Root Layout

Update the root layout to include the navigation component:

```typescript
// src/app/layout.tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';
import Navigation from '@/components/Navigation';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Cardano NFT Minting Platform',
  description: 'Mint your own NFTs on the Cardano blockchain',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Navigation />
        {children}
      </body>
    </html>
  );
}
```
