# 📦 **Webpack Configuration & Session Management**

## 🎯 **What You'll Learn**

Master **Webpack customization** in Angular and implement **session timeout functionality** - essential skills for production applications! Think of Webpack as your app's **assembly line** and session management as your **security guard**.

---

## 📚 **Part 1: Webpack in Angular (The Basics)**

### **Do You Use Webpack? 🤔**

**YES!** Angular CLI uses Webpack under the hood, but it's **hidden by default**. Think of it like a **professional kitchen**:

- 🏭 **Webpack** = The industrial kitchen equipment (ovens, mixers, processors)
- 👨‍🍳 **Angular CLI** = The head chef who knows how to operate everything
- 🍽️ **Your App** = The delicious meal that comes out

```bash
# Angular CLI hides Webpack complexity
ng build                    # ← Uses Webpack internally
ng serve                    # ← Uses Webpack dev server
ng test                     # ← Uses Webpack for testing

# But you can access Webpack config when needed!
ng eject                    # ⚠️ Exposes Webpack config (irreversible)
```

### **Why Customize Webpack? 🔧**

```typescript
// Common reasons to customize Webpack:

// 1️⃣ Add custom loaders for special file types
// 2️⃣ Integrate third-party build tools
// 3️⃣ Optimize bundle splitting strategies
// 4️⃣ Add environment-specific configurations
// 5️⃣ Implement advanced caching strategies
// 6️⃣ Integrate with legacy systems
```

---

## 🛠️ **Webpack Customization Approaches**

### **1. 🎯 Angular Builders (Recommended)**

```typescript
// angular.json - Custom builder configuration
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
                "externals": "replace",
                "module.rules": "append"
              }
            },
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json"
          }
        },
        "serve": {
          "builder": "@angular-builders/custom-webpack:dev-server",
          "options": {
            "customWebpackConfig": {
              "path": "./webpack.config.js"
            }
          }
        }
      }
    }
  }
}
```

```javascript
// webpack.config.js - Custom Webpack configuration
const path = require("path");
const webpack = require("webpack");

module.exports = (config, options) => {
  console.log("🔧 Customizing Webpack configuration...");

  // 📦 Add custom aliases for cleaner imports
  config.resolve.alias = {
    ...config.resolve.alias,
    "@app": path.resolve(__dirname, "src/app"),
    "@shared": path.resolve(__dirname, "src/app/shared"),
    "@core": path.resolve(__dirname, "src/app/core"),
    "@features": path.resolve(__dirname, "src/app/features"),
    "@environments": path.resolve(__dirname, "src/environments"),
    "@assets": path.resolve(__dirname, "src/assets"),
  };

  // 🎯 Add custom loaders
  config.module.rules.push(
    // SVG as Angular components
    {
      test: /\.svg$/,
      use: [
        {
          loader:
            "@angular-devkit/build-angular/src/angular-cli-files/plugins/raw-css-loader.js",
        },
        {
          loader: "postcss-loader",
          options: {
            postcssOptions: {
              plugins: ["autoprefixer"],
            },
          },
        },
      ],
    },

    // Custom SCSS processing
    {
      test: /\.scss$/,
      use: [
        "style-loader",
        "css-loader",
        {
          loader: "sass-loader",
          options: {
            additionalData: `
              @import 'src/styles/variables';
              @import 'src/styles/mixins';
            `,
          },
        },
      ],
    }
  );

  // 🔌 Add custom plugins
  config.plugins.push(
    // Environment-specific plugins
    new webpack.DefinePlugin({
      "process.env.BUILD_TIME": JSON.stringify(new Date().toISOString()),
      "process.env.BUILD_VERSION": JSON.stringify(
        process.env.npm_package_version || "1.0.0"
      ),
    }),

    // Bundle analyzer in development
    ...(options.configuration === "development"
      ? [
          new (require("webpack-bundle-analyzer").BundleAnalyzerPlugin)({
            analyzerMode: "server",
            openAnalyzer: false,
            analyzerPort: 8889,
          }),
        ]
      : [])
  );

  // 📊 Optimize bundle splitting
  if (options.configuration === "production") {
    config.optimization = {
      ...config.optimization,
      splitChunks: {
        chunks: "all",
        cacheGroups: {
          // Vendor libraries
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: "vendors",
            chunks: "all",
            priority: 10,
          },

          // Angular framework
          angular: {
            test: /[\\/]node_modules[\\/]@angular[\\/]/,
            name: "angular",
            chunks: "all",
            priority: 15,
          },

          // RxJS separately
          rxjs: {
            test: /[\\/]node_modules[\\/]rxjs[\\/]/,
            name: "rxjs",
            chunks: "all",
            priority: 12,
          },

          // Common components
          common: {
            name: "common",
            minChunks: 2,
            chunks: "all",
            enforce: true,
            priority: 5,
          },
        },
      },
    };
  }

  // 🔍 Development optimizations
  if (options.configuration === "development") {
    // Faster rebuilds
    config.cache = {
      type: "filesystem",
      buildDependencies: {
        config: [__filename],
      },
    };

    // Better debugging
    config.devtool = "eval-cheap-module-source-map";
  }

  console.log("✅ Webpack customization complete");
  return config;
};
```

