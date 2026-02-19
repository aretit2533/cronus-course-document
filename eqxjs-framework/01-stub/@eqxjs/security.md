# @eqxjs/security

A comprehensive security utility library for NestJS applications that provides asset validation, permission management, and product normalization capabilities for IPMS (Integrated Product Management System) integration.

## Installation

```bash
npm install @eqxjs/security
```

## Description

This module serves as the entry point for security-related functionalities including:

- Data Transfer Objects (DTOs) for security options, asset permissions, and product management
- Enumerations for failed reason messages and product mapper states
- Services for authentication and security utility operations
- Interfaces for asset list permission responses
- The main security module for NestJS integration

## Usage

### Basic Setup

```typescript
import { Module } from '@nestjs/common';
import { SecurityModule } from '@eqxjs/security';

@Module({
  imports: [
    SecurityModule.register({
      // Security configuration options
    })
  ]
})
export class AppModule {}
```

### Using Security Services

```typescript
import { Injectable } from '@nestjs/common';
import { SecurityUtilService } from '@eqxjs/security';

@Injectable()
export class AssetService {
  constructor(private readonly securityUtil: SecurityUtilService) {}

  async validateAssets(assets: any[]) {
    // Validate managed asset group
    const validation = this.securityUtil.validateManagedAssetGroupEntiy(assets);
    
    if (!validation.valid) {
      throw new Error(`Validation failed: ${validation.message}`);
    }

    // Find managed asset group entities
    const entities = this.securityUtil.findManagedAssetGroupEntiy(assets);
    
    // Normalize permission entities
    const normalizedPermissions = this.securityUtil.normalizePermissionEntity(entities);
    
    return normalizedPermissions;
  }

  async processProductList(ipmsResponse: IIpmsProductResponse) {
    // Normalize product list from IPMS response
    const normalizedProducts = this.securityUtil.normalizeProductList(ipmsResponse);
    
    return normalizedProducts;
  }
}
```

## API Reference

### Services

#### SecurityUtilService

Main utility service for security-related operations including asset validation and product normalization.

**Methods:**

##### validateManagedAssetGroupEntiy(assets: Array<any>): ValidateManagedAssetGroup

Validates a group of managed assets to ensure they belong to a single managed asset group.

**Parameters:**
- `assets: Array<any>` - An array of assets to be validated

**Returns:** `ValidateManagedAssetGroup` - Object containing validation result:
- `valid: boolean` - Indicates whether validation was successful
- `message?: EFailedReasonMessage` - Optional failure reason message

**Validation Conditions:**
1. The assets array should not be empty or null
2. All assets should belong to a single managed asset group

**Example:**
```typescript
const assets = [
  { managedAssetGroup: { id: 1, entity: [...] } },
  { managedAssetGroup: { id: 1, entity: [...] } }
];

const result = securityUtil.validateManagedAssetGroupEntiy(assets);
if (result.valid) {
  console.log('Assets are valid');
} else {
  console.log(`Validation failed: ${result.message}`);
}
```

##### findManagedAssetGroupEntiy(assets: Array<any>): Array<any>

Finds and returns the entities from the managed asset group of the first asset that has a managed asset group.

**Parameters:**
- `assets: Array<any>` - Array of assets to search through

**Returns:** `Array<any>` - Array of entities from the managed asset group, or empty array if none found

**Example:**
```typescript
const entities = securityUtil.findManagedAssetGroupEntiy(assets);
console.log('Found entities:', entities);
```

##### normalizePermissionEntity(entities: Array<any>): Array<any>

Normalizes an array of permission entities by extracting unique 'number' characteristics.

**Parameters:**
- `entities: Array<any>` - Array of entities with productCharacteristic properties

**Returns:** `Array<any>` - Array of unique 'number' characteristic values

**Example:**
```typescript
const entities = [
  { productCharacteristic: [{ name: 'number', value: '123' }] },
  { productCharacteristic: [{ name: 'number', value: '456' }] }
];

const normalized = securityUtil.normalizePermissionEntity(entities);
// Result: ['123', '456']
```

##### normalizeProductList(impsGetProductsResponse: IIpmsProductResponse): INormalizedIpmsProducts

Normalizes the product list from the IPMS product response.

**Parameters:**
- `impsGetProductsResponse: IIpmsProductResponse` - The response object from IPMS get products API

**Returns:** `INormalizedIpmsProducts` - Object containing normalized billing accounts and product numbers

**Example:**
```typescript
const response: IIpmsProductResponse = {
  resultData: {
    customer: [
      { billingAccount: ['123', '456'] },
      { billingAccount: ['789'] }
    ],
    product: [
      { productCharacteristic: { value: 'ABC' } },
      { productCharacteristic: { value: 'DEF' } }
    ]
  }
};

const normalized = securityUtil.normalizeProductList(response);
// Result: { billingAccounts: ['123', '456', '789'], numbers: ['ABC', 'DEF'] }
```

##### findProductMapperState(inputMatch: { ba: boolean; nums: boolean }): EProductMapperState

