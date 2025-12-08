# ⚡ **Angular Build Optimization Strategies**

## 🎯 **What You'll Learn**

Master **production-ready build optimization** for Angular applications! Learn webpack tuning, bundle splitting, CI/CD optimization, and performance monitoring. Think of build optimization as **fine-tuning a race car** - every millisecond counts! 🏎️

---

## 📚 **The Build Performance Challenge**

### **🤔 Why Build Optimization Matters**

```bash
# ❌ Before optimization
ng build --prod
✓ Initial chunk files   | Names         |  Raw Size
  main.js               | main          |   2.1 MB  😱
  vendor.js             | vendor        |   8.5 MB  😱

Build time: 4m 23s      | Bundle size: 10.6 MB 😱
First Contentful Paint: 8.2s
Time to Interactive: 12.5s

# ✅ After optimization
ng build --prod
✓ Initial chunk files   | Names         |  Raw Size
  main.js               | main          |   245 KB  ✨
  vendor.js             | vendor        |   1.8 MB  ✨
  feature-*.js          | lazy chunks   |   ~150 KB each ✨

Build time: 45s         | Bundle size: 2.9 MB ✨
First Contentful Paint: 1.8s
Time to Interactive: 3.2s
```

### **🎯 Optimization Goals**

- 🚀 **Build Speed** - Faster development and CI/CD pipelines
- 📦 **Bundle Size** - Smaller JavaScript chunks for faster loading
- ⚡ **Runtime Performance** - Optimized code execution
- 🔄 **Cache Efficiency** - Better browser and CDN caching
- 🛠️ **Developer Experience** - Fast rebuilds and hot reloading

---

## 🛠️ **Webpack Optimization Strategies**

### **1. ⚡ Development Build Optimization**

```javascript
// webpack.dev.js - Development-optimized configuration
const path = require("path");
const webpack = require("webpack");

module.exports = (config, options) => {
  console.log("🔧 Applying development optimizations...");

  // 🚀 Faster compilation with eval-cheap-source-map
  config.devtool = "eval-cheap-module-source-map";

  // 🎯 Faster rebuilds with persistent caching
  config.cache = {
    type: "filesystem",
    version: "1.0",
    cacheDirectory: path.resolve(__dirname, ".webpack-cache"),
    store: "pack",
    buildDependencies: {
      config: [__filename, path.resolve(__dirname, "package-lock.json")],
    },
  };

  // ⚡ Module resolution optimizations
  config.resolve = {
    ...config.resolve,
    // Faster module resolution
    extensions: [".ts", ".js", ".json"],
    modules: ["node_modules"],
    // Skip expensive lookups
    symlinks: false,
    // Use absolute paths for better caching
    alias: {
      "@app": path.resolve(__dirname, "src/app"),
      "@shared": path.resolve(__dirname, "src/app/shared"),
      "@core": path.resolve(__dirname, "src/app/core"),
      "@environments": path.resolve(__dirname, "src/environments"),
    },
  };

  // 🔄 Hot Module Replacement optimization
  config.plugins.push(
    new webpack.HotModuleReplacementPlugin(),

    // Faster development server
    new webpack.ProgressPlugin((percentage, message) => {
      if (percentage === 1) {
        console.log("✅ Build completed!");
      }
    })
  );

  // 📊 Development server optimizations
  config.devServer = {
    ...config.devServer,
    // Faster compilation
    lazy: false,
    // Better caching
    watchOptions: {
      poll: false,
      ignored: /node_modules/,
      aggregateTimeout: 300,
    },
    // Faster serving
    compress: true,
    // Memory optimizations
    watchContentBase: false,
  };

  // 🎯 TypeScript optimization for development
  const tsRule = config.module.rules.find(
    (rule) => rule.test && rule.test.toString().includes("ts")
  );

  if (tsRule && tsRule.use) {
    const angularCompilerOptions = tsRule.use.find(
      (use) => use.loader && use.loader.includes("angular-webpack-loader")
    );

    if (angularCompilerOptions && angularCompilerOptions.options) {
      // Faster compilation in development
      angularCompilerOptions.options = {
        ...angularCompilerOptions.options,
        skipCodeGeneration: true,
        skipTemplateCodegen: true,
        preserveSymlinks: true,
      };
    }
  }

  console.log("✅ Development optimizations applied");
  return config;
};
```

