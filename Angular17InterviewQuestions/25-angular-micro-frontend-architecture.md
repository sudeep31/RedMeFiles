# 🏗️ Angular Micro-Frontend Architecture: Complete Implementation Guide

## 🎯 **Question Overview**

_"How do you design and implement a scalable micro-frontend architecture in Angular using Module Federation, shared libraries, and independent deployments?"_

## 🔍 **Understanding Micro-Frontend Architecture**

### **What Are Micro-Frontends? 🤔**

Micro-frontends are an architectural pattern where a **single application** is broken down into **smaller, independent frontend applications**. Think of it like having **multiple mini-websites** that work together to create **one cohesive user experience**.

**Real-World Analogy:**
Imagine a shopping mall where each store (micro-frontend) is independently owned and operated, but they all contribute to the overall mall experience. Each store can:

- Update their displays independently
- Hire their own staff (developers)
- Choose their own inventory (technologies)
- Operate on their own schedule (deployments)

### **Why Use Micro-Frontends? 🎯**

**Traditional Monolithic Frontend:**

```
┌─────────────────────────────────┐
│        SINGLE ANGULAR APP       │
│  ┌─────────────────────────────┐ │
│  │     Products Component      │ │
│  │     Users Component         │ │
│  │     Orders Component        │ │
│  │     Analytics Component     │ │
│  │     Notifications Component │ │
│  └─────────────────────────────┘ │
│         One Deployment          │
│         One Team                │
│         One Codebase            │
└─────────────────────────────────┘
```

**Micro-Frontend Architecture:**

```
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ Shell   │  │Products │  │ Users   │  │ Orders  │
│ App     │  │Micro-FE │  │Micro-FE │  │Micro-FE │
│         │  │         │  │         │  │         │
│ Team A  │  │ Team B  │  │ Team C  │  │ Team D  │
│Deploy 1 │  │Deploy 2 │  │Deploy 3 │  │Deploy 4 │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
```

### **Key Benefits 🚀**

**1. Team Independence:**

- Each team owns their micro-frontend completely
- Teams can choose their own technology stack
- No coordination needed for deployments
- Faster development cycles

**2. Scalability:**

- Scale individual features independently
- Add new features without affecting existing ones
- Handle different load patterns per feature

**3. Resilience:**

- If one micro-frontend fails, others continue working
- Gradual rollouts and A/B testing
- Easier to isolate and fix bugs

**4. Technology Flexibility:**

- Mix Angular, React, Vue in the same application
- Upgrade frameworks incrementally
- Try new technologies on small features first

### **How Module Federation Works 🔧**

**Module Federation** is like having **smart connectors** that allow different applications to **share code at runtime**. Here's how it works:

```typescript
// Think of it like this:
// App A says: "I have a ProductService, anyone can use it"
// App B says: "I need a ProductService, let me borrow it from App A"

// App A (Product Micro-Frontend):
exposes: {
  './ProductService': './src/product.service.ts'
}

// App B (Shell Application):
remotes: {
  'products': 'productApp@http://localhost:4201/remoteEntry.js'
}

// App B can now use:
import { ProductService } from 'products/ProductService';
```

### **Core Concepts Explained 📚**

**1. Shell/Host Application:**

- The **main container** that loads other micro-frontends
- Handles **routing** and **navigation**
- Manages **shared state** and **authentication**
- Think of it as the "mall management" that coordinates everything

**2. Remote Applications:**

- **Independent Angular applications** that are loaded by the shell
- Each handles a **specific business domain** (products, users, orders)
- Can be developed and deployed **separately**
- Think of them as individual "stores" in the mall

**3. Shared Dependencies:**

- **Common libraries** used by multiple applications (Angular, RxJS)
- **Shared UI components** and **design systems**
- **Utility functions** and **business logic**
- Think of them as "shared utilities" like electricity and internet in the mall

## 🏗️ **Project Structure & Setup Guide**

### **Step 1: Setting Up the Workspace 📁**

Let's create a **complete micro-frontend workspace**. Here's the **recommended folder structure**:

```
micro-frontend-workspace/
├── apps/                          # All applications
│   ├── shell/                     # Main host application
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── shared/
│   │   │   │   │   ├── services/  # Shell services
│   │   │   │   │   ├── guards/    # Route guards
│   │   │   │   │   └── state/     # Global state
│   │   │   │   ├── pages/         # Shell pages
│   │   │   │   └── app.routes.ts  # Main routing
│   │   │   └── main.ts
│   │   ├── webpack.config.js      # Module Federation config
│   │   └── package.json
│   │
│   ├── product-catalog/           # Product micro-frontend
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── features/
│   │   │   │   │   └── product/   # Product domain
│   │   │   │   ├── shared/        # Local shared code
│   │   │   │   └── app.module.ts
│   │   │   └── main.ts
│   │   ├── webpack.config.js      # Module Federation config
│   │   └── package.json
│   │
│   ├── user-management/           # User micro-frontend
│   ├── order-processing/          # Order micro-frontend
│   └── analytics-dashboard/       # Analytics micro-frontend
│
├── libs/                          # Shared libraries
│   ├── shared-ui/                 # Common UI components
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── components/    # Reusable components
│   │   │   │   ├── directives/    # Custom directives
│   │   │   │   ├── pipes/         # Common pipes
│   │   │   │   └── styles/        # Shared CSS/SCSS
│   │   │   └── public-api.ts      # Public exports
│   │   └── package.json
│   │
│   ├── shared-utils/              # Common utilities
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── constants/     # App constants
│   │   │   │   ├── interfaces/    # TypeScript interfaces
│   │   │   │   ├── validators/    # Form validators
│   │   │   │   └── helpers/       # Utility functions
│   │   │   └── public-api.ts
│   │   └── package.json
│   │
│   └── shared-state/              # Shared state management
│       ├── src/
│       │   ├── lib/
│       │   │   ├── auth/          # Authentication state
│       │   │   ├── user/          # User state
│       │   │   └── theme/         # Theme state
│       │   └── public-api.ts
│       └── package.json
│
├── tools/                         # Build and deployment tools
│   ├── scripts/                   # Build scripts
│   └── webpack/                   # Shared webpack configs
│
├── nx.json                        # Nx workspace config
├── angular.json                   # Angular workspace config
└── package.json                   # Root package.json
```

### **Step 2: Understanding the Architecture Flow 🔄**

Here's **how everything connects**:

```
┌─────────────────────────────────────────────────────────────┐
│                    BROWSER                                   │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐│
│  │                 SHELL APP (Port 4200)                   ││
│  │                                                         ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     ││
│  │  │  Routing    │  │ Event Bus   │  │ Auth State  │     ││
│  │  │  Service    │  │ Service     │  │ Management  │     ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘     ││
│  │                                                         ││
│  │  Dynamic Loading of Micro-Frontends:                   ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │        http://localhost:4201/remoteEntry.js         │││
│  │  │        http://localhost:4202/remoteEntry.js         │││
│  │  │        http://localhost:4203/remoteEntry.js         │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Products   │  │   Users     │  │   Orders    │
│ Micro-FE    │  │ Micro-FE    │  │ Micro-FE    │
│             │  │             │  │             │
│ Port 4201   │  │ Port 4202   │  │ Port 4203   │
│             │  │             │  │             │
│Independent  │  │Independent  │  │Independent  │
│Deployment   │  │Deployment   │  │Deployment   │
└─────────────┘  └─────────────┘  └─────────────┘
```

### **Step 3: Creating the Workspace 🛠️**

**1. Initialize the Workspace:**

```bash
# Create new Angular workspace
npx @angular/cli new micro-frontend-workspace --routing --style=scss --skip-git

cd micro-frontend-workspace

# Install Module Federation
npm install @angular-architects/module-federation --save-dev

# Install Nx for better workspace management (optional but recommended)
npm install nx --save-dev
```

**2. Generate Shell Application:**

```bash
# Generate the shell (host) application
ng generate application shell --routing --style=scss

# Add Module Federation to shell
ng add @angular-architects/module-federation --project shell --port 4200 --type host
```

**3. Generate Micro-Frontend Applications:**

```bash
# Generate product catalog micro-frontend
ng generate application product-catalog --routing --style=scss
ng add @angular-architects/module-federation --project product-catalog --port 4201 --type remote

# Generate user management micro-frontend
ng generate application user-management --routing --style=scss
ng add @angular-architects/module-federation --project user-management --port 4202 --type remote

# Generate order processing micro-frontend
ng generate application order-processing --routing --style=scss
ng add @angular-architects/module-federation --project order-processing --port 4203 --type remote
```

**4. Generate Shared Libraries:**

```bash
# Generate shared UI library
ng generate library shared-ui
ng generate library shared-utils
ng generate library shared-state
```

## 🏗️ **Module Federation Configuration Explained**

### **Understanding Webpack Configuration 📦**

**Module Federation** works through **special webpack configurations**. Let's understand each part:

**What Each Part Does:**

```typescript
// apps/shell/webpack.config.js
const ModuleFederationPlugin = require("@module-federation/webpack");

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      // 1. NAME: Unique identifier for this application
      name: "shell", // This is like giving your app a name tag

      // 2. REMOTES: Other micro-frontends this app wants to use
      remotes: {
        // "nickname": "realname@url"
        "product-catalog":
          "productCatalog@http://localhost:4201/remoteEntry.js",
        "user-management":
          "userManagement@http://localhost:4202/remoteEntry.js",
      },

      // 3. SHARED: Libraries that should be shared between apps
      shared: {
        "@angular/core": {
          singleton: true, // Only one copy in memory
          strictVersion: true, // Must match exact version
        },
        // More shared libraries...
      },
    }),
  ],
};
```

**Breaking Down the Concepts:**

**🎯 Name:**

- Think of this as your app's **unique ID card**
- Other apps will reference this name when they want to use your code
- Must be **unique** across all micro-frontends

**🔗 Remotes:**

- These are **other micro-frontends** you want to load
- Format: `"localAlias": "remoteName@remoteURL"`
- `localAlias` = what you call it in your code
- `remoteName` = the actual name from the remote's webpack config
- `remoteURL` = where the remote app is hosted

**📚 Shared:**

- **Libraries that all apps need** (like Angular, RxJS)
- `singleton: true` = only **one copy** loaded in the browser
- `strictVersion: true` = all apps must use **same version**
- This prevents version conflicts and reduces bundle size

