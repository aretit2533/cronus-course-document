# @eqxjs/utils

A comprehensive utility library for NestJS applications that provides JWT token management, encryption/decryption services, instance data management, credential loading, and message context handling.

## Installation

```bash
npm install @eqxjs/utils
```

## Description

This module re-exports various utilities, DTOs, enums, exceptions, and services from the 'utils' directory, providing:

- JWT token signing, verification, and decoding with HS256 algorithm
- AES-256 encryption and decryption services
- Instance data management capabilities
- Credential loading from Azure Key Vault
- Message context handling for M2M communications
- Random Nano ID generation
- Exception handling for decryption operations

## Usage

### Basic Setup

```typescript
import { Module } from '@nestjs/common';
import { UtilModule } from '@eqxjs/utils';

@Module({
  imports: [
    UtilModule.register({
      nanoid: {
        alphanum: '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz',
        len: 12
      },
      encryption: {
        algorithm: 'aes-256-cbc'
      }
    })
  ]
})
export class AppModule {}
```

### JWT Token Management

```typescript
import { Injectable } from '@nestjs/common';
import { UtilService } from '@eqxjs/utils';

@Injectable()
export class AuthService {
  constructor(private readonly utilService: UtilService) {}

  async createToken(userId: string) {
    // Sign JWT token
    const payload = { 
      sub: userId, 
      iat: Math.floor(Date.now() / 1000),
      exp: Math.floor(Date.now() / 1000) + (60 * 60) // 1 hour
    };
    
    const token = await this.utilService.signJwtHs256Token(payload, {
      expiresIn: '1h',
      issuer: 'my-app'
    });
    
    return token;
  }

  async validateToken(token: string) {
    try {
      // Verify JWT token
      const decoded = await this.utilService.verifyJwtHs256Token(token);
      return { valid: true, payload: decoded };
    } catch (error) {
      return { valid: false, error: error.message };
    }
  }

  decodeTokenUnsafe(token: string) {
    // Decode without verification (for debugging)
    return this.utilService.decodeJwtHs256Token(token);
  }
}
```

### Encryption and Decryption

```typescript
import { Injectable } from '@nestjs/common';
import { EncryptionService, Aes256CredentialsDto } from '@eqxjs/utils';

@Injectable()
export class DataProtectionService {
  constructor(private readonly encryptionService: EncryptionService) {}

  async encryptSensitiveData(data: string) {
    const credentials: Aes256CredentialsDto = {
      key: 'your-256-bit-key-here',
      iv: 'your-initialization-vector'
    };

    const encrypted = await this.encryptionService.encrypt(data, credentials);
    return encrypted;
  }

  async decryptSensitiveData(encryptedData: string) {
    const credentials: Aes256CredentialsDto = {
      key: 'your-256-bit-key-here',
      iv: 'your-initialization-vector'
    };

    try {
      const decrypted = await this.encryptionService.decrypt(encryptedData, credentials);
      return decrypted.data;
    } catch (error) {
      throw new Error(`Decryption failed: ${error.message}`);
    }
  }
}
```

### Instance Data Management

```typescript
import { Injectable } from '@nestjs/common';
import { InstanceDataManagerService } from '@eqxjs/utils';

@Injectable()
export class ConfigService {
  constructor(private readonly instanceManager: InstanceDataManagerService) {}

  async loadInstanceConfiguration() {
    // Load instance-specific configuration data
    const instanceData = await this.instanceManager.loadInstanceData();
    return instanceData;
  }

  async updateInstanceData(data: any) {
    // Update instance configuration
    await this.instanceManager.updateInstanceData(data);
  }
}
```

### Message Context Handling

```typescript
import { Injectable } from '@nestjs/common';
import { M2MessageContextService, M2MessageContextDto, MessageTypeEnum } from '@eqxjs/utils';

@Injectable()
export class MessageService {
  constructor(private readonly messageContext: M2MessageContextService) {}

  async processMessage(messageData: any) {
    const context: M2MessageContextDto = {
      messageId: this.utilService.randomNanoId(),
      messageType: MessageTypeEnum.REQUEST,
      timestamp: new Date(),
      correlationId: messageData.correlationId
    };

    // Process message with context
    return await this.messageContext.processMessage(context, messageData);
  }
}
```

