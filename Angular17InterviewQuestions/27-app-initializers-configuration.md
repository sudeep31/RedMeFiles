# 🚀 **Angular App Initializers Configuration**

## 🎯 **What You'll Learn**

Master **App Initializers** - the special services that run during app startup! These are like your app's "morning routine" that happens before anything else loads. Perfect for loading configuration, checking authentication, or setting up critical services.

---

## 📚 **The Basics (Start Here If You're New)**

### **What Are App Initializers? 🤔**

Think of App Initializers like **your morning checklist before work**:

- ☕ Make coffee (load configuration)
- 📧 Check important emails (verify authentication)
- 🌡️ Check weather (fetch critical data)
- 🚗 Start car (initialize services)

**Only after completing this checklist do you actually start your workday!**

```typescript
// Think of it like this morning routine
async function morningRoutine() {
  await makeCoffee(); // ← App Initializer 1
  await checkEmails(); // ← App Initializer 2
  await checkWeather(); // ← App Initializer 3

  // NOW we can start the actual day (render Angular app)
  startWorkDay();
}
```

---

## 🛠️ **How App Initializers Work**

### **1. 🎯 Basic App Initializer Setup**

```typescript
// src/app/app.config.ts - Basic Configuration
import { ApplicationConfig, APP_INITIALIZER } from "@angular/core";
import { ConfigurationService } from "./services/configuration.service";

// Simple function that runs at startup
function initializeApp(
  configService: ConfigurationService
): () => Promise<void> {
  return () => configService.loadConfiguration();
}

export const appConfig: ApplicationConfig = {
  providers: [
    // Other providers...

    // 🚀 Register App Initializer
    {
      provide: APP_INITIALIZER,
      useFactory: initializeApp,
      deps: [ConfigurationService],
      multi: true, // ← Important! Allows multiple initializers
    },
  ],
};
```

### **2. 📋 Configuration Service Implementation**

