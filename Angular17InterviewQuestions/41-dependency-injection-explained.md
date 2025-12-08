# 💉 **Angular Dependency Injection: Complete Guide**

## 🎯 **What You'll Learn**

This comprehensive guide covers Angular's powerful Dependency Injection (DI) system, from basic concepts to advanced patterns, with practical examples and best practices for enterprise applications.

---

## 📚 **The Basics: DI Fundamentals**

### **🤔 What is Dependency Injection?**

**Dependency Injection** is a design pattern where objects receive their dependencies from external sources rather than creating them internally.

**Without DI (Tightly Coupled):**

```typescript
// ❌ Bad: Hard to test and maintain
class UserComponent {
  private userService: UserService;

  constructor() {
    this.userService = new UserService(); // Hard-coded dependency
  }
}
```

**With DI (Loosely Coupled):**

```typescript
// ✅ Good: Flexible and testable
class UserComponent {
  constructor(private userService: UserService) {
    // Dependency injected automatically
  }
}
```

**Benefits:**

- ✅ **Testability** - Easy to mock dependencies
- ✅ **Flexibility** - Swap implementations easily
- ✅ **Maintainability** - Centralized configuration
- ✅ **Separation of Concerns** - Clear responsibility boundaries

---

## 🏗️ **Angular DI System Architecture**

### **🎯 Core DI Concepts**

```typescript
// src/app/models/di-concepts.ts

// 🎯 DEPENDENCY - What we want to inject
export interface UserRepository {
  getUsers(): Observable<User[]>;
  getUserById(id: string): Observable<User>;
  createUser(user: User): Observable<User>;
}

// 🏭 PROVIDER - How to create the dependency
export const USER_REPOSITORY_PROVIDER = {
  provide: UserRepository,
  useClass: HttpUserRepository,
};

// 💉 INJECTOR - Container that manages dependencies
export interface UserService {
  users$: Observable<User[]>;
  getUser(id: string): Observable<User>;
}

// 🎯 INJECTION TOKEN - Unique identifier for dependencies
export const API_BASE_URL = new InjectionToken<string>("ApiBaseUrl");
export const USER_CONFIG = new InjectionToken<UserConfig>("UserConfig");
```

### **🛠️ Provider Types Deep Dive**

```typescript
// src/app/providers/provider-examples.ts
import { Injectable, InjectionToken, Provider } from "@angular/core";

// 🎯 1. CLASS PROVIDER (Most Common)
@Injectable({
  providedIn: "root",
})
export class DatabaseService {
  connect(): void {
    console.log("🔗 Connected to database");
  }
}

// Provider configuration
const classProvider: Provider = {
  provide: DatabaseService,
  useClass: DatabaseService,
};

// 🎯 2. VALUE PROVIDER (Constants & Configuration)
export interface AppConfig {
  apiUrl: string;
  timeout: number;
  retryAttempts: number;
}

const APP_CONFIG: AppConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retryAttempts: 3,
};

const valueProvider: Provider = {
  provide: "APP_CONFIG",
  useValue: APP_CONFIG,
};

// 🎯 3. FACTORY PROVIDER (Dynamic Creation)
export function createLogger(isDev: boolean): Logger {
  return isDev ? new ConsoleLogger() : new FileLogger();
}

const factoryProvider: Provider = {
  provide: Logger,
  useFactory: (env: EnvironmentService) => createLogger(env.isDevelopment()),
  deps: [EnvironmentService],
};

// 🎯 4. EXISTING PROVIDER (Aliases)
const aliasProvider: Provider = {
  provide: "Logger",
  useExisting: Logger,
};

// 🎯 5. MULTI PROVIDER (Multiple Values)
export const HTTP_INTERCEPTORS_PROVIDER: Provider = {
  provide: HTTP_INTERCEPTORS,
  useClass: AuthInterceptor,
  multi: true,
};
```

---

## 💉 **Advanced DI Patterns**