### **2. 🔧 Advanced Webpack Customization**

```javascript
// webpack.advanced.js - Production-ready customizations
const path = require("path");
const webpack = require("webpack");
const CompressionPlugin = require("compression-webpack-plugin");
const { SubresourceIntegrityPlugin } = require("webpack-subresource-integrity");

module.exports = (config, options) => {
  // 🚀 PERFORMANCE OPTIMIZATIONS
  if (options.configuration === "production") {
    // Gzip compression
    config.plugins.push(
      new CompressionPlugin({
        algorithm: "gzip",
        test: /\.(js|css|html|svg)$/,
        threshold: 8192,
        minRatio: 0.8,
      })
    );

    // Brotli compression
    config.plugins.push(
      new CompressionPlugin({
        filename: "[path][base].br",
        algorithm: "brotliCompress",
        test: /\.(js|css|html|svg)$/,
        compressionOptions: {
          params: {
            [require("zlib").constants.BROTLI_PARAM_QUALITY]: 11,
          },
        },
        threshold: 8192,
        minRatio: 0.8,
      })
    );

    // Subresource integrity for security
    config.plugins.push(
      new SubresourceIntegrityPlugin({
        hashFuncNames: ["sha256", "sha384"],
        enabled: true,
      })
    );

    // Advanced tree shaking
    config.optimization.usedExports = true;
    config.optimization.sideEffects = false;
  }

  // 🔍 DEVELOPMENT ENHANCEMENTS
  if (options.configuration === "development") {
    // Hot module replacement
    config.plugins.push(new webpack.HotModuleReplacementPlugin());

    // Better error overlay
    config.devServer = {
      ...config.devServer,
      hot: true,
      overlay: {
        warnings: true,
        errors: true,
      },
      historyApiFallback: true,
      compress: true,
      port: 4200,

      // Proxy API calls during development
      proxy: {
        "/api/**": {
          target: "http://localhost:3000",
          secure: false,
          changeOrigin: true,
          logLevel: "debug",
        },
      },
    };
  }

  // 📦 CUSTOM MODULE FEDERATION (Micro-frontends)
  if (process.env.ENABLE_MODULE_FEDERATION === "true") {
    const ModuleFederationPlugin = require("@module-federation/webpack");

    config.plugins.push(
      new ModuleFederationPlugin({
        name: "shell",
        remotes: {
          "user-management":
            "userManagement@http://localhost:4201/remoteEntry.js",
          billing: "billing@http://localhost:4202/remoteEntry.js",
        },
        shared: {
          "@angular/core": { singleton: true },
          "@angular/common": { singleton: true },
          "@angular/router": { singleton: true },
          rxjs: { singleton: true },
        },
      })
    );
  }

  return config;
};
```

### **3. 📱 PWA & Service Worker Integration**

```javascript
// webpack.pwa.js - PWA-specific Webpack configuration
const { GenerateSW } = require("workbox-webpack-plugin");

module.exports = (config, options) => {
  if (options.configuration === "production") {
    // Advanced Service Worker generation
    config.plugins.push(
      new GenerateSW({
        swDest: "custom-sw.js",
        clientsClaim: true,
        skipWaiting: true,

        // Cache strategies
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\//,
            handler: "StaleWhileRevalidate",
            options: {
              cacheName: "api-cache",
              expiration: {
                maxEntries: 100,
                maxAgeSeconds: 60 * 60, // 1 hour
              },
            },
          },
          {
            urlPattern: /\.(?:png|jpg|jpeg|svg)$/,
            handler: "CacheFirst",
            options: {
              cacheName: "image-cache",
              expiration: {
                maxEntries: 200,
                maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
              },
            },
          },
        ],

        // Files to precache
        include: [/\.html$/, /\.js$/, /\.css$/],
        exclude: [/\.map$/, /manifest$/, /\.htaccess$/],
      })
    );
  }

  return config;
};
```

