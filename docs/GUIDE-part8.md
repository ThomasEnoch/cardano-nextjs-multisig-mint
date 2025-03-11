## Running Your Application

Before starting the application, make sure you have Redis running:

```bash
redis-server
```

Then run the development server:

```bash
npm run dev
```

Your application should now be running at [http://localhost:3000](http://localhost:3000).

## Security Considerations

Security is critical when dealing with blockchain applications. Our implementation follows these key security principles:

1. **API Key Protection**: We use server-side API routes to prevent exposing your Anvil API key in client code, following the principle of never exposing sensitive data.

2. **Input Validation**: All user inputs are validated before processing to prevent errors and security issues.

3. **Error Handling**: Comprehensive error handling is implemented throughout the application to prevent unexpected behavior and provide clear feedback.

4. **Transaction Verification**: We verify that transactions are valid before submitting them to the blockchain.

5. **HTTPS for External Requests**: All API calls use HTTPS to ensure encrypted communication.

6. **Temporary Token Storage**: Redis provides secure temporary storage for transaction data with automatic expiration.

For production deployments, consider these additional security measures:

- Implement rate limiting to prevent abuse
- Add authentication for admin operations
- Set up monitoring and logging for suspicious activities
- Regularly update dependencies to patch security vulnerabilities

## Testing Your Implementation

To ensure your application works correctly, follow these testing steps:

1. **Connect Wallet**: Verify that wallet connection works properly with Eternl

2. **Create Transaction**: Test the transaction creation process

3. **Sign Transaction**: Confirm that the wallet correctly signs the transaction

4. **Submit Transaction**: Verify that signed transactions are successfully submitted to the blockchain

5. **Check Blockchain**: Confirm that the NFT appears in your wallet after successful minting

Consider writing automated tests for critical components:

```typescript
// Example test for the createTransaction function
import { describe, it, expect, vi } from 'vitest';
import { createTransaction } from '@/lib/anvil';

// Mock fetch
global.fetch = vi.fn();

describe('createTransaction', () => {
  it('should create a transaction successfully', async () => {
    // Mock successful response
    (fetch as any).mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        hash: 'test-hash',
        cbor: 'test-cbor',
      }),
    });

    const result = await createTransaction(
      'addr_test1...',
      { assetName: { name: 'Test' }, policyId: 'test', quantity: 1 },
      'key-hash',
      12345678,
      'policy-id'
    );

    expect(result).toEqual({
      hash: 'test-hash',
      cbor: 'test-cbor',
    });
  });

  it('should handle API errors', async () => {
    // Mock error response
    (fetch as any).mockResolvedValueOnce({
      ok: false,
      json: async () => ({ error: 'API Error' }),
    });

    await expect(createTransaction(
      'addr_test1...',
      { assetName: { name: 'Test' }, policyId: 'test', quantity: 1 },
      'key-hash',
      12345678,
      'policy-id'
    )).rejects.toThrow();
  });
});
```

## Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Cardano Developer Portal](https://developers.cardano.org/)
- [Anvil API Documentation](https://docs.ada-anvil.app/)
- [CIP-25: NFT Metadata Standard](https://cips.cardano.org/cips/cip25/)
- [Weld Library Documentation](https://www.npmjs.com/package/@ada-anvil/weld)

## Conclusion

You've successfully built a Cardano NFT minting platform using Next.js Server Components and the Anvil API. The application demonstrates proper separation of concerns, with server components handling API requests and client components managing user interactions.

This implementation follows best practices for:

- Security: Protecting API keys and validating inputs
- Performance: Using server components for data-intensive operations
- Accessibility: Following WCAG guidelines with semantic HTML and ARIA attributes
- Error handling: Providing clear feedback and graceful failure states
- Code quality: Using TypeScript, ESLint, and Prettier for consistent, maintainable code

This guide serves as a starting point. Consider expanding your application with features like:

- User authentication
- NFT collection management
- Transaction history and status tracking
- Custom metadata editors
- Multi-wallet support