### **🏭 Factory Pattern with Complex Dependencies**

```typescript
// src/app/factories/user-service.factory.ts
import { Injectable, Inject, InjectionToken } from "@angular/core";
import { Observable } from "rxjs";

// 🎯 Configuration interfaces
export interface UserServiceConfig {
  baseUrl: string;
  cacheTimeout: number;
  retryAttempts: number;
  enableOfflineMode: boolean;
}

export interface CacheStrategy {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T, ttl?: number): void;
  clear(): void;
}

// 🔧 Injection tokens
export const USER_SERVICE_CONFIG = new InjectionToken<UserServiceConfig>(
  "UserServiceConfig"
);
export const CACHE_STRATEGY = new InjectionToken<CacheStrategy>(
  "CacheStrategy"
);

// 🏭 Advanced service factory
export function createAdvancedUserService(
  http: HttpClient,
  config: UserServiceConfig,
  cache: CacheStrategy,
  logger: Logger,
  errorHandler: ErrorHandler
): UserService {
  console.log("🏭 Creating advanced user service with config:", config);

  return new AdvancedUserService(http, config, cache, logger, errorHandler);
}

// 🔧 Factory provider configuration
export const ADVANCED_USER_SERVICE_PROVIDER: Provider = {
  provide: UserService,
  useFactory: createAdvancedUserService,
  deps: [HttpClient, USER_SERVICE_CONFIG, CACHE_STRATEGY, Logger, ErrorHandler],
};

// 🏗️ Implementation
@Injectable()
export class AdvancedUserService implements UserService {
  private users$ = new BehaviorSubject<User[]>([]);

  constructor(
    private http: HttpClient,
    private config: UserServiceConfig,
    private cache: CacheStrategy,
    private logger: Logger,
    private errorHandler: ErrorHandler
  ) {
    this.initializeService();
  }

  // 👥 GET USERS with caching
  getUsers(): Observable<User[]> {
    const cacheKey = "users_list";
    const cached = this.cache.get<User[]>(cacheKey);

    if (cached) {
      this.logger.info("📋 Serving users from cache");
      return of(cached);
    }

    return this.http.get<User[]>(`${this.config.baseUrl}/users`).pipe(
      tap((users) => {
        this.cache.set(cacheKey, users, this.config.cacheTimeout);
        this.users$.next(users);
        this.logger.info(`👥 Loaded ${users.length} users from API`);
      }),
      retry(this.config.retryAttempts),
      catchError((error) => {
        this.logger.error("Failed to load users:", error);

        if (this.config.enableOfflineMode) {
          return this.getOfflineUsers();
        }

        return this.errorHandler.handleError(error);
      })
    );
  }

  // 👤 GET USER BY ID
  getUserById(id: string): Observable<User> {
    const cacheKey = `user_${id}`;
    const cached = this.cache.get<User>(cacheKey);

    if (cached) {
      this.logger.info(`👤 Serving user ${id} from cache`);
      return of(cached);
    }

    return this.http.get<User>(`${this.config.baseUrl}/users/${id}`).pipe(
      tap((user) => {
        this.cache.set(cacheKey, user, this.config.cacheTimeout);
        this.logger.info(`👤 Loaded user ${id} from API`);
      }),
      retry(this.config.retryAttempts),
      catchError((error) => this.handleUserError(error, id))
    );
  }

  // ➕ CREATE USER
  createUser(user: Partial<User>): Observable<User> {
    this.logger.info("➕ Creating new user:", user.email);

    return this.http.post<User>(`${this.config.baseUrl}/users`, user).pipe(
      tap((newUser) => {
        this.invalidateUsersCache();
        this.logger.info(`✅ User created: ${newUser.id}`);
      }),
      catchError((error) => this.handleCreateError(error, user))
    );
  }

  // 🔄 UPDATE USER
  updateUser(id: string, updates: Partial<User>): Observable<User> {
    this.logger.info(`🔄 Updating user ${id}:`, updates);

    return this.http
      .patch<User>(`${this.config.baseUrl}/users/${id}`, updates)
      .pipe(
        tap((updatedUser) => {
          this.cache.set(`user_${id}`, updatedUser, this.config.cacheTimeout);
          this.invalidateUsersCache();
          this.logger.info(`✅ User updated: ${id}`);
        }),
        catchError((error) => this.handleUpdateError(error, id))
      );
  }

  // 🗑️ DELETE USER
  deleteUser(id: string): Observable<void> {
    this.logger.info(`🗑️ Deleting user ${id}`);

    return this.http.delete<void>(`${this.config.baseUrl}/users/${id}`).pipe(
      tap(() => {
        this.cache.set(`user_${id}`, null);
        this.invalidateUsersCache();
        this.logger.info(`✅ User deleted: ${id}`);
      }),
      catchError((error) => this.handleDeleteError(error, id))
    );
  }

  // 🔄 Private Helper Methods

  private initializeService(): void {
    this.logger.info("🚀 Initializing AdvancedUserService", this.config);
  }

  private getOfflineUsers(): Observable<User[]> {
    this.logger.warn("📱 Using offline mode");

    const offlineUsers = this.cache.get<User[]>("offline_users") || [];
    return of(offlineUsers);
  }

  private invalidateUsersCache(): void {
    this.cache.set("users_list", null);
    this.logger.info("🗑️ Users cache invalidated");
  }

  private handleUserError(error: any, id: string): Observable<User> {
    this.logger.error(`Failed to load user ${id}:`, error);
    return this.errorHandler.handleError(error);
  }

  private handleCreateError(error: any, user: Partial<User>): Observable<User> {
    this.logger.error("Failed to create user:", error);
    return this.errorHandler.handleError(error);
  }

  private handleUpdateError(error: any, id: string): Observable<User> {
    this.logger.error(`Failed to update user ${id}:`, error);
    return this.errorHandler.handleError(error);
  }

  private handleDeleteError(error: any, id: string): Observable<void> {
    this.logger.error(`Failed to delete user ${id}:`, error);
    return this.errorHandler.handleError(error);
  }
}
```