### **2. 🚀 Production Build Optimization**

```javascript
// webpack.prod.js - Production-optimized configuration
const path = require("path");
const webpack = require("webpack");
const CompressionPlugin = require("compression-webpack-plugin");
const BrotliPlugin = require("brotli-webpack-plugin");
const BundleAnalyzerPlugin =
  require("webpack-bundle-analyzer").BundleAnalyzerPlugin;
const { SubresourceIntegrityPlugin } = require("webpack-subresource-integrity");

module.exports = (config, options) => {
  console.log("🚀 Applying production optimizations...");

  // 📦 Advanced bundle splitting
  config.optimization = {
    ...config.optimization,

    // Split chunks strategy
    splitChunks: {
      chunks: "all",
      minSize: 20000,
      minRemainingSize: 0,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,

      cacheGroups: {
        // 🏗️ Framework code (Angular, RxJS)
        framework: {
          test: /[\\/]node_modules[\\/](@angular|rxjs)[\\/]/,
          name: "framework",
          priority: 40,
          chunks: "all",
          reuseExistingChunk: true,
        },

        // 🔧 Vendor libraries
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          priority: 30,
          chunks: "all",
          reuseExistingChunk: true,
          // Split large vendors
          maxSize: 250000,
        },

        // 🎨 UI libraries (Material, etc.)
        ui: {
          test: /[\\/]node_modules[\\/](@angular\/material|@angular\/cdk)[\\/]/,
          name: "ui-framework",
          priority: 35,
          chunks: "all",
          reuseExistingChunk: true,
        },

        // 🔄 Common utilities
        common: {
          name: "common",
          minChunks: 2,
          priority: 20,
          chunks: "all",
          reuseExistingChunk: true,
          maxSize: 200000,
        },

        // 📊 Polyfills
        polyfills: {
          test: /[\\/]node_modules[\\/](core-js|zone\.js)[\\/]/,
          name: "polyfills",
          priority: 45,
          chunks: "all",
        },
      },
    },

    // 🎯 Advanced minification
    minimize: true,
    minimizer: [
      // JavaScript minification
      new (require("terser-webpack-plugin"))({
        terserOptions: {
          parse: {
            ecma: 2017,
          },
          compress: {
            ecma: 2017,
            warnings: false,
            comparisons: false,
            inline: 2,
            drop_console: true,
            drop_debugger: true,
            pure_getters: true,
            passes: 3,
          },
          mangle: {
            safari10: true,
            properties: {
              regex: /^_/,
            },
          },
          output: {
            ecma: 2017,
            comments: false,
            ascii_only: true,
          },
        },
        parallel: true,
        extractComments: false,
      }),

      // CSS minification
      new (require("css-minimizer-webpack-plugin"))({
        minimizerOptions: {
          preset: [
            "default",
            {
              discardComments: { removeAll: true },
              normalizeCharset: false,
            },
          ],
        },
      }),
    ],

    // 📝 Better module IDs for caching
    moduleIds: "deterministic",
    chunkIds: "deterministic",
  };

  // 🗜️ Compression plugins
  config.plugins.push(
    // Gzip compression
    new CompressionPlugin({
      filename: "[path][base].gz",
      algorithm: "gzip",
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8,
      compressionOptions: {
        level: 9,
      },
    }),

    // Brotli compression (better than gzip)
    new BrotliPlugin({
      asset: "[path].br[query]",
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8,
      quality: 11,
    }),

    // Security: Subresource integrity
    new SubresourceIntegrityPlugin({
      hashFuncNames: ["sha256", "sha384"],
      enabled: true,
    }),

    // Bundle analysis (conditional)
    ...(process.env.ANALYZE_BUNDLE === "true"
      ? [
          new BundleAnalyzerPlugin({
            analyzerMode: "static",
            openAnalyzer: false,
            reportFilename: "bundle-report.html",
            generateStatsFile: true,
            statsFilename: "bundle-stats.json",
          }),
        ]
      : []),

    // Build progress
    new webpack.ProgressPlugin({
      activeModules: false,
      entries: true,
      handler(percentage, message, ...args) {
        console.info(`🔨 ${(percentage * 100).toFixed(1)}%`, message, ...args);
      },
    })
  );

  // 🎯 Performance hints
  config.performance = {
    hints: "warning",
    maxEntrypointSize: 512000, // 500kb
    maxAssetSize: 250000, // 250kb
    assetFilter: (assetFilename) => {
      // Only warn about JS and CSS files
      return /\.(js|css)$/.test(assetFilename);
    },
  };

  console.log("✅ Production optimizations applied");
  return config;
};
```