## API Reference

### Classes

#### UtilService

Main utility service providing JWT token management and Nano ID generation.

**Methods:**

##### signJwtHs256Token(payload: any, options?: SignOptions): Promise<string>

Signs a JWT token using the HS256 algorithm.

**Parameters:**
- `payload: any` - The payload to include in the JWT
- `options?: SignOptions` - Optional settings for signing the JWT

**Returns:** `Promise<string>` - The signed JWT token

**Example:**
```typescript
const token = await utilService.signJwtHs256Token(
  { userId: '123', role: 'admin' },
  { expiresIn: '2h', issuer: 'my-app' }
);
```

##### verifyJwtHs256Token(token: string): Promise<any>

Verifies a JWT token using the HS256 algorithm.

**Parameters:**
- `token: string` - The JWT token to verify

**Returns:** `Promise<any>` - The decoded token payload

**Throws:** `Error` if token verification fails

**Example:**
```typescript
try {
  const payload = await utilService.verifyJwtHs256Token(token);
  console.log('User ID:', payload.userId);
} catch (error) {
  console.error('Token verification failed:', error.message);
}
```

##### decodeJwtHs256Token(token: string): any

Decodes a JWT token without verification.

**Parameters:**
- `token: string` - The JWT token to decode

**Returns:** `any` - The decoded token payload

**Example:**
```typescript
const payload = utilService.decodeJwtHs256Token(token);
console.log('Token expires at:', new Date(payload.exp * 1000));
```

##### getCacheHS256Key(): Promise<string>

Retrieves the HS256 key from cache or loads it from Azure Key Vault.

**Returns:** `Promise<string>` - The HS256 key for JWT signing/verification

##### randomNanoId(): string

Generates a random Nano ID string.

**Returns:** `string` - A randomly generated Nano ID

**Example:**
```typescript
const id = utilService.randomNanoId();
console.log('Generated ID:', id); // e.g., "V1StGXR8_Z5j"
```

#### EncryptionService

Service for AES-256 encryption and decryption operations.

**Methods:**

##### encrypt(data: string, credentials: Aes256CredentialsDto): Promise<string>

Encrypts data using AES-256 encryption.

**Parameters:**
- `data: string` - Data to encrypt
- `credentials: Aes256CredentialsDto` - Encryption credentials

**Returns:** `Promise<string>` - Encrypted data

##### decrypt(encryptedData: string, credentials: Aes256CredentialsDto): Promise<DecryptionResponseDto>

Decrypts data using AES-256 decryption.

**Parameters:**
- `encryptedData: string` - Data to decrypt
- `credentials: Aes256CredentialsDto` - Decryption credentials

**Returns:** `Promise<DecryptionResponseDto>` - Decryption response with data

**Throws:** `CannotDecryptException` if decryption fails

#### InstanceDataManagerService

Service for managing instance-specific data and configuration.

#### LoadCredentialService

Service for loading credentials from secure storage (Azure Key Vault).

#### M2MessageContextService

Service for handling message-to-message (M2M) communication context.

### DTOs and Interfaces

#### Aes256CredentialsDto

Credentials for AES-256 encryption/decryption.

```typescript
interface Aes256CredentialsDto {
  key: string;    // 256-bit encryption key
  iv: string;     // Initialization vector
}
```

#### DecryptionResponseDto

Response object for decryption operations.

```typescript
interface DecryptionResponseDto {
  data: string;           // Decrypted data
  success: boolean;       // Operation success status
  metadata?: any;         // Optional metadata
}
```

#### M2MessageContextDto

Context information for message-to-message communication.

```typescript
interface M2MessageContextDto {
  messageId: string;              // Unique message identifier
  messageType: MessageTypeEnum;   // Type of message
  timestamp: Date;                // Message timestamp
  correlationId?: string;         // Optional correlation ID
  source?: string;               // Message source
  destination?: string;          // Message destination
}
```

#### UtilOptionDto

Configuration options for the utility module.

```typescript
interface UtilOptionDto {
  nanoid: {
    alphanum: string;    // Character set for Nano ID generation
    len: number;         // Length of generated Nano IDs
  };
  encryption?: {
    algorithm: string;   // Encryption algorithm (default: aes-256-cbc)
  };
}
```