```typescript
// src/app/services/configuration.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { BehaviorSubject, Observable } from "rxjs";
import { tap, catchError } from "rxjs/operators";

export interface AppConfiguration {
  apiBaseUrl: string;
  featureFlags: {
    enableNewDashboard: boolean;
    enableBetaFeatures: boolean;
    maintenanceMode: boolean;
  };
  theme: {
    primaryColor: string;
    secondaryColor: string;
    darkMode: boolean;
  };
  integrations: {
    googleAnalyticsId?: string;
    stripePublishableKey?: string;
    mapboxApiKey?: string;
  };
  limits: {
    maxFileUploadSize: number;
    requestTimeoutMs: number;
    maxRetryAttempts: number;
  };
}

@Injectable({
  providedIn: "root",
})
export class ConfigurationService {
  private config$ = new BehaviorSubject<AppConfiguration | null>(null);
  private isLoaded = false;

  constructor(private http: HttpClient) {}

  // 🔄 LOAD CONFIGURATION: Main initializer method
  async loadConfiguration(): Promise<void> {
    console.log("🚀 Loading app configuration...");

    try {
      // Load from multiple sources
      const [envConfig, userPrefs, featureFlags] = await Promise.all([
        this.loadEnvironmentConfig(),
        this.loadUserPreferences(),
        this.loadFeatureFlags(),
      ]);

      // Merge all configurations
      const finalConfig: AppConfiguration = {
        ...envConfig,
        featureFlags: {
          ...envConfig.featureFlags,
          ...featureFlags,
        },
        theme: {
          ...envConfig.theme,
          ...userPrefs.theme,
        },
      };

      // Store configuration
      this.config$.next(finalConfig);
      this.isLoaded = true;

      console.log("✅ Configuration loaded successfully:", finalConfig);

      // Apply immediate configuration
      this.applyGlobalConfiguration(finalConfig);
    } catch (error) {
      console.error("❌ Failed to load configuration:", error);

      // Load fallback configuration
      this.loadFallbackConfiguration();
    }
  }

  // 📡 GET CONFIGURATION: Access loaded config
  getConfiguration(): Observable<AppConfiguration | null> {
    return this.config$.asObservable();
  }

  // ✅ IS READY: Check if configuration is loaded
  isConfigurationLoaded(): boolean {
    return this.isLoaded && this.config$.value !== null;
  }

  // 🎯 GET SPECIFIC VALUES: Helper methods
  getApiBaseUrl(): string {
    const config = this.config$.value;
    return config?.apiBaseUrl || "https://api.fallback.com";
  }

  isFeatureEnabled(
    featureName: keyof AppConfiguration["featureFlags"]
  ): boolean {
    const config = this.config$.value;
    return config?.featureFlags[featureName] || false;
  }

  getTheme(): AppConfiguration["theme"] {
    const config = this.config$.value;
    return (
      config?.theme || {
        primaryColor: "#007bff",
        secondaryColor: "#6c757d",
        darkMode: false,
      }
    );
  }

  // 🔍 Private Methods

  private async loadEnvironmentConfig(): Promise<AppConfiguration> {
    const response = await this.http
      .get<AppConfiguration>("/api/config/environment")
      .pipe(
        catchError(() => {
          console.warn("Environment config not available, using defaults");
          return Promise.resolve(this.getDefaultConfiguration());
        })
      )
      .toPromise();

    return response || this.getDefaultConfiguration();
  }

  private async loadUserPreferences(): Promise<Partial<AppConfiguration>> {
    try {
      // Try to load from localStorage first
      const saved = localStorage.getItem("userPreferences");
      if (saved) {
        return JSON.parse(saved);
      }

      // Fallback to API
      const response = await this.http
        .get<Partial<AppConfiguration>>("/api/user/preferences")
        .toPromise();

      return response || {};
    } catch (error) {
      console.warn("User preferences not available:", error);
      return {};
    }
  }

  private async loadFeatureFlags(): Promise<
    Partial<AppConfiguration["featureFlags"]>
  > {
    try {
      const response = await this.http
        .get<AppConfiguration["featureFlags"]>("/api/config/features")
        .toPromise();

      return response || {};
    } catch (error) {
      console.warn("Feature flags not available:", error);
      return {};
    }
  }

  private applyGlobalConfiguration(config: AppConfiguration): void {
    // Apply theme
    this.applyThemeConfiguration(config.theme);

    // Set up global error handling
    this.setupGlobalErrorHandling(config);

    // Configure analytics if enabled
    if (config.integrations.googleAnalyticsId) {
      this.initializeAnalytics(config.integrations.googleAnalyticsId);
    }

    // Apply maintenance mode if needed
    if (config.featureFlags.maintenanceMode) {
      this.showMaintenanceNotice();
    }
  }

  private applyThemeConfiguration(theme: AppConfiguration["theme"]): void {
    // Apply CSS custom properties
    const root = document.documentElement;
    root.style.setProperty("--primary-color", theme.primaryColor);
    root.style.setProperty("--secondary-color", theme.secondaryColor);

    // Apply dark mode class
    if (theme.darkMode) {
      document.body.classList.add("dark-theme");
    } else {
      document.body.classList.remove("dark-theme");
    }
  }

  private setupGlobalErrorHandling(config: AppConfiguration): void {
    // Configure global HTTP timeouts
    console.log(
      `Setting request timeout to ${config.limits.requestTimeoutMs}ms`
    );

    // This would typically be done in an HTTP interceptor
    // but we're showing the concept here
  }

  private initializeAnalytics(analyticsId: string): void {
    // Initialize Google Analytics or other tracking
    console.log(`Initializing analytics with ID: ${analyticsId}`);

    // In real implementation, you'd load the analytics script
    // and configure it with the provided ID
  }

  private showMaintenanceNotice(): void {
    // Show maintenance mode notice
    console.log("🚨 Application is in maintenance mode");

    // Could show a global banner or redirect to maintenance page
  }

  private loadFallbackConfiguration(): void {
    console.log("📦 Loading fallback configuration");

    const fallbackConfig = this.getDefaultConfiguration();
    this.config$.next(fallbackConfig);
    this.isLoaded = true;

    this.applyGlobalConfiguration(fallbackConfig);
  }

  private getDefaultConfiguration(): AppConfiguration {
    return {
      apiBaseUrl: window.location.origin + "/api",
      featureFlags: {
        enableNewDashboard: false,
        enableBetaFeatures: false,
        maintenanceMode: false,
      },
      theme: {
        primaryColor: "#007bff",
        secondaryColor: "#6c757d",
        darkMode: false,
      },
      integrations: {},
      limits: {
        maxFileUploadSize: 10 * 1024 * 1024, // 10MB
        requestTimeoutMs: 30000, // 30 seconds
        maxRetryAttempts: 3,
      },
    };
  }
}
```