```typescript
// apps/shell/webpack.config.js - Main Shell Configuration
const ModuleFederationPlugin = require("@module-federation/webpack");
const path = require("path");

module.exports = {
  mode: "development",

  plugins: [
    new ModuleFederationPlugin({
      name: "shell",

      // Remote module definitions
      remotes: {
        "product-catalog":
          "productCatalog@http://localhost:4201/remoteEntry.js",
        "user-management":
          "userManagement@http://localhost:4202/remoteEntry.js",
        "order-processing":
          "orderProcessing@http://localhost:4203/remoteEntry.js",
        "analytics-dashboard":
          "analyticsDashboard@http://localhost:4204/remoteEntry.js",
        "notification-center":
          "notificationCenter@http://localhost:4205/remoteEntry.js",
      },

      // Shared dependencies with version control
      shared: {
        "@angular/core": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
          eager: true,
        },
        "@angular/common": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
          eager: true,
        },
        "@angular/router": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        "@angular/platform-browser": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        rxjs: {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^7.8.0",
          eager: true,
        },
        "@ngrx/store": {
          singleton: true,
          strictVersion: false,
          requiredVersion: "^17.0.0",
        },

        // Shared UI library
        "@company/shared-ui": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^1.0.0",
          eager: false,
        },

        // Shared utilities
        "@company/shared-utils": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^1.0.0",
        },

        // Third-party libraries
        lodash: { singleton: false }, // Allow multiple versions
        moment: { singleton: true, requiredVersion: "^2.29.0" },
        "chart.js": { singleton: true, requiredVersion: "^4.0.0" },
      },

      // Expose shell utilities to micro-frontends
      exposes: {
        "./ShellService": "./src/app/shared/services/shell.service.ts",
        "./EventBus": "./src/app/shared/services/event-bus.service.ts",
        "./ThemeService": "./src/app/shared/services/theme.service.ts",
        "./AuthenticationState": "./src/app/shared/state/auth.state.ts",
      },
    }),
  ],

  // Development server configuration
  devServer: {
    port: 4200,
    historyApiFallback: true,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers":
        "X-Requested-With, content-type, Authorization",
    },
  },

  // Optimization for micro-frontends
  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        // Vendor chunk optimization
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          priority: 10,
          enforce: true,
          maxSize: 500000, // 500KB max
        },

        // Shared components chunk
        shared: {
          test: /[\\/]src[\\/]app[\\/]shared[\\/]/,
          name: "shared",
          priority: 20,
          minChunks: 2,
          maxSize: 200000, // 200KB max
        },
      },
    },

    runtimeChunk: {
      name: "runtime",
    },
  },
};
```

### **Understanding the Shell Configuration 📚**

Let's break down what each part of the shell configuration does:

**1. Remote Definitions Explained:**

```typescript
remotes: {
  "product-catalog": "productCatalog@http://localhost:4201/remoteEntry.js"
}
```

**What this means:**

- `"product-catalog"` = **nickname** you use in your code
- `"productCatalog"` = **actual name** from the remote app's webpack config
- `"http://localhost:4201/remoteEntry.js"` = **where to find** the remote app

**Think of it like a phone book:**

- You want to call "John" (nickname)
- But his real name is "John Smith" (actual name)
- And his number is "555-1234" (URL)

**2. Shared Dependencies Strategy:**

```typescript
shared: {
  "@angular/core": {
    singleton: true,        // Only one copy allowed
    strictVersion: true,    // Must match exactly
    requiredVersion: "^17.0.0", // Which version
    eager: true,           // Load immediately
  }
}
```

**Why this matters:**

- **singleton: true** = Prevents multiple Angular instances (would break the app)
- **strictVersion: true** = Ensures compatibility between micro-frontends
- **eager: true** = Loads immediately when app starts (for critical libraries)

**3. Exposing Shell Services:**

```typescript
exposes: {
  "./EventBus": "./src/app/shared/services/event-bus.service.ts",
  "./AuthenticationState": "./src/app/shared/state/auth.state.ts"
}
```

**What this enables:**

- Other micro-frontends can **import and use** these services
- Creates a **shared communication layer**
- Enables **centralized authentication** and **cross-app messaging**

### **Shell Routing Configuration 🛣️**

The shell manages **routing to different micro-frontends**. Here's how it works:

**Traditional vs Micro-Frontend Routing:**

```typescript
// Traditional Routing (everything in one app):
const routes = [
  {
    path: "products",
    loadChildren: () => import("./products/products.module"),
  },
  { path: "users", loadChildren: () => import("./users/users.module") },
];

// Micro-Frontend Routing (loading from remote apps):
const routes = [
  {
    path: "products",
    loadChildren: () =>
      loadRemoteModule({
        remoteEntry: "http://localhost:4201/remoteEntry.js",
        exposedModule: "./ProductModule",
      }),
  },
];
```

**Key Differences:**

- **Traditional:** Loads modules from **same application**
- **Micro-Frontend:** Loads modules from **different applications** running on different ports
- **Benefits:** Each team can deploy their feature **independently**

````

```typescript
// apps/shell/src/app/app.routes.ts - Shell Routing Configuration
import { Routes } from "@angular/router";
import { loadRemoteModule } from "@angular-architects/module-federation";

import { AuthGuard } from "./guards/auth.guard";
import { FeatureToggleGuard } from "./guards/feature-toggle.guard";
import { MicroFrontendGuard } from "./guards/micro-frontend.guard";

export const routes: Routes = [
  // Shell routes
  {
    path: "",
    loadComponent: () =>
      import("./pages/dashboard/dashboard.component").then(
        (m) => m.DashboardComponent
      ),
    title: "Dashboard",
  },

  // Product Catalog Micro-Frontend
  {
    path: "products",
    canMatch: [MicroFrontendGuard],
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4201/remoteEntry.js",
        exposedModule: "./ProductModule",
      })
        .then((m) => m.ProductModule)
        .catch(() => {
          // Fallback to local implementation
          console.warn("Product micro-frontend unavailable, loading fallback");
          return import("./fallbacks/products/products.routes").then(
            (m) => m.PRODUCT_FALLBACK_ROUTES
          );
        }),
    data: {
      microFrontend: "product-catalog",
      version: "^2.1.0",
      fallback: true,
    },
  },

  // User Management Micro-Frontend
  {
    path: "users",
    canMatch: [AuthGuard, MicroFrontendGuard],
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4202/remoteEntry.js",
        exposedModule: "./UserModule",
      }).then((m) => m.UserModule),
    data: {
      microFrontend: "user-management",
      version: "^1.5.0",
      requiredRoles: ["admin", "user-manager"],
    },
  },

  // Order Processing Micro-Frontend
  {
    path: "orders",
    canMatch: [AuthGuard, MicroFrontendGuard],
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4203/remoteEntry.js",
        exposedModule: "./OrderModule",
      }).then((m) => m.OrderModule),
    data: {
      microFrontend: "order-processing",
      version: "^3.0.0",
      preloadStrategy: "onDemand",
    },
  },

  // Analytics Dashboard Micro-Frontend
  {
    path: "analytics",
    canMatch: [AuthGuard, FeatureToggleGuard, MicroFrontendGuard],
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4204/remoteEntry.js",
        exposedModule: "./AnalyticsModule",
      }).then((m) => m.AnalyticsModule),
    data: {
      microFrontend: "analytics-dashboard",
      version: "^1.8.0",
      featureFlag: "analytics-enabled",
      requiredRoles: ["analyst", "admin"],
    },
  },

  // Notification Center Micro-Frontend
  {
    path: "notifications",
    canMatch: [AuthGuard],
    loadComponent: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4205/remoteEntry.js",
        exposedModule: "./NotificationComponent",
      }).then((m) => m.NotificationComponent),
    data: {
      microFrontend: "notification-center",
      version: "^0.9.0",
      standalone: true,
    },
  },

  // Error handling
  {
    path: "micro-frontend-error",
    loadComponent: () =>
      import(
        "./pages/micro-frontend-error/micro-frontend-error.component"
      ).then((m) => m.MicroFrontendErrorComponent),
  },

  // Wildcard
  { path: "**", redirectTo: "" },
];
````

### **2. 🎮 Shell Application Core Services**

```typescript
// apps/shell/src/app/shared/services/micro-frontend-manager.service.ts
import { Injectable, inject, computed, signal } from "@angular/core";
import { Router } from "@angular/router";
import { HttpClient } from "@angular/common/http";
import { Observable, BehaviorSubject, interval, of } from "rxjs";
import { map, catchError, switchMap, retry } from "rxjs/operators";

export interface MicroFrontendManifest {
  name: string;
  version: string;
  remoteEntry: string;
  exposedModules: string[];
  dependencies: { [key: string]: string };
  status: "available" | "unavailable" | "loading" | "error";
  healthEndpoint?: string;
  fallbackRoute?: string;
  requiredFeatureFlags?: string[];
}

export interface MicroFrontendHealth {
  name: string;
  status: "healthy" | "unhealthy" | "degraded";
  version: string;
  lastCheck: Date;
  responseTime: number;
  uptime: number;
}

@Injectable({
  providedIn: "root",
})
export class MicroFrontendManagerService {
  private http = inject(HttpClient);
  private router = inject(Router);

  // Signal-based reactive state
  private readonly _manifests = signal<Map<string, MicroFrontendManifest>>(
    new Map()
  );
  private readonly _healthStatuses = signal<Map<string, MicroFrontendHealth>>(
    new Map()
  );
  private readonly _loadingStates = signal<Map<string, boolean>>(new Map());

  // Public readonly computed signals
  readonly manifests = computed(() => Array.from(this._manifests().values()));
  readonly healthStatuses = computed(() =>
    Array.from(this._healthStatuses().values())
  );
  readonly availableMicroFrontends = computed(() =>
    this.manifests().filter((mf) => mf.status === "available")
  );
  readonly unhealthyMicroFrontends = computed(() =>
    this.healthStatuses().filter((health) => health.status !== "healthy")
  );

  private readonly manifestRegistry$ = new BehaviorSubject<
    MicroFrontendManifest[]
  >([]);
  private readonly healthCheckInterval = 30000; // 30 seconds

  constructor() {
    this.initializeMicroFrontends();
    this.startHealthMonitoring();
  }

  // Initialize micro-frontends from registry
  private async initializeMicroFrontends(): Promise<void> {
    try {
      const manifests = await this.loadManifestRegistry();

      const manifestMap = new Map<string, MicroFrontendManifest>();
      manifests.forEach((manifest) => {
        manifestMap.set(manifest.name, manifest);
      });

      this._manifests.set(manifestMap);

      // Perform initial health checks
      await this.performHealthChecks();
    } catch (error) {
      console.error("Failed to initialize micro-frontends:", error);
    }
  }

  // Load manifest registry from server
  private async loadManifestRegistry(): Promise<MicroFrontendManifest[]> {
    return (
      this.http
        .get<MicroFrontendManifest[]>("/api/micro-frontends/manifests")
        .pipe(
          retry(3),
          catchError((error) => {
            console.error("Failed to load manifest registry:", error);
            return of(this.getDefaultManifests());
          })
        )
        .toPromise() || []
    );
  }

  // Default manifests for fallback
  private getDefaultManifests(): MicroFrontendManifest[] {
    return [
      {
        name: "product-catalog",
        version: "2.1.0",
        remoteEntry: "http://localhost:4201/remoteEntry.js",
        exposedModules: ["./ProductModule", "./ProductService"],
        dependencies: { "@angular/core": "^17.0.0", rxjs: "^7.8.0" },
        status: "available",
        healthEndpoint: "http://localhost:4201/health",
        fallbackRoute: "/fallbacks/products",
      },
      {
        name: "user-management",
        version: "1.5.0",
        remoteEntry: "http://localhost:4202/remoteEntry.js",
        exposedModules: ["./UserModule", "./UserService"],
        dependencies: { "@angular/core": "^17.0.0", "@ngrx/store": "^17.0.0" },
        status: "available",
        healthEndpoint: "http://localhost:4202/health",
      },
      {
        name: "order-processing",
        version: "3.0.0",
        remoteEntry: "http://localhost:4203/remoteEntry.js",
        exposedModules: ["./OrderModule"],
        dependencies: { "@angular/core": "^17.0.0", "chart.js": "^4.0.0" },
        status: "available",
        healthEndpoint: "http://localhost:4203/health",
      },
    ];
  }

