# Chrome Extension Development Standards

## Purpose
This document defines standards and best practices for developing Chrome extensions using Manifest V3. It covers architecture patterns, UI/UX guidelines, security, performance optimisation, and deployment processes.

## Technology Stack

### Required Technologies
- **Manifest V3**: Required for all new Chrome extensions
- **TypeScript**: Type-safe development with ES2021+ features
- **Build Tools**: Webpack 5+ or Vite for modern bundling
- **Code Quality**: ESLint + Prettier with TypeScript support
- **Testing**: Jest (unit) + Playwright (E2E)

### Key Dependencies
```bash
# Core development
typescript ts-loader @types/chrome
webpack webpack-cli webpack-merge

# Code quality
eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
prettier eslint-config-prettier

# Testing
jest @types/jest playwright @playwright/test
```

## Project Structure

### Standard Directory Layout
```
/extension-project
├── manifest.json              # Extension manifest (V3)
├── package.json
├── tsconfig.json
├── webpack.config.js
├── .eslintrc.json
├── .prettierrc
├── /src/                      # Source files
│   ├── /popup/
│   │   ├── popup.html
│   │   ├── popup.ts
│   │   └── popup.css
│   ├── /scripts/
│   │   ├── background.ts      # Service worker
│   │   └── content.ts         # Content scripts
│   ├── /options/
│   │   ├── options.html
│   │   ├── options.ts
│   │   └── options.css
│   ├── /newtab/              # Optional: New tab override
│   │   ├── newtab.html
│   │   ├── newtab.ts
│   │   └── newtab.css
│   ├── /utils/
│   │   ├── storage.ts
│   │   ├── messaging.ts
│   │   └── types.ts
│   └── /assets/
│       ├── /icons/
│       ├── /images/
│       └── /fonts/
├── /dist/                     # Build output
├── /_locales/                 # Internationalization
│   └── /en/
│       └── messages.json
├── /tests/
│   ├── /unit/
│   ├── /e2e/
│   └── jest.config.js
└── /.github/
    └── /workflows/
        └── ci.yml
```

## Manifest V3 Architecture

### Service Worker (Background Script)
Service workers replace background pages in Manifest V3 and have specific constraints:

#### Key Characteristics
- Non-persistent: Can be terminated by Chrome at any time
- No DOM access: Cannot use `window` or `document`
- Limited execution time: ~30 seconds per event
- Must be registered in manifest as `background.service_worker`

#### Best Practices
```typescript
// background.ts - Service Worker patterns

// Use Chrome alarms API instead of setInterval/setTimeout
chrome.alarms.create('periodicTask', { periodInMinutes: 5 });

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'periodicTask') {
    performPeriodicTask();
  }
});

// Handle extension lifecycle events
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'install') {
    // First install
    initialiseExtension();
  } else if (details.reason === 'update') {
    // Extension updated
    migrateData(details.previousVersion);
  }
});

// Manage persistent state across sessions
async function saveState(state: AppState): Promise<void> {
  await chrome.storage.local.set({ appState: state });
}

async function loadState(): Promise<AppState | null> {
  const result = await chrome.storage.local.get('appState');
  return result.appState || null;
}

// Process messages from UI components
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  handleMessage(message, sender).then(sendResponse);
  return true; // Required for async sendResponse
});

async function handleMessage(message: Message, sender: chrome.runtime.MessageSender) {
  switch (message.type) {
    case 'GET_DATA':
      return await fetchData();
    case 'SAVE_SETTINGS':
      return await saveSettings(message.data);
    default:
      throw new Error(`Unknown message type: ${message.type}`);
  }
}
```

### Content Scripts
Content scripts run in the context of web pages and must follow specific patterns:

#### Injection Strategies
```json
// manifest.json - Static injection
{
  "content_scripts": [
    {
      "matches": ["https://example.com/*"],
      "js": ["content.js"],
      "run_at": "document_idle",
      "all_frames": false
    }
  ]
}
```

```typescript
// Dynamic injection from background script
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  if (tabs[0]?.id) {
    chrome.scripting.executeScript({
      target: { tabId: tabs[0].id },
      files: ['content.js'],
    });
  }
});
```

