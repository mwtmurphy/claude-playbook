# JavaScript/TypeScript Testing Standards

## Purpose
This document defines testing standards for JavaScript and TypeScript projects using Jest for unit testing and Playwright for end-to-end testing. These standards ensure code quality, reliability, and maintainability.

## Testing Philosophy

### Test Coverage Goals
- **Minimum coverage**: 80% for all projects
- **Business logic**: 100% coverage for critical paths
- **UI components**: Test behaviour, not implementation
- **Integration tests**: Cover component interactions
- **E2E tests**: Focus on critical user workflows

### Coverage Priorities

Different code types warrant different coverage targets:

1. **Critical business logic**: 90-100%
2. **Component logic**: 80-90%
3. **Utility functions**: 80-90%
4. **UI glue code**: 50-70%

### Testing Pyramid
```
      /\
     /E2E\         (Few) - Critical user paths
    /------\
   /Integra-\      (Some) - Component interactions
  /----------\
 /Unit Tests \     (Many) - Business logic, utilities
/--------------\
```

### Arrange-Act-Assert (AAA) Pattern

Structure all tests consistently using the AAA pattern:

**Why**: Makes tests easier to read and understand at a glance, matching Python test conventions.

```typescript
it('should calculate total with tax', () => {
  // Arrange - Set up test data
  const price = 100;
  const taxRate = 0.1;
  const calculator = new PriceCalculator();

  // Act - Perform the action being tested
  const total = calculator.calculateTotal(price, taxRate);

  // Assert - Verify the result
  expect(total).toBe(110);
});
```

## Unit Testing with Jest

### Configuration

#### jest.config.js
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.ts',
    '!src/**/index.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
};
```

#### Test Setup (tests/setup.ts)
```typescript
// Mock Chrome APIs for extension testing
global.chrome = {
  runtime: {
    sendMessage: jest.fn(),
    onMessage: {
      addListener: jest.fn(),
      removeListener: jest.fn(),
    },
    getManifest: jest.fn(() => ({
      version: '1.0.0',
      name: 'Test Extension',
    })),
  },
  storage: {
    local: {
      get: jest.fn((keys) => Promise.resolve({})),
      set: jest.fn((items) => Promise.resolve()),
      remove: jest.fn((keys) => Promise.resolve()),
      clear: jest.fn(() => Promise.resolve()),
      getBytesInUse: jest.fn(() => Promise.resolve(0)),
      QUOTA_BYTES: 5242880,
    },
    sync: {
      get: jest.fn((keys) => Promise.resolve({})),
      set: jest.fn((items) => Promise.resolve()),
      remove: jest.fn((keys) => Promise.resolve()),
      clear: jest.fn(() => Promise.resolve()),
    },
    onChanged: {
      addListener: jest.fn(),
      removeListener: jest.fn(),
    },
  },
  tabs: {
    query: jest.fn(() => Promise.resolve([])),
    sendMessage: jest.fn(),
  },
} as any;

// Mock DOM APIs
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### Test Structure

#### File Organisation
```
tests/
├── unit/
│   ├── utils/
│   │   ├── storage.test.ts
│   │   └── messaging.test.ts
│   ├── components/
│   │   └── button.test.ts
│   └── services/
│       └── api.test.ts
├── integration/
│   └── popup-workflow.test.ts
└── setup.ts
```

#### Naming Conventions
- Test files: `*.test.ts` or `*.spec.ts`
- Test suites: Descriptive `describe` blocks
- Test cases: Start with "should" for clarity