---

## 🔥 **Advanced Multi-Initializer Setup**

### **Multiple App Initializers with Dependencies 🎯**

```typescript
// src/app/app.config.ts - Advanced Configuration
import { ApplicationConfig, APP_INITIALIZER } from "@angular/core";
import { ConfigurationService } from "./services/configuration.service";
import { AuthenticationService } from "./services/authentication.service";
import { CacheService } from "./services/cache.service";
import { FeatureFlagService } from "./services/feature-flag.service";

// 1️⃣ Configuration Initializer (runs first)
function initializeConfiguration(
  configService: ConfigurationService
): () => Promise<void> {
  return async () => {
    console.log("1️⃣ Initializing configuration...");
    await configService.loadConfiguration();
  };
}

// 2️⃣ Authentication Initializer (runs after config)
function initializeAuthentication(
  authService: AuthenticationService,
  configService: ConfigurationService
): () => Promise<void> {
  return async () => {
    console.log("2️⃣ Initializing authentication...");

    // Wait for config to be ready
    if (!configService.isConfigurationLoaded()) {
      throw new Error("Configuration must be loaded before authentication");
    }

    await authService.initializeAuth();
  };
}

// 3️⃣ Cache Initializer (runs in parallel with others)
function initializeCache(cacheService: CacheService): () => Promise<void> {
  return async () => {
    console.log("3️⃣ Initializing cache...");
    await cacheService.initializeCache();
  };
}

// 4️⃣ Feature Flags Initializer (depends on auth and config)
function initializeFeatureFlags(
  featureFlagService: FeatureFlagService,
  authService: AuthenticationService,
  configService: ConfigurationService
): () => Promise<void> {
  return async () => {
    console.log("4️⃣ Initializing feature flags...");

    // Ensure dependencies are ready
    if (
      !configService.isConfigurationLoaded() ||
      !authService.isInitialized()
    ) {
      throw new Error("Config and Auth must be ready before feature flags");
    }

    await featureFlagService.loadUserSpecificFlags();
  };
}

export const appConfig: ApplicationConfig = {
  providers: [
    // Services
    ConfigurationService,
    AuthenticationService,
    CacheService,
    FeatureFlagService,

    // 📋 App Initializers (order matters!)
    {
      provide: APP_INITIALIZER,
      useFactory: initializeConfiguration,
      deps: [ConfigurationService],
      multi: true,
    },
    {
      provide: APP_INITIALIZER,
      useFactory: initializeAuthentication,
      deps: [AuthenticationService, ConfigurationService],
      multi: true,
    },
    {
      provide: APP_INITIALIZER,
      useFactory: initializeCache,
      deps: [CacheService],
      multi: true,
    },
    {
      provide: APP_INITIALIZER,
      useFactory: initializeFeatureFlags,
      deps: [FeatureFlagService, AuthenticationService, ConfigurationService],
      multi: true,
    },
  ],
};
```

### **Complex Authentication Initializer 🔐**