Determines the product mapper state based on input match criteria.

**Parameters:**
- `inputMatch: { ba: boolean; nums: boolean }` - Object indicating matches for billing account and numbers

**Returns:** `EProductMapperState` - Corresponding product mapper state

**Example:**
```typescript
const match = { ba: true, nums: false };
const state = securityUtil.findProductMapperState(match);
// Result: EProductMapperState.BA_MATCH_ONLY
```

### Types and Interfaces

#### ValidateManagedAssetGroup

Represents the result of validating a managed asset group.

```typescript
export type ValidateManagedAssetGroup = {
  valid: boolean;
  message?: EFailedReasonMessage;
}
```

#### IIpmsProductResponse

Interface for IPMS product response structure.

```typescript
interface IIpmsProductResponse {
  resultData?: {
    customer?: Array<{
      billingAccount?: Array<string>;
    }>;
    product?: Array<{
      productCharacteristic?: {
        value: string;
      };
    }>;
  };
}
```

#### INormalizedIpmsProducts

Interface for normalized IPMS products.

```typescript
interface INormalizedIpmsProducts {
  billingAccounts: Array<string>;
  numbers: Array<string>;
}
```

### Enums

#### EFailedReasonMessage

Enum representing various failure reason messages.

```typescript
export enum EFailedReasonMessage {
  MULTIPLE_ASSET_GROUP = 'Data incorrect format: IPMS response multiple `managedAssetGroup`',
  CANNOT_NORMALIZE = 'Data incorrect format: cannot normalize assets from IPMS',
  DATA_NOT_FOUND = 'Data not found: cannot map target asset with IMPS response',
  DATA_PARTIALLY_MATCHING = 'Response data partial matching'
}
```

#### EProductMapperState

Enum representing different product mapper states.

```typescript
export enum EProductMapperState {
  BOTH_BA_AND_NUMBER_MATCH = 'both_ba_and_number_match',
  BA_MATCH_ONLY = 'ba_match_only',
  NUMBER_MATCH_ONLY = 'number_match_only',
  ALL_NOT_MATCH = 'all_not_match'
}
```

## Features

### Asset Group Validation
Validates that assets belong to a single managed asset group, preventing data inconsistencies:

```typescript
// Validate multiple assets
const validation = securityUtil.validateManagedAssetGroupEntiy(assets);
if (!validation.valid) {
  switch (validation.message) {
    case EFailedReasonMessage.MULTIPLE_ASSET_GROUP:
      console.log('Assets belong to multiple groups');
      break;
    case EFailedReasonMessage.CANNOT_NORMALIZE:
      console.log('Cannot normalize assets');
      break;
  }
}
```

### Permission Normalization
Extracts and normalizes permission data from complex asset structures:

```typescript
// Extract unique permission numbers
const entities = securityUtil.findManagedAssetGroupEntiy(assets);
const permissions = securityUtil.normalizePermissionEntity(entities);
```

### Product List Processing
Processes IPMS product responses to extract billing accounts and product numbers:

```typescript
// Process IPMS response
const normalizedData = securityUtil.normalizeProductList(ipmsResponse);
console.log('Billing Accounts:', normalizedData.billingAccounts);
console.log('Product Numbers:', normalizedData.numbers);
```

### Product Matching State Detection
Determines the state of product matching for business logic decisions:

```typescript
const matchResult = { ba: true, nums: false };
const state = securityUtil.findProductMapperState(matchResult);

switch (state) {
  case EProductMapperState.BOTH_BA_AND_NUMBER_MATCH:
    // Handle complete match
    break;
  case EProductMapperState.BA_MATCH_ONLY:
    // Handle billing account match only
    break;
  case EProductMapperState.NUMBER_MATCH_ONLY:
    // Handle number match only
    break;
  case EProductMapperState.ALL_NOT_MATCH:
    // Handle no matches
    break;
}
```

## Error Handling

The library provides comprehensive error handling through typed failure messages:

```typescript
try {
  const validation = securityUtil.validateManagedAssetGroupEntiy(assets);
  if (!validation.valid) {
    throw new Error(`Security validation failed: ${validation.message}`);
  }
} catch (error) {
  // Handle specific security errors
  console.error('Security operation failed:', error.message);
}
```

## Integration with IPMS

This library is designed to work seamlessly with Integrated Product Management System (IPMS) responses:

```typescript
// Example IPMS integration
async function processIPMSResponse(response: IIpmsProductResponse) {
  // Normalize the product data
  const normalized = securityUtil.normalizeProductList(response);
  
  // Determine matching state
  const hasBA = normalized.billingAccounts.length > 0;
  const hasNumbers = normalized.numbers.length > 0;
  const state = securityUtil.findProductMapperState({ ba: hasBA, nums: hasNumbers });
  
  return {
    normalizedData: normalized,
    mappingState: state
  };
}
```

## TypeScript Support

Full TypeScript support with comprehensive type definitions for all interfaces, enums, and service methods.

## Dependencies

This library depends on:
- `@nestjs/common` - For NestJS integration and dependency injection

## License

ISC