```typescript
// storage.test.ts - Example structure
import { StorageManager } from '@/utils/storage';

describe('StorageManager', () => {
  let storageManager: StorageManager;

  beforeEach(() => {
    storageManager = new StorageManager();
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  describe('saveLocal', () => {
    it('should save data to local storage', async () => {
      const testData = { key: 'value' };
      await storageManager.saveLocal('testKey', testData);

      expect(chrome.storage.local.set).toHaveBeenCalledWith({
        testKey: testData,
      });
    });

    it('should throw error when storage fails', async () => {
      const error = new Error('Storage error');
      (chrome.storage.local.set as jest.Mock).mockRejectedValue(error);

      await expect(storageManager.saveLocal('testKey', {})).rejects.toThrow(
        'Storage error'
      );
    });
  });

  describe('loadLocal', () => {
    it('should load data from local storage', async () => {
      const testData = { key: 'value' };
      (chrome.storage.local.get as jest.Mock).mockResolvedValue({
        testKey: testData,
      });

      const result = await storageManager.loadLocal('testKey');
      expect(result).toEqual(testData);
    });

    it('should return null when key does not exist', async () => {
      (chrome.storage.local.get as jest.Mock).mockResolvedValue({});

      const result = await storageManager.loadLocal('nonexistent');
      expect(result).toBeNull();
    });
  });
});
```

### Mocking Patterns

#### Function Mocks
```typescript
// Mock external dependencies
jest.mock('@/services/api', () => ({
  fetchData: jest.fn(),
}));

import { fetchData } from '@/services/api';

describe('DataService', () => {
  it('should fetch and process data', async () => {
    const mockData = { id: 1, name: 'Test' };
    (fetchData as jest.Mock).mockResolvedValue(mockData);

    const result = await dataService.getData();
    expect(result).toEqual(mockData);
    expect(fetchData).toHaveBeenCalledTimes(1);
  });
});
```

#### Class Mocks
```typescript
// Mock entire class
jest.mock('@/utils/logger');

import { Logger } from '@/utils/logger';

describe('Service', () => {
  it('should log errors', () => {
    const mockLogger = Logger as jest.MockedClass<typeof Logger>;
    const logSpy = jest.spyOn(mockLogger.prototype, 'error');

    service.handleError(new Error('test'));
    expect(logSpy).toHaveBeenCalled();
  });
});
```

#### Partial Mocks
```typescript
// Mock specific methods while keeping others
import * as utils from '@/utils/helpers';

jest.spyOn(utils, 'formatDate').mockReturnValue('2024-01-01');

describe('Component', () => {
  it('should use formatted date', () => {
    const result = component.render();
    expect(result).toContain('2024-01-01');
    expect(utils.formatDate).toHaveBeenCalled();
  });
});
```

### Async Testing

#### Promises
```typescript
it('should handle async operations', async () => {
  const promise = asyncFunction();
  await expect(promise).resolves.toBe('success');
});

it('should handle async errors', async () => {
  const promise = failingAsyncFunction();
  await expect(promise).rejects.toThrow('Error message');
});
```

#### Timers
```typescript
describe('Timer tests', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should execute after timeout', () => {
    const callback = jest.fn();
    setTimeout(callback, 1000);

    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);
    expect(callback).toHaveBeenCalled();
  });

  it('should clear timeout', () => {
    const callback = jest.fn();
    const timer = setTimeout(callback, 1000);
    clearTimeout(timer);

    jest.advanceTimersByTime(1000);
    expect(callback).not.toHaveBeenCalled();
  });
});
```

### Test Fixtures and Factories

#### Type-Safe Factories
```typescript
// test-utils/factories.ts
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

export function createMockUser(overrides?: Partial<User>): User {
  return {
    id: '123',
    name: 'Test User',
    email: 'test@example.com',
    role: 'user',
    ...overrides,
  };
}

export function createMockUsers(count: number): User[] {
  return Array.from({ length: count }, (_, i) =>
    createMockUser({ id: `${i + 1}`, name: `User ${i + 1}` })
  );
}

// Usage in tests
const adminUser = createMockUser({ role: 'admin' });
const users = createMockUsers(5);
```

#### Fixture Files
```typescript
// test-utils/fixtures.ts
export const mockSettings = {
  theme: 'dark' as const,
  notifications: true,
  syncEnabled: true,
};

export const mockApiResponse = {
  status: 'success' as const,
  data: {
    items: [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
    ],
  },
};
```

### Snapshot Testing

#### Component Snapshots
```typescript
import { render } from './test-utils';
import { Button } from '@/components/Button';

describe('Button', () => {
  it('should match snapshot', () => {
    const { container } = render(
      <Button variant="primary">Click me</Button>
    );
    expect(container).toMatchSnapshot();
  });

  it('should match inline snapshot', () => {
    const result = formatOutput({ name: 'Test' });
    expect(result).toMatchInlineSnapshot(`"Name: Test"`);
  });
});
```

