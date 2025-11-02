# Webpack Build Configuration Standards

## Purpose
This document defines standards for Webpack configuration in JavaScript and TypeScript projects, with specific focus on Chrome extension builds, code optimisation, and modern bundling practices.

## Why Webpack

### Use Cases
- **Chrome Extensions**: Multi-entry bundling for popup, options, background, content scripts
- **TypeScript Projects**: Transpilation and type checking integration
- **Code Splitting**: Optimize bundle sizes with shared chunks
- **Asset Management**: Handle CSS, images, fonts, and other assets
- **Development Experience**: Hot reloading and source maps

### Alternatives
- **Vite**: Faster development builds, simpler configuration (good alternative for modern projects)
- **Rollup**: Library bundling (better for npm packages)
- **esbuild**: Extremely fast, minimal configuration (good for simple projects)

## Base Configuration

### webpack.config.js Structure
```javascript
const path = require('path');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    mode: argv.mode || 'development',
    devtool: isProduction ? 'source-map' : 'inline-source-map',
    entry: {
      popup: './src/popup/popup.ts',
      options: './src/options/options.ts',
      background: './src/scripts/background.ts',
      content: './src/scripts/content.ts',
    },
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: '[name].js',
      clean: true,
    },
    resolve: {
      extensions: ['.ts', '.tsx', '.js', '.jsx'],
      alias: {
        '@': path.resolve(__dirname, 'src'),
      },
    },
    module: {
      rules: [
        // TypeScript
        {
          test: /\.tsx?$/,
          use: 'ts-loader',
          exclude: /node_modules/,
        },
        // CSS
        {
          test: /\.css$/,
          use: ['style-loader', 'css-loader'],
        },
        // Images
        {
          test: /\.(png|svg|jpg|jpeg|gif)$/i,
          type: 'asset/resource',
          generator: {
            filename: 'assets/images/[name][ext]',
          },
        },
        // Fonts
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/i,
          type: 'asset/resource',
          generator: {
            filename: 'assets/fonts/[name][ext]',
          },
        },
      ],
    },
    plugins: [
      new CleanWebpackPlugin(),
      new CopyWebpackPlugin({
        patterns: [
          { from: 'manifest.json', to: 'manifest.json' },
          { from: 'src/popup/popup.html', to: 'popup.html' },
          { from: 'src/options/options.html', to: 'options.html' },
          { from: 'src/assets/icons', to: 'assets/icons' },
          { from: '_locales', to: '_locales' },
        ],
      }),
    ],
    optimisation: {
      minimise: isProduction,
      splitChunks: {
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
          common: {
            name: 'common',
            minChunks: 2,
            chunks: 'all',
            enforce: true,
          },
        },
      },
    },
  };
};
```

## Entry Points

### Multi-Entry Configuration
For Chrome extensions, define separate entries for each component:

```javascript
entry: {
  // UI components
  popup: './src/popup/popup.ts',
  options: './src/options/options.ts',
  newtab: './src/newtab/newtab.ts',

  // Scripts
  background: './src/scripts/background.ts',
  content: './src/scripts/content.ts',

  // Additional content scripts
  'content-injected': './src/scripts/content-injected.ts',
}
```

### Dynamic Entry Points
```javascript
const glob = require('glob');
const path = require('path');

const contentScripts = glob.sync('./src/content/**/*.ts').reduce((entries, file) => {
  const name = path.basename(file, '.ts');
  entries[`content/${name}`] = file;
  return entries;
}, {});

module.exports = {
  entry: {
    popup: './src/popup/popup.ts',
    background: './src/scripts/background.ts',
    ...contentScripts,
  },
};
```

## Loaders

### TypeScript Loader
```javascript
module: {
  rules: [
    {
      test: /\.tsx?$/,
      use: [
        {
          loader: 'ts-loader',
          options: {
            transpileOnly: false, // Set true for faster builds, false for type checking
            compilerOptions: {
              module: 'esnext',
            },
          },
        },
      ],
      exclude: /node_modules/,
    },
  ],
}
```

### CSS Loaders
```javascript
// For Chrome extensions - inline CSS
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
}

// For separate CSS files
{
  test: /\.css$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'postcss-loader',
  ],
}

// For SCSS/SASS
{
  test: /\.s[ac]ss$/,
  use: [
    'style-loader',
    'css-loader',
    'sass-loader',
  ],
}
```

### Asset Loaders
```javascript
module: {
  rules: [
    // Webpack 5 asset modules
    {
      test: /\.(png|svg|jpg|jpeg|gif|webp)$/i,
      type: 'asset/resource',
      generator: {
        filename: 'assets/images/[name].[hash:8][ext]',
      },
    },
    {
      test: /\.(woff|woff2|eot|ttf|otf)$/i,
      type: 'asset/resource',
      generator: {
        filename: 'assets/fonts/[name][ext]',
      },
    },
    // Inline small assets
    {
      test: /\.svg$/,
      type: 'asset',
      parser: {
        dataUrlCondition: {
          maxSize: 8 * 1024, // 8kb
        },
      },
    },
  ],
}
```