```typescript
// src/app/services/authentication.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { BehaviorSubject } from "rxjs";
import { ConfigurationService } from "./configuration.service";

export interface UserProfile {
  id: string;
  email: string;
  name: string;
  roles: string[];
  preferences: Record<string, any>;
  lastLoginAt: string;
}

export interface AuthState {
  isAuthenticated: boolean;
  user: UserProfile | null;
  token: string | null;
  refreshToken: string | null;
}

@Injectable({
  providedIn: "root",
})
export class AuthenticationService {
  private authState$ = new BehaviorSubject<AuthState>({
    isAuthenticated: false,
    user: null,
    token: null,
    refreshToken: null,
  });

  private initialized = false;

  constructor(
    private http: HttpClient,
    private configService: ConfigurationService
  ) {}

  // 🚀 INITIALIZE AUTH: Main initializer method
  async initializeAuth(): Promise<void> {
    console.log("🔐 Initializing authentication...");

    try {
      // 1. Check for existing session
      const existingToken = this.getStoredToken();

      if (existingToken) {
        console.log("🔍 Found existing token, validating...");
        await this.validateExistingSession(existingToken);
      } else {
        console.log("🆕 No existing session found");
      }

      // 2. Set up token refresh mechanism
      this.setupTokenRefresh();

      // 3. Set up session timeout monitoring
      this.setupSessionMonitoring();

      this.initialized = true;
      console.log("✅ Authentication initialization complete");
    } catch (error) {
      console.error("❌ Authentication initialization failed:", error);

      // Clear any invalid stored data
      this.clearStoredAuth();

      // Continue with unauthenticated state
      this.initialized = true;
    }
  }

  // ✅ IS INITIALIZED: Check if auth is ready
  isInitialized(): boolean {
    return this.initialized;
  }

  // 📊 GET AUTH STATE: Access current auth state
  getAuthState() {
    return this.authState$.asObservable();
  }

  // 🔍 IS AUTHENTICATED: Check if user is authenticated
  isAuthenticated(): boolean {
    return this.authState$.value.isAuthenticated;
  }

  // 👤 GET USER: Get current user profile
  getCurrentUser(): UserProfile | null {
    return this.authState$.value.user;
  }

  // 🎯 LOGIN: Authenticate user
  async login(email: string, password: string): Promise<UserProfile> {
    try {
      const response = await this.http
        .post<{
          user: UserProfile;
          token: string;
          refreshToken: string;
        }>("/api/auth/login", { email, password })
        .toPromise();

      if (!response) {
        throw new Error("Invalid login response");
      }

      // Store auth data
      this.storeAuthData(response.token, response.refreshToken);

      // Update state
      this.updateAuthState({
        isAuthenticated: true,
        user: response.user,
        token: response.token,
        refreshToken: response.refreshToken,
      });

      console.log("✅ Login successful");
      return response.user;
    } catch (error) {
      console.error("❌ Login failed:", error);
      throw error;
    }
  }

  // 🚪 LOGOUT: Clear authentication
  async logout(): Promise<void> {
    try {
      // Notify server
      if (this.authState$.value.token) {
        await this.http.post("/api/auth/logout", {}).toPromise();
      }
    } catch (error) {
      console.warn("Logout API call failed:", error);
    } finally {
      // Always clear local state
      this.clearStoredAuth();
      this.updateAuthState({
        isAuthenticated: false,
        user: null,
        token: null,
        refreshToken: null,
      });

      console.log("✅ Logout complete");
    }
  }

  // 🔄 Private Methods

  private async validateExistingSession(token: string): Promise<void> {
    try {
      const response = await this.http
        .get<{ user: UserProfile }>("/api/auth/validate", {
          headers: { Authorization: `Bearer ${token}` },
        })
        .toPromise();

      if (response?.user) {
        const refreshToken = this.getStoredRefreshToken();

        this.updateAuthState({
          isAuthenticated: true,
          user: response.user,
          token,
          refreshToken,
        });

        console.log("✅ Existing session validated");
      } else {
        throw new Error("Invalid session response");
      }
    } catch (error) {
      console.warn("❌ Session validation failed:", error);
      this.clearStoredAuth();
      throw error;
    }
  }

  private setupTokenRefresh(): void {
    // Set up automatic token refresh 5 minutes before expiry
    setInterval(() => {
      if (this.isAuthenticated()) {
        this.refreshTokenIfNeeded();
      }
    }, 5 * 60 * 1000); // Check every 5 minutes
  }

  private async refreshTokenIfNeeded(): Promise<void> {
    const refreshToken = this.authState$.value.refreshToken;
    if (!refreshToken) return;

    try {
      const response = await this.http
        .post<{
          token: string;
          refreshToken: string;
        }>("/api/auth/refresh", { refreshToken })
        .toPromise();

      if (response) {
        // Update stored tokens
        this.storeAuthData(response.token, response.refreshToken);

        // Update state
        const currentState = this.authState$.value;
        this.updateAuthState({
          ...currentState,
          token: response.token,
          refreshToken: response.refreshToken,
        });

        console.log("🔄 Token refreshed successfully");
      }
    } catch (error) {
      console.error("❌ Token refresh failed:", error);

      // Force logout on refresh failure
      await this.logout();
    }
  }

  private setupSessionMonitoring(): void {
    // Monitor for session timeouts
    let lastActivity = Date.now();

    // Track user activity
    ["click", "keydown", "mousemove", "scroll"].forEach((event) => {
      document.addEventListener(
        event,
        () => {
          lastActivity = Date.now();
        },
        true
      );
    });

    // Check for inactivity every minute
    setInterval(() => {
      if (this.isAuthenticated()) {
        const inactiveTime = Date.now() - lastActivity;
        const maxInactiveTime = 30 * 60 * 1000; // 30 minutes

        if (inactiveTime > maxInactiveTime) {
          console.warn("⏰ Session timeout due to inactivity");
          this.logout();
        }
      }
    }, 60 * 1000); // Check every minute
  }

  private updateAuthState(newState: AuthState): void {
    this.authState$.next(newState);
  }

  private storeAuthData(token: string, refreshToken: string): void {
    try {
      localStorage.setItem("auth_token", token);
      localStorage.setItem("refresh_token", refreshToken);
    } catch (error) {
      console.warn("Failed to store auth data:", error);
    }
  }

  private getStoredToken(): string | null {
    try {
      return localStorage.getItem("auth_token");
    } catch (error) {
      return null;
    }
  }

  private getStoredRefreshToken(): string | null {
    try {
      return localStorage.getItem("refresh_token");
    } catch (error) {
      return null;
    }
  }

  private clearStoredAuth(): void {
    try {
      localStorage.removeItem("auth_token");
      localStorage.removeItem("refresh_token");
    } catch (error) {
      console.warn("Failed to clear stored auth:", error);
    }
  }
}
```