### **3. 🎯 Angular-Specific Optimizations**

```typescript
// angular.json - Build configuration optimizations
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular-builders/custom-webpack:browser",
          "options": {
            "customWebpackConfig": {
              "path": "./webpack.config.js",
              "mergeStrategies": {
                "module.rules": "prepend"
              }
            },
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",

            // 🚀 Build optimizations
            "optimization": true,
            "buildOptimizer": true,
            "aot": true,
            "vendorChunk": true,
            "extractLicenses": false,
            "sourceMap": false,
            "namedChunks": false,

            // 🎯 Bundle budgets (performance monitoring)
            "budgets": [
              {
                "type": "initial",
                "maximumWarning": "2mb",
                "maximumError": "5mb"
              },
              {
                "type": "anyComponentStyle",
                "maximumWarning": "6kb",
                "maximumError": "10kb"
              },
              {
                "type": "bundle",
                "name": "main",
                "maximumWarning": "1mb",
                "maximumError": "2mb"
              },
              {
                "type": "bundle",
                "name": "vendor",
                "maximumWarning": "2mb",
                "maximumError": "3mb"
              }
            ]
          },
          "configurations": {
            "production": {
              // 🔥 Production-specific optimizations
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.prod.ts"
                }
              ],
              "optimization": true,
              "outputHashing": "all",
              "sourceMap": false,
              "namedChunks": false,
              "extractLicenses": true,
              "vendorChunk": false,
              "buildOptimizer": true,

              // 🎯 Advanced Angular optimizations
              "serviceWorker": true,
              "ngswConfigPath": "ngsw-config.json",
              "index": {
                "input": "src/index.html",
                "output": "index.html"
              }
            },

            "development": {
              // 🚀 Fast development builds
              "optimization": false,
              "extractLicenses": false,
              "sourceMap": true,
              "vendorChunk": true,
              "buildOptimizer": false,
              "namedChunks": true,
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.dev.ts"
                }
              ]
            }
          }
        },

        "serve": {
          "builder": "@angular-builders/custom-webpack:dev-server",
          "options": {
            "customWebpackConfig": {
              "path": "./webpack.dev.js"
            }
          },
          "configurations": {
            "production": {
              "buildTarget": "my-app:build:production"
            },
            "development": {
              "buildTarget": "my-app:build:development"
            }
          }
        }
      }
    }
  }
}
```

---

## 🔄 **Advanced Lazy Loading Strategies**

### **1. 🎯 Component-Level Lazy Loading**

```typescript
// src/app/shared/services/lazy-loader.service.ts
import {
  Injectable,
  ComponentFactoryResolver,
  ViewContainerRef,
} from "@angular/core";
import { from, Observable } from "rxjs";
import { map, catchError } from "rxjs/operators";

@Injectable({
  providedIn: "root",
})
export class LazyLoaderService {
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {}

  // 📦 Lazy load individual components
  loadComponent<T>(
    componentLoader: () => Promise<any>,
    viewContainer: ViewContainerRef
  ): Observable<T> {
    return from(componentLoader()).pipe(
      map(({ component }) => {
        const factory =
          this.componentFactoryResolver.resolveComponentFactory(component);
        const componentRef = viewContainer.createComponent(factory);
        return componentRef.instance as T;
      }),
      catchError((error) => {
        console.error("Failed to load component:", error);
        throw error;
      })
    );
  }

  // 🔄 Lazy load services
  loadService<T>(serviceLoader: () => Promise<any>): Observable<T> {
    return from(serviceLoader()).pipe(
      map(({ service }) => new service()),
      catchError((error) => {
        console.error("Failed to load service:", error);
        throw error;
      })
    );
  }

  // 📊 Conditional lazy loading based on features
  conditionalLoad<T>(
    condition: boolean,
    loader: () => Promise<any>
  ): Observable<T | null> {
    if (!condition) {
      return of(null);
    }
    return from(loader()).pipe(
      map((module) => module.default || module),
      catchError((error) => {
        console.warn("Conditional load failed:", error);
        return of(null);
      })
    );
  }
}

// Usage in components
@Component({
  template: `
    <div class="feature-container">
      <button (click)="loadAdvancedFeature()" [disabled]="loading">
        {{ loading ? "Loading..." : "Load Advanced Feature" }}
      </button>

      <div #dynamicComponent></div>
    </div>
  `,
})
export class FeatureHostComponent {
  @ViewChild("dynamicComponent", { read: ViewContainerRef })
  dynamicContainer!: ViewContainerRef;

  loading = false;

  constructor(private lazyLoader: LazyLoaderService) {}

  async loadAdvancedFeature() {
    this.loading = true;

    try {
      // Lazy load heavy component only when needed
      const component = await this.lazyLoader
        .loadComponent(
          () => import("./advanced-charts/advanced-charts.component"),
          this.dynamicContainer
        )
        .toPromise();

      // Configure the loaded component
      if (component) {
        component.data = this.getChartData();
        component.config = this.getChartConfig();
      }
    } catch (error) {
      console.error("Failed to load advanced feature:", error);
    } finally {
      this.loading = false;
    }
  }
}
```