#### Best Practices
- **Minimal permissions**: Only request necessary host permissions
- **Selective injection**: Use `matches` patterns to limit injection
- **Message passing**: Communicate with background script via messaging
- **DOM isolation**: Content scripts have isolated JavaScript context
- **CSS isolation**: Use Shadow DOM or unique class prefixes

```typescript
// content.ts - Content script patterns
class ContentScript {
  private shadowRoot: ShadowRoot | null = null;

  constructor() {
    this.initialise();
    this.setupMessageListener();
  }

  private initialise(): void {
    // Create isolated UI using Shadow DOM
    const container = document.createElement('div');
    container.id = 'extension-container';
    document.body.appendChild(container);

    this.shadowRoot = container.attachShadow({ mode: 'open' });
    this.renderUI();
  }

  private setupMessageListener(): void {
    chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
      if (message.type === 'UPDATE_UI') {
        this.updateUI(message.data);
        sendResponse({ success: true });
      }
      return true;
    });
  }

  private async sendToBackground(message: Message): Promise<Response> {
    return chrome.runtime.sendMessage(message);
  }

  private renderUI(): void {
    if (this.shadowRoot) {
      // Inject styles and content into Shadow DOM
      this.shadowRoot.innerHTML = `
        <style>
          /* Isolated styles */
          .container { padding: 10px; }
        </style>
        <div class="container">
          <!-- UI content -->
        </div>
      `;
    }
  }
}

// Initialise content script
new ContentScript();
```

### Storage Management

#### Storage Types
Chrome provides three storage areas:
- **local**: Local device storage (no sync, larger quota)
- **sync**: Synced across devices (100KB limit)
- **session**: Temporary storage (cleared when browser closes)

```typescript
// storage.ts - Type-safe storage utilities
interface StorageData {
  settings: UserSettings;
  cache: CacheData;
  sessionData: SessionData;
}

class StorageManager {
  // Local storage for larger data
  async saveLocal<K extends keyof StorageData>(
    key: K,
    value: StorageData[K]
  ): Promise<void> {
    await chrome.storage.local.set({ [key]: value });
  }

  async loadLocal<K extends keyof StorageData>(
    key: K
  ): Promise<StorageData[K] | null> {
    const result = await chrome.storage.local.get(key);
    return result[key] || null;
  }

  // Sync storage for user preferences
  async saveSync<K extends keyof StorageData>(
    key: K,
    value: StorageData[K]
  ): Promise<void> {
    try {
      await chrome.storage.sync.set({ [key]: value });
    } catch (error) {
      if (error instanceof Error && error.message.includes('QUOTA_EXCEEDED')) {
        // Fallback to local storage
        await this.saveLocal(key, value);
      }
      throw error;
    }
  }

  // Listen for storage changes
  onChanged<K extends keyof StorageData>(
    key: K,
    callback: (newValue: StorageData[K], oldValue: StorageData[K]) => void
  ): void {
    chrome.storage.onChanged.addListener((changes, area) => {
      if (changes[key]) {
        callback(changes[key].newValue, changes[key].oldValue);
      }
    });
  }

  // Handle quota limits
  async checkQuota(): Promise<number> {
    const usage = await chrome.storage.local.getBytesInUse();
    const quota = chrome.storage.local.QUOTA_BYTES;
    return (usage / quota) * 100;
  }
}
```

#### Data Migration
```typescript
// migrations.ts - Version-based data migration
interface Migration {
  version: string;
  migrate: (data: any) => Promise<any>;
}

const migrations: Migration[] = [
  {
    version: '2.0.0',
    migrate: async (data) => {
      // Migrate from v1 to v2 format
      return {
        ...data,
        settings: {
          ...data.settings,
          newField: 'default value',
        },
      };
    },
  },
];

async function runMigrations(currentVersion: string): Promise<void> {
  const data = await chrome.storage.local.get();
  const dataVersion = data.version || '1.0.0';

  for (const migration of migrations) {
    if (compareVersions(dataVersion, migration.version) < 0) {
      const migratedData = await migration.migrate(data);
      await chrome.storage.local.set({
        ...migratedData,
        version: migration.version,
      });
    }
  }
}
```

## UI/UX Best Practices

### Popup Design

#### Size Constraints
- **Recommended dimensions**: 320x480px (Chrome enforces 800x600px hard limit)
- **Minimum viable size**: 300x400px
- **Load time target**: Under 100ms
- **Bundle size**: Keep under 50KB total

