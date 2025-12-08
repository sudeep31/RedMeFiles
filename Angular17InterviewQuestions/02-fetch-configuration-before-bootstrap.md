# 🚀 Fetching Configuration Data Before Angular Bootstrap

## 🎯 **Question Overview**

_"In Angular, I want to fetch a configuration data (API, URL, etc.) before my root component loads (bootstrap stage). How can I achieve?"_

## 🔍 **Understanding the Problem**

When building enterprise applications, you often need to:

- Load environment-specific configurations
- Fetch API endpoints from a remote source
- Initialize authentication tokens
- Load feature flags or user permissions
- Configure third-party libraries

All of this must happen **before** Angular starts rendering components, ensuring your app has all necessary configuration data from the very beginning.

## 🛠️ **Solution Approaches**

### **1. APP_INITIALIZER (Recommended Approach)**

The `APP_INITIALIZER` token is the most robust way to execute initialization logic before Angular bootstrap.

#### **Basic Implementation**

```typescript
// config.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class ConfigService {
  private config: any = {};

  constructor(private http: HttpClient) {}

  loadConfig(): Observable<any> {
    return this.http.get("/assets/config/app-config.json");
  }

  getConfig(key: string): any {
    return this.config[key];
  }

  setConfig(config: any): void {
    this.config = config;
  }
}
```

#### **Factory Function**

```typescript
// app-init.factory.ts
import { ConfigService } from "./config.service";

export function initializeApp(
  configService: ConfigService
): () => Promise<any> {
  return (): Promise<any> => {
    return new Promise((resolve, reject) => {
      configService.loadConfig().subscribe({
        next: (config) => {
          configService.setConfig(config);
          console.log("✅ Configuration loaded successfully:", config);
          resolve(config);
        },
        error: (error) => {
          console.error("❌ Failed to load configuration:", error);
          reject(error);
        },
      });
    });
  };
}
```

#### **Provider Configuration**

```typescript
// app.config.ts (Angular 17+ Standalone)
import { ApplicationConfig, importProvidersFrom } from "@angular/core";
import { provideRouter } from "@angular/router";
import { provideHttpClient } from "@angular/common/http";
import { APP_INITIALIZER } from "@angular/core";
import { ConfigService } from "./services/config.service";
import { initializeApp } from "./factories/app-init.factory";
import { routes } from "./app.routes";

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    {
      provide: APP_INITIALIZER,
      useFactory: initializeApp,
      deps: [ConfigService],
      multi: true,
    },
  ],
};
```

#### **For NgModule-based Apps**

```typescript
// app.module.ts
import { NgModule, APP_INITIALIZER } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { HttpClientModule } from "@angular/common/http";
import { ConfigService } from "./services/config.service";
import { initializeApp } from "./factories/app-init.factory";
import { AppComponent } from "./app.component";

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: initializeApp,
      deps: [ConfigService],
      multi: true,
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### **2. Advanced Configuration with Multiple Initializers**

You can have multiple APP_INITIALIZER providers for different initialization tasks:

```typescript
// Multiple initializers
export const appConfig: ApplicationConfig = {
  providers: [
    // ... other providers
    {
      provide: APP_INITIALIZER,
      useFactory: initializeConfig,
      deps: [ConfigService],
      multi: true,
    },
    {
      provide: APP_INITIALIZER,
      useFactory: initializeAuth,
      deps: [AuthService, ConfigService],
      multi: true,
    },
    {
      provide: APP_INITIALIZER,
      useFactory: initializeFeatureFlags,
      deps: [FeatureFlagService],
      multi: true,
    },
  ],
};
```

### **3. Environment-Specific Configuration**

```typescript
// environment.ts
export const environment = {
  production: false,
  configUrl: "/assets/config/dev-config.json",
};

// environment.prod.ts
export const environment = {
  production: true,
  configUrl: "https://api.myapp.com/config",
};
```

```typescript
// config.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { environment } from "../environments/environment";

@Injectable({
  providedIn: "root",
})
export class ConfigService {
  private config: any = {};

  constructor(private http: HttpClient) {}

  async loadConfig(): Promise<any> {
    try {
      const config = await this.http.get(environment.configUrl).toPromise();
      this.config = config;
      return config;
    } catch (error) {
      console.error("Failed to load configuration:", error);
      throw error;
    }
  }

  get apiUrl(): string {
    return this.config.apiUrl || "http://localhost:3000/api";
  }