### **2. 📊 Route-Based Progressive Loading**

```typescript
// src/app/core/strategies/progressive-preload.strategy.ts
import { PreloadingStrategy, Route } from "@angular/router";
import { Observable, of, timer } from "rxjs";
import { mergeMap } from "rxjs/operators";

export interface ProgressivePreloadConfig {
  delay?: number;
  priority?: "high" | "medium" | "low";
  conditions?: {
    networkSpeed?: "fast" | "slow";
    deviceMemory?: number;
    userEngagement?: boolean;
  };
}

@Injectable()
export class ProgressivePreloadStrategy implements PreloadingStrategy {
  private networkSpeed = this.detectNetworkSpeed();
  private deviceMemory = this.getDeviceMemory();
  private userEngagement = false;

  preload(route: Route, load: Function): Observable<any> {
    const config = route.data?.preloadConfig as ProgressivePreloadConfig;

    if (!config || !this.shouldPreload(config)) {
      return of(null);
    }

    const delay = this.calculateDelay(config);

    console.log(`🚀 Progressive preloading: ${route.path} (delay: ${delay}ms)`);

    return timer(delay).pipe(
      mergeMap(() => {
        console.log(`📦 Loading module: ${route.path}`);
        return load();
      })
    );
  }

  private shouldPreload(config: ProgressivePreloadConfig): boolean {
    const { conditions } = config;

    if (!conditions) return true;

    // Check network speed
    if (conditions.networkSpeed === "fast" && this.networkSpeed !== "fast") {
      return false;
    }

    // Check device memory
    if (
      conditions.deviceMemory &&
      this.deviceMemory < conditions.deviceMemory
    ) {
      return false;
    }

    // Check user engagement
    if (conditions.userEngagement && !this.userEngagement) {
      return false;
    }

    return true;
  }

  private calculateDelay(config: ProgressivePreloadConfig): number {
    const baseDelay = config.delay || 0;

    // Adjust delay based on priority and device capabilities
    switch (config.priority) {
      case "high":
        return Math.max(0, baseDelay - 1000);
      case "low":
        return baseDelay + 2000;
      default:
        return baseDelay;
    }
  }

  private detectNetworkSpeed(): "fast" | "slow" {
    // Use Network Information API if available
    const connection = (navigator as any).connection;
    if (connection) {
      const effectiveType = connection.effectiveType;
      return ["4g", "slow-2g"].includes(effectiveType) ? "fast" : "slow";
    }
    return "fast"; // Default assumption
  }

  private getDeviceMemory(): number {
    // Use Device Memory API if available
    const memory = (navigator as any).deviceMemory;
    return memory || 4; // Default to 4GB
  }

  // Track user engagement
  trackUserEngagement() {
    let interactionCount = 0;
    const events = ["click", "scroll", "keypress"];

    const handler = () => {
      interactionCount++;
      if (interactionCount >= 3) {
        this.userEngagement = true;
        events.forEach((event) => document.removeEventListener(event, handler));
      }
    };

    events.forEach((event) =>
      document.addEventListener(event, handler, { passive: true })
    );
  }
}

// Route configuration with progressive preloading
const routes: Routes = [
  {
    path: "dashboard",
    loadChildren: () =>
      import("./features/dashboard/dashboard.module").then(
        (m) => m.DashboardModule
      ),
    data: {
      preloadConfig: {
        priority: "high",
        delay: 0,
      } as ProgressivePreloadConfig,
    },
  },
  {
    path: "reports",
    loadChildren: () =>
      import("./features/reports/reports.module").then((m) => m.ReportsModule),
    data: {
      preloadConfig: {
        priority: "medium",
        delay: 2000,
        conditions: {
          networkSpeed: "fast",
          userEngagement: true,
        },
      } as ProgressivePreloadConfig,
    },
  },
  {
    path: "analytics",
    loadChildren: () =>
      import("./features/analytics/analytics.module").then(
        (m) => m.AnalyticsModule
      ),
    data: {
      preloadConfig: {
        priority: "low",
        delay: 5000,
        conditions: {
          deviceMemory: 4,
          networkSpeed: "fast",
        },
      } as ProgressivePreloadConfig,
    },
  },
];
```

