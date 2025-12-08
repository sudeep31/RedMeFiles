# 🧩 Standalone Components in Angular: Complete Architecture Guide

## 🎯 **Question Overview**

_"How do you architect an application using standalone components in Angular?"_

## 🔍 **Understanding Standalone Components**

Standalone components represent a **paradigm shift** in Angular architecture, eliminating the need for NgModules in many scenarios. They offer **simplified bootstrapping**, **better tree-shaking**, and **more flexible component composition**.

This approach enables **micro-frontend architectures**, **component libraries**, and **simplified testing** patterns! 🚀

## 🏗️ **Core Architecture Patterns**

### **1. 📱 Application Bootstrap**

```typescript
// main.ts - Modern standalone bootstrap
import { bootstrapApplication } from "@angular/platform-browser";
import { AppComponent } from "./app/app.component";
import { provideRouter } from "@angular/router";
import { provideHttpClient, withInterceptors } from "@angular/common/http";
import { provideAnimations } from "@angular/platform-browser/animations";
import { provideExperimentalZonelessChangeDetection } from "@angular/core";
import { routes } from "./app/app.routes";
import { authInterceptor } from "./app/interceptors/auth.interceptor";
import { errorInterceptor } from "./app/interceptors/error.interceptor";

bootstrapApplication(AppComponent, {
  providers: [
    // Routing
    provideRouter(routes),

    // HTTP Client with interceptors
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor])),

    // Animations
    provideAnimations(),

    // Modern change detection (Angular 19+)
    provideExperimentalZonelessChangeDetection(),

    // Custom services
    { provide: "API_URL", useValue: environment.apiUrl },

    // Feature providers
    ...provideAuthFeature(),
    ...provideThemeFeature(),
    ...provideNotificationFeature(),
  ],
}).catch((err) => console.error(err));

// Feature providers pattern
export function provideAuthFeature() {
  return [
    AuthService,
    AuthGuard,
    { provide: "AUTH_CONFIG", useValue: { tokenKey: "auth_token" } },
  ];
}

export function provideThemeFeature() {
  return [ThemeService, { provide: "DEFAULT_THEME", useValue: "light" }];
}

export function provideNotificationFeature() {
  return [
    NotificationService,
    { provide: "NOTIFICATION_CONFIG", useValue: { duration: 3000 } },
  ];
}
```

### **2. 🎯 Root App Component**