### **🎭 Multiple Implementation Strategy**

```typescript
// src/app/strategies/storage.strategies.ts

// 🎯 Define strategy interface
export interface StorageStrategy {
  save(key: string, value: any): void;
  load(key: string): any;
  remove(key: string): void;
  clear(): void;
}

// 🗄️ Local Storage Strategy
@Injectable()
export class LocalStorageStrategy implements StorageStrategy {
  save(key: string, value: any): void {
    localStorage.setItem(key, JSON.stringify(value));
  }

  load(key: string): any {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  remove(key: string): void {
    localStorage.removeItem(key);
  }

  clear(): void {
    localStorage.clear();
  }
}

// 🗃️ Session Storage Strategy
@Injectable()
export class SessionStorageStrategy implements StorageStrategy {
  save(key: string, value: any): void {
    sessionStorage.setItem(key, JSON.stringify(value));
  }

  load(key: string): any {
    const item = sessionStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  remove(key: string): void {
    sessionStorage.removeItem(key);
  }

  clear(): void {
    sessionStorage.clear();
  }
}

// 💾 Memory Storage Strategy
@Injectable()
export class MemoryStorageStrategy implements StorageStrategy {
  private storage = new Map<string, any>();

  save(key: string, value: any): void {
    this.storage.set(key, value);
  }

  load(key: string): any {
    return this.storage.get(key) || null;
  }

  remove(key: string): void {
    this.storage.delete(key);
  }

  clear(): void {
    this.storage.clear();
  }
}

// 🏭 Storage factory
export function createStorageStrategy(
  environment: EnvironmentService
): StorageStrategy {
  if (!environment.isBrowser()) {
    return new MemoryStorageStrategy();
  }

  if (environment.isProduction()) {
    return new LocalStorageStrategy();
  }

  return new SessionStorageStrategy();
}

// 🔧 Provider configuration
export const STORAGE_STRATEGY_PROVIDER: Provider = {
  provide: StorageStrategy,
  useFactory: createStorageStrategy,
  deps: [EnvironmentService],
};
```