#### When to Use Snapshots
- **Good for**: Component HTML structure, serialized data
- **Avoid for**: Frequently changing data, implementation details
- **Review**: Always review snapshot changes in PRs

## End-to-End Testing with Playwright

### Configuration

#### playwright.config.ts
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Chrome Extension E2E Testing

#### Loading Extension
```typescript
// tests/e2e/extension.setup.ts
import { chromium, BrowserContext } from '@playwright/test';
import path from 'path';

export async function loadExtension(): Promise<BrowserContext> {
  const pathToExtension = path.join(__dirname, '../../dist');

  const context = await chromium.launchPersistentContext('', {
    headless: false,
    args: [
      `--disable-extensions-except=${pathToExtension}`,
      `--load-extension=${pathToExtension}`,
    ],
  });

  return context;
}

export async function getExtensionId(context: BrowserContext): Promise<string> {
  // Wait for background script to load
  let [background] = context.serviceWorkers();
  if (!background) background = await context.waitForEvent('serviceworker');

  const extensionId = background.url().split('/')[2];
  return extensionId;
}
```

#### Testing Popup
```typescript
// tests/e2e/popup.spec.ts
import { test, expect } from '@playwright/test';
import { loadExtension, getExtensionId } from './extension.setup';

test.describe('Extension Popup', () => {
  test('should open popup and display content', async () => {
    const context = await loadExtension();
    const extensionId = await getExtensionId(context);

    const page = await context.newPage();
    await page.goto(`chrome-extension://${extensionId}/popup.html`);

    await expect(page.locator('h1')).toContainText('Extension Title');
    await expect(page.locator('#main-content')).toBeVisible();

    await context.close();
  });

  test('should navigate between views', async () => {
    const context = await loadExtension();
    const extensionId = await getExtensionId(context);

    const page = await context.newPage();
    await page.goto(`chrome-extension://${extensionId}/popup.html`);

    // Click settings button
    await page.click('#settings-button');
    await expect(page.locator('.settings-view')).toBeVisible();

    // Navigate back
    await page.click('#back-button');
    await expect(page.locator('.main-view')).toBeVisible();

    await context.close();
  });
});
```

#### Testing Content Scripts
```typescript
// tests/e2e/content-script.spec.ts
import { test, expect } from '@playwright/test';
import { loadExtension } from './extension.setup';

test.describe('Content Script', () => {
  test('should inject into target page', async () => {
    const context = await loadExtension();
    const page = await context.newPage();

    await page.goto('https://example.com');

    // Wait for content script to inject
    await page.waitForSelector('#extension-container');

    // Verify injected content
    const container = page.locator('#extension-container');
    await expect(container).toBeVisible();

    // Test interaction with injected UI
    await page.click('#extension-button');
    await expect(page.locator('.extension-modal')).toBeVisible();

    await context.close();
  });

  test('should communicate with background script', async () => {
    const context = await loadExtension();
    const page = await context.newPage();

    await page.goto('https://example.com');
    await page.waitForSelector('#extension-container');

    // Trigger action that sends message to background
    await page.click('#fetch-data-button');

    // Wait for response and UI update
    await expect(page.locator('.data-display')).toContainText('Data loaded');

    await context.close();
  });
});
```

### Page Object Model

#### Creating Page Objects
```typescript
// tests/e2e/pages/popup.page.ts
import { Page } from '@playwright/test';

export class PopupPage {
  constructor(
    private page: Page,
    private extensionId: string
  ) {}

  async navigate(): Promise<void> {
    await this.page.goto(`chrome-extension://${this.extensionId}/popup.html`);
  }

  async clickSettings(): Promise<void> {
    await this.page.click('#settings-button');
  }

  async getTitle(): Promise<string> {
    return this.page.locator('h1').textContent() || '';
  }

  async isMainViewVisible(): Promise<boolean> {
    return this.page.locator('.main-view').isVisible();
  }

  async isSettingsViewVisible(): Promise<boolean> {
    return this.page.locator('.settings-view').isVisible();
  }
}