---

## 🏗️ **CI/CD Build Pipeline Optimization**

### **1. 🚀 Docker Multi-Stage Builds**

```dockerfile
# Dockerfile.optimized - Multi-stage build for smaller images
# 📦 Stage 1: Dependencies and build environment
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# 🎯 Copy package files first (better caching)
COPY package*.json ./
COPY angular.json ./
COPY tsconfig*.json ./

# 🔄 Install dependencies with npm ci for faster, reliable builds
RUN npm ci --only=production --silent

# 📁 Copy source code
COPY src/ src/
COPY .browserslistrc ./

# 🏗️ Build the application
ENV NODE_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=4096"

RUN npm run build:prod -- --output-path=dist

# 📊 Stage 2: Bundle analyzer (optional)
FROM builder AS analyzer
RUN npm run build:analyze

# 🗜️ Stage 3: Production runtime
FROM nginx:alpine AS production

# 🔧 Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# 📦 Copy built application
COPY --from=builder /app/dist/ /usr/share/nginx/html/

# 🔒 Security: Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S angular -u 1001
USER angular

# 🚀 Expose port
EXPOSE 80

# 🎯 Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# .github/workflows/build-optimize.yml
name: Optimized Build Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: "18"
  CACHE_VERSION: "v1"

jobs:
  # 🔍 Pre-build analysis
  analyze:
    runs-on: ubuntu-latest
    outputs:
      should-build: ${{ steps.changes.outputs.should-build }}
      cache-key: ${{ steps.cache-key.outputs.key }}
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: 📊 Analyze changes
        id: changes
        run: |
          if git diff --name-only HEAD^ | grep -E '\.(ts|js|html|scss|json)$'; then
            echo "should-build=true" >> $GITHUB_OUTPUT
          else
            echo "should-build=false" >> $GITHUB_OUTPUT
          fi

      - name: 🔑 Generate cache key
        id: cache-key
        run: |
          echo "key=${{ env.CACHE_VERSION }}-${{ runner.os }}-node-${{ env.NODE_VERSION }}-${{ hashFiles('package-lock.json') }}" >> $GITHUB_OUTPUT

  # 🏗️ Optimized build job
  build:
    needs: analyze
    if: needs.analyze.outputs.should-build == 'true'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        build-type: [production, development]

    steps:
      - uses: actions/checkout@v3

      - name: 📦 Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: 💾 Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ needs.analyze.outputs.cache-key }}
          restore-keys: |
            ${{ env.CACHE_VERSION }}-${{ runner.os }}-node-${{ env.NODE_VERSION }}-

      - name: 🔄 Install dependencies
        run: npm ci --prefer-offline --no-audit

      - name: 🧪 Run tests (parallel)
        run: |
          npm run test:ci &
          npm run lint &
          wait

      - name: 🏗️ Build application
        run: |
          if [ "${{ matrix.build-type }}" = "production" ]; then
            npm run build:prod
          else
            npm run build:dev
          fi
        env:
          NODE_OPTIONS: "--max-old-space-size=4096"

      - name: 📊 Bundle analysis
        if: matrix.build-type == 'production'
        run: |
          npm run build:analyze
          echo "## Bundle Analysis 📊" >> $GITHUB_STEP_SUMMARY
          echo "![Bundle Size](./dist/bundle-report.html)" >> $GITHUB_STEP_SUMMARY

      - name: 🗜️ Compress artifacts
        run: |
          cd dist
          tar -czf ../build-${{ matrix.build-type }}.tar.gz .

      - name: 📤 Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ matrix.build-type }}-${{ github.sha }}
          path: build-${{ matrix.build-type }}.tar.gz
          retention-days: 7

  # 🚀 Docker build with optimization
  docker:
    needs: [analyze, build]
    if: needs.analyze.outputs.should-build == 'true' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: 🐳 Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: 🔐 Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 📦 Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile.optimized
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # 📈 Performance monitoring
  performance:
    needs: docker
    runs-on: ubuntu-latest
    steps:
      - name: 🔍 Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            https://staging.example.com
          uploadDir: "./lhci_reports"
          temporaryPublicStorage: true
```

