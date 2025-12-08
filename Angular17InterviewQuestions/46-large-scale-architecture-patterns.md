# 🏗️ **Large-Scale Angular Architecture Patterns**

## 🎯 **What You'll Learn**

Master **enterprise-grade Angular architecture** for applications with 100+ components, multiple teams, and complex business domains! Think of large-scale architecture as designing a **smart city** - you need proper districts, infrastructure, traffic flow, and governance systems.

---

## 📚 **The Challenge: Why Architecture Matters**

### **🤔 The Problem with "Just Start Coding"**

```typescript
// ❌ How NOT to structure a large Angular app
src/
├── app/
│   ├── component1.ts          // 😱 Everything in one folder
│   ├── component2.ts          // 😱 No organization
│   ├── component3.ts          // 😱 Hard to find anything
│   ├── service1.ts            // 😱 Tangled dependencies
│   ├── service2.ts            // 😱 No clear boundaries
│   ├── model1.ts              // 😱 Shared state chaos
│   └── app.module.ts          // 😱 Massive monolithic module
```

### **🎯 What Happens Without Proper Architecture**

- 🔥 **Developer Burnout** - "Where does this component go?"
- 🐛 **Bug Multiplication** - Changes break unexpected parts
- 🚀 **Slow Development** - Teams stepping on each other's toes
- 📚 **Knowledge Silos** - Only certain devs understand certain areas
- 🔄 **Merge Conflicts** - Everyone editing the same files

---

## 🏛️ **Enterprise Architecture Patterns**

### **1. 🎯 Domain-Driven Design (DDD) Structure**

```typescript
// ✅ Enterprise-grade folder structure
src/
├── app/
│   ├── core/                          // 🏛️ Singleton services & guards
│   │   ├── services/
│   │   │   ├── auth/
│   │   │   │   ├── auth.service.ts
│   │   │   │   ├── token.service.ts
│   │   │   │   └── permission.service.ts
│   │   │   ├── api/
│   │   │   │   ├── api-client.service.ts
│   │   │   │   ├── error-handler.service.ts
│   │   │   │   └── request-interceptor.ts
│   │   │   └── logging/
│   │   │       ├── logger.service.ts
│   │   │       └── telemetry.service.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   │   ├── permission.guard.ts
│   │   │   └── unsaved-changes.guard.ts
│   │   ├── interceptors/
│   │   │   ├── auth-token.interceptor.ts
│   │   │   ├── error.interceptor.ts
│   │   │   └── loading.interceptor.ts
│   │   └── core.module.ts
│   │
│   ├── shared/                        // 🔄 Reusable components & utilities
│   │   ├── components/
│   │   │   ├── ui/
│   │   │   │   ├── button/
│   │   │   │   ├── modal/
│   │   │   │   ├── table/
│   │   │   │   └── form-controls/
│   │   │   └── layout/
│   │   │       ├── header/
│   │   │       ├── sidebar/
│   │   │       └── footer/
│   │   ├── pipes/
│   │   ├── directives/
│   │   ├── validators/
│   │   ├── utils/
│   │   └── shared.module.ts
│   │
│   ├── features/                      // 🎯 Business domains
│   │   ├── user-management/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   ├── store/                 // NgRx feature store
│   │   │   │   ├── user.actions.ts
│   │   │   │   ├── user.effects.ts
│   │   │   │   ├── user.reducer.ts
│   │   │   │   └── user.selectors.ts
│   │   │   ├── user-management-routing.module.ts
│   │   │   └── user-management.module.ts
│   │   │
│   │   ├── billing/
│   │   │   ├── components/
│   │   │   │   ├── billing-dashboard/
│   │   │   │   ├── invoice-list/
│   │   │   │   └── payment-form/
│   │   │   ├── services/
│   │   │   │   ├── billing.service.ts
│   │   │   │   └── payment.service.ts
│   │   │   ├── models/
│   │   │   │   ├── invoice.model.ts
│   │   │   │   └── payment.model.ts
│   │   │   └── billing.module.ts
│   │   │
│   │   └── reports/
│   │       ├── components/
│   │       ├── services/
│   │       ├── models/
│   │       └── reports.module.ts
│   │
│   ├── layout/                        // 🏠 App shell components
│   │   ├── main-layout/
│   │   ├── auth-layout/
│   │   └── error-layout/
│   │
│   └── app-routing.module.ts
```