```typescript
// app.component.ts - Standalone root component
@Component({
  selector: "app-root",
  template: `
    <div class="app-layout" [class.dark-theme]="isDarkTheme()">
      <!-- Header -->
      <app-header
        [user]="currentUser()"
        [notifications]="notifications()"
        (themeToggle)="toggleTheme()"
        (logout)="logout()"
      >
      </app-header>

      <!-- Navigation -->
      <app-sidebar
        [isCollapsed]="sidebarCollapsed()"
        [currentRoute]="currentRoute()"
        (toggleSidebar)="toggleSidebar()"
      >
      </app-sidebar>

      <!-- Main Content -->
      <main class="main-content" [class.sidebar-collapsed]="sidebarCollapsed()">
        <!-- Loading indicator -->
        <app-loading-spinner *ngIf="isLoading()"></app-loading-spinner>

        <!-- Error boundary -->
        <app-error-boundary>
          <!-- Router outlet -->
          <router-outlet></router-outlet>
        </app-error-boundary>
      </main>

      <!-- Footer -->
      <app-footer [version]="appVersion" [buildInfo]="buildInfo"> </app-footer>

      <!-- Global notifications -->
      <app-notification-container></app-notification-container>

      <!-- Modals -->
      <app-modal-container></app-modal-container>
    </div>
  `,
  styles: [
    `
      .app-layout {
        display: grid;
        grid-template-areas:
          "header header"
          "sidebar main"
          "footer footer";
        grid-template-rows: auto 1fr auto;
        grid-template-columns: 250px 1fr;
        min-height: 100vh;
        transition: grid-template-columns 0.3s ease;
      }

      .app-layout.dark-theme {
        background: #121212;
        color: #ffffff;
      }

      .main-content.sidebar-collapsed {
        grid-column: 1 / -1;
      }

      .main-content {
        grid-area: main;
        padding: 20px;
        overflow-y: auto;
        position: relative;
      }

      app-header {
        grid-area: header;
      }

      app-sidebar {
        grid-area: sidebar;
      }

      app-footer {
        grid-area: footer;
      }
    `,
  ],
  standalone: true,
  imports: [
    CommonModule,
    RouterOutlet,
    HeaderComponent,
    SidebarComponent,
    FooterComponent,
    LoadingSpinnerComponent,
    ErrorBoundaryComponent,
    NotificationContainerComponent,
    ModalContainerComponent,
  ],
})
export class AppComponent implements OnInit {
  // Application state signals
  currentUser = signal<User | null>(null);
  notifications = signal<Notification[]>([]);
  isLoading = signal(false);
  isDarkTheme = signal(false);
  sidebarCollapsed = signal(false);
  currentRoute = signal("");

  // Static properties
  appVersion = environment.version;
  buildInfo = environment.buildInfo;

  private authService = inject(AuthService);
  private themeService = inject(ThemeService);
  private notificationService = inject(NotificationService);
  private router = inject(Router);

  constructor() {
    this.setupApplicationEffects();
  }

  ngOnInit() {
    this.initializeApplication();
  }

  private setupApplicationEffects() {
    // Track route changes
    effect(() => {
      this.router.events
        .pipe(
          filter(
            (event): event is NavigationEnd => event instanceof NavigationEnd
          ),
          map((event) => event.url),
          takeUntilDestroyed()
        )
        .subscribe((url) => {
          this.currentRoute.set(url);
        });
    });

    // Track authentication state
    effect(() => {
      this.authService.currentUser$
        .pipe(takeUntilDestroyed())
        .subscribe((user) => {
          this.currentUser.set(user);
        });
    });

    // Track theme changes
    effect(() => {
      this.themeService.isDarkTheme$
        .pipe(takeUntilDestroyed())
        .subscribe((isDark) => {
          this.isDarkTheme.set(isDark);
        });
    });

    // Track notifications
    effect(() => {
      this.notificationService.notifications$
        .pipe(takeUntilDestroyed())
        .subscribe((notifications) => {
          this.notifications.set(notifications);
        });
    });

    // Track loading state
    effect(() => {
      this.authService.loading$
        .pipe(takeUntilDestroyed())
        .subscribe((loading) => {
          this.isLoading.set(loading);
        });
    });
  }

  private async initializeApplication() {
    // Initialize authentication
    await this.authService.initialize();

    // Load user preferences
    await this.loadUserPreferences();

    // Setup error handling
    this.setupErrorHandling();
  }

  private async loadUserPreferences() {
    const preferences = await this.authService.getUserPreferences();
    if (preferences) {
      this.themeService.setTheme(preferences.theme);
      this.sidebarCollapsed.set(preferences.sidebarCollapsed);
    }
  }

  private setupErrorHandling() {
    // Global error handling
    window.addEventListener("error", (event) => {
      console.error("Global error:", event.error);
      this.notificationService.showError("An unexpected error occurred");
    });

    window.addEventListener("unhandledrejection", (event) => {
      console.error("Unhandled promise rejection:", event.reason);
      this.notificationService.showError("An unexpected error occurred");
    });
  }

  // Event handlers
  toggleTheme() {
    this.themeService.toggleTheme();
  }

  toggleSidebar() {
    this.sidebarCollapsed.update((collapsed) => !collapsed);
    this.authService.updateUserPreferences({
      sidebarCollapsed: this.sidebarCollapsed(),
    });
  }

  logout() {
    this.authService.logout();
    this.router.navigate(["/login"]);
  }
}
```