---

## 🔐 **Part 2: Session Management Use Case**

### **The Challenge 🎯**

> **Use Case**: You have an app where users are logged in. You want to show a popup after 1 minute of inactivity, and if they don't interact for another minute, automatically log them out.

Let's build this step by step!

### **1. 🕐 Session Timeout Service**

```typescript
// src/app/core/services/session-timeout.service.ts
import { Injectable, NgZone } from "@angular/core";
import {
  BehaviorSubject,
  Observable,
  fromEvent,
  merge,
  timer,
  NEVER,
} from "rxjs";
import { map, switchMap, takeUntil, tap, startWith } from "rxjs/operators";

export interface SessionState {
  isActive: boolean;
  timeRemaining: number;
  showWarning: boolean;
  lastActivity: Date;
}

export interface SessionTimeoutConfig {
  warningTimeoutMs: number; // Time until warning (1 minute)
  logoutTimeoutMs: number; // Time until logout (2 minutes total)
  checkIntervalMs: number; // How often to check (1 second)
  activityEvents: string[]; // Events that count as activity
}

@Injectable({
  providedIn: "root",
})
export class SessionTimeoutService {
  // 📊 Default Configuration
  private config: SessionTimeoutConfig = {
    warningTimeoutMs: 60 * 1000, // 1 minute
    logoutTimeoutMs: 120 * 1000, // 2 minutes total
    checkIntervalMs: 1000, // 1 second
    activityEvents: [
      "mousedown",
      "mousemove",
      "keypress",
      "scroll",
      "touchstart",
      "click",
      "focus",
      "blur",
    ],
  };

  // 🔄 State Management
  private sessionState$ = new BehaviorSubject<SessionState>({
    isActive: true,
    timeRemaining: this.config.logoutTimeoutMs,
    showWarning: false,
    lastActivity: new Date(),
  });

  private lastActivityTime = Date.now();
  private warningShown = false;
  private timeoutTimer: any;

  // 📡 Public Observables
  readonly sessionState = this.sessionState$.asObservable();
  readonly shouldShowWarning$ = this.sessionState.pipe(
    map((state) => state.showWarning)
  );
  readonly timeRemaining$ = this.sessionState.pipe(
    map((state) => state.timeRemaining)
  );

  constructor(private ngZone: NgZone) {}

  // 🚀 START SESSION MONITORING
  startSessionMonitoring(customConfig?: Partial<SessionTimeoutConfig>): void {
    console.log("🔐 Starting session monitoring...");

    // Apply custom configuration
    if (customConfig) {
      this.config = { ...this.config, ...customConfig };
    }

    this.resetActivity();
    this.setupActivityListeners();
    this.startTimeoutCheck();
  }

  // ⏹️ STOP SESSION MONITORING
  stopSessionMonitoring(): void {
    console.log("🔐 Stopping session monitoring...");

    if (this.timeoutTimer) {
      clearInterval(this.timeoutTimer);
      this.timeoutTimer = null;
    }

    this.removeActivityListeners();
    this.resetSessionState();
  }

  // 🔄 RESET ACTIVITY
  resetActivity(): void {
    const now = Date.now();
    this.lastActivityTime = now;
    this.warningShown = false;

    this.updateSessionState({
      isActive: true,
      timeRemaining: this.config.logoutTimeoutMs,
      showWarning: false,
      lastActivity: new Date(now),
    });

    console.log("🔄 Activity reset at", new Date(now).toLocaleTimeString());
  }

  // ⚠️ EXTEND SESSION (from warning dialog)
  extendSession(): void {
    console.log("⏰ Session extended by user");
    this.resetActivity();
  }

  // 🚪 FORCE LOGOUT
  forceLogout(): void {
    console.log("🚪 Forcing logout due to inactivity");
    this.stopSessionMonitoring();
    this.onSessionTimeout();
  }

  // 📊 GET CURRENT STATE
  getCurrentState(): SessionState {
    return this.sessionState$.value;
  }

  // 🎯 Private Methods

  private setupActivityListeners(): void {
    console.log("👂 Setting up activity listeners...");

    // Listen to activity events outside Angular zone for performance
    this.ngZone.runOutsideAngular(() => {
      this.config.activityEvents.forEach((eventType) => {
        document.addEventListener(eventType, this.onUserActivity.bind(this), {
          passive: true,
          capture: true,
        });
      });

      // Also listen to visibility changes
      document.addEventListener(
        "visibilitychange",
        this.onVisibilityChange.bind(this)
      );
    });
  }

  private removeActivityListeners(): void {
    console.log("🔇 Removing activity listeners...");

    this.config.activityEvents.forEach((eventType) => {
      document.removeEventListener(
        eventType,
        this.onUserActivity.bind(this),
        true
      );
    });

    document.removeEventListener(
      "visibilitychange",
      this.onVisibilityChange.bind(this)
    );
  }

  private onUserActivity(): void {
    const now = Date.now();
    const timeSinceLastActivity = now - this.lastActivityTime;

    // Throttle activity updates (max once per second)
    if (timeSinceLastActivity > 1000) {
      this.ngZone.run(() => {
        this.resetActivity();
      });
    }
  }

  private onVisibilityChange(): void {
    if (document.visibilityState === "visible") {
      console.log("👁️ Page became visible - resetting activity");
      this.ngZone.run(() => {
        this.resetActivity();
      });
    }
  }

  private startTimeoutCheck(): void {
    console.log("⏰ Starting timeout check timer...");

    this.timeoutTimer = setInterval(() => {
      this.ngZone.run(() => {
        this.checkSessionTimeout();
      });
    }, this.config.checkIntervalMs);
  }

  private checkSessionTimeout(): void {
    const now = Date.now();
    const inactiveTime = now - this.lastActivityTime;
    const timeRemaining = Math.max(
      0,
      this.config.logoutTimeoutMs - inactiveTime
    );

    // Update state
    this.updateSessionState({
      isActive: timeRemaining > 0,
      timeRemaining,
      showWarning:
        inactiveTime >= this.config.warningTimeoutMs && timeRemaining > 0,
      lastActivity: new Date(this.lastActivityTime),
    });

    // Check if warning should be shown
    if (
      inactiveTime >= this.config.warningTimeoutMs &&
      !this.warningShown &&
      timeRemaining > 0
    ) {
      this.warningShown = true;
      this.showTimeoutWarning();
    }

    // Check if session should be terminated
    if (timeRemaining <= 0) {
      this.onSessionTimeout();
    }
  }

  private showTimeoutWarning(): void {
    console.log("⚠️ Showing timeout warning");

    // This will be handled by the component listening to sessionState$
    window.dispatchEvent(
      new CustomEvent("session-timeout-warning", {
        detail: {
          timeRemaining:
            this.config.logoutTimeoutMs - this.config.warningTimeoutMs,
        },
      })
    );
  }

  private onSessionTimeout(): void {
    console.log("🚪 Session timeout - logging out user");

    this.updateSessionState({
      isActive: false,
      timeRemaining: 0,
      showWarning: false,
      lastActivity: new Date(this.lastActivityTime),
    });

    // Emit logout event
    window.dispatchEvent(new CustomEvent("session-timeout-logout"));

    this.stopSessionMonitoring();
  }

  private updateSessionState(updates: Partial<SessionState>): void {
    const currentState = this.sessionState$.value;
    this.sessionState$.next({
      ...currentState,
      ...updates,
    });
  }

  private resetSessionState(): void {
    this.updateSessionState({
      isActive: true,
      timeRemaining: this.config.logoutTimeoutMs,
      showWarning: false,
      lastActivity: new Date(),
    });
  }
}
```