### **2. 🔧 Core Module Pattern**

```typescript
// src/app/core/core.module.ts
import { NgModule, Optional, SkipSelf } from "@angular/core";
import { CommonModule } from "@angular/common";
import { HTTP_INTERCEPTORS } from "@angular/common/http";

// Services
import { AuthService } from "./services/auth/auth.service";
import { ApiClientService } from "./services/api/api-client.service";
import { LoggerService } from "./services/logging/logger.service";
import { ErrorHandlerService } from "./services/api/error-handler.service";

// Guards
import { AuthGuard } from "./guards/auth.guard";
import { PermissionGuard } from "./guards/permission.guard";

// Interceptors
import { AuthTokenInterceptor } from "./interceptors/auth-token.interceptor";
import { ErrorInterceptor } from "./interceptors/error.interceptor";
import { LoadingInterceptor } from "./interceptors/loading.interceptor";

@NgModule({
  imports: [CommonModule],
  providers: [
    // 🔐 Authentication Services
    AuthService,
    ApiClientService,
    LoggerService,
    ErrorHandlerService,

    // 🛡️ Guards
    AuthGuard,
    PermissionGuard,

    // 🔄 HTTP Interceptors (Order matters!)
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthTokenInterceptor,
      multi: true,
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoadingInterceptor,
      multi: true,
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true,
    },

    // 🎯 Global Error Handling
    {
      provide: ErrorHandler,
      useClass: ErrorHandlerService,
    },
  ],
})
export class CoreModule {
  /**
   * 🚨 CRITICAL: Prevent CoreModule from being imported more than once
   * This ensures singleton services remain singletons
   */
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error(
        "🚫 CoreModule is already loaded. Import it ONLY in AppModule!"
      );
    }
  }

  /**
   * ⚡ For root-level import with configuration
   */
  static forRoot(config?: CoreConfig): ModuleWithProviders<CoreModule> {
    return {
      ngModule: CoreModule,
      providers: [
        {
          provide: CORE_CONFIG,
          useValue: config || DEFAULT_CORE_CONFIG,
        },
      ],
    };
  }
}

// 🎯 Core configuration interface
export interface CoreConfig {
  apiBaseUrl: string;
  enableLogging: boolean;
  logLevel: "debug" | "info" | "warn" | "error";
  sessionTimeoutMinutes: number;
  enableAnalytics: boolean;
}

export const DEFAULT_CORE_CONFIG: CoreConfig = {
  apiBaseUrl: "/api",
  enableLogging: true,
  logLevel: "info",
  sessionTimeoutMinutes: 30,
  enableAnalytics: false,
};

export const CORE_CONFIG = new InjectionToken<CoreConfig>("CORE_CONFIG");
```

### **3. 🎯 Feature Module Pattern**

```typescript
// src/app/features/user-management/user-management.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { ReactiveFormsModule } from "@angular/forms";
import { StoreModule } from "@ngrx/store";
import { EffectsModule } from "@ngrx/effects";

// Routing
import { UserManagementRoutingModule } from "./user-management-routing.module";

// Components
import { UserDashboardComponent } from "./components/user-dashboard/user-dashboard.component";
import { UserListComponent } from "./components/user-list/user-list.component";
import { UserFormComponent } from "./components/user-form/user-form.component";
import { UserDetailComponent } from "./components/user-detail/user-detail.component";

// Services
import { UserService } from "./services/user.service";
import { UserValidationService } from "./services/user-validation.service";
import { UserExportService } from "./services/user-export.service";

// Store
import { userReducer } from "./store/user.reducer";
import { UserEffects } from "./store/user.effects";

// Shared Module
import { SharedModule } from "../../shared/shared.module";

// Feature-specific components that aren't reusable
const COMPONENTS = [
  UserDashboardComponent,
  UserListComponent,
  UserFormComponent,
  UserDetailComponent,
];

// Feature-specific services
const SERVICES = [UserService, UserValidationService, UserExportService];

@NgModule({
  declarations: [...COMPONENTS],
  imports: [
    // Angular modules
    CommonModule,
    ReactiveFormsModule,

    // Routing
    UserManagementRoutingModule,

    // Shared components & utilities
    SharedModule,

    // State management (feature-specific)
    StoreModule.forFeature("users", userReducer),
    EffectsModule.forFeature([UserEffects]),
  ],
  providers: [...SERVICES],
})
export class UserManagementModule {
  constructor() {
    console.log("🧑‍👥 UserManagementModule loaded");
  }
}
```