### Enums

#### MessageTypeEnum

Enumeration of message types for M2M communication.

```typescript
export enum MessageTypeEnum {
  REQUEST = 'request',
  RESPONSE = 'response',
  EVENT = 'event',
  COMMAND = 'command'
}
```

### Exceptions

#### CannotDecryptException

Exception thrown when decryption operations fail.

```typescript
try {
  const result = await encryptionService.decrypt(encryptedData, credentials);
} catch (error) {
  if (error instanceof CannotDecryptException) {
    console.error('Decryption failed:', error.message);
  }
}
```

## Environment Variables

The utility service supports the following environment variables:

```bash
# Key Vault Configuration
KEY_VAULT_URL=https://your-keyvault.vault.azure.net/
MFAF_JWT_HMACSHA256_SECRET_KV_NAME=jwt-signing-key

# JWT Key Caching (optional)
MFAF_JWT_HMACSHA256_KV_CACHE_ENABLE=true
MFAF_JWT_HMACSHA256_KV_CACHE_EXP_SEC=3600

# Debug JWT Key (for development)
R4NDR4NDS33D_DEBUG_JWT_KEY_SERVICE=your-debug-key-here
```

## Features

### JWT Token Management with Azure Key Vault Integration

Secure JWT token handling with key retrieval from Azure Key Vault:

```typescript
// Keys are automatically loaded from Azure Key Vault
const token = await utilService.signJwtHs256Token({ userId: '123' });

// Verification uses the same secure key
const payload = await utilService.verifyJwtHs256Token(token);
```

### Key Caching for Performance

Intelligent caching of JWT signing keys for improved performance:

```typescript
// First call loads from Key Vault
const token1 = await utilService.signJwtHs256Token(payload1);

// Subsequent calls use cached key (much faster)
const token2 = await utilService.signJwtHs256Token(payload2);
```

### Secure Data Encryption

AES-256 encryption for protecting sensitive data:

```typescript
const sensitiveData = "user-password-123";
const encrypted = await encryptionService.encrypt(sensitiveData, credentials);
const decrypted = await encryptionService.decrypt(encrypted, credentials);
```

### Custom Nano ID Generation

Configurable random ID generation:

```typescript
// Configure character set and length in module registration
UtilModule.register({
  nanoid: {
    alphanum: '0123456789ABCDEF',  // Hexadecimal only
    len: 16                        // 16 characters long
  }
})

// Generate IDs
const id = utilService.randomNanoId(); // e.g., "A1B2C3D4E5F6A7B8"
```

### Message Context Management

Structured message handling for microservice communication:

```typescript
const context: M2MessageContextDto = {
  messageId: utilService.randomNanoId(),
  messageType: MessageTypeEnum.REQUEST,
  timestamp: new Date(),
  correlationId: 'req-123'
};

await messageContext.processMessage(context, messageData);
```

## Error Handling

Comprehensive error handling with typed exceptions:

```typescript
try {
  const result = await utilService.verifyJwtHs256Token(token);
} catch (error) {
  if (error.message.includes('Failed to verify JWT')) {
    // Handle JWT verification errors
  } else if (error.message.includes('Cannot get HS256Key')) {
    // Handle key loading errors
  }
}
```

## Advanced Configuration

### Custom Cache Configuration

```typescript
@Module({
  imports: [
    CacheModule.register({
      ttl: 3600, // 1 hour
      max: 100,  // Maximum 100 items in cache
    }),
    UtilModule.register(config)
  ]
})
export class AppModule {}
```

### Azure Key Vault Integration

Ensure proper Azure authentication for Key Vault access:

```typescript
// Use Azure Managed Identity or Service Principal
// The service automatically handles authentication
```

## Dependencies

This library requires:
- `@nestjs/common` - NestJS common utilities
- `@nestjs/cache-manager` - Caching functionality
- `@eqxjs/azure-manage-identity` - Azure Key Vault integration
- `jsonwebtoken` - JWT token handling
- `nanoid` - Nano ID generation
- Node.js crypto module - For AES encryption

## TypeScript Support

Full TypeScript support with comprehensive type definitions for all interfaces, classes, and service methods.

## License

ISC