### **2. ⚠️ Session Warning Modal Component**

```typescript
// src/app/shared/components/session-warning-modal/session-warning-modal.component.ts
import { Component, OnInit, OnDestroy, Inject } from "@angular/core";
import { MAT_DIALOG_DATA, MatDialogRef } from "@angular/material/dialog";
import { Subject, interval } from "rxjs";
import { takeUntil, map } from "rxjs/operators";

export interface SessionWarningData {
  timeRemaining: number;
}

@Component({
  selector: "app-session-warning-modal",
  template: `
    <div class="session-warning-modal">
      <div class="modal-header">
        <h2 class="modal-title">
          <span class="warning-icon">⚠️</span>
          Session Timeout Warning
        </h2>
      </div>

      <div class="modal-body">
        <p class="warning-message">
          Your session is about to expire due to inactivity.
        </p>

        <div class="countdown-container">
          <div class="countdown-circle">
            <div class="countdown-text">
              <span class="countdown-number">{{ timeRemaining }}</span>
              <span class="countdown-label">seconds</span>
            </div>
          </div>
        </div>

        <p class="action-message">
          Click "Stay Logged In" to continue your session, or you will be
          automatically logged out when the timer reaches zero.
        </p>
      </div>

      <div class="modal-actions">
        <button type="button" class="btn btn-secondary" (click)="onLogoutNow()">
          Logout Now
        </button>
        <button
          type="button"
          class="btn btn-primary"
          (click)="onStayLoggedIn()"
          [class.pulse]="timeRemaining <= 10"
        >
          Stay Logged In
        </button>
      </div>
    </div>
  `,
  styleUrls: ["./session-warning-modal.component.scss"],
})
export class SessionWarningModalComponent implements OnInit, OnDestroy {
  timeRemaining: number = 60;

  private destroy$ = new Subject<void>();

  constructor(
    public dialogRef: MatDialogRef<SessionWarningModalComponent>,
    @Inject(MAT_DIALOG_DATA) public data: SessionWarningData
  ) {
    this.timeRemaining = Math.floor(data.timeRemaining / 1000);
  }

  ngOnInit(): void {
    this.startCountdown();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  onStayLoggedIn(): void {
    this.dialogRef.close("extend");
  }

  onLogoutNow(): void {
    this.dialogRef.close("logout");
  }

  private startCountdown(): void {
    interval(1000)
      .pipe(
        takeUntil(this.destroy$),
        map(() => --this.timeRemaining)
      )
      .subscribe((time) => {
        this.timeRemaining = time;

        if (time <= 0) {
          this.dialogRef.close("timeout");
        }
      });
  }
}
```