### **4. 🔄 Shared Module Pattern**

```typescript
// src/app/shared/shared.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { FormsModule, ReactiveFormsModule } from "@angular/forms";
import { RouterModule } from "@angular/router";

// Angular Material (example)
import { MatButtonModule } from "@angular/material/button";
import { MatInputModule } from "@angular/material/input";
import { MatTableModule } from "@angular/material/table";
import { MatDialogModule } from "@angular/material/dialog";
import { MatSnackBarModule } from "@angular/material/snack-bar";

// Custom UI Components
import { LoadingSpinnerComponent } from "./components/ui/loading-spinner/loading-spinner.component";
import { ConfirmDialogComponent } from "./components/ui/confirm-dialog/confirm-dialog.component";
import { DataTableComponent } from "./components/ui/data-table/data-table.component";
import { FormFieldComponent } from "./components/ui/form-field/form-field.component";

// Layout Components
import { PageHeaderComponent } from "./components/layout/page-header/page-header.component";
import { EmptyStateComponent } from "./components/layout/empty-state/empty-state.component";

// Pipes
import { TruncatePipe } from "./pipes/truncate.pipe";
import { HighlightPipe } from "./pipes/highlight.pipe";
import { SafeUrlPipe } from "./pipes/safe-url.pipe";
import { CurrencyFormatPipe } from "./pipes/currency-format.pipe";

// Directives
import { ClickOutsideDirective } from "./directives/click-outside.directive";
import { AutoFocusDirective } from "./directives/auto-focus.directive";
import { PermissionDirective } from "./directives/permission.directive";

// Third-party modules that are commonly used
const MATERIAL_MODULES = [
  MatButtonModule,
  MatInputModule,
  MatTableModule,
  MatDialogModule,
  MatSnackBarModule,
];

const UI_COMPONENTS = [
  LoadingSpinnerComponent,
  ConfirmDialogComponent,
  DataTableComponent,
  FormFieldComponent,
];

const LAYOUT_COMPONENTS = [PageHeaderComponent, EmptyStateComponent];

const PIPES = [TruncatePipe, HighlightPipe, SafeUrlPipe, CurrencyFormatPipe];

const DIRECTIVES = [
  ClickOutsideDirective,
  AutoFocusDirective,
  PermissionDirective,
];

@NgModule({
  declarations: [
    ...UI_COMPONENTS,
    ...LAYOUT_COMPONENTS,
    ...PIPES,
    ...DIRECTIVES,
  ],
  imports: [
    // Angular core modules
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModule,

    // Third-party modules
    ...MATERIAL_MODULES,
  ],
  exports: [
    // ⚡ Re-export commonly used Angular modules
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModule,

    // ⚡ Re-export Material modules
    ...MATERIAL_MODULES,

    // ⚡ Export our custom components
    ...UI_COMPONENTS,
    ...LAYOUT_COMPONENTS,

    // ⚡ Export our pipes & directives
    ...PIPES,
    ...DIRECTIVES,
  ],
})
export class SharedModule {}
```

---

## 🏗️ **Advanced Architecture Patterns**

### **1. 🎯 Micro-Frontend Architecture**

```typescript
// src/app/shell/shell.module.ts - Main Shell Application
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { ModuleFederationPlugin } from "@module-federation/webpack";

// Shell components
import { ShellComponent } from "./shell.component";
import { NavigationComponent } from "./navigation/navigation.component";

// Lazy-loaded micro-frontends
const routes: Routes = [
  {
    path: "users",
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4001/remoteEntry.js",
        remoteName: "userManagement",
        exposedModule: "./UserManagementModule",
      }).then((m) => m.UserManagementModule),
  },
  {
    path: "billing",
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4002/remoteEntry.js",
        remoteName: "billing",
        exposedModule: "./BillingModule",
      }).then((m) => m.BillingModule),
  },
  {
    path: "reports",
    loadChildren: () =>
      loadRemoteModule({
        type: "module",
        remoteEntry: "http://localhost:4003/remoteEntry.js",
        remoteName: "reports",
        exposedModule: "./ReportsModule",
      }).then((m) => m.ReportsModule),
  },
];

@NgModule({
  declarations: [ShellComponent, NavigationComponent],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes, {
      enableTracing: false,
      preloadingStrategy: PreloadAllModules,
    }),
  ],
  bootstrap: [ShellComponent],
})
export class ShellModule {}

// webpack.config.js - Module Federation Configuration
const ModuleFederationPlugin = require("@module-federation/webpack");

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: "shell",
      remotes: {
        userManagement: "userManagement@http://localhost:4001/remoteEntry.js",
        billing: "billing@http://localhost:4002/remoteEntry.js",
        reports: "reports@http://localhost:4003/remoteEntry.js",
      },
      shared: {
        "@angular/core": { singleton: true, strictVersion: true },
        "@angular/common": { singleton: true, strictVersion: true },
        "@angular/router": { singleton: true, strictVersion: true },
        rxjs: { singleton: true, strictVersion: true },
      },
    }),
  ],
};
```