---

## 🎯 **Real-World Usage Examples**

### **1. 🏢 Enterprise App Configuration**

```typescript
// src/app/initializers/enterprise.initializer.ts
import { APP_INITIALIZER } from "@angular/core";
import { ConfigurationService } from "../services/configuration.service";
import { LicenseService } from "../services/license.service";
import { TenantService } from "../services/tenant.service";
import { SecurityService } from "../services/security.service";

// Multi-tenant enterprise initialization
export function enterpriseInitializer(
  configService: ConfigurationService,
  licenseService: LicenseService,
  tenantService: TenantService,
  securityService: SecurityService
) {
  return async (): Promise<void> => {
    console.log("🏢 Starting enterprise initialization...");

    try {
      // 1. Load base configuration
      await configService.loadConfiguration();

      // 2. Determine tenant from URL/subdomain
      const tenantInfo = await tenantService.resolveTenant();

      // 3. Load tenant-specific configuration
      await configService.loadTenantConfiguration(tenantInfo.tenantId);

      // 4. Validate license
      const licenseValid = await licenseService.validateLicense(
        tenantInfo.tenantId
      );
      if (!licenseValid) {
        throw new Error("Invalid or expired license");
      }

      // 5. Initialize security policies
      await securityService.loadSecurityPolicies(tenantInfo.tenantId);

      // 6. Apply tenant branding
      await tenantService.applyTenantBranding(tenantInfo);

      console.log("✅ Enterprise initialization complete");
    } catch (error) {
      console.error("❌ Enterprise initialization failed:", error);

      // Redirect to error page or show maintenance message
      window.location.href = "/maintenance";
    }
  };
}

// Register in app.config.ts
export const enterpriseAppConfig: ApplicationConfig = {
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: enterpriseInitializer,
      deps: [
        ConfigurationService,
        LicenseService,
        TenantService,
        SecurityService,
      ],
      multi: true,
    },
  ],
};
```

### **2. 📱 PWA Initialization**