```css
/* popup.css - Optimal sizing */
body {
  width: 320px;
  min-height: 400px;
  max-height: 600px;
  margin: 0;
  overflow-y: auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}
```

#### Navigation Patterns
- **Single-page navigation**: Use tabs or collapsible sections
- **Back button simulation**: Breadcrumb navigation for deep workflows
- **Modal overlays**: For complex forms within popup constraints
- **Quick actions**: Most features accessible within 2 clicks

```typescript
// popup.ts - State management
interface PopupState {
  currentView: 'main' | 'settings' | 'help';
  previousView?: string;
  data: Record<string, unknown>;
}

class PopupController {
  private state: PopupState = { currentView: 'main', data: {} };

  navigate(view: PopupState['currentView']): void {
    this.state.previousView = this.state.currentView;
    this.state.currentView = view;
    this.render();
  }

  goBack(): void {
    if (this.state.previousView) {
      this.navigate(this.state.previousView as PopupState['currentView']);
    }
  }

  private render(): void {
    // Render current view
    const container = document.getElementById('app');
    if (container) {
      container.innerHTML = this.getViewHTML(this.state.currentView);
      this.attachEventListeners();
    }
  }

  private getViewHTML(view: PopupState['currentView']): string {
    switch (view) {
      case 'main':
        return '<div>Main View</div>';
      case 'settings':
        return '<div>Settings View</div>';
      case 'help':
        return '<div>Help View</div>';
    }
  }
}
```

#### State Synchronization
```typescript
// Sync state across popup instances
chrome.storage.onChanged.addListener((changes, area) => {
  if (area === 'local' && changes.popupState) {
    updateUI(changes.popupState.newValue);
  }
});

async function syncState(state: PopupState): Promise<void> {
  await chrome.storage.local.set({ popupState: state });
}
```

### Options Page Design

#### Layout Organisation
- **Progressive disclosure**: Group advanced settings under expandable sections
- **Visual hierarchy**: Typography and spacing for clarity
- **Form validation**: Real-time feedback with error messaging
- **Save states**: Indicate unsaved changes and implement auto-save

```html
<!-- options.html - Well-structured settings -->
<main class="settings-container">
  <nav class="settings-nav" role="tablist">
    <button role="tab" aria-selected="true">General</button>
    <button role="tab" aria-selected="false">Privacy</button>
    <button role="tab" aria-selected="false">Advanced</button>
  </nav>

  <section class="settings-content" role="tabpanel">
    <div class="setting-group">
      <h3>Display Preferences</h3>
      <label>
        <input type="checkbox" id="dark-mode" />
        Enable dark mode
        <span class="setting-description">Applies to popup and new tab page</span>
      </label>
    </div>
  </section>
</main>
```

#### Settings Management
```typescript
// settings.ts - Robust settings management
interface Settings {
  theme: 'light' | 'dark' | 'auto';
  notifications: boolean;
  syncEnabled: boolean;
}

class SettingsManager {
  private static readonly SETTINGS_KEY = 'extensionSettings';

  async loadSettings(): Promise<Settings> {
    const result = await chrome.storage.sync.get(SettingsManager.SETTINGS_KEY);
    return { ...this.getDefaultSettings(), ...result[SettingsManager.SETTINGS_KEY] };
  }

  async saveSettings(settings: Partial<Settings>): Promise<void> {
    const current = await this.loadSettings();
    const updated = { ...current, ...settings };
    await chrome.storage.sync.set({ [SettingsManager.SETTINGS_KEY]: updated });

    // Notify all extension contexts
    chrome.runtime.sendMessage({
      type: 'SETTINGS_CHANGED',
      settings: updated,
    });
  }

  private getDefaultSettings(): Settings {
    return {
      theme: 'auto',
      notifications: true,
      syncEnabled: true,
    };
  }

  // Import/Export functionality
  async exportSettings(): Promise<string> {
    const settings = await this.loadSettings();
    return JSON.stringify({
      version: chrome.runtime.getManifest().version,
      timestamp: new Date().toISOString(),
      settings,
    }, null, 2);
  }

  async importSettings(jsonString: string): Promise<void> {
    const data = JSON.parse(jsonString);
    if (this.validateSettings(data.settings)) {
      await this.saveSettings(data.settings);
    } else {
      throw new Error('Invalid settings format');
    }
  }

  private validateSettings(settings: unknown): settings is Settings {
    // Implement validation logic
    return (
      typeof settings === 'object' &&
      settings !== null &&
      'theme' in settings &&
      'notifications' in settings
    );
  }
}
```