### **2. 🎨 Design System Integration**

```typescript
// src/app/design-system/design-system.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";

// Design System Components
import { DSButtonComponent } from "./components/button/ds-button.component";
import { DSInputComponent } from "./components/input/ds-input.component";
import { DSCardComponent } from "./components/card/ds-card.component";
import { DSModalComponent } from "./components/modal/ds-modal.component";
import { DSDataTableComponent } from "./components/data-table/ds-data-table.component";

// Design System Services
import { ThemeService } from "./services/theme.service";
import { BreakpointService } from "./services/breakpoint.service";
import { DesignTokenService } from "./services/design-token.service";

// Design System Directives
import { ResponsiveDirective } from "./directives/responsive.directive";
import { ThemeDirective } from "./directives/theme.directive";

const DESIGN_SYSTEM_COMPONENTS = [
  DSButtonComponent,
  DSInputComponent,
  DSCardComponent,
  DSModalComponent,
  DSDataTableComponent,
];

const DESIGN_SYSTEM_DIRECTIVES = [ResponsiveDirective, ThemeDirective];

@NgModule({
  declarations: [...DESIGN_SYSTEM_COMPONENTS, ...DESIGN_SYSTEM_DIRECTIVES],
  imports: [CommonModule],
  providers: [ThemeService, BreakpointService, DesignTokenService],
  exports: [...DESIGN_SYSTEM_COMPONENTS, ...DESIGN_SYSTEM_DIRECTIVES],
})
export class DesignSystemModule {
  static forRoot(): ModuleWithProviders<DesignSystemModule> {
    return {
      ngModule: DesignSystemModule,
      providers: [ThemeService, BreakpointService, DesignTokenService],
    };
  }
}

// Design Token Service
@Injectable({
  providedIn: "root",
})
export class DesignTokenService {
  private tokens = {
    colors: {
      primary: {
        50: "#e3f2fd",
        100: "#bbdefb",
        500: "#2196f3",
        900: "#0d47a1",
      },
      semantic: {
        success: "#4caf50",
        warning: "#ff9800",
        error: "#f44336",
        info: "#2196f3",
      },
    },
    spacing: {
      xs: "4px",
      sm: "8px",
      md: "16px",
      lg: "24px",
      xl: "32px",
      xxl: "48px",
    },
    typography: {
      fontFamily: "Inter, sans-serif",
      fontSize: {
        xs: "0.75rem",
        sm: "0.875rem",
        md: "1rem",
        lg: "1.125rem",
        xl: "1.25rem",
      },
    },
    breakpoints: {
      sm: "576px",
      md: "768px",
      lg: "992px",
      xl: "1200px",
    },
  };

  getToken(path: string): any {
    return this.getNestedProperty(this.tokens, path);
  }

  private getNestedProperty(obj: any, path: string): any {
    return path.split(".").reduce((current, prop) => current?.[prop], obj);
  }
}
```

### **3. 🗄️ Enterprise State Management**