### **3. 🗺️ Routing Architecture**

```typescript
// app.routes.ts - Standalone routing configuration
import { Routes } from "@angular/router";
import { AuthGuard } from "./guards/auth.guard";
import { AdminGuard } from "./guards/admin.guard";

export const routes: Routes = [
  // Public routes
  {
    path: "",
    redirectTo: "/dashboard",
    pathMatch: "full",
  },
  {
    path: "login",
    loadComponent: () =>
      import("./pages/auth/login.component").then((m) => m.LoginComponent),
    title: "Login - MyApp",
  },
  {
    path: "register",
    loadComponent: () =>
      import("./pages/auth/register.component").then(
        (m) => m.RegisterComponent
      ),
    title: "Register - MyApp",
  },
  {
    path: "forgot-password",
    loadComponent: () =>
      import("./pages/auth/forgot-password.component").then(
        (m) => m.ForgotPasswordComponent
      ),
    title: "Forgot Password - MyApp",
  },

  // Protected routes
  {
    path: "dashboard",
    canActivate: [AuthGuard],
    loadComponent: () =>
      import("./pages/dashboard/dashboard.component").then(
        (m) => m.DashboardComponent
      ),
    title: "Dashboard - MyApp",
  },

  // Feature modules with lazy loading
  {
    path: "users",
    canActivate: [AuthGuard],
    loadChildren: () =>
      import("./features/users/users.routes").then((m) => m.userRoutes),
  },
  {
    path: "products",
    canActivate: [AuthGuard],
    loadChildren: () =>
      import("./features/products/products.routes").then(
        (m) => m.productRoutes
      ),
  },
  {
    path: "analytics",
    canActivate: [AuthGuard],
    loadChildren: () =>
      import("./features/analytics/analytics.routes").then(
        (m) => m.analyticsRoutes
      ),
  },

  // Admin routes
  {
    path: "admin",
    canActivate: [AuthGuard, AdminGuard],
    loadChildren: () =>
      import("./features/admin/admin.routes").then((m) => m.adminRoutes),
  },

  // Settings
  {
    path: "settings",
    canActivate: [AuthGuard],
    loadComponent: () =>
      import("./pages/settings/settings.component").then(
        (m) => m.SettingsComponent
      ),
    title: "Settings - MyApp",
  },

  // Error pages
  {
    path: "404",
    loadComponent: () =>
      import("./pages/errors/not-found.component").then(
        (m) => m.NotFoundComponent
      ),
    title: "Page Not Found - MyApp",
  },
  {
    path: "500",
    loadComponent: () =>
      import("./pages/errors/server-error.component").then(
        (m) => m.ServerErrorComponent
      ),
    title: "Server Error - MyApp",
  },

  // Catch all
  {
    path: "**",
    redirectTo: "/404",
  },
];

// Feature route files example
// features/users/users.routes.ts
export const userRoutes: Routes = [
  {
    path: "",
    loadComponent: () =>
      import("./user-list/user-list.component").then(
        (m) => m.UserListComponent
      ),
    title: "Users - MyApp",
  },
  {
    path: "create",
    loadComponent: () =>
      import("./user-form/user-form.component").then(
        (m) => m.UserFormComponent
      ),
    title: "Create User - MyApp",
  },
  {
    path: ":id",
    loadComponent: () =>
      import("./user-detail/user-detail.component").then(
        (m) => m.UserDetailComponent
      ),
    title: "User Details - MyApp",
  },
  {
    path: ":id/edit",
    loadComponent: () =>
      import("./user-form/user-form.component").then(
        (m) => m.UserFormComponent
      ),
    title: "Edit User - MyApp",
  },
];
```

### **4. 🎨 Component Library Architecture**