### New Tab Page Override

#### Manifest Configuration
```json
{
  "chrome_url_overrides": {
    "newtab": "newtab.html"
  },
  "permissions": ["storage", "topSites", "bookmarks"]
}
```

#### Performance Optimisation
- **Critical path**: Render basic layout within 50ms
- **Progressive enhancement**: Load features asynchronously
- **Image optimisation**: WebP format with lazy loading
- **Search integration**: Debounced search with caching

```typescript
// newtab.ts - Performance-optimized loading
class NewTabController {
  private isInitialLoad = true;

  async initialise(): Promise<void> {
    // Critical path - render immediately
    this.renderBasicLayout();

    // Progressive enhancement
    if (this.isInitialLoad) {
      await this.loadEssentialData();
      this.renderMainContent();

      // Load secondary data in background
      this.loadSecondaryData().then(() => this.renderWidgets());
      this.isInitialLoad = false;
    }
  }

  private renderBasicLayout(): void {
    document.body.innerHTML = `
      <div id="search-container"></div>
      <div id="main-content"></div>
      <div id="widgets"></div>
    `;
  }

  private async loadEssentialData(): Promise<{
    bookmarks: chrome.bookmarks.BookmarkTreeNode[];
    topSites: chrome.topSites.MostVisitedURL[];
  }> {
    const [bookmarks, topSites] = await Promise.all([
      chrome.bookmarks.getTree(),
      chrome.topSites.get(),
    ]);
    return { bookmarks, topSites };
  }

  private async loadSecondaryData(): Promise<void> {
    // Load widgets, weather, news, etc.
  }
}
```

### Consistent Design System

#### CSS Variables
```css
/* design-system.css - Consistent theming */
:root {
  /* Color palette */
  --primary-color: #4285f4;
  --secondary-color: #34a853;
  --error-color: #ea4335;
  --warning-color: #fbbc05;
  --text-primary: #202124;
  --text-secondary: #5f6368;
  --background: #ffffff;
  --surface: #f8f9fa;

  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;

  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-base: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  /* Borders */
  --border-radius: 4px;
  --border-color: #dadce0;
}

/* Dark theme */
@media (prefers-color-scheme: dark) {
  :root {
    --text-primary: #e8eaed;
    --text-secondary: #9aa0a6;
    --background: #202124;
    --surface: #292a2d;
    --border-color: #5f6368;
  }
}
```

#### Reusable Components
```css
/* Common button styles */
.btn {
  padding: var(--spacing-sm) var(--spacing-base);
  border-radius: var(--border-radius);
  font-size: var(--font-size-sm);
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}

.btn-primary {
  background: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background: #3367d6;
}

.btn-secondary {
  background: var(--surface);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
}
```

## Message Passing Patterns

### Type-Safe Messaging
```typescript
// messaging.ts - Type-safe message contracts
interface MessageMap {
  GET_SETTINGS: { request: void; response: Settings };
  UPDATE_BADGE: { request: { text: string; color: string }; response: void };
  FETCH_DATA: { request: { url: string }; response: { success: boolean; data?: unknown } };
}

class MessageHandler {
  static async sendMessage<K extends keyof MessageMap>(
    type: K,
    data: MessageMap[K]['request']
  ): Promise<MessageMap[K]['response']> {
    try {
      const response = await chrome.runtime.sendMessage({ type, data });
      if (response?.error) {
        throw new Error(response.error);
      }
      return response;
    } catch (error) {
      console.error(`Message sending failed for ${type}:`, error);
      throw error;
    }
  }

  static addListener<K extends keyof MessageMap>(
    type: K,
    handler: (
      data: MessageMap[K]['request'],
      sender: chrome.runtime.MessageSender
    ) => Promise<MessageMap[K]['response']>
  ): void {
    chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
      if (message.type === type) {
        handler(message.data, sender)
          .then(sendResponse)
          .catch((error) => sendResponse({ error: error.message }));
        return true; // Required for async response
      }
    });
  }
}

// Usage
MessageHandler.addListener('GET_SETTINGS', async () => {
  return await new SettingsManager().loadSettings();
});

// From popup or content script
const settings = await MessageHandler.sendMessage('GET_SETTINGS', undefined);
```