---

## 🏗️ **Hierarchical Injection**

### **🌳 Component Level Providers**

```typescript
// src/app/components/user-profile/user-profile.component.ts
import { Component, OnInit, Injector } from "@angular/core";

// 🎯 Component-specific service
@Injectable()
export class UserProfileService {
  constructor(private http: HttpClient, private userService: UserService) {}

  getProfile(userId: string): Observable<UserProfile> {
    return this.http.get<UserProfile>(`/api/users/${userId}/profile`);
  }
}

@Component({
  selector: "app-user-profile",
  template: `
    <div class="user-profile">
      <h2>User Profile</h2>

      <div *ngIf="profile$ | async as profile" class="profile-content">
        <img [src]="profile.avatar" [alt]="profile.name" />
        <h3>{{ profile.name }}</h3>
        <p>{{ profile.email }}</p>

        <!-- Child component inherits providers -->
        <app-user-settings [userId]="profile.id"></app-user-settings>
      </div>
    </div>
  `,
  providers: [
    // 🎯 Component-level provider (new instance per component)
    UserProfileService,

    // 🔧 Custom configuration for this component
    {
      provide: "PROFILE_CONFIG",
      useValue: { enableEdit: true, showPrivateInfo: false },
    },
  ],
})
export class UserProfileComponent implements OnInit {
  profile$!: Observable<UserProfile>;

  constructor(
    private userProfileService: UserProfileService,
    private route: ActivatedRoute,
    @Inject("PROFILE_CONFIG") private config: any
  ) {}

  ngOnInit(): void {
    const userId = this.route.snapshot.paramMap.get("id")!;
    this.profile$ = this.userProfileService.getProfile(userId);

    console.log("Profile config:", this.config);
  }
}
```

### **🔄 Dynamic Provider Creation**

```typescript
// src/app/services/dynamic-injector.service.ts
import { Injectable, Injector, Type } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class DynamicInjectorService {
  constructor(private injector: Injector) {}

  // 🔧 Create dynamic injector with custom providers
  createChildInjector(providers: Provider[]): Injector {
    return Injector.create({
      providers,
      parent: this.injector,
    });
  }

  // 🏭 Create service with dynamic configuration
  createConfiguredService<T>(serviceClass: Type<T>, config: any): T {
    const childInjector = this.createChildInjector([
      {
        provide: serviceClass,
        useClass: serviceClass,
      },
      {
        provide: "SERVICE_CONFIG",
        useValue: config,
      },
    ]);

    return childInjector.get(serviceClass);
  }

  // 🎭 Create service with different implementation
  createServiceWithImplementation<T>(
    token: any,
    implementation: Type<T>,
    dependencies: Provider[] = []
  ): T {
    const childInjector = this.createChildInjector([
      {
        provide: token,
        useClass: implementation,
      },
      ...dependencies,
    ]);

    return childInjector.get(token);
  }
}

// Usage example
@Component({
  selector: "app-dynamic-example",
  template: `
    <button (click)="createDynamicService()">Create Dynamic Service</button>
    <div *ngIf="result">{{ result }}</div>
  `,
})
export class DynamicExampleComponent {
  result = "";

  constructor(private dynamicInjector: DynamicInjectorService) {}

  createDynamicService(): void {
    // Create service with specific configuration
    const customService = this.dynamicInjector.createConfiguredService(
      UserService,
      { baseUrl: "https://custom-api.com", timeout: 10000 }
    );

    this.result = "Dynamic service created with custom config!";
  }
}
```

---

## 🚀 **Modern DI with Angular 17+ Features**

### **⚡ inject() Function Usage**