### **2. 🎯 Build Performance Monitoring**

```typescript
// scripts/build-monitor.ts - Build performance tracking
import * as fs from "fs";
import * as path from "path";
import { execSync } from "child_process";

interface BuildMetrics {
  timestamp: number;
  buildTime: number;
  bundleSize: {
    total: number;
    main: number;
    vendor: number;
    lazy: number;
  };
  chunks: Array<{
    name: string;
    size: number;
    type: "initial" | "async";
  }>;
  performance: {
    firstContentfulPaint: number;
    timeToInteractive: number;
    totalBlockingTime: number;
  };
}

class BuildMonitor {
  private metricsFile = "build-metrics.json";
  private previousMetrics: BuildMetrics[] = [];

  constructor() {
    this.loadPreviousMetrics();
  }

  async measureBuild(): Promise<BuildMetrics> {
    console.log("📊 Starting build performance measurement...");

    const startTime = Date.now();

    // Run the build
    try {
      execSync("npm run build:prod", { stdio: "inherit" });
    } catch (error) {
      throw new Error(`Build failed: ${error}`);
    }

    const buildTime = Date.now() - startTime;

    // Analyze bundle
    const bundleAnalysis = this.analyzeBundles();

    // Measure runtime performance
    const performanceMetrics = await this.measurePerformance();

    const metrics: BuildMetrics = {
      timestamp: Date.now(),
      buildTime,
      bundleSize: bundleAnalysis.bundleSize,
      chunks: bundleAnalysis.chunks,
      performance: performanceMetrics,
    };

    this.saveMetrics(metrics);
    this.compareWithPrevious(metrics);

    return metrics;
  }

  private analyzeBundles() {
    const distPath = path.join(process.cwd(), "dist");
    const files = fs.readdirSync(distPath);

    let totalSize = 0;
    let mainSize = 0;
    let vendorSize = 0;
    let lazySize = 0;

    const chunks: Array<{
      name: string;
      size: number;
      type: "initial" | "async";
    }> = [];

    files.forEach((file) => {
      if (!file.endsWith(".js")) return;

      const filePath = path.join(distPath, file);
      const size = fs.statSync(filePath).size;

      totalSize += size;

      if (file.includes("main")) {
        mainSize += size;
        chunks.push({ name: file, size, type: "initial" });
      } else if (file.includes("vendor")) {
        vendorSize += size;
        chunks.push({ name: file, size, type: "initial" });
      } else {
        lazySize += size;
        chunks.push({ name: file, size, type: "async" });
      }
    });

    return {
      bundleSize: {
        total: totalSize,
        main: mainSize,
        vendor: vendorSize,
        lazy: lazySize,
      },
      chunks,
    };
  }

  private async measurePerformance() {
    // This would integrate with tools like Lighthouse or Puppeteer
    // For demo, return mock metrics
    return {
      firstContentfulPaint: 1200,
      timeToInteractive: 2800,
      totalBlockingTime: 150,
    };
  }

  private loadPreviousMetrics() {
    try {
      if (fs.existsSync(this.metricsFile)) {
        const data = fs.readFileSync(this.metricsFile, "utf8");
        this.previousMetrics = JSON.parse(data);
      }
    } catch (error) {
      console.warn("Could not load previous metrics:", error);
    }
  }

  private saveMetrics(metrics: BuildMetrics) {
    this.previousMetrics.push(metrics);

    // Keep only last 50 builds
    if (this.previousMetrics.length > 50) {
      this.previousMetrics = this.previousMetrics.slice(-50);
    }

    fs.writeFileSync(
      this.metricsFile,
      JSON.stringify(this.previousMetrics, null, 2)
    );
  }

  private compareWithPrevious(current: BuildMetrics) {
    if (this.previousMetrics.length < 2) return;

    const previous = this.previousMetrics[this.previousMetrics.length - 2];

    console.log("\n📈 Build Performance Comparison:");
    console.log("═".repeat(50));

    // Build time comparison
    const buildTimeDiff = current.buildTime - previous.buildTime;
    const buildTimePercent = (
      (buildTimeDiff / previous.buildTime) *
      100
    ).toFixed(1);

    console.log(
      `⏱️  Build Time: ${current.buildTime}ms (${
        buildTimeDiff > 0 ? "+" : ""
      }${buildTimePercent}%)`
    );

    // Bundle size comparison
    const bundleSizeDiff = current.bundleSize.total - previous.bundleSize.total;
    const bundleSizePercent = (
      (bundleSizeDiff / previous.bundleSize.total) *
      100
    ).toFixed(1);

    console.log(
      `📦 Bundle Size: ${this.formatBytes(current.bundleSize.total)} (${
        bundleSizeDiff > 0 ? "+" : ""
      }${bundleSizePercent}%)`
    );

    // Performance comparison
    const fcpDiff =
      current.performance.firstContentfulPaint -
      previous.performance.firstContentfulPaint;
    console.log(
      `🎨 First Contentful Paint: ${
        current.performance.firstContentfulPaint
      }ms (${fcpDiff > 0 ? "+" : ""}${fcpDiff}ms)`
    );

    const ttiDiff =
      current.performance.timeToInteractive -
      previous.performance.timeToInteractive;
    console.log(
      `⚡ Time to Interactive: ${current.performance.timeToInteractive}ms (${
        ttiDiff > 0 ? "+" : ""
      }${ttiDiff}ms)`
    );

    // Warnings
    if (buildTimeDiff > 10000) {
      console.log("⚠️  WARNING: Build time increased significantly!");
    }

    if (bundleSizeDiff > 100000) {
      console.log("⚠️  WARNING: Bundle size increased significantly!");
    }

    console.log("═".repeat(50));
  }

  private formatBytes(bytes: number): string {
    if (bytes === 0) return "0 Bytes";
    const k = 1024;
    const sizes = ["Bytes", "KB", "MB", "GB"];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
  }
}

// Usage
const monitor = new BuildMonitor();
monitor
  .measureBuild()
  .then((metrics) => {
    console.log("✅ Build monitoring completed");
    process.exit(0);
  })
  .catch((error) => {
    console.error("❌ Build monitoring failed:", error);
    process.exit(1);
  });
```