```typescript
// shared/components/button/button.component.ts
@Component({
  selector: "app-button",
  template: `
    <button
      [type]="type"
      [class]="buttonClasses()"
      [disabled]="disabled || loading()"
      (click)="handleClick($event)"
    >
      <!-- Loading spinner -->
      <span *ngIf="loading()" class="loading-spinner" aria-hidden="true"></span>

      <!-- Icon -->
      <span
        *ngIf="icon && !loading()"
        [class]="iconClasses()"
        aria-hidden="true"
      >
        {{ icon }}
      </span>

      <!-- Text content -->
      <span class="button-text" [class.sr-only]="iconOnly">
        <ng-content></ng-content>
      </span>

      <!-- Badge -->
      <span *ngIf="badge" class="badge">{{ badge }}</span>
    </button>
  `,
  styles: [
    `
      button {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: 8px;
        border: none;
        border-radius: 6px;
        font-weight: 500;
        text-decoration: none;
        transition: all 0.2s ease;
        cursor: pointer;
        position: relative;
        white-space: nowrap;
      }

      button:disabled {
        opacity: 0.6;
        cursor: not-allowed;
      }

      /* Sizes */
      .size-small {
        padding: 6px 12px;
        font-size: 14px;
        line-height: 1.25;
      }

      .size-medium {
        padding: 8px 16px;
        font-size: 16px;
        line-height: 1.5;
      }

      .size-large {
        padding: 12px 24px;
        font-size: 18px;
        line-height: 1.5;
      }

      /* Variants */
      .variant-primary {
        background: #007acc;
        color: white;
      }

      .variant-primary:hover:not(:disabled) {
        background: #005999;
      }

      .variant-secondary {
        background: #6c757d;
        color: white;
      }

      .variant-secondary:hover:not(:disabled) {
        background: #545b62;
      }

      .variant-outline {
        background: transparent;
        color: #007acc;
        border: 1px solid #007acc;
      }

      .variant-outline:hover:not(:disabled) {
        background: #007acc;
        color: white;
      }

      .variant-ghost {
        background: transparent;
        color: #007acc;
      }

      .variant-ghost:hover:not(:disabled) {
        background: rgba(0, 122, 204, 0.1);
      }

      .variant-danger {
        background: #dc3545;
        color: white;
      }

      .variant-danger:hover:not(:disabled) {
        background: #c82333;
      }

      /* Loading spinner */
      .loading-spinner {
        width: 16px;
        height: 16px;
        border: 2px solid transparent;
        border-top: 2px solid currentColor;
        border-radius: 50%;
        animation: spin 1s linear infinite;
      }

      @keyframes spin {
        to {
          transform: rotate(360deg);
        }
      }

      /* Badge */
      .badge {
        position: absolute;
        top: -4px;
        right: -4px;
        background: #dc3545;
        color: white;
        font-size: 12px;
        padding: 2px 6px;
        border-radius: 10px;
        min-width: 16px;
        text-align: center;
      }

      /* Icon only */
      .icon-only {
        padding: 8px;
        aspect-ratio: 1;
      }

      .sr-only {
        position: absolute;
        width: 1px;
        height: 1px;
        padding: 0;
        margin: -1px;
        overflow: hidden;
        clip: rect(0, 0, 0, 0);
        white-space: nowrap;
        border: 0;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ButtonComponent {
  @Input() type: "button" | "submit" | "reset" = "button";
  @Input() variant: "primary" | "secondary" | "outline" | "ghost" | "danger" =
    "primary";
  @Input() size: "small" | "medium" | "large" = "medium";
  @Input() disabled = false;
  @Input() icon?: string;
  @Input() iconOnly = false;
  @Input() badge?: string | number;

  // Reactive state
  loading = signal(false);

  @Output() buttonClick = new EventEmitter<MouseEvent>();

  // Computed classes
  buttonClasses = computed(() =>
    [
      `variant-${this.variant}`,
      `size-${this.size}`,
      this.iconOnly ? "icon-only" : "",
      this.loading() ? "loading" : "",
    ]
      .filter(Boolean)
      .join(" ")
  );

  iconClasses = computed(() =>
    ["icon", this.iconOnly ? "icon-standalone" : ""].join(" ")
  );

  handleClick(event: MouseEvent) {
    if (!this.disabled && !this.loading()) {
      this.buttonClick.emit(event);
    }
  }

  // Public methods for loading state
  setLoading(loading: boolean) {
    this.loading.set(loading);
  }

  async performAsyncAction<T>(action: () => Promise<T>): Promise<T> {
    this.loading.set(true);
    try {
      return await action();
    } finally {
      this.loading.set(false);
    }
  }
}

// Usage example
@Component({
  selector: "app-example",
  template: `
    <div class="button-examples">
      <!-- Basic buttons -->
      <app-button (buttonClick)="handlePrimary()"> Primary Button </app-button>

      <app-button variant="secondary" (buttonClick)="handleSecondary()">
        Secondary Button
      </app-button>

      <!-- Button with icon -->
      <app-button variant="outline" icon="🔍" (buttonClick)="handleSearch()">
        Search
      </app-button>

      <!-- Icon only button -->
      <app-button
        variant="ghost"
        icon="❤️"
        iconOnly="true"
        (buttonClick)="handleLike()"
      >
        Like
      </app-button>

      <!-- Button with badge -->
      <app-button
        variant="primary"
        badge="3"
        (buttonClick)="handleNotifications()"
      >
        Notifications
      </app-button>

      <!-- Async action button -->
      <app-button
        #saveBtn
        variant="primary"
        (buttonClick)="handleSave(saveBtn)"
      >
        Save
      </app-button>
    </div>
  `,
  standalone: true,
  imports: [ButtonComponent],
})
export class ButtonExampleComponent {
  handlePrimary() {
    console.log("Primary clicked");
  }

  handleSecondary() {
    console.log("Secondary clicked");
  }

  handleSearch() {
    console.log("Search clicked");
  }

  handleLike() {
    console.log("Like clicked");
  }

  handleNotifications() {
    console.log("Notifications clicked");
  }

  async handleSave(button: ButtonComponent) {
    await button.performAsyncAction(async () => {
      // Simulate save operation
      await new Promise((resolve) => setTimeout(resolve, 2000));
      console.log("Saved successfully");
    });
  }
}
```

### **5. 🔧 Service Architecture**

```typescript
// services/feature.service.ts - Standalone-compatible services
@Injectable({
  providedIn: "root",
})
export class FeatureService {
  private httpClient = inject(HttpClient);
  private notificationService = inject(NotificationService);

  // State management with signals
  private _items = signal<Item[]>([]);
  private _loading = signal(false);
  private _error = signal<string | null>(null);
  private _selectedItem = signal<Item | null>(null);

  // Public readonly signals
  readonly items = this._items.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();
  readonly selectedItem = this._selectedItem.asReadonly();

  // Computed state
  readonly itemCount = computed(() => this._items().length);
  readonly hasItems = computed(() => this._items().length > 0);
  readonly hasError = computed(() => this._error() !== null);

  // Filter and search
  private _searchTerm = signal("");
  private _filterStatus = signal<string>("all");

  readonly searchTerm = this._searchTerm.asReadonly();
  readonly filterStatus = this._filterStatus.asReadonly();

  readonly filteredItems = computed(() => {
    const items = this._items();
    const search = this._searchTerm().toLowerCase();
    const status = this._filterStatus();

    return items.filter((item) => {
      const matchesSearch =
        !search ||
        item.name.toLowerCase().includes(search) ||
        item.description.toLowerCase().includes(search);

      const matchesStatus = status === "all" || item.status === status;

      return matchesSearch && matchesStatus;
    });
  });

  // CRUD operations
  async loadItems(): Promise<void> {
    this._loading.set(true);
    this._error.set(null);

    try {
      const items = await firstValueFrom(
        this.httpClient.get<Item[]>("/api/items")
      );
      this._items.set(items);
    } catch (error) {
      const message = "Failed to load items";
      this._error.set(message);
      this.notificationService.showError(message);
      throw error;
    } finally {
      this._loading.set(false);
    }
  }

  async createItem(itemData: CreateItemRequest): Promise<Item> {
    this._loading.set(true);
    this._error.set(null);

    try {
      const newItem = await firstValueFrom(
        this.httpClient.post<Item>("/api/items", itemData)
      );

      this._items.update((items) => [...items, newItem]);
      this.notificationService.showSuccess("Item created successfully");

      return newItem;
    } catch (error) {
      const message = "Failed to create item";
      this._error.set(message);
      this.notificationService.showError(message);
      throw error;
    } finally {
      this._loading.set(false);
    }
  }

  async updateItem(id: string, updates: UpdateItemRequest): Promise<Item> {
    this._loading.set(true);
    this._error.set(null);

    try {
      const updatedItem = await firstValueFrom(
        this.httpClient.patch<Item>(`/api/items/${id}`, updates)
      );

      this._items.update((items) =>
        items.map((item) => (item.id === id ? updatedItem : item))
      );

      this.notificationService.showSuccess("Item updated successfully");

      return updatedItem;
    } catch (error) {
      const message = "Failed to update item";
      this._error.set(message);
      this.notificationService.showError(message);
      throw error;
    } finally {
      this._loading.set(false);
    }
  }

  async deleteItem(id: string): Promise<void> {
    this._loading.set(true);
    this._error.set(null);

    try {
      await firstValueFrom(this.httpClient.delete(`/api/items/${id}`));

      this._items.update((items) => items.filter((item) => item.id !== id));

      if (this._selectedItem()?.id === id) {
        this._selectedItem.set(null);
      }

      this.notificationService.showSuccess("Item deleted successfully");
    } catch (error) {
      const message = "Failed to delete item";
      this._error.set(message);
      this.notificationService.showError(message);
      throw error;
    } finally {
      this._loading.set(false);
    }
  }

  // Selection and filters
  selectItem(item: Item | null): void {
    this._selectedItem.set(item);
  }

  setSearchTerm(term: string): void {
    this._searchTerm.set(term);
  }

  setFilterStatus(status: string): void {
    this._filterStatus.set(status);
  }

  clearFilters(): void {
    this._searchTerm.set("");
    this._filterStatus.set("all");
  }

  // Utility methods
  getItemById(id: string): Item | undefined {
    return this._items().find((item) => item.id === id);
  }

  refreshItems(): void {
    this.loadItems();
  }

  clearError(): void {
    this._error.set(null);
  }
}

interface Item {
  id: string;
  name: string;
  description: string;
  status: "active" | "inactive" | "pending";
  createdAt: Date;
  updatedAt: Date;
}

interface CreateItemRequest {
  name: string;
  description: string;
  status?: "active" | "inactive" | "pending";
}

interface UpdateItemRequest {
  name?: string;
  description?: string;
  status?: "active" | "inactive" | "pending";
}
```

## 🧪 **Testing Standalone Components**

### **1. 🔬 Component Testing**

```typescript
// button.component.spec.ts
import { TestBed } from "@angular/core/testing";
import { ComponentFixture } from "@angular/core/testing";
import { ButtonComponent } from "./button.component";

describe("ButtonComponent", () => {
  let component: ButtonComponent;
  let fixture: ComponentFixture<ButtonComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ButtonComponent], // Import standalone component directly
    }).compileComponents();

    fixture = TestBed.createComponent(ButtonComponent);
    component = fixture.componentInstance;
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });

  it("should render button text", () => {
    fixture.detectChanges();
    const button = fixture.nativeElement.querySelector("button");

    // Set content through ng-content
    const span = document.createElement("span");
    span.textContent = "Test Button";
    button.appendChild(span);

    expect(button).toBeTruthy();
    expect(button.textContent).toContain("Test Button");
  });

  it("should emit click event", () => {
    spyOn(component.buttonClick, "emit");

    const button = fixture.nativeElement.querySelector("button");
    button.click();

    expect(component.buttonClick.emit).toHaveBeenCalled();
  });

  it("should show loading spinner when loading", () => {
    component.setLoading(true);
    fixture.detectChanges();

    const spinner = fixture.nativeElement.querySelector(".loading-spinner");
    expect(spinner).toBeTruthy();
  });

  it("should disable button when loading", () => {
    component.setLoading(true);
    fixture.detectChanges();

    const button = fixture.nativeElement.querySelector("button");
    expect(button.disabled).toBeTruthy();
  });

  it("should apply correct CSS classes", () => {
    component.variant = "primary";
    component.size = "large";
    fixture.detectChanges();

    const button = fixture.nativeElement.querySelector("button");
    expect(button.className).toContain("variant-primary");
    expect(button.className).toContain("size-large");
  });

  it("should perform async action with loading state", async () => {
    const mockAction = jest.fn().mockResolvedValue("success");

    expect(component.loading()).toBeFalsy();

    const promise = component.performAsyncAction(mockAction);

    expect(component.loading()).toBeTruthy();

    const result = await promise;

    expect(component.loading()).toBeFalsy();
    expect(result).toBe("success");
    expect(mockAction).toHaveBeenCalled();
  });
});
```

### **2. 🔧 Service Testing**

```typescript
// feature.service.spec.ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { FeatureService } from "./feature.service";
import { NotificationService } from "./notification.service";