  // Health monitoring
  private startHealthMonitoring(): void {
    interval(this.healthCheckInterval).subscribe(() => {
      this.performHealthChecks();
    });
  }

  private async performHealthChecks(): Promise<void> {
    const manifests = this._manifests();
    const healthChecks = Array.from(manifests.values())
      .filter((manifest) => manifest.healthEndpoint)
      .map((manifest) => this.checkMicroFrontendHealth(manifest));

    const results = await Promise.allSettled(healthChecks);

    const healthMap = new Map(this._healthStatuses());
    results.forEach((result, index) => {
      if (result.status === "fulfilled") {
        healthMap.set(result.value.name, result.value);
      }
    });

    this._healthStatuses.set(healthMap);
  }

  private async checkMicroFrontendHealth(
    manifest: MicroFrontendManifest
  ): Promise<MicroFrontendHealth> {
    const startTime = performance.now();

    try {
      const response = await this.http
        .get(manifest.healthEndpoint!, {
          timeout: 5000, // 5 second timeout
        })
        .toPromise();

      const responseTime = performance.now() - startTime;

      return {
        name: manifest.name,
        status: "healthy",
        version: manifest.version,
        lastCheck: new Date(),
        responseTime,
        uptime: (response as any)?.uptime || 0,
      };
    } catch (error) {
      const responseTime = performance.now() - startTime;

      return {
        name: manifest.name,
        status: responseTime > 10000 ? "degraded" : "unhealthy",
        version: manifest.version,
        lastCheck: new Date(),
        responseTime,
        uptime: 0,
      };
    }
  }

  // Public API methods
  async loadMicroFrontend(name: string): Promise<boolean> {
    const manifest = this._manifests().get(name);
    if (!manifest) {
      console.error(`Micro-frontend '${name}' not found in registry`);
      return false;
    }

    if (manifest.status === "loading") {
      return false; // Already loading
    }

    try {
      // Update loading state
      this.setLoadingState(name, true);
      this.updateManifestStatus(name, "loading");

      // Dynamic import of the remote module
      const module = await import(
        /* webpackIgnore: true */ manifest.remoteEntry
      );

      // Update success state
      this.updateManifestStatus(name, "available");
      this.setLoadingState(name, false);

      return true;
    } catch (error) {
      console.error(`Failed to load micro-frontend '${name}':`, error);

      // Update error state
      this.updateManifestStatus(name, "error");
      this.setLoadingState(name, false);

      // Navigate to fallback if available
      if (manifest.fallbackRoute) {
        this.router.navigate([manifest.fallbackRoute]);
      }

      return false;
    }
  }

  getMicroFrontendManifest(name: string): MicroFrontendManifest | undefined {
    return this._manifests().get(name);
  }

  getMicroFrontendHealth(name: string): MicroFrontendHealth | undefined {
    return this._healthStatuses().get(name);
  }

  isMicroFrontendAvailable(name: string): boolean {
    const manifest = this._manifests().get(name);
    return manifest?.status === "available";
  }

  isMicroFrontendLoading(name: string): boolean {
    return this._loadingStates().get(name) || false;
  }

  async refreshManifest(name: string): Promise<void> {
    try {
      const updatedManifest = await this.http
        .get<MicroFrontendManifest>(`/api/micro-frontends/manifests/${name}`)
        .toPromise();

      if (updatedManifest) {
        const manifests = new Map(this._manifests());
        manifests.set(name, updatedManifest);
        this._manifests.set(manifests);
      }
    } catch (error) {
      console.error(`Failed to refresh manifest for '${name}':`, error);
    }
  }

  // Graceful degradation
  async enableGracefulDegradation(name: string): Promise<void> {
    const manifest = this._manifests().get(name);
    if (!manifest || !manifest.fallbackRoute) {
      return;
    }

    console.log(`Enabling graceful degradation for '${name}'`);
    this.router.navigate([manifest.fallbackRoute]);
  }

  // Version compatibility checking
  checkVersionCompatibility(name: string, requiredVersion: string): boolean {
    const manifest = this._manifests().get(name);
    if (!manifest) return false;

    // Simple version comparison (in production, use semver library)
    return manifest.version >= requiredVersion;
  }

  // Utility methods
  private updateManifestStatus(
    name: string,
    status: MicroFrontendManifest["status"]
  ): void {
    const manifests = new Map(this._manifests());
    const manifest = manifests.get(name);

    if (manifest) {
      manifests.set(name, { ...manifest, status });
      this._manifests.set(manifests);
    }
  }

  private setLoadingState(name: string, loading: boolean): void {
    const loadingStates = new Map(this._loadingStates());
    loadingStates.set(name, loading);
    this._loadingStates.set(loadingStates);
  }

  // Debug and monitoring
  getSystemOverview(): {
    totalMicroFrontends: number;
    availableCount: number;
    errorCount: number;
    loadingCount: number;
    healthyCount: number;
    degradedCount: number;
  } {
    const manifests = this.manifests();
    const healthStatuses = this.healthStatuses();

    return {
      totalMicroFrontends: manifests.length,
      availableCount: manifests.filter((m) => m.status === "available").length,
      errorCount: manifests.filter((m) => m.status === "error").length,
      loadingCount: manifests.filter((m) => m.status === "loading").length,
      healthyCount: healthStatuses.filter((h) => h.status === "healthy").length,
      degradedCount: healthStatuses.filter((h) => h.status === "degraded")
        .length,
    };
  }

  exportHealthReport(): string {
    const overview = this.getSystemOverview();
    const healthStatuses = this.healthStatuses();

    return JSON.stringify(
      {
        timestamp: new Date().toISOString(),
        overview,
        details: healthStatuses,
      },
      null,
      2
    );
  }
}
```

### **3. 🔄 Inter-Micro-Frontend Communication**

```typescript
// apps/shell/src/app/shared/services/event-bus.service.ts
import { Injectable, inject } from "@angular/core";
import { BehaviorSubject, Subject, Observable, filter, map } from "rxjs";
import { share, takeUntil } from "rxjs/operators";

export interface EventMessage {
  id: string;
  type: string;
  source: string;
  target?: string; // Specific target or broadcast if undefined
  payload: any;
  timestamp: number;
  priority: "low" | "medium" | "high" | "critical";
  persistent?: boolean; // Whether to store for late subscribers
}

export interface EventSubscription {
  id: string;
  type: string;
  source: string;
  callback: (event: EventMessage) => void;
  once?: boolean;
}

@Injectable({
  providedIn: "root",
})
export class EventBusService {
  private eventStream$ = new Subject<EventMessage>();
  private persistentEvents$ = new BehaviorSubject<Map<string, EventMessage>>(
    new Map()
  );
  private subscriptions = new Map<string, EventSubscription>();
  private eventHistory: EventMessage[] = [];

  private readonly maxHistorySize = 1000;
  private readonly maxPersistentEvents = 100;

  constructor() {
    // Log all events for debugging
    this.eventStream$.subscribe((event) => {
      this.addToHistory(event);
      console.log(
        `[EventBus] ${event.source} -> ${event.target || "broadcast"}: ${
          event.type
        }`,
        event.payload
      );
    });
  }

  // Publish events
  publish<T = any>(
    type: string,
    payload: T,
    options: {
      source: string;
      target?: string;
      priority?: EventMessage["priority"];
      persistent?: boolean;
    }
  ): void {
    const event: EventMessage = {
      id: this.generateEventId(),
      type,
      source: options.source,
      target: options.target,
      payload,
      timestamp: Date.now(),
      priority: options.priority || "medium",
      persistent: options.persistent || false,
    };

    // Store persistent events
    if (event.persistent) {
      this.storePersistentEvent(event);
    }

    // Emit the event
    this.eventStream$.next(event);
  }

  // Subscribe to events
  subscribe<T = any>(
    type: string,
    callback: (payload: T, event: EventMessage) => void,
    options: {
      source?: string;
      target?: string;
      once?: boolean;
      includePersistent?: boolean;
    } = {}
  ): () => void {
    const subscriptionId = this.generateSubscriptionId();

    // Create subscription
    const subscription: EventSubscription = {
      id: subscriptionId,
      type,
      source: options.source || "*",
      callback: (event) => callback(event.payload, event),
      once: options.once,
    };

    this.subscriptions.set(subscriptionId, subscription);

    // Subscribe to future events
    const eventSubscription = this.eventStream$
      .pipe(
        filter((event) => this.matchesSubscription(event, subscription)),
        takeUntil(new Subject()) // Will be completed by unsubscribe
      )
      .subscribe((event) => {
        subscription.callback(event);

        // Remove one-time subscriptions
        if (subscription.once) {
          this.unsubscribe(subscriptionId);
        }
      });

    // Send persistent events if requested
    if (options.includePersistent) {
      this.sendPersistentEvents(subscription);
    }

    // Return unsubscribe function
    return () => {
      eventSubscription.unsubscribe();
      this.unsubscribe(subscriptionId);
    };
  }

  // Subscribe to multiple event types
  subscribeToMany<T = any>(
    types: string[],
    callback: (payload: T, event: EventMessage) => void,
    options: {
      source?: string;
      target?: string;
      once?: boolean;
    } = {}
  ): () => void {
    const unsubscribeFunctions = types.map((type) =>
      this.subscribe(type, callback, options)
    );

    return () => {
      unsubscribeFunctions.forEach((unsubscribe) => unsubscribe());
    };
  }

  // Observable streams for reactive programming
  on<T = any>(
    type: string,
    source?: string
  ): Observable<{ payload: T; event: EventMessage }> {
    return this.eventStream$.pipe(
      filter((event) => {
        return event.type === type && (!source || event.source === source);
      }),
      map((event) => ({ payload: event.payload as T, event })),
      share()
    );
  }

