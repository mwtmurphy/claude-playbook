# TypeScript Style Guide

## Purpose
This document defines TypeScript coding standards, conventions, and best practices for all TypeScript projects in the workspace. These standards ensure type safety, maintainability, and consistency across projects.

## Code Style & Formatting

### ESLint + Prettier Configuration
All TypeScript projects must use ESLint with TypeScript support and Prettier for formatting:

```json
// .eslintrc.json
{
  "parser": "@typescript-eslint/parser",
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "plugins": ["@typescript-eslint"],
  "parserOptions": {
    "ecmaVersion": 2021,
    "sourceType": "module"
  }
}
```

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false
}
```

### Formatting Standards
- **Indentation**: 2 spaces (TypeScript/JavaScript convention)
- **Line length**: Maximum 100 characters
- **Semicolons**: Required at end of statements
- **Quotes**: Single quotes for strings, double quotes in JSON
- **Trailing commas**: ES5 style (multiline only)

## TypeScript Configuration

### tsconfig.json Standards
All projects should use strict TypeScript configuration:

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ESNext",
    "lib": ["ES2021", "DOM"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Key Compiler Options
- **strict**: Always enabled for maximum type safety
- **noUnusedLocals/Parameters**: Prevent unused code
- **noImplicitReturns**: Ensure all code paths return values
- **esModuleInterop**: Better CommonJS/ES module compatibility

## Type System Best Practices

### Type Annotations
Always provide explicit type annotations for:
- Function parameters
- Function return types
- Class properties
- Complex object literals
- Exported constants

```typescript
// Good: Explicit types
function calculateTotal(items: Item[], taxRate: number): number {
  return items.reduce((sum, item) => sum + item.price, 0) * (1 + taxRate);
}

// Avoid: Implicit types
function calculateTotal(items, taxRate) {
  return items.reduce((sum, item) => sum + item.price, 0) * (1 + taxRate);
}
```

### Interface vs Type
- **Prefer interfaces** for object shapes and contracts
- **Use type aliases** for unions, intersections, and primitives
- **Interfaces** can be extended and merged, types cannot

```typescript
// Interfaces for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

interface AdminUser extends User {
  permissions: string[];
}

// Type aliases for unions and complex types
type Status = 'pending' | 'active' | 'inactive';
type Result<T> = { success: true; data: T } | { success: false; error: string };
```

### Generic Types
Use generics for reusable, type-safe components:

```typescript
// Generic function
function getFirstElement<T>(array: T[]): T | undefined {
  return array[0];
}

// Generic interface
interface Repository<T> {
  getById(id: string): Promise<T>;
  getAll(): Promise<T[]>;
  save(item: T): Promise<void>;
}

// Generic class
class DataStore<T> {
  private items: Map<string, T> = new Map();

  set(key: string, value: T): void {
    this.items.set(key, value);
  }

  get(key: string): T | undefined {
    return this.items.get(key);
  }
}
```

### Union and Literal Types
Leverage TypeScript's powerful type system:

```typescript
// Literal types for constants
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Theme = 'light' | 'dark' | 'auto';

// Discriminated unions
type ApiResponse<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; message: string }
  | { status: 'loading' };

function handleResponse<T>(response: ApiResponse<T>): void {
  switch (response.status) {
    case 'success':
      console.log(response.data); // TypeScript knows data exists
      break;
    case 'error':
      console.error(response.message); // TypeScript knows message exists
      break;
    case 'loading':
      console.log('Loading...');
      break;
  }
}
```

### Type Guards
Use type guards for runtime type checking:

```typescript
// User-defined type guard
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// Type guard with objects
interface Cat {
  type: 'cat';
  meow(): void;
}

interface Dog {
  type: 'dog';
  bark(): void;
}

type Animal = Cat | Dog;

function isCat(animal: Animal): animal is Cat {
  return animal.type === 'cat';
}

function makeSound(animal: Animal): void {
  if (isCat(animal)) {
    animal.meow(); // TypeScript knows this is Cat
  } else {
    animal.bark(); // TypeScript knows this is Dog
  }
}
```

## Naming Conventions

### Files and Directories
- **Lowercase with dashes**: `user-service.ts`, `api-client.ts`
- **Test files**: `user-service.test.ts`, `api-client.spec.ts`
- **Type definition files**: `types.ts`, `user.types.ts`

### Variables and Functions
- **camelCase**: `userName`, `getUserById`, `fetchData`
- **Descriptive names**: Avoid abbreviations unless well-known
- **Boolean variables**: Prefix with `is`, `has`, `should`

```typescript
// Good naming
const isAuthenticated = true;
const hasPermission = false;
const shouldShowModal = true;

function getUserById(userId: string): User | null {
  // Implementation
}
```

### Classes and Interfaces
- **PascalCase**: `UserService`, `ApiClient`, `DataStore`
- **Interfaces**: No "I" prefix (use descriptive names)
- **Type aliases**: PascalCase like interfaces

```typescript
// Good naming
interface User {
  id: string;
  name: string;
}

class UserRepository {
  // Implementation
}

type UserRole = 'admin' | 'user' | 'guest';
```

### Constants and Enums
- **UPPER_SNAKE_CASE** for true constants
- **PascalCase** for enums
- **camelCase** for configuration objects

```typescript
// Constants
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = 'https://api.example.com';

// Enums
enum UserRole {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest',
}