## Plugins

### Essential Plugins

#### CopyWebpackPlugin
```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin');

plugins: [
  new CopyWebpackPlugin({
    patterns: [
      {
        from: 'manifest.json',
        to: 'manifest.json',
        transform(content) {
          // Optionally transform manifest
          const manifest = JSON.parse(content);
          manifest.version = process.env.npm_package_version;
          return JSON.stringify(manifest, null, 2);
        },
      },
      { from: 'src/**/*.html', to: '[name][ext]' },
      { from: 'src/assets/icons', to: 'assets/icons' },
      { from: '_locales', to: '_locales' },
    ],
  }),
]
```

#### CleanWebpackPlugin
```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

plugins: [
  new CleanWebpackPlugin({
    cleanOnceBeforeBuildPatterns: ['**/*', '!.gitkeep'],
  }),
]
```

#### HtmlWebpackPlugin
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
  new HtmlWebpackPlugin({
    template: './src/popup/popup.html',
    filename: 'popup.html',
    chunks: ['popup'],
    inject: 'body',
  }),
  new HtmlWebpackPlugin({
    template: './src/options/options.html',
    filename: 'options.html',
    chunks: ['options'],
    inject: 'body',
  }),
]
```

#### DefinePlugin
```javascript
const webpack = require('webpack');

plugins: [
  new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    'process.env.VERSION': JSON.stringify(require('./package.json').version),
    __DEV__: JSON.stringify(!isProduction),
  }),
]
```

## Optimisation

### Code Splitting
```javascript
optimisation: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      // Vendor code (node_modules)
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        priority: 10,
        reuseExistingChunk: true,
      },
      // Common code shared across entries
      common: {
        name: 'common',
        minChunks: 2,
        priority: 5,
        reuseExistingChunk: true,
        enforce: true,
      },
      // Specific libraries
      react: {
        test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
        name: 'react-vendor',
        priority: 20,
      },
    },
  },
  // Runtime chunk for better caching
  runtimeChunk: {
    name: 'runtime',
  },
}
```

### Minimization
```javascript
const TerserPlugin = require('terser-webpack-plugin');

optimisation: {
  minimise: isProduction,
  minimiser: [
    new TerserPlugin({
      terserOptions: {
        compress: {
          drop_console: isProduction, // Remove console.log in production
          drop_debugger: true,
        },
        format: {
          comments: false, // Remove comments
        },
      },
      extractComments: false,
    }),
  ],
}
```

## Development Configuration

### Development Server (Web Projects)
```javascript
devServer: {
  static: {
    directory: path.join(__dirname, 'dist'),
  },
  hot: true,
  port: 3000,
  open: true,
  historyApiFallback: true,
}
```

### Watch Mode (Chrome Extensions)
```javascript
// For Chrome extensions, use watch mode instead of dev server
module.exports = {
  watch: process.env.NODE_ENV === 'development',
  watchOptions: {
    ignored: /node_modules/,
    aggregateTimeout: 300,
    poll: 1000, // Use polling if file watching doesn't work
  },
}
```

### Source Maps
```javascript
module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    // Development: Fast rebuild with inline source maps
    devtool: isProduction
      ? 'source-map'           // Production: Separate .map files
      : 'eval-source-map',     // Development: Fast rebuild

    // Alternative options:
    // 'cheap-module-source-map'  - Faster, less detailed
    // 'inline-source-map'        - Inline for debugging
    // 'nosources-source-map'     - Hide source code
  };
};
```

## Performance Optimisation

### Bundle Analysis
```javascript
const { BundleAnalyserPlugin } = require('webpack-bundle-analyser');

plugins: [
  new BundleAnalyserPlugin({
    analyserMode: process.env.ANALYZE ? 'server' : 'disabled',
    openAnalyser: true,
  }),
]

// Usage: ANALYZE=true npm run build
```

### Performance Budgets
```javascript
performance: {
  hints: 'warning',
  maxEntrypointSize: 512000,  // 500kb
  maxAssetSize: 512000,
  assetFilter: (assetFilename) => {
    // Only check JS files
    return assetFilename.endsWith('.js');
  },
}
```

### Tree Shaking
```javascript
// In package.json
{
  "sideEffects": [
    "*.css",
    "*.scss"
  ]
}

// In webpack config
optimisation: {
  usedExports: true,  // Mark unused exports
  sideEffects: true,   // Remove unused code
}
```

## Environment-Specific Configurations

### Multiple Config Files
```javascript
// webpack.common.js - Shared configuration
module.exports = {
  entry: { /* ... */ },
  module: { /* ... */ },
  plugins: [ /* ... */ ],
};

// webpack.dev.js - Development
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-source-map',
  watch: true,
});