```typescript
// src/app/store/app.state.ts - Global State Structure
export interface AppState {
  // Global app state
  ui: UIState;
  auth: AuthState;
  router: RouterState;

  // Feature states (lazy-loaded)
  users?: UserState;
  billing?: BillingState;
  reports?: ReportState;
}

export interface UIState {
  loading: boolean;
  sidebarCollapsed: boolean;
  theme: "light" | "dark";
  breadcrumbs: Breadcrumb[];
  notifications: Notification[];
}

// src/app/store/app.reducer.ts - Root Reducer
import { ActionReducerMap, MetaReducer } from "@ngrx/store";
import { environment } from "../../environments/environment";

// Global reducers
import { uiReducer } from "./ui/ui.reducer";
import { authReducer } from "./auth/auth.reducer";

// Import hydration and router state
import { routerReducer } from "@ngrx/router-store";

export const appReducers: ActionReducerMap<AppState> = {
  ui: uiReducer,
  auth: authReducer,
  router: routerReducer,
};

// Meta reducers for global functionality
export const metaReducers: MetaReducer<AppState>[] = !environment.production
  ? [storeFreeze, logger]
  : [storageSync];

// Local storage sync for persistence
function storageSync(reducer: any) {
  return localStorageSync({
    keys: [
      {
        auth: {
          serialize: (auth: AuthState) => ({
            user: auth.user,
            token: auth.token,
            // Don't persist sensitive data
          }),
        },
      },
      "ui.theme",
      "ui.sidebarCollapsed",
    ],
    rehydrate: true,
    storage: window.localStorage,
  })(reducer);
}

// Development logging
function logger(reducer: any) {
  return storeLogger({
    collapsed: true,
    filter: {
      blacklist: [
        "@ngrx/router-store/request",
        "@ngrx/router-store/navigation",
      ],
    },
  })(reducer);
}
```

### **4. 🔄 Advanced Routing Strategies**

```typescript
// src/app/app-routing.module.ts - Enterprise Routing
import { NgModule } from "@angular/core";
import { RouterModule, Routes, PreloadingStrategy } from "@angular/router";

// Guards
import { AuthGuard } from "./core/guards/auth.guard";
import { PermissionGuard } from "./core/guards/permission.guard";
import { FeatureFlagGuard } from "./core/guards/feature-flag.guard";

// Resolvers
import { UserProfileResolver } from "./core/resolvers/user-profile.resolver";
import { OrganizationResolver } from "./core/resolvers/organization.resolver";

// Custom preloading strategy
import { CustomPreloadingStrategy } from "./core/strategies/custom-preloading.strategy";

const routes: Routes = [
  {
    path: "",
    redirectTo: "/dashboard",
    pathMatch: "full",
  },

  // Public routes
  {
    path: "auth",
    loadChildren: () =>
      import("./features/auth/auth.module").then((m) => m.AuthModule),
    data: { preload: false }, // Don't preload auth module
  },

  // Protected routes with layout
  {
    path: "",
    component: MainLayoutComponent,
    canActivate: [AuthGuard],
    resolve: {
      userProfile: UserProfileResolver,
      organization: OrganizationResolver,
    },
    children: [
      {
        path: "dashboard",
        loadChildren: () =>
          import("./features/dashboard/dashboard.module").then(
            (m) => m.DashboardModule
          ),
        data: {
          preload: true,
          breadcrumb: "Dashboard",
          permissions: ["dashboard.view"],
        },
      },
      {
        path: "users",
        loadChildren: () =>
          import("./features/user-management/user-management.module").then(
            (m) => m.UserManagementModule
          ),
        canActivate: [PermissionGuard],
        data: {
          preload: true,
          breadcrumb: "User Management",
          permissions: ["users.view"],
        },
      },
      {
        path: "billing",
        loadChildren: () =>
          import("./features/billing/billing.module").then(
            (m) => m.BillingModule
          ),
        canActivate: [PermissionGuard, FeatureFlagGuard],
        data: {
          preload: false, // Heavy module, load on demand
          breadcrumb: "Billing",
          permissions: ["billing.view"],
          featureFlag: "billing-enabled",
        },
      },
      {
        path: "reports",
        loadChildren: () =>
          import("./features/reports/reports.module").then(
            (m) => m.ReportsModule
          ),
        canActivate: [PermissionGuard],
        data: {
          preload: false,
          breadcrumb: "Reports",
          permissions: ["reports.view"],
        },
      },
      {
        path: "settings",
        loadChildren: () =>
          import("./features/settings/settings.module").then(
            (m) => m.SettingsModule
          ),
        canActivate: [PermissionGuard],
        data: {
          preload: false,
          breadcrumb: "Settings",
          permissions: ["settings.view"],
        },
      },
    ],
  },

  // Error routes
  {
    path: "error",
    component: ErrorLayoutComponent,
    children: [
      {
        path: "403",
        component: ForbiddenComponent,
      },
      {
        path: "404",
        component: NotFoundComponent,
      },
      {
        path: "500",
        component: ServerErrorComponent,
      },
    ],
  },

  // Fallback
  {
    path: "**",
    redirectTo: "/error/404",
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      // Performance optimizations
      enableTracing: false, // Set to true for debugging
      preloadingStrategy: CustomPreloadingStrategy,

      // Router configuration
      paramsInheritanceStrategy: "always",
      relativeLinkResolution: "legacy",
      malformedUriErrorHandler: (error, router, serializer) => {
        console.error("Malformed URI:", error);
        return router.parseUrl("/error/404");
      },

      // Scroll behavior
      scrollPositionRestoration: "top",
      anchorScrolling: "enabled",

      // Route reuse strategy for performance
      onSameUrlNavigation: "reload",
    }),
  ],
  exports: [RouterModule],
  providers: [CustomPreloadingStrategy],
})
export class AppRoutingModule {}

// Custom preloading strategy
@Injectable()
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: Function): Observable<any> {
    if (route.data && route.data["preload"]) {
      console.log("🚀 Preloading module:", route.path);
      return load();
    }
    return of(null);
  }
}
```