---

## 🎉 **Summary: Build Optimization Mastery**

### **🚀 What You've Mastered:**

#### **⚡ Build Performance:**

✅ **Webpack Optimization** - Custom configurations for dev/prod builds  
✅ **Bundle Splitting** - Smart code chunking strategies  
✅ **Compression** - Gzip and Brotli for smaller assets  
✅ **Caching** - Filesystem and HTTP caching strategies

#### **🔄 Advanced Strategies:**

✅ **Progressive Loading** - Smart preloading based on user behavior  
✅ **Component Lazy Loading** - On-demand component initialization  
✅ **Build Monitoring** - Performance tracking and regression detection  
✅ **CI/CD Optimization** - Faster pipelines with intelligent caching

#### **📦 Production Optimizations:**

✅ **Tree Shaking** - Eliminate unused code automatically  
✅ **Docker Multi-Stage** - Smaller production images  
✅ **Security** - Subresource integrity and CSP headers  
✅ **Performance Budgets** - Automatic size limit enforcement

### **🎯 Key Performance Gains:**

- **🚀 Build Speed**: 70-80% faster builds with proper caching
- **📦 Bundle Size**: 60-70% smaller with advanced optimization
- **⚡ Load Time**: 3-5x faster initial page loads
- **🔄 CI/CD**: 50% faster deployment pipelines

### **💡 Pro Tips:**

1. **📊 Monitor Everything** - Track metrics to catch regressions early
2. **🎯 Progressive Enhancement** - Load heavy features only when needed
3. **🔄 Cache Strategically** - Balance between freshness and performance
4. **🧪 Test Impact** - Always measure real-world performance impact

**Remember**: Optimization is an iterative process - measure, optimize, measure again! 🌟