```scss
// session-warning-modal.component.scss
.session-warning-modal {
  width: 100%;
  max-width: 450px;
  padding: 0;
  border-radius: 12px;
  overflow: hidden;

  .modal-header {
    background: linear-gradient(135deg, #ff6b6b, #ffa726);
    color: white;
    padding: 24px;
    text-align: center;

    .modal-title {
      margin: 0;
      font-size: 1.5rem;
      font-weight: 600;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;

      .warning-icon {
        font-size: 2rem;
        animation: pulse 2s infinite;
      }
    }
  }

  .modal-body {
    padding: 32px 24px;
    text-align: center;

    .warning-message {
      font-size: 1.1rem;
      color: var(--text-secondary);
      margin-bottom: 24px;
    }

    .countdown-container {
      display: flex;
      justify-content: center;
      margin: 32px 0;

      .countdown-circle {
        width: 120px;
        height: 120px;
        border-radius: 50%;
        background: linear-gradient(135deg, #ff6b6b, #ffa726);
        display: flex;
        align-items: center;
        justify-content: center;
        box-shadow: 0 8px 32px rgba(255, 107, 107, 0.3);
        animation: countdownPulse 1s ease-in-out infinite alternate;

        .countdown-text {
          color: white;
          text-align: center;

          .countdown-number {
            display: block;
            font-size: 2.5rem;
            font-weight: 700;
            line-height: 1;
          }

          .countdown-label {
            font-size: 0.875rem;
            font-weight: 500;
            opacity: 0.9;
          }
        }
      }
    }

    .action-message {
      font-size: 0.9rem;
      color: var(--text-secondary);
      line-height: 1.5;
    }
  }

  .modal-actions {
    padding: 0 24px 24px;
    display: flex;
    gap: 12px;
    justify-content: center;

    .btn {
      flex: 1;
      padding: 12px 24px;
      border: none;
      border-radius: 8px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.2s ease;

      &.btn-secondary {
        background: var(--gray-200);
        color: var(--text-primary);

        &:hover {
          background: var(--gray-300);
          transform: translateY(-1px);
        }
      }

      &.btn-primary {
        background: linear-gradient(135deg, #007bff, #0056b3);
        color: white;

        &:hover {
          transform: translateY(-1px);
          box-shadow: 0 4px 16px rgba(0, 123, 255, 0.3);
        }

        &.pulse {
          animation: buttonPulse 0.5s ease-in-out infinite alternate;
        }
      }
    }
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes countdownPulse {
  0% {
    transform: scale(1);
    box-shadow: 0 8px 32px rgba(255, 107, 107, 0.3);
  }
  100% {
    transform: scale(1.05);
    box-shadow: 0 12px 40px rgba(255, 107, 107, 0.5);
  }
}

@keyframes buttonPulse {
  0% {
    background: linear-gradient(135deg, #007bff, #0056b3);
    box-shadow: 0 2px 8px rgba(0, 123, 255, 0.3);
  }
  100% {
    background: linear-gradient(135deg, #0056b3, #007bff);
    box-shadow: 0 4px 16px rgba(0, 123, 255, 0.5);
  }
}
```