---

## 🎯 **Performance & Scalability Patterns**

### **1. 🚀 Lazy Loading & Code Splitting**

```typescript
// Dynamic imports for better bundle splitting
export class FeatureLazyLoader {
  // ✅ Lazy load heavy libraries
  async loadChartLibrary() {
    const chartModule = await import("chart.js");
    return chartModule.Chart;
  }

  // ✅ Lazy load feature components
  async loadAdvancedReportsComponent() {
    const { AdvancedReportsComponent } = await import(
      "./advanced-reports/advanced-reports.component"
    );
    return AdvancedReportsComponent;
  }

  // ✅ Conditional loading based on user permissions
  async loadAdminComponents(userPermissions: string[]) {
    if (userPermissions.includes("admin.access")) {
      const { AdminModule } = await import("./admin/admin.module");
      return AdminModule;
    }
    return null;
  }
}

// Component-level lazy loading
@Component({
  template: `
    <div>
      <button (click)="loadHeavyComponent()">Load Heavy Component</button>
      <ng-container *ngIf="heavyComponent">
        <ng-container *ngComponentOutlet="heavyComponent"></ng-container>
      </ng-container>
    </div>
  `,
})
export class LazyComponentHostComponent {
  heavyComponent: any = null;

  constructor(
    private componentFactoryResolver: ComponentFactoryResolver,
    private lazyLoader: FeatureLazyLoader
  ) {}

  async loadHeavyComponent() {
    if (!this.heavyComponent) {
      this.heavyComponent =
        await this.lazyLoader.loadAdvancedReportsComponent();
    }
  }
}
```

### **2. 📊 Memory Management & Resource Cleanup**

```typescript
// Base component with automatic cleanup
export abstract class BaseComponent implements OnDestroy {
  protected destroy$ = new Subject<void>();
  private subscriptions: Subscription[] = [];
  private intervals: number[] = [];
  private timeouts: number[] = [];

  constructor() {
    // Automatic cleanup when component is destroyed
  }

  ngOnDestroy(): void {
    console.log(`🧹 Cleaning up ${this.constructor.name}`);

    // Clean up observables
    this.destroy$.next();
    this.destroy$.complete();

    // Clean up manual subscriptions
    this.subscriptions.forEach((sub) => {
      if (!sub.closed) {
        sub.unsubscribe();
      }
    });

    // Clean up intervals
    this.intervals.forEach((interval) => clearInterval(interval));

    // Clean up timeouts
    this.timeouts.forEach((timeout) => clearTimeout(timeout));
  }

  // Helper methods for tracked subscriptions
  protected addSubscription(subscription: Subscription): void {
    this.subscriptions.push(subscription);
  }

  protected addInterval(callback: () => void, ms: number): number {
    const interval = window.setInterval(callback, ms);
    this.intervals.push(interval);
    return interval;
  }

  protected addTimeout(callback: () => void, ms: number): number {
    const timeout = window.setTimeout(callback, ms);
    this.timeouts.push(timeout);
    return timeout;
  }
}

// Usage in feature components
@Component({
  selector: "app-user-dashboard",
  template: `
    <div class="dashboard">
      <h1>User Dashboard</h1>
      <app-real-time-stats [data]="stats$ | async"></app-real-time-stats>
    </div>
  `,
})
export class UserDashboardComponent extends BaseComponent implements OnInit {
  stats$ = this.userService.getRealtimeStats().pipe(
    takeUntil(this.destroy$) // Automatic cleanup!
  );

  constructor(
    private userService: UserService,
    private websocketService: WebSocketService
  ) {
    super();
  }

  ngOnInit(): void {
    // Set up periodic data refresh
    this.addInterval(() => {
      this.userService.refreshStats();
    }, 30000); // Every 30 seconds

    // Manual subscription with tracking
    const socketSub = this.websocketService
      .connect("user-updates")
      .subscribe((update) => {
        console.log("User update:", update);
      });

    this.addSubscription(socketSub);
  }
}
```