describe("FeatureService", () => {
  let service: FeatureService;
  let httpMock: HttpTestingController;
  let notificationService: jasmine.SpyObj<NotificationService>;

  beforeEach(() => {
    const notificationSpy = jasmine.createSpyObj("NotificationService", [
      "showSuccess",
      "showError",
    ]);

    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        FeatureService,
        { provide: NotificationService, useValue: notificationSpy },
      ],
    });

    service = TestBed.inject(FeatureService);
    httpMock = TestBed.inject(HttpTestingController);
    notificationService = TestBed.inject(
      NotificationService
    ) as jasmine.SpyObj<NotificationService>;
  });

  afterEach(() => {
    httpMock.verify();
  });

  it("should be created", () => {
    expect(service).toBeTruthy();
  });

  it("should load items successfully", async () => {
    const mockItems = [
      {
        id: "1",
        name: "Item 1",
        description: "Description 1",
        status: "active",
      },
      {
        id: "2",
        name: "Item 2",
        description: "Description 2",
        status: "inactive",
      },
    ];

    const loadPromise = service.loadItems();

    const req = httpMock.expectOne("/api/items");
    expect(req.request.method).toBe("GET");
    req.flush(mockItems);

    await loadPromise;

    expect(service.items()).toEqual(mockItems);
    expect(service.loading()).toBeFalsy();
    expect(service.error()).toBeNull();
  });

  it("should handle load items error", async () => {
    const loadPromise = service.loadItems();

    const req = httpMock.expectOne("/api/items");
    req.flush("Error", { status: 500, statusText: "Internal Server Error" });

    try {
      await loadPromise;
    } catch (error) {
      expect(service.loading()).toBeFalsy();
      expect(service.error()).toBe("Failed to load items");
      expect(notificationService.showError).toHaveBeenCalledWith(
        "Failed to load items"
      );
    }
  });

  it("should filter items based on search term", () => {
    const mockItems = [
      { id: "1", name: "Apple", description: "Fruit", status: "active" },
      { id: "2", name: "Banana", description: "Fruit", status: "active" },
      { id: "3", name: "Carrot", description: "Vegetable", status: "active" },
    ];

    service["_items"].set(mockItems);
    service.setSearchTerm("apple");

    const filtered = service.filteredItems();
    expect(filtered).toHaveLength(1);
    expect(filtered[0].name).toBe("Apple");
  });

  it("should create item successfully", async () => {
    const newItemData = { name: "New Item", description: "New Description" };
    const createdItem = { id: "1", ...newItemData, status: "active" };

    const createPromise = service.createItem(newItemData);

    const req = httpMock.expectOne("/api/items");
    expect(req.request.method).toBe("POST");
    expect(req.request.body).toEqual(newItemData);
    req.flush(createdItem);

    const result = await createPromise;

    expect(result).toEqual(createdItem);
    expect(service.items()).toContain(createdItem);
    expect(notificationService.showSuccess).toHaveBeenCalledWith(
      "Item created successfully"
    );
  });

  it("should update item successfully", async () => {
    const existingItem = {
      id: "1",
      name: "Old Name",
      description: "Old Description",
      status: "active",
    };
    const updates = { name: "New Name" };
    const updatedItem = { ...existingItem, ...updates };

    service["_items"].set([existingItem]);

    const updatePromise = service.updateItem("1", updates);

    const req = httpMock.expectOne("/api/items/1");
    expect(req.request.method).toBe("PATCH");
    expect(req.request.body).toEqual(updates);
    req.flush(updatedItem);

    const result = await updatePromise;

    expect(result).toEqual(updatedItem);
    expect(service.items()[0]).toEqual(updatedItem);
    expect(notificationService.showSuccess).toHaveBeenCalledWith(
      "Item updated successfully"
    );
  });

  it("should delete item successfully", async () => {
    const existingItem = {
      id: "1",
      name: "Item 1",
      description: "Description 1",
      status: "active",
    };
    service["_items"].set([existingItem]);

    const deletePromise = service.deleteItem("1");

    const req = httpMock.expectOne("/api/items/1");
    expect(req.request.method).toBe("DELETE");
    req.flush({});

    await deletePromise;

    expect(service.items()).not.toContain(existingItem);
    expect(service.items()).toHaveLength(0);
    expect(notificationService.showSuccess).toHaveBeenCalledWith(
      "Item deleted successfully"
    );
  });

  it("should clear selected item when deleted", async () => {
    const existingItem = {
      id: "1",
      name: "Item 1",
      description: "Description 1",
      status: "active",
    };
    service["_items"].set([existingItem]);
    service.selectItem(existingItem);

    expect(service.selectedItem()).toBe(existingItem);

    const deletePromise = service.deleteItem("1");

    const req = httpMock.expectOne("/api/items/1");
    req.flush({});

    await deletePromise;

    expect(service.selectedItem()).toBeNull();
  });
});
```

## 📊 **Angular Version Comparison**

| Feature                   | Angular 14   | Angular 17    | Angular 19    |
| ------------------------- | ------------ | ------------- | ------------- |
| **Standalone Components** | Stable       | Enhanced      | Optimized     |
| **Bootstrap API**         | Basic        | Improved      | Full-featured |
| **Router Integration**    | Good         | Excellent     | Native        |
| **Testing Support**       | Manual setup | Simplified    | Streamlined   |
| **Bundle Size**           | Baseline     | 10% reduction | 15% reduction |

## 🎯 **Key Takeaways**

### **🏗️ Architecture Benefits:**

1. **Simplified Structure** - No NgModule boilerplate
2. **Better Tree Shaking** - Only import what you need
3. **Flexible Composition** - Mix and match components easily
4. **Micro-Frontend Ready** - Perfect for distributed architectures
5. **Testing Simplified** - Direct component imports

### **📋 Best Practices:**

1. **Use standalone for all new components**
2. **Organize by feature boundaries**
3. **Implement proper lazy loading**
4. **Create reusable component libraries**
5. **Follow consistent naming conventions**

### **🚨 Migration Strategy:**

1. **Start with leaf components** (no dependencies)
2. **Convert feature modules gradually**
3. **Update routing configuration**
4. **Refactor services for standalone compatibility**
5. **Update testing infrastructure**

Standalone components represent the future of Angular architecture - embrace them for cleaner, more maintainable applications! 🚀