## Performance Optimisation

### Bundle Size Optimisation
- **Code splitting**: Separate bundles per component
- **Tree shaking**: Remove unused code
- **Dynamic imports**: Load features on-demand
- **Asset compression**: Optimize images and minimise CSS/JS

### Memory Management
```typescript
// Proper cleanup of event listeners
class Component {
  private listener: ((message: Message) => void) | null = null;

  mount(): void {
    this.listener = (message: Message) => this.handleMessage(message);
    chrome.runtime.onMessage.addListener(this.listener);
  }

  unmount(): void {
    if (this.listener) {
      chrome.runtime.onMessage.removeListener(this.listener);
      this.listener = null;
    }
  }

  private handleMessage(message: Message): void {
    // Handle message
  }
}
```

## Security & Privacy

### Permissions
- **Minimal permissions**: Only request what's necessary
- **Optional permissions**: Use `chrome.permissions.request()` for optional features
- **Document permissions**: Explain in privacy policy

```typescript
// Request optional permission at runtime
async function requestNotificationPermission(): Promise<boolean> {
  return chrome.permissions.request({ permissions: ['notifications'] });
}
```

### Content Security Policy
```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  }
}
```

### Data Handling
- **HTTPS only**: All external requests must use HTTPS
- **Data sanitization**: Sanitize user input before storage or display
- **Quota handling**: Handle storage limits gracefully
- **Export functionality**: Allow users to export their data

## Accessibility (WCAG 2.1)

### Requirements
- **Color contrast**: Minimum 4.5:1 for normal text, 3:1 for large text
- **Keyboard navigation**: All functionality accessible via keyboard
- **Screen readers**: Proper semantic HTML and ARIA attributes
- **Focus indicators**: Clear visual focus states
- **Text alternatives**: Alt text for images, labels for controls

```html
<!-- Accessible form controls -->
<label for="setting-input">
  Setting Name
  <input id="setting-input" type="text" aria-describedby="setting-help" />
</label>
<p id="setting-help" class="help-text">Additional context for screen readers</p>
```

## Testing

### Unit Testing (Jest)
```typescript
// Mock Chrome APIs
global.chrome = {
  storage: {
    local: {
      get: jest.fn(),
      set: jest.fn(),
    },
  },
  runtime: {
    sendMessage: jest.fn(),
  },
} as any;

// Test storage manager
describe('StorageManager', () => {
  it('should save and load settings', async () => {
    const manager = new StorageManager();
    const settings: Settings = { theme: 'dark', notifications: true, syncEnabled: true };

    await manager.saveSettings(settings);
    expect(chrome.storage.sync.set).toHaveBeenCalledWith({
      extensionSettings: settings,
    });
  });
});
```

### E2E Testing (Playwright)
```typescript
// Load extension for testing
const context = await chromium.launchPersistentContext('', {
  headless: false,
  args: ['--disable-extensions-except=./dist', '--load-extension=./dist'],
});

// Test extension popup
const extensionId = 'your-extension-id';
const popup = await context.newPage();
await popup.goto(`chrome-extension://${extensionId}/popup.html`);

await popup.click('#settings-button');
await expect(popup.locator('.settings-view')).toBeVisible();
```

## Deployment

### Chrome Web Store
See `git_workflow.md` for semantic versioning and conventional commits.

#### Pre-submission Checklist
- [ ] Manifest V3 compliance
- [ ] All tests passing
- [ ] Privacy policy updated
- [ ] Screenshots prepared
- [ ] Performance metrics acceptable

## Related Standards
- See `typescript_style.md` for TypeScript coding standards
- See `javascript_testing_standards.md` for testing patterns
- See `javascript_webpack_standards.md` for build configuration
- See `git_workflow.md` for versioning and commits
- See `permissions_patterns.md` for permission configuration

## References
- [Chrome Extension Documentation](https://developer.chrome.com/docs/extensions/)
- [Manifest V3 Migration Guide](https://developer.chrome.com/docs/extensions/mv3/intro/)
- [Chrome Extension Best Practices](https://developer.chrome.com/docs/extensions/mv3/best_practices/)