// Usage in tests
test('should navigate to settings', async () => {
  const context = await loadExtension();
  const extensionId = await getExtensionId(context);
  const page = await context.newPage();

  const popupPage = new PopupPage(page, extensionId);
  await popupPage.navigate();

  await expect(await popupPage.isMainViewVisible()).toBe(true);

  await popupPage.clickSettings();
  await expect(await popupPage.isSettingsViewVisible()).toBe(true);

  await context.close();
});
```

### E2E Best Practices

#### Wait Strategies
```typescript
// Good: Wait for specific condition
await page.waitForSelector('.data-loaded', { state: 'visible' });

// Good: Wait for network
await page.waitForResponse((response) =>
  response.url().includes('/api/data') && response.status() === 200
);

// Avoid: Arbitrary timeouts
await page.waitForTimeout(1000); // Only use as last resort
```

#### Test Isolation
```typescript
test.beforeEach(async ({ page }) => {
  // Clear storage before each test
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
});

test.afterEach(async ({ page }) => {
  // Cleanup after test
  await page.close();
});
```

#### Screenshots and Traces
```typescript
test('visual regression test', async ({ page }) => {
  await page.goto('/');

  // Take screenshot for comparison
  await expect(page).toHaveScreenshot('homepage.png');
});

test('debug failing test', async ({ page }, testInfo) => {
  await page.goto('/');

  // Capture trace on failure
  if (testInfo.status !== 'passed') {
    await testInfo.attach('trace', {
      path: await page.video()?.path(),
    });
  }
});
```

## Test Organisation

### Test Categories

#### Unit Tests
- Pure functions and utilities
- Class methods in isolation
- Component logic without DOM

#### Integration Tests
- Component interactions
- Service layer integration
- Storage operations

#### E2E Tests
- Critical user workflows
- Cross-component scenarios
- Full extension functionality

### Running Tests

```bash
# Unit tests
npm test                    # Run all unit tests
npm test -- --watch        # Watch mode
npm test -- --coverage     # With coverage

# Specific tests
npm test storage           # Tests matching pattern
npm test -- path/to/file   # Specific file

# E2E tests
npm run test:e2e           # All E2E tests
npm run test:e2e -- --ui   # Interactive mode
npm run test:e2e -- --debug # Debug mode
```

## Code Coverage

### Coverage Reports
```bash
npm test -- --coverage
```

#### Coverage Thresholds
```javascript
// jest.config.js
coverageThreshold: {
  global: {
    branches: 80,
    functions: 80,
    lines: 80,
    statements: 80,
  },
  './src/critical/': {
    branches: 100,
    functions: 100,
    lines: 100,
    statements: 100,
  },
}
```

#### Excluding from Coverage
```typescript
/* istanbul ignore next */
function debugOnlyFunction() {
  console.log('Debug info');
}
```

## Testing Checklist

### Before Committing
- [ ] All tests pass locally
- [ ] Coverage meets thresholds
- [ ] No skipped tests (`.skip`) in committed code
- [ ] No focused tests (`.only`) in committed code
- [ ] Mock data is realistic
- [ ] Tests are deterministic (no flakiness)

### Code Review
- [ ] Tests cover happy path and edge cases
- [ ] Error scenarios tested
- [ ] Async operations properly tested
- [ ] Proper cleanup in afterEach/afterAll
- [ ] Tests are readable and maintainable

## Common Pitfalls

### Avoid
- Testing implementation details
- Overly complex test setup
- Shared mutable state between tests
- Arbitrary timeouts in E2E tests
- Snapshots for frequently changing data

### Prefer
- Testing behaviour and outcomes
- Simple, focused test cases
- Isolated tests with proper cleanup
- Waiting for specific conditions
- Explicit assertions over snapshots

## Related Standards
- See `typescript_style.md` for TypeScript coding standards
- See `chrome_extension_standards.md` for extension-specific patterns
- See `git_workflow.md` for testing in CI/CD

## References
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Playwright Documentation](https://playwright.dev/docs/intro)
- [Testing Library](https://testing-library.com/)
- [Chrome Extension Testing](https://developer.chrome.com/docs/extensions/mv3/tut_debugging/)