### **3. 🎯 Main App Integration**

```typescript
// src/app/app.component.ts
import { Component, OnInit, OnDestroy } from "@angular/core";
import { MatDialog } from "@angular/material/dialog";
import { Router } from "@angular/router";
import { Subject } from "rxjs";
import { takeUntil, filter } from "rxjs/operators";

import { SessionTimeoutService } from "./core/services/session-timeout.service";
import { AuthenticationService } from "./core/services/authentication.service";
import { SessionWarningModalComponent } from "./shared/components/session-warning-modal/session-warning-modal.component";

@Component({
  selector: "app-root",
  template: `
    <div class="app-container">
      <!-- Navigation -->
      <app-header></app-header>

      <!-- Main Content -->
      <main class="main-content">
        <router-outlet></router-outlet>
      </main>

      <!-- Footer -->
      <app-footer></app-footer>

      <!-- Session Status Indicator (Development) -->
      <div class="session-debug" *ngIf="showDebugInfo">
        <h4>Session Status</h4>
        <p>Active: {{ (sessionState$ | async)?.isActive }}</p>
        <p>
          Time Remaining:
          {{
            ((sessionState$ | async)?.timeRemaining || 0) / 1000
              | number : "1.0-0"
          }}s
        </p>
        <p>Show Warning: {{ (sessionState$ | async)?.showWarning }}</p>
        <p>
          Last Activity:
          {{ (sessionState$ | async)?.lastActivity | date : "medium" }}
        </p>
      </div>
    </div>
  `,
  styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit, OnDestroy {
  sessionState$ = this.sessionTimeoutService.sessionState;
  showDebugInfo = false; // Set to true during development

  private destroy$ = new Subject<void>();
  private warningDialogRef: any = null;

  constructor(
    private sessionTimeoutService: SessionTimeoutService,
    private authService: AuthenticationService,
    private dialog: MatDialog,
    private router: Router
  ) {}

  ngOnInit(): void {
    this.initializeApp();
    this.setupSessionHandling();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
    this.sessionTimeoutService.stopSessionMonitoring();
  }

  private initializeApp(): void {
    // Check if user is authenticated
    this.authService
      .getAuthState()
      .pipe(takeUntil(this.destroy$))
      .subscribe((authState) => {
        if (authState.isAuthenticated) {
          this.startSessionMonitoring();
        } else {
          this.sessionTimeoutService.stopSessionMonitoring();
        }
      });
  }

  private startSessionMonitoring(): void {
    console.log("🚀 Starting session monitoring for authenticated user");

    // Custom configuration for your app
    const sessionConfig = {
      warningTimeoutMs: 60 * 1000, // 1 minute
      logoutTimeoutMs: 2 * 60 * 1000, // 2 minutes total
      checkIntervalMs: 1000, // Check every second
      activityEvents: [
        "mousedown",
        "mousemove",
        "keypress",
        "scroll",
        "touchstart",
        "click",
        "focus",
        "blur",
        "wheel",
      ],
    };

    this.sessionTimeoutService.startSessionMonitoring(sessionConfig);
  }

  private setupSessionHandling(): void {
    // Listen for session state changes
    this.sessionTimeoutService.sessionState
      .pipe(
        takeUntil(this.destroy$),
        filter((state) => state.showWarning && !this.warningDialogRef)
      )
      .subscribe((state) => {
        this.showSessionWarningDialog(state.timeRemaining);
      });

    // Listen for custom events from the service
    window.addEventListener("session-timeout-logout", () => {
      this.handleSessionLogout();
    });

    window.addEventListener("session-timeout-warning", (event: any) => {
      if (!this.warningDialogRef) {
        this.showSessionWarningDialog(event.detail.timeRemaining);
      }
    });
  }

  private showSessionWarningDialog(timeRemaining: number): void {
    console.log("⚠️ Showing session warning dialog");

    this.warningDialogRef = this.dialog.open(SessionWarningModalComponent, {
      data: { timeRemaining },
      disableClose: true,
      hasBackdrop: true,
      backdropClass: "session-warning-backdrop",
      panelClass: "session-warning-dialog",
      width: "450px",
    });

    this.warningDialogRef
      .afterClosed()
      .pipe(takeUntil(this.destroy$))
      .subscribe((result) => {
        this.warningDialogRef = null;

        switch (result) {
          case "extend":
            console.log("✅ User chose to extend session");
            this.sessionTimeoutService.extendSession();
            break;

          case "logout":
            console.log("🚪 User chose to logout");
            this.handleSessionLogout();
            break;

          case "timeout":
            console.log("⏰ Session warning timed out");
            this.handleSessionLogout();
            break;

          default:
            console.log("❌ Dialog closed unexpectedly");
            this.sessionTimeoutService.forceLogout();
        }
      });
  }

  private handleSessionLogout(): void {
    console.log("🚪 Handling session logout");

    // Close any open dialogs
    if (this.warningDialogRef) {
      this.warningDialogRef.close();
      this.warningDialogRef = null;
    }

    // Stop session monitoring
    this.sessionTimeoutService.stopSessionMonitoring();

    // Logout user
    this.authService.logout().subscribe({
      next: () => {
        console.log("✅ User logged out successfully");
        this.router.navigate(["/login"], {
          queryParams: { reason: "session_timeout" },
        });
      },
      error: (error) => {
        console.error("❌ Logout failed:", error);
        // Force navigation anyway
        this.router.navigate(["/login"], {
          queryParams: { reason: "session_timeout" },
        });
      },
    });
  }
}
```