```typescript
// src/app/services/modern-service.ts
import { Injectable, inject, DestroyRef } from "@angular/core";
import { takeUntilDestroyed } from "@angular/core/rxjs-interop";

@Injectable({
  providedIn: "root",
})
export class ModernUserService {
  // 🎯 Modern dependency injection
  private http = inject(HttpClient);
  private router = inject(Router);
  private destroyRef = inject(DestroyRef);
  private notificationService = inject(NotificationService);

  private users$ = new BehaviorSubject<User[]>([]);

  constructor() {
    this.initializeService();
  }

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>("/api/users").pipe(
      tap((users) => this.users$.next(users)),
      takeUntilDestroyed(this.destroyRef), // ✨ Automatic cleanup
      catchError(this.handleError.bind(this))
    );
  }

  private initializeService(): void {
    console.log("🚀 Modern service initialized with inject()");
  }

  private handleError(error: any): Observable<never> {
    this.notificationService.showError("Failed to load users");
    return EMPTY;
  }
}

// 🎯 Modern component with inject()
@Component({
  selector: "app-modern-user-list",
  template: `
    <div class="user-list">
      <h2>Users</h2>
      <div *ngFor="let user of users$ | async" class="user-item">
        {{ user.name }} - {{ user.email }}
      </div>
    </div>
  `,
})
export class ModernUserListComponent {
  // ⚡ Use inject() instead of constructor injection
  private userService = inject(ModernUserService);
  private cd = inject(ChangeDetectorRef);

  users$ = this.userService.getUsers();

  // 🔄 Method can use injected dependencies
  refreshUsers(): void {
    this.users$ = this.userService.getUsers();
    this.cd.markForCheck();
  }
}
```

### **🎯 Standalone Components with Providers**

```typescript
// src/app/components/standalone-user.component.ts
import { Component, inject } from "@angular/core";
import { CommonModule } from "@angular/common";
import { HttpClient } from "@angular/common/http";

@Component({
  selector: "app-standalone-user",
  standalone: true,
  imports: [CommonModule],
  providers: [
    // 🎯 Component-specific providers
    {
      provide: "API_CONFIG",
      useValue: { baseUrl: "/api/v2", timeout: 5000 },
    },
  ],
  template: `
    <div class="standalone-component">
      <h2>Standalone User Component</h2>
      <div *ngFor="let user of users$ | async">
        {{ user.name }}
      </div>
    </div>
  `,
})
export class StandaloneUserComponent {
  private http = inject(HttpClient);
  private config = inject("API_CONFIG");

  users$ = this.http.get<User[]>(`${this.config.baseUrl}/users`);
}

// 🏗️ Bootstrap with providers
import { bootstrapApplication } from "@angular/platform-browser";

bootstrapApplication(AppComponent, {
  providers: [
    // 🌍 Global providers
    provideHttpClient(),
    provideRouter(routes),

    // 🔧 Custom providers
    {
      provide: Logger,
      useClass: ConsoleLogger,
    },
    {
      provide: "APP_VERSION",
      useValue: "1.0.0",
    },
  ],
});
```

---

## 🎉 **Summary: DI Mastery**

### **✅ What We've Covered:**

💉 **DI Fundamentals** - Core concepts and benefits  
🏭 **Provider Types** - Class, value, factory, existing, multi providers  
🎯 **Advanced Patterns** - Factory functions and strategy patterns  
🌳 **Hierarchical Injection** - Component and service level providers  
⚡ **Modern Features** - inject() function and standalone components

### **🎯 Best Practices:**

- ✅ **Single Responsibility** - One service, one purpose
- ✅ **Interface Segregation** - Define clear contracts
- ✅ **Dependency Inversion** - Depend on abstractions
- ✅ **Testability** - Easy to mock and test
- ✅ **Performance** - Use providedIn: 'root' for singletons
- ✅ **Memory Management** - Proper cleanup with takeUntilDestroyed

**You're now equipped to master Angular's DI system!** 🚀