```typescript
// src/app/initializers/pwa.initializer.ts
import { APP_INITIALIZER } from "@angular/core";
import { ServiceWorkerService } from "../services/service-worker.service";
import { NotificationService } from "../services/notification.service";
import { OfflineService } from "../services/offline.service";

export function pwaInitializer(
  swService: ServiceWorkerService,
  notificationService: NotificationService,
  offlineService: OfflineService
) {
  return async (): Promise<void> => {
    console.log("📱 Initializing PWA features...");

    try {
      // 1. Register service worker
      await swService.register();

      // 2. Set up offline detection
      offlineService.initialize();

      // 3. Request notification permissions
      if ("Notification" in window) {
        await notificationService.requestPermission();
      }

      // 4. Preload critical resources
      await swService.preloadCriticalResources([
        "/assets/offline.html",
        "/assets/icons/icon-192x192.png",
        "/api/config/offline",
      ]);

      // 5. Set up background sync
      await swService.setupBackgroundSync();

      console.log("✅ PWA initialization complete");
    } catch (error) {
      console.warn("⚠️ Some PWA features may not be available:", error);
      // Continue without PWA features
    }
  };
}
```

---

## ⚡ **Performance Tips & Best Practices**

### **🎯 Do's and Don'ts**

#### **✅ DO:**

- **Keep initializers fast** - Users are waiting for your app to start
- **Handle errors gracefully** - Don't break the entire app
- **Use Promise.all()** for parallel operations
- **Provide fallback configurations**
- **Log initialization progress** for debugging

#### **❌ DON'T:**

- **Make unnecessary HTTP calls** - Cache what you can
- **Block on non-critical operations** - Some things can load later
- **Forget error handling** - Always have a fallback plan
- **Load heavy resources** - Keep it lightweight
- **Chain too many dependencies** - Parallel is better

### **📊 Performance Optimization**

```typescript
// Optimized parallel initialization
export function optimizedInitializer(
  configService: ConfigurationService,
  cacheService: CacheService,
  analyticsService: AnalyticsService
) {
  return async (): Promise<void> => {
    console.log("⚡ Starting optimized initialization...");

    // 🚀 Run critical operations in parallel
    const criticalOperations = await Promise.allSettled([
      configService.loadEssentialConfig(),
      cacheService.initializeCache(),
    ]);

    // Check if critical operations succeeded
    const configResult = criticalOperations[0];
    if (configResult.status === "rejected") {
      console.error("Critical config load failed:", configResult.reason);
      // Load minimal fallback config
      await configService.loadFallbackConfig();
    }

    // 🎯 Run non-critical operations (don't await)
    Promise.allSettled([
      analyticsService.initialize(),
      configService.loadUserPreferences(),
      cacheService.warmupCache(),
    ])
      .then((results) => {
        console.log("Non-critical initializations complete:", results);
      })
      .catch((error) => {
        console.warn("Some non-critical initializations failed:", error);
      });

    console.log("✅ Core initialization complete");
  };
}
```

---

## 🎉 **Summary: App Initializer Mastery**

You now know how to:

### **🏗️ What You've Mastered:**

✅ **Basic App Initializers** - Simple startup configuration  
✅ **Complex Multi-Initializers** - Coordinated startup sequence  
✅ **Authentication Integration** - Secure app initialization  
✅ **Enterprise Patterns** - Multi-tenant and licensed apps  
✅ **PWA Features** - Modern web app capabilities  
✅ **Performance Optimization** - Fast and reliable startup

### **🚀 Real-World Benefits:**

- **Faster app startup** with optimized initialization
- **Better user experience** with proper loading states
- **More reliable apps** with fallback configurations
- **Enterprise-ready** with security and licensing
- **Modern capabilities** with PWA features

### **🎯 When to Use App Initializers:**

- Loading critical configuration from APIs
- Authenticating users before app starts
- Setting up third-party services (analytics, monitoring)
- Initializing caching and offline capabilities
- Applying tenant-specific branding and configuration
- Validating licenses and security policies

**Remember**: App Initializers are your app's foundation - make them solid, fast, and reliable! 🌟