  // Wait for a specific event (Promise-based)
  waitFor<T = any>(
    type: string,
    options: {
      source?: string;
      timeout?: number;
      condition?: (payload: T) => boolean;
    } = {}
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      const timeout = options.timeout || 30000; // 30 second default timeout

      let timeoutId: NodeJS.Timeout;

      const unsubscribe = this.subscribe<T>(
        type,
        (payload, event) => {
          // Check condition if provided
          if (options.condition && !options.condition(payload)) {
            return;
          }

          clearTimeout(timeoutId);
          unsubscribe();
          resolve(payload);
        },
        {
          source: options.source,
          once: true,
        }
      );

      // Set timeout
      timeoutId = setTimeout(() => {
        unsubscribe();
        reject(new Error(`Timeout waiting for event: ${type}`));
      }, timeout);
    });
  }

  // Request-Response pattern
  async request<TRequest, TResponse>(
    requestType: string,
    payload: TRequest,
    options: {
      source: string;
      target: string;
      timeout?: number;
    }
  ): Promise<TResponse> {
    const correlationId = this.generateEventId();
    const responseType = `${requestType}_response`;

    // Set up response listener
    const responsePromise = this.waitFor<TResponse>(responseType, {
      source: options.target,
      timeout: options.timeout,
      condition: (responsePayload: any) => {
        return responsePayload.correlationId === correlationId;
      },
    });

    // Send request
    this.publish(
      requestType,
      {
        ...payload,
        correlationId,
        responseType,
      },
      {
        source: options.source,
        target: options.target,
        priority: "high",
      }
    );

    return responsePromise;
  }

  // Batch operations
  publishBatch(
    events: Array<{
      type: string;
      payload: any;
      source: string;
      target?: string;
      priority?: EventMessage["priority"];
    }>
  ): void {
    events.forEach((eventData) => {
      this.publish(eventData.type, eventData.payload, eventData);
    });
  }

  // Event filtering and querying
  getEventHistory(
    filter: {
      type?: string;
      source?: string;
      target?: string;
      since?: number;
      limit?: number;
    } = {}
  ): EventMessage[] {
    let filtered = this.eventHistory;

    if (filter.type) {
      filtered = filtered.filter((e) => e.type === filter.type);
    }

    if (filter.source) {
      filtered = filtered.filter((e) => e.source === filter.source);
    }

    if (filter.target) {
      filtered = filtered.filter((e) => e.target === filter.target);
    }

    if (filter.since) {
      filtered = filtered.filter((e) => e.timestamp >= filter.since);
    }

    if (filter.limit) {
      filtered = filtered.slice(-filter.limit);
    }

    return filtered;
  }

  // Debug and monitoring
  getActiveSubscriptions(): EventSubscription[] {
    return Array.from(this.subscriptions.values());
  }

  getSubscriptionsByType(type: string): EventSubscription[] {
    return Array.from(this.subscriptions.values()).filter(
      (sub) => sub.type === type
    );
  }

  clearEventHistory(): void {
    this.eventHistory = [];
  }

  clearPersistentEvents(): void {
    this.persistentEvents$.next(new Map());
  }

  // Private helper methods
  private matchesSubscription(
    event: EventMessage,
    subscription: EventSubscription
  ): boolean {
    // Check event type
    if (subscription.type !== "*" && subscription.type !== event.type) {
      return false;
    }

    // Check source
    if (subscription.source !== "*" && subscription.source !== event.source) {
      return false;
    }

    return true;
  }

  private storePersistentEvent(event: EventMessage): void {
    const persistentEvents = new Map(this.persistentEvents$.value);

    // Use type + source as key for uniqueness
    const key = `${event.type}:${event.source}`;
    persistentEvents.set(key, event);

    // Limit persistent events to prevent memory leaks
    if (persistentEvents.size > this.maxPersistentEvents) {
      const firstKey = persistentEvents.keys().next().value;
      persistentEvents.delete(firstKey);
    }

    this.persistentEvents$.next(persistentEvents);
  }

  private sendPersistentEvents(subscription: EventSubscription): void {
    const persistentEvents = this.persistentEvents$.value;

    persistentEvents.forEach((event) => {
      if (this.matchesSubscription(event, subscription)) {
        // Send persistent event asynchronously
        setTimeout(() => subscription.callback(event), 0);
      }
    });
  }

  private unsubscribe(subscriptionId: string): void {
    this.subscriptions.delete(subscriptionId);
  }

  private addToHistory(event: EventMessage): void {
    this.eventHistory.push(event);

    // Limit history size to prevent memory leaks
    if (this.eventHistory.length > this.maxHistorySize) {
      this.eventHistory = this.eventHistory.slice(-this.maxHistorySize / 2);
    }
  }

  private generateEventId(): string {
    return `event_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private generateSubscriptionId(): string {
    return `sub_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### **4. 🛡️ Micro-Frontend Guard**

```typescript
// apps/shell/src/app/guards/micro-frontend.guard.ts
import { Injectable, inject } from "@angular/core";
import { CanMatch, Route, UrlSegment, Router } from "@angular/router";
import { Observable, of } from "rxjs";
import { map, catchError, timeout } from "rxjs/operators";

import { MicroFrontendManagerService } from "../shared/services/micro-frontend-manager.service";
import { FeatureToggleService } from "../shared/services/feature-toggle.service";
import { UserService } from "../shared/services/user.service";

@Injectable({
  providedIn: "root",
})
export class MicroFrontendGuard implements CanMatch {
  private microFrontendManager = inject(MicroFrontendManagerService);
  private featureToggleService = inject(FeatureToggleService);
  private userService = inject(UserService);
  private router = inject(Router);

  canMatch(route: Route, segments: UrlSegment[]): Observable<boolean> {
    const microFrontendName = route.data?.["microFrontend"];

    if (!microFrontendName) {
      console.warn("Route missing micro-frontend name in data");
      return of(false);
    }

    return this.checkMicroFrontendAccess(microFrontendName, route).pipe(
      timeout(10000), // 10 second timeout
      catchError((error) => {
        console.error(
          `Micro-frontend guard error for ${microFrontendName}:`,
          error
        );
        this.handleGuardError(microFrontendName, route);
        return of(false);
      })
    );
  }

  private checkMicroFrontendAccess(
    microFrontendName: string,
    route: Route
  ): Observable<boolean> {
    // Check if micro-frontend is available
    if (
      !this.microFrontendManager.isMicroFrontendAvailable(microFrontendName)
    ) {
      console.warn(`Micro-frontend '${microFrontendName}' is not available`);
      this.handleUnavailableMicroFrontend(microFrontendName, route);
      return of(false);
    }

    // Check version compatibility
    const requiredVersion = route.data?.["version"];
    if (requiredVersion) {
      const isCompatible = this.microFrontendManager.checkVersionCompatibility(
        microFrontendName,
        requiredVersion
      );

      if (!isCompatible) {
        console.warn(`Version incompatibility for '${microFrontendName}'`);
        return of(false);
      }
    }

    // Check feature flags
    const featureFlag = route.data?.["featureFlag"];
    if (featureFlag && !this.featureToggleService.isEnabled(featureFlag)) {
      console.warn(
        `Feature flag '${featureFlag}' is disabled for ${microFrontendName}`
      );
      return of(false);
    }

    // Check user roles
    const requiredRoles = route.data?.["requiredRoles"] as string[];
    if (requiredRoles && requiredRoles.length > 0) {
      return this.userService.getCurrentUser().pipe(
        map((user) => {
          if (!user) return false;

          const hasRequiredRole = requiredRoles.some((role) =>
            user.roles.includes(role)
          );

          if (!hasRequiredRole) {
            console.warn(
              `User lacks required roles for ${microFrontendName}: ${requiredRoles.join(
                ", "
              )}`
            );
            this.router.navigate(["/unauthorized"]);
          }

          return hasRequiredRole;
        })
      );
    }

    // All checks passed
    return of(true);
  }

  private handleUnavailableMicroFrontend(
    microFrontendName: string,
    route: Route
  ): void {
    const fallbackRoute = route.data?.["fallback"];

    if (fallbackRoute) {
      console.log(
        `Redirecting to fallback route for ${microFrontendName}: ${fallbackRoute}`
      );
      this.router.navigate([fallbackRoute]);
    } else {
      console.log(
        `No fallback available for ${microFrontendName}, showing error page`
      );
      this.router.navigate(["/micro-frontend-error"], {
        queryParams: { microFrontend: microFrontendName },
      });
    }
  }

  private handleGuardError(microFrontendName: string, route: Route): void {
    console.error(`Guard error for micro-frontend: ${microFrontendName}`);

    // Try graceful degradation
    this.microFrontendManager.enableGracefulDegradation(microFrontendName);
  }
}
```

## 🚀 **Remote Application Setup**

### **What are Remote Applications? 🤔**

**Remote applications** are independent Angular apps that can be loaded by other apps. Think of them like **LEGO blocks** - each one is self-contained but can connect to build something bigger!

**Key Characteristics:**

- 🏠 **Independent**: Can run on their own
- 🔄 **Reusable**: Multiple shell apps can use them
- 📦 **Deployable**: Can be deployed separately
- 🔗 **Connectable**: Can share code with other apps

**Real-World Example:**

- **Netflix** = Shell app (main website)
- **Recommendations Engine** = Remote app (suggests movies)
- **Payment System** = Remote app (handles payments)
- **User Profiles** = Remote app (manages user data)

Each team can work on their remote app **independently** and deploy **whenever they're ready**!

### **1. 📦 Product Catalog Micro-Frontend - Step by Step**

#### **Setting Up the Webpack Configuration 🛠️**

```typescript
// apps/product-catalog/webpack.config.js - Remote Configuration
const ModuleFederationPlugin = require("@module-federation/webpack");

module.exports = {
  mode: "development",

  plugins: [
    new ModuleFederationPlugin({
      // 🏷️ UNIQUE IDENTIFIER: What other apps call this app
      name: "productCatalog", // Must match what shell expects!

      // 📁 ENTRY POINT: The file other apps download first
      filename: "remoteEntry.js", // Standard naming convention

      // 📤 WHAT WE EXPORT: Features other apps can use
      exposes: {
        // Format: "./PublicName": "./actual/file/path"
        "./ProductModule": "./src/app/features/product/product.module.ts",
        "./ProductService":
          "./src/app/features/product/services/product.service.ts",
        "./ProductComponent":
          "./src/app/features/product/components/product-list/product-list.component.ts",
      },

      // 📥 WHAT WE IMPORT: Other micro-frontends we want to use
      remotes: {
        // Can import services/components from the shell app
        shell: "shell@http://localhost:4200/remoteEntry.js",
      },

      // 📚 SHARED LIBRARIES: What we share with other apps
      shared: {
        "@angular/core": {
          singleton: true, // Only one copy in browser memory
          strictVersion: true, // Must match exact version
          requiredVersion: "^17.0.0", // Which version we expect
        },
        "@angular/common": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        "@angular/router": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        rxjs: {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^7.8.0",
        },

        // 🔄 IMPORT SHELL SERVICES: Use services from shell
        "shell/EventBus": { singleton: true },
        "shell/ShellService": { singleton: true },
        "shell/ThemeService": { singleton: true },
      },
    }),
  ],

  // 🌐 DEVELOPMENT SERVER: Where this micro-frontend runs
  devServer: {
    port: 4201, // Unique port (different from shell's 4200)
    historyApiFallback: true, // Handle Angular routing properly
    headers: {
      "Access-Control-Allow-Origin": "*", // Let shell access this app
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers":
        "X-Requested-With, content-type, Authorization",
    },
  },

  // 🎯 OPTIMIZATION: Make loading faster
  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
  },
};
```

#### **Understanding Remote Configuration 📚**

**🎯 The `name` Property:**

```typescript
name: "productCatalog";
```

- This is your app's **unique identifier** in the micro-frontend world
- Must match **exactly** what the shell app expects in its `remotes` config
- Like a **phone number** - other apps use this to "call" your app

**📤 The `exposes` Section:**

```typescript
exposes: {
  "./ProductModule": "./src/app/features/product/product.module.ts",
  "./ProductService": "./src/app/features/product/services/product.service.ts"
}
```

**What this does:**

- **Left side** = **Public API** name (what other apps import)
- **Right side** = **Actual file** in your project
- It's like creating a **menu** - you list what others can "order"

**Example Usage:**

```typescript
// In the shell app, you can now do:
const ProductModule = await import("productCatalog/ProductModule");
const ProductService = await import("productCatalog/ProductService");
```

**🔄 The `remotes` Section:**

```typescript
remotes: {
  shell: "shell@http://localhost:4200/remoteEntry.js";
}
```

**Why import from shell?**

- Get **shared services** (authentication, theming, navigation)
- Use **common components** (headers, footers, modals)
- Access **global state** (user info, app settings)

#### **Creating the Product Module 🛠️**

````

```typescript
// apps/product-catalog/src/app/features/product/product.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { RouterModule } from "@angular/router";

// Import local components
import { ProductListComponent } from "./components/product-list/product-list.component";
import { ProductDetailComponent } from "./components/product-detail/product-detail.component";
import { ProductService } from "./services/product.service";

@NgModule({
  declarations: [
    ProductListComponent,
    ProductDetailComponent
  ],
  imports: [
    CommonModule,

    // 🛣️ INTERNAL ROUTING: Routes within this micro-frontend
    RouterModule.forChild([
      {
        path: "",                    // /products (handled by shell)
        component: ProductListComponent,
      },
      {
        path: ":id",                 // /products/123 (product detail)
        component: ProductDetailComponent,
      },
    ]),
  ],
  providers: [ProductService],       // Services specific to products

  // 📤 EXPORT: What other micro-frontends can use
  exports: [ProductListComponent, ProductDetailComponent],
})
export class ProductModule {
  constructor() {
    console.log("🚀 Product Module loaded from micro-frontend");
  }
}
```

#### **Understanding Module Structure 📚**

**🔍 Key Parts Explained:**

**1. Declarations:**
- Lists all **components, directives, pipes** this module owns
- Think of it as the module's "**inventory**"

**2. Imports:**
- **CommonModule**: Basic Angular directives (*ngIf, *ngFor)
- **RouterModule.forChild()**: Sets up **internal routing**

**3. Providers:**
- **Services** that this module provides
- These are **injectable** throughout the micro-frontend

**4. Exports:**
- What **other modules can use**
- Like making components "**public**" for sharing

#### **Creating the Product Service 🛠️**
````

```typescript
// apps/product-catalog/src/app/features/product/services/product.service.ts
import { Injectable, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable, BehaviorSubject } from "rxjs";
import { map, tap } from "rxjs/operators";

// 🔗 SHELL INTEGRATION: Import shared services from shell
declare const loadRemoteModule: any;

// 📋 PRODUCT INTERFACE: What a product looks like
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  rating: number;
  inStock: boolean;
}

@Injectable({
  providedIn: "root", // 🌍 GLOBAL SERVICE: Available throughout this micro-frontend
})
export class ProductService {
  private http = inject(HttpClient);

  // 📊 STATE MANAGEMENT: Local state for products
  private products$ = new BehaviorSubject<Product[]>([]);
  private eventBusService: any; // Will hold shell's EventBus service

  constructor() {
    this.initializeShellServices();
  }

  // 🔄 SHELL INTEGRATION: Connect to shell services
  private async initializeShellServices(): Promise<void> {
    try {
      // 📡 IMPORT SHELL SERVICES: Get shared services from shell
      const eventBusModule = await import("shell/EventBus");
      this.eventBusService = eventBusModule.EventBusService;

      // 🎧 SETUP COMMUNICATION: Listen for events from other micro-frontends
      this.setupEventListeners();

      console.log("✅ Successfully connected to shell services");
    } catch (error) {
      console.warn(
        "⚠️ Shell services unavailable, running in standalone mode:",
        error
      );
    }
  }

  // 🎧 EVENT LISTENERS: How micro-frontends talk to each other
  private setupEventListeners(): void {
    if (!this.eventBusService) return;

    // 🛒 CART EVENTS: Listen when items are added to cart
    this.eventBusService.subscribe(
      "cart.item.added",
      (payload: { productId: string; quantity: number }) => {
        console.log(`🛒 Product ${payload.productId} added to cart`);
        this.updateProductStock(payload.productId, payload.quantity);
      },
      { source: "shopping-cart" } // Which micro-frontend sent this event
    );

    // 📦 INVENTORY EVENTS: Listen for stock updates
    this.eventBusService.subscribe(
      "inventory.updated",
      (payload: { productId: string; stock: number }) => {
        console.log(`📦 Inventory updated for product ${payload.productId}`);
        this.updateProductInList(payload.productId, {
          inStock: payload.stock > 0,
        });
      },
      { source: "inventory-management" }
    );
  }

  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>("/api/products").pipe(
      tap((products) => {
        this.products$.next(products);

        // Notify shell about products loaded
        this.publishEvent("products.loaded", { count: products.length });
      })
    );
  }

  getProduct(id: string): Observable<Product> {
    return this.http.get<Product>(`/api/products/${id}`).pipe(
      tap((product) => {
        // Notify shell about product view
        this.publishEvent("product.viewed", {
          productId: id,
          name: product.name,
        });
      })
    );
  }

  searchProducts(query: string): Observable<Product[]> {
    return this.http.get<Product[]>(`/api/products/search?q=${query}`).pipe(
      tap((results) => {
        this.publishEvent("products.searched", {
          query,
          resultCount: results.length,
        });
      })
    );
  }

  addToWishlist(productId: string): Observable<void> {
    return this.http.post<void>(`/api/wishlist/${productId}`, {}).pipe(
      tap(() => {
        this.publishEvent("wishlist.item.added", { productId });
      })
    );
  }

  private updateProductStock(productId: string, quantity: number): void {
    const currentProducts = this.products$.value;
    const updatedProducts = currentProducts.map((product) => {
      if (product.id === productId) {
        // Simple stock reduction (in real app, this would come from backend)
        return { ...product, inStock: product.inStock };
      }
      return product;
    });

    this.products$.next(updatedProducts);
  }

  private updateProductInList(
    productId: string,
    updates: Partial<Product>
  ): void {
    const currentProducts = this.products$.value;
    const updatedProducts = currentProducts.map((product) => {
      if (product.id === productId) {
        return { ...product, ...updates };
      }
      return product;
    });

    this.products$.next(updatedProducts);
  }

  // 📤 PUBLISHING EVENTS: How this micro-frontend talks to others
  private publishEvent(type: string, payload: any): void {
    if (this.eventBusService) {
      this.eventBusService.publish(type, payload, {
        source: "product-catalog", // Who is sending this event
        priority: "medium", // How important is this event
        timestamp: new Date().toISOString(),
      });
    }
  }

  // 🔍 PUBLIC METHODS: What other parts of the app can use

  getProducts(): Observable<Product[]> {
    return this.products$.asObservable();
  }

  getProductById(id: string): Observable<Product | undefined> {
    return this.products$.pipe(
      map((products) => products.find((p) => p.id === id))
    );
  }

  // 🛒 Add to cart and notify other micro-frontends
  addToCart(productId: string, quantity: number): void {
    this.publishEvent("product.added-to-cart", {
      productId,
      quantity,
      source: "product-catalog",
    });

    console.log(`📣 Published: Product ${productId} added to cart`);
  }
}
```

#### **Understanding Inter-Micro-Frontend Communication 🗣️**

**🎯 The Event Bus Pattern:**

Think of the EventBus like a **radio station**:

- **Publishers** = **Radio DJ** (sends messages)
- **Subscribers** = **Radio listeners** (receive messages)
- **Events** = **Songs being played** (the actual messages)

**Example Communication Flow:**

```
1. User clicks "Add to Cart" in Product Catalog
2. Product Catalog publishes "product.added-to-cart" event
3. Shopping Cart micro-frontend receives the event
4. Shopping Cart updates its state
5. Header micro-frontend gets notified and updates cart count
```

**🔄 Event Types in This Example:**

1. **`cart.item.added`** - Shopping cart tells others an item was added
2. **`inventory.updated`** - Inventory system reports stock changes
3. **`product.added-to-cart`** - Product catalog reports user action

---

## 🚀 **Getting Started: Your First Micro-Frontend**

### **Step 1: Create the Shell Application 📱**

```bash
# Create new Angular workspace
ng new my-micro-frontend-app --routing --style=scss
cd my-micro-frontend-app

# Install Module Federation
npm install @angular-architects/module-federation

# Setup Module Federation for shell
ng add @angular-architects/module-federation --project my-micro-frontend-app --type host
```

### **Step 2: Create Your First Remote App 🏪**

```bash
# Create remote application
ng generate application product-catalog
cd product-catalog

# Setup Module Federation for remote
ng add @angular-architects/module-federation --project product-catalog --type remote --port 4201
```

### **Step 3: Configure the Connection 🔗**

**Shell webpack.config.js:**

```javascript
remotes: {
  "productCatalog": "productCatalog@http://localhost:4201/remoteEntry.js"
}
```

**Remote webpack.config.js:**

```javascript
name: "productCatalog",
exposes: {
  "./ProductModule": "./src/app/product/product.module.ts"
}
```

### **Step 4: Load Remote in Shell 🎯**

```typescript
// In shell routing
{
  path: 'products',
  loadChildren: () =>
    import('productCatalog/ProductModule').then(m => m.ProductModule)
}
```

### **Step 5: Start Everything 🏁**

```bash
# Terminal 1: Start shell
ng serve

# Terminal 2: Start remote
ng serve product-catalog --port 4201
```

**🎉 Congratulations!** You now have:

- A **shell app** running on `localhost:4200`
- A **remote app** running on `localhost:4201`
- **Module Federation** connecting them together

---

## 📚 **Quick Reference Guide**

### **🔧 Essential Commands**

```bash
# Create shell app
ng add @angular-architects/module-federation --type host

# Create remote app
ng add @angular-architects/module-federation --type remote --port 4201

# Run in development
ng serve                    # Shell (port 4200)
ng serve product-catalog    # Remote (port 4201)

# Build for production
ng build                    # Shell
ng build product-catalog    # Remote
```

### **🎯 Key Concepts Summary**

| Concept               | What It Does                       | Why Important                       |
| --------------------- | ---------------------------------- | ----------------------------------- |
| **Shell App**         | Main application that hosts others | Central navigation & auth           |
| **Remote App**        | Independent micro-frontend         | Feature isolation & team autonomy   |
| **Module Federation** | Webpack plugin for sharing code    | Enables micro-frontend architecture |
| **Exposes**           | Makes code available to others     | Defines public API                  |
| **Remotes**           | Imports code from other apps       | Consumes external features          |
| **Shared**            | Libraries used by multiple apps    | Prevents version conflicts          |

### **🚀 Benefits You Get**

✅ **Team Independence** - Each team owns their micro-frontend  
✅ **Separate Deployments** - Deploy features independently  
✅ **Technology Flexibility** - Different tech stacks per team  
✅ **Scalable Development** - Multiple teams work in parallel  
✅ **Fault Isolation** - One micro-frontend failure doesn't break everything

---

## 🎯 **Part 1 Summary**

This comprehensive micro-frontend implementation covers:

- **🏗️ Module Federation Setup** - Complete webpack configuration for shell and remotes
- **🎮 Micro-Frontend Manager** - Advanced service for health monitoring and lifecycle management
- **🔄 Event Bus Communication** - Sophisticated inter-micro-frontend messaging system
- **🛡️ Security & Guards** - Comprehensive access control and version compatibility checking
- **📚 Step-by-Step Guide** - Practical setup instructions for beginners

**Coming in Part 2:**

- **📦 Shared library architecture** - Creating reusable component libraries
- **🔄 State synchronization patterns** - Managing state across micro-frontends
- **⚡ Performance optimization** - Loading strategies and caching
- **🧪 Testing strategies** - Unit, integration, and E2E testing for micro-frontends

---

# 🚀 **Part 2: Advanced Micro-Frontend Patterns**

## 📦 **Shared Library Architecture**

### **What are Shared Libraries? 🤔**

Think of shared libraries like a **company's supply closet** - instead of each department buying their own pens, staplers, and paper, everyone shares from one central location.

**In Micro-Frontends:**

- **Shared UI Components** = Company-wide design system
- **Shared Utilities** = Common tools everyone needs
- **Shared Services** = Centralized business logic

### **Creating Shared Libraries 🛠️**

#### **1. Shared UI Component Library**

```typescript
// libs/shared-ui/src/lib/components/button/button.component.ts
import { Component, Input, Output, EventEmitter } from "@angular/core";

export type ButtonVariant = "primary" | "secondary" | "danger" | "success";
export type ButtonSize = "small" | "medium" | "large";

@Component({
  selector: "app-button",
  template: `
    <button
      [class]="getButtonClasses()"
      [disabled]="disabled"
      [attr.aria-label]="ariaLabel"
      (click)="handleClick($event)"
    >
      <!-- 🔄 Loading State -->
      <span *ngIf="loading" class="spinner"></span>

      <!-- 📦 Icon Support -->
      <ng-container *ngIf="icon && !loading">
        <i
          [class]="'icon ' + icon"
          [class.icon-left]="iconPosition === 'left'"
        ></i>
      </ng-container>

      <!-- 📝 Button Content -->
      <span class="button-content">
        <ng-content></ng-content>
      </span>

      <!-- ➡️ Right Icon -->
      <ng-container *ngIf="icon && iconPosition === 'right' && !loading">
        <i [class]="'icon ' + icon"></i>
      </ng-container>
    </button>
  `,
  styles: [
    `
      .button {
        /* Base styles that all micro-frontends inherit */
        font-family: var(--font-family-primary);
        border-radius: var(--border-radius-medium);
        transition: all 0.2s ease;
        cursor: pointer;
        display: inline-flex;
        align-items: center;
        gap: 8px;
      }

      /* 🎨 Variant Styles */
      .button--primary {
        background: var(--color-primary);
        color: white;
      }
      .button--secondary {
        background: var(--color-secondary);
        color: var(--text-primary);
      }
      .button--danger {
        background: var(--color-danger);
        color: white;
      }

      /* 📏 Size Styles */
      .button--small {
        padding: 6px 12px;
        font-size: 14px;
      }
      .button--medium {
        padding: 8px 16px;
        font-size: 16px;
      }
      .button--large {
        padding: 12px 24px;
        font-size: 18px;
      }

      /* ⏳ Loading State */
      .spinner {
        width: 16px;
        height: 16px;
        border: 2px solid transparent;
        border-top: 2px solid currentColor;
        border-radius: 50%;
        animation: spin 1s linear infinite;
      }
    `,
  ],
})
export class ButtonComponent {
  @Input() variant: ButtonVariant = "primary";
  @Input() size: ButtonSize = "medium";
  @Input() disabled = false;
  @Input() loading = false;
  @Input() icon?: string;
  @Input() iconPosition: "left" | "right" = "left";
  @Input() ariaLabel?: string;

  @Output() clicked = new EventEmitter<Event>();

  handleClick(event: Event): void {
    if (!this.disabled && !this.loading) {
      this.clicked.emit(event);
    }
  }

  getButtonClasses(): string {
    return [
      "button",
      `button--${this.variant}`,
      `button--${this.size}`,
      this.disabled ? "button--disabled" : "",
      this.loading ? "button--loading" : "",
    ]
      .filter(Boolean)
      .join(" ");
  }
}
```

#### **2. Shared Utilities Library**

```typescript
// libs/shared-utils/src/lib/validators/custom-validators.ts

import { AbstractControl, ValidationErrors, ValidatorFn } from "@angular/forms";

export class CustomValidators {
  // 📧 Advanced Email Validation
  static email(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      if (!control.value) return null;

      const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
      const isValid = emailRegex.test(control.value);

      return isValid
        ? null
        : {
            email: {
              value: control.value,
              message: "Please enter a valid email address",
            },
          };
    };
  }

  // 🔒 Password Strength Validation
  static strongPassword(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      if (!control.value) return null;

      const value = control.value;
      const hasLower = /[a-z]/.test(value);
      const hasUpper = /[A-Z]/.test(value);
      const hasNumber = /\d/.test(value);
      const hasSpecial = /[!@#$%^&*(),.?":{}|<>]/.test(value);
      const isLongEnough = value.length >= 8;

      const errors: any = {};

      if (!hasLower) errors.missingLowercase = true;
      if (!hasUpper) errors.missingUppercase = true;
      if (!hasNumber) errors.missingNumber = true;
      if (!hasSpecial) errors.missingSpecial = true;
      if (!isLongEnough) errors.tooShort = true;

      return Object.keys(errors).length > 0 ? { strongPassword: errors } : null;
    };
  }

  // 📱 Phone Number Validation
  static phoneNumber(format: "US" | "international" = "US"): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      if (!control.value) return null;

      const patterns = {
        US: /^\+?1?[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$/,
        international: /^\+?[1-9]\d{1,14}$/,
      };

      const isValid = patterns[format].test(control.value);
      return isValid
        ? null
        : {
            phoneNumber: {
              value: control.value,
              format,
              message: `Please enter a valid ${format} phone number`,
            },
          };
    };
  }
}
```

#### **3. Shared State Management**

```typescript
// libs/shared-state/src/lib/auth/auth.state.ts
import { Injectable, computed, signal, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Router } from "@angular/router";
import { catchError, tap } from "rxjs/operators";
import { of } from "rxjs";

export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  roles: string[];
  permissions: string[];
  lastLoginAt: Date;
  avatarUrl?: string;
}

export interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  isLoading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: "root",
})
export class AuthStateService {
  private http = inject(HttpClient);
  private router = inject(Router);

  // 📊 REACTIVE STATE: Using Angular 17+ signals
  private state = signal<AuthState>({
    user: null,
    accessToken: localStorage.getItem("accessToken"),
    refreshToken: localStorage.getItem("refreshToken"),
    isLoading: false,
    error: null,
  });

  // 🔍 COMPUTED VALUES: Derived from state
  readonly isAuthenticated = computed(
    () => !!this.state().accessToken && !!this.state().user
  );
  readonly currentUser = computed(() => this.state().user);
  readonly isLoading = computed(() => this.state().isLoading);
  readonly userRoles = computed(() => this.state().user?.roles ?? []);
  readonly userPermissions = computed(
    () => this.state().user?.permissions ?? []
  );

  constructor() {
    // 🔄 Initialize authentication state on app start
    this.initializeAuth();
  }

  // 🔐 LOGIN: Authenticate user
  async login(email: string, password: string): Promise<boolean> {
    this.updateState({ isLoading: true, error: null });

    try {
      const response = await this.http
        .post<{
          user: User;
          accessToken: string;
          refreshToken: string;
        }>("/api/auth/login", { email, password })
        .toPromise();

      if (response) {
        // 💾 Store tokens
        localStorage.setItem("accessToken", response.accessToken);
        localStorage.setItem("refreshToken", response.refreshToken);

        // 📊 Update state
        this.updateState({
          user: response.user,
          accessToken: response.accessToken,
          refreshToken: response.refreshToken,
          isLoading: false,
          error: null,
        });

        // 🎉 Notify all micro-frontends
        this.broadcastAuthChange("login", response.user);

        return true;
      }
    } catch (error: any) {
      this.updateState({
        isLoading: false,
        error: error.message || "Login failed",
      });
    }

    return false;
  }

  // 🚪 LOGOUT: Clear authentication
  logout(): void {
    // 🗑️ Clear tokens
    localStorage.removeItem("accessToken");
    localStorage.removeItem("refreshToken");

    // 📊 Reset state
    this.updateState({
      user: null,
      accessToken: null,
      refreshToken: null,
      isLoading: false,
      error: null,
    });

    // 🎉 Notify all micro-frontends
    this.broadcastAuthChange("logout", null);

    // 🔄 Redirect to login
    this.router.navigate(["/login"]);
  }

  // 🔄 REFRESH TOKEN: Maintain session
  async refreshAccessToken(): Promise<boolean> {
    const refreshToken = this.state().refreshToken;
    if (!refreshToken) return false;

    try {
      const response = await this.http
        .post<{
          accessToken: string;
          refreshToken: string;
        }>("/api/auth/refresh", { refreshToken })
        .toPromise();

      if (response) {
        localStorage.setItem("accessToken", response.accessToken);
        localStorage.setItem("refreshToken", response.refreshToken);

        this.updateState({
          accessToken: response.accessToken,
          refreshToken: response.refreshToken,
        });

        return true;
      }
    } catch (error) {
      console.warn("Token refresh failed, logging out");
      this.logout();
    }

    return false;
  }

  // 🔐 PERMISSION CHECKING: Authorization helpers
  hasPermission(permission: string): boolean {
    return this.userPermissions().includes(permission);
  }

  hasRole(role: string): boolean {
    return this.userRoles().includes(role);
  }

  hasAnyRole(roles: string[]): boolean {
    const userRoles = this.userRoles();
    return roles.some((role) => userRoles.includes(role));
  }

  // 📡 CROSS-MICRO-FRONTEND COMMUNICATION
  private broadcastAuthChange(
    type: "login" | "logout",
    user: User | null
  ): void {
    // Send to all micro-frontends via BroadcastChannel
    const channel = new BroadcastChannel("auth-state");
    channel.postMessage({
      type: "auth-state-change",
      payload: { type, user },
      timestamp: Date.now(),
      source: "shared-auth-state",
    });
  }

  // 🔄 PRIVATE HELPERS
  private updateState(partialState: Partial<AuthState>): void {
    this.state.update((current) => ({ ...current, ...partialState }));
  }

  private async initializeAuth(): Promise<void> {
    const token = this.state().accessToken;
    if (!token) return;

    this.updateState({ isLoading: true });

    try {
      // Verify token and get user info
      const user = await this.http.get<User>("/api/auth/me").toPromise();

      if (user) {
        this.updateState({
          user,
          isLoading: false,
          error: null,
        });
      } else {
        this.logout();
      }
    } catch (error) {
      // Token is invalid
      this.logout();
}
```

### **How Shared Libraries Work in Module Federation 🔄**

```javascript
// webpack.config.js - Shared library configuration
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      // 📦 EXPOSING SHARED LIBRARIES
      exposes: {
        "./SharedUI": "./libs/shared-ui/src/public-api.ts",
        "./SharedUtils": "./libs/shared-utils/src/public-api.ts",
        "./SharedState": "./libs/shared-state/src/public-api.ts",
      },

      // 🔄 SHARING BETWEEN APPS
      shared: {
        // 🎨 Share the design system
        "@company/shared-ui": {
          singleton: true,
          strictVersion: true,
          eager: true, // Load immediately
        },

        // 🛠️ Share utilities
        "@company/shared-utils": {
          singleton: true,
          strictVersion: true,
        },

        // 📊 Share state management
        "@company/shared-state": {
          singleton: true,
          strictVersion: true,
          eager: true, // Critical for consistent state
        },
      },
    }),
  ],
};
```

**🎯 Usage in Micro-Frontends:**

```typescript
// Any micro-frontend can now use shared components
import { ButtonComponent } from "@company/shared-ui";
import { CustomValidators } from "@company/shared-utils";
import { AuthStateService } from "@company/shared-state";

@Component({
  template: `
    <!-- 🎨 Shared UI Component -->
    <app-button
      variant="primary"
      size="large"
      [loading]="isLoading"
      (clicked)="handleLogin()"
    >
      Login
    </app-button>
  `,
})
export class LoginComponent {
  constructor(
    private authState: AuthStateService // 📊 Shared state
  ) {}

  // Form with shared validators
  loginForm = this.fb.group({
    email: ["", [Validators.required, CustomValidators.email()]],
    password: ["", [Validators.required, CustomValidators.strongPassword()]],
  });
}
```

---

## 🔄 **State Synchronization Patterns**

### **The Challenge 🤔**

Imagine you're at a **group dinner** where everyone orders separately, but you need to keep track of:

- Who ordered what
- Total bill amount
- Who's paying for what

**In Micro-Frontends:**

- **Cart state** needs to be shared between product catalog and checkout
- **User authentication** must be consistent across all apps
- **Theme preferences** should apply everywhere

### **Pattern 1: Event-Driven State Sync 📡**

```typescript
// libs/shared-state/src/lib/event-bus/state-sync.service.ts
import { Injectable, signal, computed } from "@angular/core";

export interface StateChangeEvent<T = any> {
  type: string;
  payload: T;
  source: string;
  timestamp: number;
  version: number;
}

@Injectable({
  providedIn: "root",
})
export class StateSyncService {
  private broadcastChannel: BroadcastChannel;
  private subscribers = new Map<
    string,
    Set<(event: StateChangeEvent) => void>
  >();

  // 📊 Global state signals
  private sharedState = signal<{ [key: string]: any }>({});

  constructor() {
    this.broadcastChannel = new BroadcastChannel("micro-frontend-state");
    this.broadcastChannel.onmessage = (event) => {
      this.handleIncomingStateChange(event.data);
    };
  }

  // 📤 PUBLISH: Send state changes to all micro-frontends
  publishStateChange<T>(type: string, payload: T, source: string): void {
    const event: StateChangeEvent<T> = {
      type,
      payload,
      source,
      timestamp: Date.now(),
      version: this.getNextVersion(type),
    };

    // 🔄 Update local state
    this.updateLocalState(type, payload);

    // 📡 Broadcast to other micro-frontends
    this.broadcastChannel.postMessage(event);

    // 🎧 Notify local subscribers
    this.notifySubscribers(type, event);

    console.log(`📤 [${source}] Published state change:`, type, payload);
  }

  // 🎧 SUBSCRIBE: Listen for specific state changes
  subscribe<T>(
    eventType: string,
    callback: (event: StateChangeEvent<T>) => void
  ): () => void {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, new Set());
    }

    this.subscribers.get(eventType)!.add(callback);

    // Return unsubscribe function
    return () => {
      this.subscribers.get(eventType)?.delete(callback);
    };
  }

  // 📊 GET STATE: Access shared state reactively
  getStateSignal<T>(key: string): T | null {
    return computed(() => (this.sharedState()[key] as T) || null)();
  }

  // 🔄 Private methods
  private handleIncomingStateChange(event: StateChangeEvent): void {
    console.log(`📥 Received state change from ${event.source}:`, event.type);

    // Update local state
    this.updateLocalState(event.type, event.payload);

    // Notify subscribers
    this.notifySubscribers(event.type, event);
  }

  private updateLocalState(type: string, payload: any): void {
    this.sharedState.update((current) => ({
      ...current,
      [type]: payload,
    }));
  }

  private notifySubscribers(type: string, event: StateChangeEvent): void {
    const typeSubscribers = this.subscribers.get(type);
    if (typeSubscribers) {
      typeSubscribers.forEach((callback) => {
        try {
          callback(event);
        } catch (error) {
          console.error(`Error in state subscriber for ${type}:`, error);
        }
      });
    }
  }

  private getNextVersion(type: string): number {
    const currentState = this.sharedState();
    const currentVersion = currentState[`${type}_version`] || 0;
    const nextVersion = currentVersion + 1;

    this.sharedState.update((current) => ({
      ...current,
      [`${type}_version`]: nextVersion,
    }));

    return nextVersion;
  }
}
```

### **Pattern 2: Shopping Cart State Sync 🛒**

```typescript
// Example: Shopping cart shared across micro-frontends
import { Injectable, inject, signal, computed } from "@angular/core";
import { StateSyncService } from "./state-sync.service";

export interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
  imageUrl: string;
}

export interface CartState {
  items: CartItem[];
  totalItems: number;
  totalPrice: number;
  lastUpdated: Date;
}

@Injectable({
  providedIn: "root",
})
export class CartStateService {
  private stateSync = inject(StateSyncService);

  // 🛒 Local cart state
  private cartState = signal<CartState>({
    items: [],
    totalItems: 0,
    totalPrice: 0,
    lastUpdated: new Date(),
  });

  // 🔍 Computed values
  readonly items = computed(() => this.cartState().items);
  readonly totalItems = computed(() => this.cartState().totalItems);
  readonly totalPrice = computed(() => this.cartState().totalPrice);
  readonly isEmpty = computed(() => this.cartState().items.length === 0);

  constructor() {
    // 🎧 Listen for cart changes from other micro-frontends
    this.stateSync.subscribe("cart.state.changed", (event) => {
      console.log("🛒 Cart state updated from", event.source);
      this.cartState.set(event.payload);
    });

    // 🔄 Initialize from shared state
    this.loadInitialState();
  }

  // ➕ ADD ITEM: Add product to cart
  addItem(item: Omit<CartItem, "quantity">, quantity: number = 1): void {
    const currentItems = this.cartState().items;
    const existingItemIndex = currentItems.findIndex(
      (i) => i.productId === item.productId
    );

    let updatedItems: CartItem[];

    if (existingItemIndex >= 0) {
      // Update existing item
      updatedItems = currentItems.map((cartItem, index) =>
        index === existingItemIndex
          ? { ...cartItem, quantity: cartItem.quantity + quantity }
          : cartItem
      );
    } else {
      // Add new item
      updatedItems = [...currentItems, { ...item, quantity }];
    }

    this.updateCartState(updatedItems);

    // 📤 Broadcast change
    this.stateSync.publishStateChange(
      "cart.item.added",
      { productId: item.productId, quantity },
      "cart-service"
    );

    console.log(`➕ Added ${quantity}x ${item.name} to cart`);
  }

  // ➖ REMOVE ITEM: Remove product from cart
  removeItem(productId: string): void {
    const updatedItems = this.cartState().items.filter(
      (item) => item.productId !== productId
    );
    this.updateCartState(updatedItems);

    this.stateSync.publishStateChange(
      "cart.item.removed",
      { productId },
      "cart-service"
    );

    console.log(`➖ Removed product ${productId} from cart`);
  }

  // 🔄 UPDATE QUANTITY: Change item quantity
  updateQuantity(productId: string, quantity: number): void {
    if (quantity <= 0) {
      this.removeItem(productId);
      return;
    }

    const updatedItems = this.cartState().items.map((item) =>
      item.productId === productId ? { ...item, quantity } : item
    );

    this.updateCartState(updatedItems);

    this.stateSync.publishStateChange(
      "cart.quantity.updated",
      { productId, quantity },
      "cart-service"
    );
  }

  // 🗑️ CLEAR CART: Empty the cart
  clearCart(): void {
    this.updateCartState([]);

    this.stateSync.publishStateChange("cart.cleared", {}, "cart-service");

    console.log("🗑️ Cart cleared");
  }

  // 🔄 Private helpers
  private updateCartState(items: CartItem[]): void {
    const totalItems = items.reduce((sum, item) => sum + item.quantity, 0);
    const totalPrice = items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );

    const newState: CartState = {
      items,
      totalItems,
      totalPrice,
      lastUpdated: new Date(),
    };

    this.cartState.set(newState);

    // 📤 Sync with other micro-frontends
    this.stateSync.publishStateChange(
      "cart.state.changed",
      newState,
      "cart-service"
    );
  }

  private loadInitialState(): void {
    const savedCart =
      this.stateSync.getStateSignal<CartState>("cart.state.changed");
    if (savedCart) {
      this.cartState.set(savedCart);
      console.log("🔄 Loaded cart state from shared storage");
    }
  }
}
```

---

## ⚡ **Performance Optimization**

### **Loading Strategies 🚀**

#### **1. Lazy Loading with Preloading**

```typescript
// apps/shell/src/app/core/preloading/smart-preloading.strategy.ts
import { Injectable, inject } from "@angular/core";
import { PreloadingStrategy, Route } from "@angular/router";
import { Observable, of, EMPTY, timer } from "rxjs";
import { mergeMap, switchMap } from "rxjs/operators";

@Injectable({
  providedIn: "root",
})
export class SmartPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // 🎯 Check if route should be preloaded
    if (this.shouldPreload(route)) {
      console.log(`🚀 Preloading micro-frontend: ${route.path}`);

      // ⏱️ Delay preloading to avoid blocking initial render
      return timer(2000).pipe(
        switchMap(() => load()),
        mergeMap(() => EMPTY) // Don't emit the module, just preload
      );
    }

    return EMPTY; // Don't preload
  }

  private shouldPreload(route: Route): boolean {
    // 🔍 Preload based on user behavior and route data
    const routeData = route.data || {};

    // High-priority routes (user likely to visit)
    if (routeData["priority"] === "high") return true;

    // User's role determines what to preload
    if (routeData["preloadForRoles"]) {
      const userRoles = this.getCurrentUserRoles();
      return routeData["preloadForRoles"].some((role: string) =>
        userRoles.includes(role)
      );
    }

    // Analytics-based preloading
    if (routeData["preloadBasedOnAnalytics"]) {
      return this.shouldPreloadBasedOnAnalytics(route.path);
    }

    return false;
  }

  private getCurrentUserRoles(): string[] {
    // Get from auth service
    return ["user", "customer"]; // Example
  }

  private shouldPreloadBasedOnAnalytics(routePath?: string): boolean {
    // Example: preload if route has >70% visit rate
    const analyticsData = {
      "/products": 0.85, // 85% of users visit products
      "/profile": 0.72, // 72% visit profile
      "/admin": 0.15, // Only 15% visit admin
    };

    return (analyticsData[routePath as keyof typeof analyticsData] || 0) > 0.7;
  }
}
```

#### **2. Module Federation Caching**

```typescript
// apps/shell/src/app/core/federation/federation-cache.service.ts
import { Injectable } from "@angular/core";

interface CacheEntry {
  module: any;
  timestamp: number;
  version: string;
}

@Injectable({
  providedIn: "root",
})
export class FederationCacheService {
  private cache = new Map<string, CacheEntry>();
  private readonly CACHE_DURATION = 5 * 60 * 1000; // 5 minutes

  async loadRemoteModule(
    remoteName: string,
    exposedModule: string,
    version?: string
  ): Promise<any> {
    const cacheKey = `${remoteName}/${exposedModule}`;

    // 🔍 Check cache first
    const cached = this.getCachedModule(cacheKey, version);
    if (cached) {
      console.log(`💾 Using cached module: ${cacheKey}`);
      return cached;
    }

    // 📥 Load fresh module
    console.log(`📥 Loading remote module: ${cacheKey}`);

    try {
      // Dynamic import with error handling
      const remoteModule = await this.importWithRetry(
        remoteName,
        exposedModule
      );

      // 💾 Cache the module
      this.cacheModule(cacheKey, remoteModule, version);

      return remoteModule;
    } catch (error) {
      console.error(`❌ Failed to load ${cacheKey}:`, error);
      throw error;
    }
  }

  private getCachedModule(cacheKey: string, version?: string): any | null {
    const entry = this.cache.get(cacheKey);

    if (!entry) return null;

    // ⏰ Check if cache is expired
    const isExpired = Date.now() - entry.timestamp > this.CACHE_DURATION;
    if (isExpired) {
      this.cache.delete(cacheKey);
      return null;
    }

    // 📋 Check version compatibility
    if (version && entry.version !== version) {
      console.log(
        `🔄 Version mismatch for ${cacheKey}: cached=${entry.version}, required=${version}`
      );
      this.cache.delete(cacheKey);
      return null;
    }

    return entry.module;
  }

  private cacheModule(cacheKey: string, module: any, version = "1.0.0"): void {
    this.cache.set(cacheKey, {
      module,
      timestamp: Date.now(),
      version,
    });

    // 🧹 Cleanup old entries
    this.cleanupExpiredEntries();
  }

  private async importWithRetry(
    remoteName: string,
    exposedModule: string,
    maxRetries = 3
  ): Promise<any> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        // @ts-ignore - Dynamic import
        return await import(`${remoteName}/${exposedModule}`);
      } catch (error) {
        console.warn(
          `⚠️ Attempt ${attempt} failed for ${remoteName}/${exposedModule}`
        );

        if (attempt === maxRetries) {
          throw new Error(
            `Failed to load ${remoteName}/${exposedModule} after ${maxRetries} attempts`
          );
        }

        // ⏱️ Wait before retry (exponential backoff)
        await this.wait(Math.pow(2, attempt) * 1000);
      }
    }
  }

  private wait(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  private cleanupExpiredEntries(): void {
    const now = Date.now();
    for (const [key, entry] of this.cache.entries()) {
      if (now - entry.timestamp > this.CACHE_DURATION) {
        this.cache.delete(key);
      }
    }
  }

  // 🗑️ Manual cache management
  clearCache(): void {
    this.cache.clear();
    console.log("🗑️ Federation cache cleared");
  }

  getCacheStats(): { size: number; entries: string[] } {
    return {
      size: this.cache.size,
      entries: Array.from(this.cache.keys()),
    };
  }
}
```

---

## 🧪 **Testing Strategies for Micro-Frontends**

### **The Testing Challenge 🤔**

Testing micro-frontends is like **testing a rock band** where:

- Each musician practices **individually** (unit tests)
- They rehearse **in pairs** (integration tests)
- They perform the **full concert** together (E2E tests)
- And sometimes the **sound system fails** (micro-frontend unavailable)

### **1. Unit Testing Individual Micro-Frontends 🎯**

```typescript
// apps/product-catalog/src/app/features/product/services/product.service.spec.ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { ProductService, Product } from "./product.service";

describe("ProductService", () => {
  let service: ProductService;
  let httpMock: HttpTestingController;

  const mockProducts: Product[] = [
    {
      id: "1",
      name: "Test Product",
      price: 99.99,
      description: "A test product",
      category: "electronics",
      imageUrl: "test.jpg",
      rating: 4.5,
      inStock: true,
    },
  ];

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ProductService],
    });

    service = TestBed.inject(ProductService);
    httpMock = TestBed.inject(HttpTestingController);

    // 🎭 Mock shell services
    (service as any).eventBusService = {
      subscribe: jasmine.createSpy("subscribe"),
      publish: jasmine.createSpy("publish"),
    };
  });

  afterEach(() => {
    httpMock.verify();
  });

  it("should load products", () => {
    // 🎬 Arrange
    service.loadProducts().subscribe((products) => {
      // 🔍 Assert
      expect(products).toEqual(mockProducts);
      expect(products.length).toBe(1);
    });

    // 🎭 Act & Mock HTTP
    const req = httpMock.expectOne("/api/products");
    expect(req.request.method).toBe("GET");
    req.flush(mockProducts);
  });

  it("should handle shell service unavailable gracefully", () => {
    // 🎬 Arrange - Simulate shell services not available
    (service as any).eventBusService = null;

    // 🎭 Act - Should not throw error
    expect(() => service.addToCart("1", 2)).not.toThrow();
  });

  it("should publish events when shell services available", () => {
    // 🎭 Act
    service.addToCart("1", 2);

    // 🔍 Assert
    expect((service as any).eventBusService.publish).toHaveBeenCalledWith(
      "product.added-to-cart",
      jasmine.objectContaining({
        productId: "1",
        quantity: 2,
      }),
      jasmine.any(Object)
    );
  });
});
```

### **2. Integration Testing with Mock Shell 🔗**

```typescript
// apps/product-catalog/src/app/integration/shell-integration.spec.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { Router } from "@angular/router";
import { Component } from "@angular/core";
import { BehaviorSubject } from "rxjs";

// 🎭 Mock Shell Services
class MockShellServices {
  private eventBus = new BehaviorSubject<any>(null);

  // Mock EventBus
  eventBusService = {
    subscribe: jasmine
      .createSpy("subscribe")
      .and.callFake((event: string, callback: Function) => {
        return this.eventBus.subscribe(callback);
      }),

    publish: jasmine
      .createSpy("publish")
      .and.callFake((event: string, data: any) => {
        this.eventBus.next({ event, data });
      }),
  };

  // Mock Authentication
  authService = {
    isAuthenticated: jasmine.createSpy("isAuthenticated").and.returnValue(true),
    getCurrentUser: jasmine.createSpy("getCurrentUser").and.returnValue({
      id: "test-user",
      email: "test@example.com",
      roles: ["user"],
    }),
  };

  // Mock Theme Service
  themeService = {
    getCurrentTheme: jasmine
      .createSpy("getCurrentTheme")
      .and.returnValue("light"),
    setTheme: jasmine.createSpy("setTheme"),
  };
}

@Component({
  template: `
    <div>
      <h1>Product Catalog Integration Test</h1>
      <!-- Test component that uses shell services -->
    </div>
  `,
})
class TestHostComponent {}

describe("Product Catalog Shell Integration", () => {
  let component: TestHostComponent;
  let fixture: ComponentFixture<TestHostComponent>;
  let mockShell: MockShellServices;
  let router: Router;

  beforeEach(async () => {
    mockShell = new MockShellServices();

    await TestBed.configureTestingModule({
      declarations: [TestHostComponent],
      providers: [
        // 🎭 Provide mock shell services
        { provide: "shell/EventBus", useValue: mockShell.eventBusService },
        { provide: "shell/AuthService", useValue: mockShell.authService },
        { provide: "shell/ThemeService", useValue: mockShell.themeService },
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(TestHostComponent);
    component = fixture.componentInstance;
    router = TestBed.inject(Router);
  });

  it("should integrate with shell authentication", () => {
    // 🎭 Act
    fixture.detectChanges();

    // 🔍 Assert
    expect(mockShell.authService.isAuthenticated).toHaveBeenCalled();
    expect(mockShell.authService.getCurrentUser).toHaveBeenCalled();
  });

  it("should handle shell service communication", () => {
    // 🎭 Act - Simulate event from shell
    mockShell.eventBusService.publish("theme.changed", { theme: "dark" });

    // 🔍 Assert - Should receive and handle the event
    expect(mockShell.eventBusService.subscribe).toHaveBeenCalled();
  });

  it("should gracefully handle shell service failures", () => {
    // 🎬 Arrange - Mock service failure
    mockShell.authService.isAuthenticated.and.throwError("Service unavailable");

    // 🎭 Act & Assert - Should not crash
    expect(() => fixture.detectChanges()).not.toThrow();
  });
});
```

---

## 🎉 **Final Summary: Your Micro-Frontend Mastery**

Congratulations! 🎊 You've just learned how to build **enterprise-grade micro-frontend architecture** with Angular. Here's what you've mastered:

### **🏗️ Architecture Foundations**

- **Module Federation setup** for shell and remote applications
- **Project structure** that scales with multiple teams
- **Development workflow** that enables independent deployments

### **🔄 Communication Patterns**

- **Event-driven architecture** for micro-frontend communication
- **Shared state management** with reactive patterns
- **Cross-app messaging** with BroadcastChannel API

### **📦 Code Sharing Strategy**

- **Shared component libraries** for consistent UI
- **Utility libraries** for common functionality
- **State management** libraries for cross-app consistency

### **⚡ Performance & Optimization**

- **Smart preloading** strategies for better UX
- **Caching mechanisms** for faster load times
- **Error boundaries** and graceful degradation

### **🧪 Testing Excellence**

- **Unit testing** for individual micro-frontends
- **Integration testing** with mock shell services
- **End-to-end testing** for full user journeys
- **Performance testing** for load time optimization

### **🚀 Production Readiness**

- **Health monitoring** and service discovery
- **Version compatibility** checking
- **Fallback strategies** for service unavailability
- **Security patterns** and access control

**You're now ready to:**
✅ **Lead micro-frontend initiatives** in your organization  
✅ **Architecture discussions** with confidence  
✅ **Implement scalable solutions** for large teams  
✅ **Optimize performance** for better user experience  
✅ **Test effectively** across the entire ecosystem

**Next Steps:**

- 🔧 **Implement** these patterns in your current project
- 📚 **Share knowledge** with your team
- 🌟 **Contribute** to micro-frontend community
- 📈 **Monitor** and optimize your implementations

Remember: Great architecture isn't about the complexity you add—it's about the complexity you **eliminate** while enabling teams to work **independently** and **efficiently**! 🌟

```

```