// webpack.prod.js - Production
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const TerserPlugin = require('terser-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  optimisation: {
    minimise: true,
    minimiser: [new TerserPlugin()],
  },
});
```

### package.json Scripts
```json
{
  "scripts": {
    "build": "webpack --config webpack.prod.js",
    "build:dev": "webpack --config webpack.dev.js",
    "build:watch": "webpack --config webpack.dev.js --watch",
    "build:analyse": "ANALYZE=true webpack --config webpack.prod.js"
  }
}
```

## Chrome Extension Specific

### Manifest Transformation
```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin');

plugins: [
  new CopyWebpackPlugin({
    patterns: [
      {
        from: 'manifest.json',
        to: 'manifest.json',
        transform(content) {
          const manifest = JSON.parse(content.toString());

          // Update version from package.json
          manifest.version = process.env.npm_package_version;

          // Add development CSP for hot reload
          if (process.env.NODE_ENV === 'development') {
            manifest.content_security_policy = {
              extension_pages: "script-src 'self' 'unsafe-eval'; object-src 'self'",
            };
          }

          return JSON.stringify(manifest, null, 2);
        },
      },
    ],
  }),
]
```

### Content Script CSS Injection
```javascript
// For content scripts, inline CSS to avoid CSP issues
{
  test: /\.css$/,
  include: path.resolve(__dirname, 'src/scripts'),
  use: [
    'style-loader',
    'css-loader',
  ],
}
```

## TypeScript Integration

### Path Aliases
```javascript
// webpack.config.js
resolve: {
  alias: {
    '@': path.resolve(__dirname, 'src'),
    '@utils': path.resolve(__dirname, 'src/utils'),
    '@components': path.resolve(__dirname, 'src/components'),
  },
}

// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@utils/*": ["src/utils/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```

### Type Checking
```javascript
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

plugins: [
  new ForkTsCheckerWebpackPlugin({
    typescript: {
      configFile: path.resolve(__dirname, 'tsconfig.json'),
      diagnosticOptions: {
        semantic: true,
        syntactic: true,
      },
    },
  }),
]
```

## Common Patterns

### Development vs Production
```javascript
module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';
  const isDevelopment = !isProduction;

  return {
    mode: isProduction ? 'production' : 'development',
    devtool: isProduction ? 'source-map' : 'eval-source-map',
    output: {
      filename: isProduction ? '[name].[contenthash].js' : '[name].js',
    },
    optimisation: {
      minimise: isProduction,
    },
    plugins: [
      isDevelopment && new SomeDevelopmentPlugin(),
      isProduction && new SomeProductionPlugin(),
    ].filter(Boolean),
  };
};
```

### Cache Busting
```javascript
output: {
  filename: isProduction
    ? '[name].[contenthash:8].js'
    : '[name].js',
  chunkFilename: isProduction
    ? '[name].[contenthash:8].chunk.js'
    : '[name].chunk.js',
}
```

## Troubleshooting

### Common Issues

#### Module Not Found
```javascript
// Ensure proper resolve configuration
resolve: {
  extensions: ['.ts', '.tsx', '.js', '.jsx', '.json'],
  modules: [path.resolve(__dirname, 'src'), 'node_modules'],
}
```

#### CSS Loading Issues
```javascript
// For Chrome extensions, ensure CSS is imported in TS files
// popup.ts
import './popup.css';

// Not in HTML:
// <link rel="stylesheet" href="popup.css"> ❌
```

#### Large Bundle Size
```bash
# Analyse bundle
ANALYZE=true npm run build

# Solutions:
# - Enable code splitting
# - Use dynamic imports
# - Remove unused dependencies
# - Use lighter alternatives
```

## Build Performance

### Faster Builds
```javascript
module.exports = {
  cache: {
    type: 'filesystem',
    cacheDirectory: path.resolve(__dirname, '.webpack_cache'),
  },
  optimisation: {
    moduleIds: 'deterministic',
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: {
          loader: 'ts-loader',
          options: {
            transpileOnly: true, // Faster, skip type checking
          },
        },
      },
    ],
  },
}
```

## Migration to Vite

### When to Consider Vite
- New projects without legacy requirements
- Need faster development builds
- Simpler configuration desired
- Native ES modules support

### Basic Vite Config for Chrome Extension
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        popup: resolve(__dirname, 'src/popup/popup.html'),
        options: resolve(__dirname, 'src/options/options.html'),
        background: resolve(__dirname, 'src/scripts/background.ts'),
        content: resolve(__dirname, 'src/scripts/content.ts'),
      },
      output: {
        entryFileNames: '[name].js',
        chunkFileNames: '[name].js',
        assetFileNames: 'assets/[name].[ext]',
      },
    },
    outDir: 'dist',
  },
});
```

## Related Standards
- See `typescript_style.md` for TypeScript configuration
- See `chrome_extension_standards.md` for extension-specific patterns
- See `javascript_testing.md` for test configuration
- See `performance_considerations.md` for optimisation strategies

## References
- [Webpack Documentation](https://webpack.js.org/concepts/)
- [Webpack Chrome Extension Template](https://github.com/webpack/webpack/tree/main/examples)
- [Vite Documentation](https://vitejs.dev/)
- [Bundle Optimisation Guide](https://webpack.js.org/guides/build-performance/)
