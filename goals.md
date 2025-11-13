# Goals

1. **Origin enforcement :** Enable websites to verify and block bots at the origin server level, without relying solely on edge or proxy-based filtering.
2. **Intent based auth:** Allow origin to selectively reveal or hide content based on bot's intent.
3. **DNS-based open registry:** Build an open bot registry using DNS TXT records for publishing and discovery, ensuring no single entity controls the verification infrastructure.
4. **Registry indexer verification :** Independent indexers verify DNS TXT records containing keyid information for bot registrations. Bot operators cryptographically prove their identity through public key signing, allowing third-party audit without centralized authority.