// Configuration objects
const apiConfig = {
  baseUrl: 'https://api.example.com',
  timeout: 5000,
  retryAttempts: 3,
};
```

## Error Handling

### Custom Error Classes
Create typed error classes for better error handling:

```typescript
class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public response?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: unknown
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

### Error Handling Patterns
```typescript
// Use Result type for expected errors
type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

async function fetchUser(id: string): Promise<Result<User, ApiError>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return {
        success: false,
        error: new ApiError('User not found', response.status),
      };
    }
    const user = await response.json();
    return { success: true, value: user };
  } catch (error) {
    return {
      success: false,
      error: new ApiError('Network error', 0, error),
    };
  }
}

// Usage
const result = await fetchUser('123');
if (result.success) {
  console.log(result.value); // TypeScript knows this is User
} else {
  console.error(result.error); // TypeScript knows this is ApiError
}
```

## Async/Await and Promises

### Prefer async/await
```typescript
// Good: async/await
async function fetchUserData(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new ApiError('Failed to fetch user', response.status);
  }
  return await response.json();
}

// Avoid: Promise chains (unless necessary)
function fetchUserData(userId: string): Promise<User> {
  return fetch(`/api/users/${userId}`)
    .then(response => {
      if (!response.ok) {
        throw new ApiError('Failed to fetch user', response.status);
      }
      return response.json();
    });
}
```

### Proper error handling
```typescript
async function processData(): Promise<void> {
  try {
    const data = await fetchData();
    await validateData(data);
    await saveData(data);
  } catch (error) {
    if (error instanceof ValidationError) {
      console.error('Validation failed:', error.field);
    } else if (error instanceof ApiError) {
      console.error('API error:', error.statusCode);
    } else {
      console.error('Unexpected error:', error);
    }
    throw error; // Re-throw for caller to handle
  }
}
```

## Module Organisation

### Barrel Exports
Use index.ts for clean exports:

```typescript
// utils/index.ts
export { formatDate } from './date-utils';
export { validateEmail } from './validation';
export { createLogger } from './logger';

// Usage
import { formatDate, validateEmail } from './utils';
```

### Avoid Circular Dependencies
- Structure imports to flow in one direction
- Use dependency injection to break cycles
- Consider splitting modules if cycles occur

## Documentation

### TSDoc Comments
Use TSDoc for public APIs:

```typescript
/**
 * Calculates the total price including tax
 *
 * @param items - Array of items to calculate total for
 * @param taxRate - Tax rate as decimal (e.g., 0.1 for 10%)
 * @returns Total price including tax
 * @throws {ValidationError} If taxRate is negative
 *
 * @example
 * ```typescript
 * const total = calculateTotal([{ price: 100 }], 0.1);
 * console.log(total); // 110
 * ```
 */
function calculateTotal(items: Item[], taxRate: number): number {
  if (taxRate < 0) {
    throw new ValidationError('Tax rate cannot be negative', 'taxRate', taxRate);
  }
  return items.reduce((sum, item) => sum + item.price, 0) * (1 + taxRate);
}
```

## Performance Considerations

### Avoid Type Assertions
Type assertions (`as`) bypass type checking - use sparingly:

```typescript
// Avoid: Type assertions
const user = JSON.parse(data) as User;

// Prefer: Validation
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj
  );
}

const parsed = JSON.parse(data);
if (isUser(parsed)) {
  const user = parsed; // TypeScript knows this is User
}
```

### Use const assertions
For literal types and readonly arrays:

```typescript
// Const assertion for literal types
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
} as const;

// config.apiUrl is now type 'https://api.example.com', not string
// config is readonly

// Readonly arrays
const colors = ['red', 'green', 'blue'] as const;
// colors is readonly ['red', 'green', 'blue']
```

## Testing Considerations

### Type-safe test utilities
```typescript
// Type-safe mock factory
function createMockUser(overrides?: Partial<User>): User {
  return {
    id: '123',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides,
  };
}

// Usage in tests
const mockUser = createMockUser({ name: 'Custom Name' });
```

### Type-safe assertions
```typescript
// Type guard for testing
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error('Value is not a User');
  }
}

// Usage
const data = await fetchData();
assertIsUser(data);
// TypeScript now knows data is User
console.log(data.name);
```

## Common Patterns

### Singleton Pattern
```typescript
class Logger {
  private static instance: Logger;

  private constructor() {
    // Private constructor
  }

  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }

  log(message: string): void {
    console.log(`[${new Date().toISOString()}] ${message}`);
  }
}

// Usage
const logger = Logger.getInstance();
logger.log('Application started');
```

### Factory Pattern
```typescript
interface Shape {
  area(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle implements Shape {
  constructor(
    private width: number,
    private height: number
  ) {}

  area(): number {
    return this.width * this.height;
  }
}

class ShapeFactory {
  static createCircle(radius: number): Circle {
    return new Circle(radius);
  }

  static createRectangle(width: number, height: number): Rectangle {
    return new Rectangle(width, height);
  }
}
```

## Related Standards
- See `javascript_testing.md` for Jest and Playwright testing patterns
- See `git_workflow.md` for commit and versioning standards
- See `chrome_extension_standards.md` for Chrome extension specific TypeScript patterns
- See `webpack_standards.md` for build configuration

## References
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript ESLint](https://typescript-eslint.io/)
- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