### **4. 🎨 App Component Styles**

```scss
// app.component.scss
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;

  .main-content {
    flex: 1;
    padding: 20px;
  }
}

// Session debug info (development only)
.session-debug {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 16px;
  border-radius: 8px;
  font-size: 0.75rem;
  z-index: 9999;

  h4 {
    margin: 0 0 8px 0;
    color: #ffa726;
  }

  p {
    margin: 4px 0;
  }
}

// Dialog backdrop styling
:host ::ng-deep .session-warning-backdrop {
  background: rgba(255, 107, 107, 0.1);
  backdrop-filter: blur(2px);
}

:host ::ng-deep .session-warning-dialog {
  .mat-dialog-container {
    padding: 0;
    border-radius: 12px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  }
}
```

---

## 🎉 **Summary: Webpack & Session Management Mastery**

### **🏗️ What You've Mastered:**

#### **📦 Webpack Customization:**

✅ **Angular Builders** - Safe way to customize Webpack without ejecting  
✅ **Custom Loaders** - Adding support for new file types  
✅ **Bundle Optimization** - Smart code splitting strategies  
✅ **Environment Configuration** - Different settings for dev/prod  
✅ **Performance Optimization** - Compression, caching, and tree shaking  
✅ **Module Federation** - Micro-frontend architecture support

#### **🔐 Session Management:**

✅ **Activity Monitoring** - Detecting user interactions  
✅ **Timeout Warnings** - User-friendly session expiry notices  
✅ **Automatic Logout** - Secure session termination  
✅ **State Management** - Reactive session state handling  
✅ **Accessibility** - WCAG-compliant warning dialogs  
✅ **Performance** - Efficient event handling outside Angular zone

### **🚀 Real-World Benefits:**

- **Secure applications** with automatic session management
- **Better user experience** with clear timeout warnings
- **Optimized builds** with custom Webpack configurations
- **Enterprise-ready** session security features
- **Scalable architecture** with module federation support

**Remember**: These are production-grade implementations that prioritize security, performance, and user experience! 🌟