### **3. 🔧 Configuration Management**

```typescript
// Environment-specific configuration
export interface AppConfig {
  production: boolean;
  apiUrl: string;
  wsUrl: string;
  features: {
    billing: boolean;
    reports: boolean;
    analytics: boolean;
  };
  limits: {
    maxFileSize: number;
    maxConcurrentRequests: number;
    sessionTimeoutMinutes: number;
  };
  integrations: {
    stripe: {
      publishableKey: string;
    };
    analytics: {
      trackingId: string;
    };
  };
}

// Runtime configuration service
@Injectable({
  providedIn: "root",
})
export class ConfigService {
  private config!: AppConfig;

  async loadConfig(): Promise<AppConfig> {
    try {
      // Load runtime configuration from server
      const response = await fetch("/api/config");
      const runtimeConfig = await response.json();

      // Merge with compile-time config
      this.config = {
        ...environment,
        ...runtimeConfig,
      };

      return this.config;
    } catch (error) {
      console.error("Failed to load runtime config:", error);
      // Fallback to compile-time config
      this.config = environment as AppConfig;
      return this.config;
    }
  }

  get<T>(path: string): T {
    return this.getNestedProperty(this.config, path);
  }

  getFeatureFlag(feature: string): boolean {
    return this.config.features[feature] || false;
  }

  private getNestedProperty(obj: any, path: string): any {
    return path.split(".").reduce((current, prop) => current?.[prop], obj);
  }
}

// App initialization factory
export function appInitializerFactory(
  configService: ConfigService
): () => Promise<any> {
  return () => configService.loadConfig();
}

// app.module.ts
@NgModule({
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: appInitializerFactory,
      deps: [ConfigService],
      multi: true,
    },
  ],
})
export class AppModule {}
```

---

## 🎉 **Summary: Enterprise Architecture Mastery**

### **🏗️ What You've Mastered:**

#### **🏛️ Structural Patterns:**

✅ **Domain-Driven Design** - Organize by business domains, not technical layers  
✅ **Module Federation** - Micro-frontend architecture for team autonomy  
✅ **Core/Shared/Feature** - Clear separation of concerns and responsibilities  
✅ **Lazy Loading** - Performance optimization through code splitting

#### **🔧 Implementation Patterns:**

✅ **Singleton Services** - Core module pattern prevents duplicate imports  
✅ **State Management** - NgRx with feature stores and global state  
✅ **Configuration** - Runtime config loading and feature flags  
✅ **Memory Management** - Automatic cleanup and resource management

#### **🚀 Scalability Features:**

✅ **Design System** - Consistent UI components across teams  
✅ **Route Strategies** - Smart preloading and permission-based routing  
✅ **Error Boundaries** - Graceful error handling and recovery  
✅ **Performance** - Bundle optimization and efficient data flow

### **💡 Key Architectural Principles:**

1. **🎯 Single Responsibility** - Each module has one reason to change
2. **🔒 Dependency Inversion** - Depend on abstractions, not concretions
3. **📦 Encapsulation** - Hide implementation details, expose clean APIs
4. **🔄 Loose Coupling** - Modules interact through well-defined interfaces
5. **⚡ Performance First** - Lazy load everything, optimize bundle sizes

### **🎭 When to Use Each Pattern:**

- **🏢 Large Enterprise (100+ devs)**: Full DDD + Module Federation
- **🚀 Growing Startup (20-50 devs)**: Core/Shared/Feature pattern
- **👥 Small Team (5-20 devs)**: Simplified feature modules
- **🔧 Proof of Concept**: Start simple, refactor as you grow

**Remember**: Great architecture evolves with your team and requirements - start simple and add complexity only when needed! 🌟