  get features(): any {
    return this.config.features || {};
  }
}
```

### **4. Complex Initialization with Dependencies**

```typescript
// auth-init.factory.ts
export function initializeAuth(
  authService: AuthService,
  configService: ConfigService
): () => Promise<any> {
  return async (): Promise<any> => {
    // Wait for config to be loaded first
    const config = configService.getConfig("auth");

    if (config?.autoLogin) {
      try {
        await authService.initializeFromStorage();
        console.log("✅ Auto-login completed");
      } catch (error) {
        console.warn("⚠️ Auto-login failed:", error);
      }
    }
  };
}
```

## 🚨 **Common Pitfalls & Best Practices**

### **❌ Common Mistakes**

1. **Not returning a Promise**: APP_INITIALIZER expects a function that returns a Promise
2. **Forgetting `multi: true`**: Without this, you can only have one initializer
3. **Not handling errors**: Failed initialization can prevent app bootstrap

### **✅ Best Practices**

1. **Always handle errors gracefully**
2. **Use loading indicators during initialization**
3. **Cache configuration data appropriately**
4. **Make initialization idempotent**

```typescript
// Enhanced factory with error handling
export function initializeApp(
  configService: ConfigService
): () => Promise<any> {
  return (): Promise<any> => {
    return configService
      .loadConfig()
      .toPromise()
      .catch((error) => {
        console.error("Configuration loading failed, using defaults:", error);
        // Return default configuration instead of failing
        return { apiUrl: "http://localhost:3000", features: {} };
      })
      .then((config) => {
        configService.setConfig(config);
        return config;
      });
  };
}
```

## 🎨 **Real-World Example: Multi-Tenant Application**

```typescript
// tenant-config.service.ts
@Injectable({
  providedIn: "root",
})
export class TenantConfigService {
  private tenantConfig: any = {};

  constructor(private http: HttpClient) {}

  async loadTenantConfig(): Promise<any> {
    const hostname = window.location.hostname;
    const subdomain = hostname.split(".")[0];

    try {
      const config = await this.http
        .get(`/api/tenant/${subdomain}/config`)
        .toPromise();
      this.tenantConfig = config;

      // Apply tenant-specific theme
      this.applyTheme(config.theme);

      return config;
    } catch (error) {
      console.error("Failed to load tenant configuration:", error);
      throw error;
    }
  }

  private applyTheme(theme: any): void {
    if (theme?.primaryColor) {
      document.documentElement.style.setProperty(
        "--primary-color",
        theme.primaryColor
      );
    }
  }

  getTenantFeatures(): string[] {
    return this.tenantConfig.features || [];
  }
}
```

## 🔧 **Testing APP_INITIALIZER**

```typescript
// config.service.spec.ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { ConfigService } from "./config.service";
import { initializeApp } from "./app-init.factory";

describe("App Initialization", () => {
  let service: ConfigService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ConfigService],
    });

    service = TestBed.inject(ConfigService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("should load configuration successfully", async () => {
    const mockConfig = { apiUrl: "https://api.test.com" };
    const initFunction = initializeApp(service);

    const initPromise = initFunction();

    const req = httpMock.expectOne("/assets/config/app-config.json");
    req.flush(mockConfig);

    await initPromise;

    expect(service.getConfig("apiUrl")).toBe(mockConfig.apiUrl);
  });
});
```

## 📊 **Angular Version Comparison**

| Feature                  | Angular 15                | Angular 17-19                                   |
| ------------------------ | ------------------------- | ----------------------------------------------- |
| **Module System**        | NgModule required         | Standalone components preferred                 |
| **Provider Setup**       | `providers` in NgModule   | `ApplicationConfig` with `bootstrapApplication` |
| **HTTP Client**          | `HttpClientModule` import | `provideHttpClient()` function                  |
| **APP_INITIALIZER**      | Same syntax               | Same syntax, cleaner setup                      |
| **Dependency Injection** | Constructor injection     | Enhanced with `inject()` function               |
| **Bundle Size**          | Larger due to modules     | Smaller with tree-shaking                       |

### **Modern Angular 19 Approach**

```typescript
// main.ts (Angular 19)
import { bootstrapApplication } from "@angular/platform-browser";
import { AppComponent } from "./app/app.component";
import { appConfig } from "./app/app.config";

bootstrapApplication(AppComponent, appConfig).catch((err) =>
  console.error("Bootstrap failed:", err)
);
```

## 🎯 **Key Takeaways**

1. **APP_INITIALIZER is the standard way** to run code before Angular bootstrap
2. **Always return a Promise** from your initialization function
3. **Handle errors gracefully** to prevent bootstrap failure
4. **Use dependency injection** to access services in your initializers
5. **Consider multiple initializers** for complex initialization logic
6. **Test your initialization code** thoroughly

This approach ensures your Angular application has all necessary configuration data loaded before any components render, providing a smooth and reliable user experience! 🚀
