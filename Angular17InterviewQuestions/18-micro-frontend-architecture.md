# 🏗️ Micro-Frontend Architecture with Angular: Complete Guide

## 🎯 **Question Overview**

_"How do you implement micro-frontend architecture with Angular applications?"_

## 🔍 **Understanding Micro-Frontend Architecture**

Micro-frontend architecture is a **design approach** where a front-end application is decomposed into **individual, loosely coupled applications** that work together to form a cohesive user experience.

It's like having multiple **independent Angular applications** that can be developed, tested, and deployed separately while sharing a common shell! 🌟

## 🏛️ **Architectural Patterns**

### **1. 🗂️ Shell/Container Pattern**

```typescript
// shell-app/src/app/app.module.ts - Shell Application
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { RouterModule } from "@angular/router";
import { loadRemoteModule } from "@angular-architects/module-federation";

import { AppComponent } from "./app.component";
import { HeaderComponent } from "./shared/header/header.component";
import { NavigationComponent } from "./shared/navigation/navigation.component";
import { FooterComponent } from "./shared/footer/footer.component";

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    NavigationComponent,
    FooterComponent,
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(
      [
        {
          path: "products",
          loadChildren: () =>
            loadRemoteModule({
              type: "module",
              remoteEntry: "http://localhost:4201/remoteEntry.js",
              remoteName: "productsMfe",
              exposedModule: "./ProductsModule",
            }).then((m) => m.ProductsModule),
        },
        {
          path: "orders",
          loadChildren: () =>
            loadRemoteModule({
              type: "module",
              remoteEntry: "http://localhost:4202/remoteEntry.js",
              remoteName: "ordersMfe",
              exposedModule: "./OrdersModule",
            }).then((m) => m.OrdersModule),
        },
        {
          path: "users",
          loadChildren: () =>
            loadRemoteModule({
              type: "module",
              remoteEntry: "http://localhost:4203/remoteEntry.js",
              remoteName: "usersMfe",
              exposedModule: "./UsersModule",
            }).then((m) => m.UsersModule),
        },
        {
          path: "",
          redirectTo: "/products",
          pathMatch: "full",
        },
        {
          path: "**",
          loadChildren: () =>
            import("./not-found/not-found.module").then(
              (m) => m.NotFoundModule
            ),
        },
      ],
      {
        enableTracing: false,
        errorHandler: (error) => {
          console.error("Router error:", error);
        },
      }
    ),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

```typescript
// shell-app/src/app/app.component.ts - Main Shell Component
import { Component, OnInit, inject } from "@angular/core";
import { Router, NavigationEnd } from "@angular/router";
import { filter } from "rxjs/operators";

import { AuthService } from "./core/services/auth.service";
import { ThemeService } from "./core/services/theme.service";
import { NotificationService } from "./core/services/notification.service";
import { LoadingService } from "./core/services/loading.service";

@Component({
  selector: "shell-root",
  template: `
    <div class="app-shell" [class.dark-theme]="isDarkMode">
      <!-- Global Loading Indicator -->
      <div *ngIf="loadingService.isLoading$ | async" class="global-loading">
        <div class="spinner"></div>
      </div>

      <!-- Shell Header -->
      <shell-header
        [user]="authService.currentUser$ | async"
        [notifications]="notificationCount$ | async"
        (menuToggle)="toggleSidebar()"
        (profileClick)="openProfile()"
        (logout)="handleLogout()"
      >
      </shell-header>

      <!-- Main Content Area -->
      <div class="app-content" [class.sidebar-open]="sidebarOpen">
        <!-- Navigation Sidebar -->
        <shell-navigation
          [isOpen]="sidebarOpen"
          [currentRoute]="currentRoute"
          (navigationClick)="handleNavigation($event)"
          (closeSidebar)="closeSidebar()"
        >
        </shell-navigation>

        <!-- Micro-Frontend Content -->
        <main class="main-content" role="main">
          <!-- Breadcrumb -->
          <shell-breadcrumb [route]="currentRoute"></shell-breadcrumb>

          <!-- Router Outlet for Micro-Frontends -->
          <router-outlet></router-outlet>
        </main>
      </div>

      <!-- Global Footer -->
      <shell-footer></shell-footer>

      <!-- Global Notifications -->
      <shell-notifications></shell-notifications>

      <!-- Global Modals -->
      <shell-modal-container></shell-modal-container>
    </div>
  `,
  styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit {
  authService = inject(AuthService);
  themeService = inject(ThemeService);
  loadingService = inject(LoadingService);
  notificationService = inject(NotificationService);

  sidebarOpen = false;
  currentRoute = "";
  isDarkMode = false;

  notificationCount$ = this.notificationService.unreadCount$;

  constructor(private router: Router) {}

  ngOnInit(): void {
    // Track route changes
    this.router.events
      .pipe(filter((event) => event instanceof NavigationEnd))
      .subscribe((event: NavigationEnd) => {
        this.currentRoute = event.url;
        this.updatePageTitle(event.url);
      });

    // Subscribe to theme changes
    this.themeService.isDarkMode$.subscribe(
      (isDark) => (this.isDarkMode = isDark)
    );

    // Initialize shell services
    this.initializeShell();
  }

  private initializeShell(): void {
    // Initialize authentication
    this.authService.initializeAuth();

    // Load user preferences
    this.loadUserPreferences();

    // Setup global error handling
    this.setupGlobalErrorHandling();

    // Setup cross-MFE communication
    this.setupCrossMfeComm();
  }

  toggleSidebar(): void {
    this.sidebarOpen = !this.sidebarOpen;
  }

  closeSidebar(): void {
    this.sidebarOpen = false;
  }

  handleNavigation(route: string): void {
    this.router.navigate([route]);
    this.closeSidebar();
  }

  openProfile(): void {
    this.router.navigate(["/profile"]);
  }

  async handleLogout(): Promise<void> {
    try {
      await this.authService.logout();
      this.router.navigate(["/login"]);
    } catch (error) {
      this.notificationService.showError("Logout failed. Please try again.");
    }
  }

  private updatePageTitle(url: string): void {
    const titles: Record<string, string> = {
      "/products": "Products - E-Commerce Platform",
      "/orders": "Orders - E-Commerce Platform",
      "/users": "Users - E-Commerce Platform",
    };

    const title = titles[url] || "E-Commerce Platform";
    document.title = title;
  }

  private loadUserPreferences(): void {
    // Load theme preference
    const savedTheme = localStorage.getItem("user-theme");
    if (savedTheme) {
      this.themeService.setTheme(savedTheme as "light" | "dark");
    }

    // Load sidebar preference
    const sidebarPref = localStorage.getItem("sidebar-open");
    if (sidebarPref) {
      this.sidebarOpen = JSON.parse(sidebarPref);
    }
  }

  private setupGlobalErrorHandling(): void {
    // Global error handling for micro-frontends
    window.addEventListener("unhandledrejection", (event) => {
      console.error("Unhandled promise rejection:", event.reason);
      this.notificationService.showError("An unexpected error occurred.");
    });
  }

  private setupCrossMfeComm(): void {
    // Setup event bus for cross-MFE communication
    // This will be covered in the communication section
  }
}
```

### **2. 🔧 Module Federation Configuration**

```javascript
// shell-app/webpack.config.js - Shell Application Webpack Config
const ModuleFederationPlugin = require("@module-federation/webpack");

module.exports = {
  mode: "development",
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
  plugins: [
    new ModuleFederationPlugin({
      name: "shell",
      remotes: {
        productsMfe: "products@http://localhost:4201/remoteEntry.js",
        ordersMfe: "orders@http://localhost:4202/remoteEntry.js",
        usersMfe: "users@http://localhost:4203/remoteEntry.js",
        sharedMfe: "shared@http://localhost:4204/remoteEntry.js",
      },
      shared: {
        "@angular/core": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
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
        "@angular/common/http": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        rxjs: {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^7.8.0",
        },
        "rxjs/operators": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^7.8.0",
        },
        "@my-org/shared-lib": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^1.0.0",
        },
      },
    }),
  ],
};
```

```javascript
// products-mfe/webpack.config.js - Products Micro-Frontend
const ModuleFederationPlugin = require("@module-federation/webpack");

module.exports = {
  mode: "development",
  devServer: {
    port: 4201,
    historyApiFallback: true,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers":
        "X-Requested-With, content-type, Authorization",
    },
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "products",
      filename: "remoteEntry.js",
      exposes: {
        "./ProductsModule": "./src/app/products/products.module.ts",
        "./ProductService": "./src/app/products/services/product.service.ts",
        "./ProductListComponent":
          "./src/app/products/components/product-list/product-list.component.ts",
      },
      remotes: {
        shell: "shell@http://localhost:4200/remoteEntry.js",
        sharedMfe: "shared@http://localhost:4204/remoteEntry.js",
      },
      shared: {
        "@angular/core": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
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
        "@angular/common/http": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^17.0.0",
        },
        rxjs: {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^7.8.0",
        },
        "@my-org/shared-lib": {
          singleton: true,
          strictVersion: true,
          requiredVersion: "^1.0.0",
        },
      },
    }),
  ],
};
```

### **3. 📦 Micro-Frontend Module Structure**

```typescript
// products-mfe/src/app/products/products.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { RouterModule } from "@angular/router";
import { HttpClientModule } from "@angular/common/http";
import { ReactiveFormsModule } from "@angular/forms";

// Shared library
import { SharedUiModule } from "@my-org/shared-lib";

// Products components
import { ProductListComponent } from "./components/product-list/product-list.component";
import { ProductDetailComponent } from "./components/product-detail/product-detail.component";
import { ProductFormComponent } from "./components/product-form/product-form.component";
import { ProductCardComponent } from "./components/product-card/product-card.component";
import { ProductFilterComponent } from "./components/product-filter/product-filter.component";

// Products services
import { ProductService } from "./services/product.service";
import { ProductResolver } from "./services/product.resolver";
import { ProductGuard } from "./guards/product.guard";

// Products routes
const routes = [
  {
    path: "",
    component: ProductListComponent,
    data: { title: "Products" },
  },
  {
    path: "new",
    component: ProductFormComponent,
    canActivate: [ProductGuard],
    data: { title: "New Product" },
  },
  {
    path: ":id",
    component: ProductDetailComponent,
    resolve: { product: ProductResolver },
    data: { title: "Product Details" },
  },
  {
    path: ":id/edit",
    component: ProductFormComponent,
    canActivate: [ProductGuard],
    resolve: { product: ProductResolver },
    data: { title: "Edit Product" },
  },
];

@NgModule({
  declarations: [
    ProductListComponent,
    ProductDetailComponent,
    ProductFormComponent,
    ProductCardComponent,
    ProductFilterComponent,
  ],
  imports: [
    CommonModule,
    HttpClientModule,
    ReactiveFormsModule,
    SharedUiModule,
    RouterModule.forChild(routes),
  ],
  providers: [ProductService, ProductResolver, ProductGuard],
  exports: [
    // Export components for potential direct use
    ProductListComponent,
    ProductCardComponent,
  ],
})
export class ProductsModule {}
```

```typescript
// products-mfe/src/app/products/components/product-list/product-list.component.ts
import { Component, OnInit, signal, computed, inject } from "@angular/core";
import { Router } from "@angular/router";
import { Observable, BehaviorSubject, combineLatest } from "rxjs";
import {
  debounceTime,
  distinctUntilChanged,
  startWith,
  map,
} from "rxjs/operators";

import { ProductService } from "../../services/product.service";
import { EventBusService } from "@my-org/shared-lib";
import { LoadingService } from "@my-org/shared-lib";
import { NotificationService } from "@my-org/shared-lib";

export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  stock: number;
  rating: number;
  createdAt: Date;
  updatedAt: Date;
}

export interface ProductFilter {
  search?: string;
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  inStock?: boolean;
  sortBy?: "name" | "price" | "rating" | "created";
  sortOrder?: "asc" | "desc";
}

@Component({
  selector: "products-list",
  template: `
    <div class="products-page">
      <!-- Page Header -->
      <div class="page-header">
        <h1>Products</h1>
        <div class="header-actions">
          <button
            class="btn btn-primary"
            (click)="createProduct()"
            *ngIf="canCreateProduct"
          >
            <i class="icon-plus"></i>
            New Product
          </button>
        </div>
      </div>

      <!-- Filter & Search -->
      <products-filter
        [filter]="currentFilter()"
        [categories]="categories()"
        (filterChange)="updateFilter($event)"
        class="filter-section"
      >
      </products-filter>

      <!-- Products Grid -->
      <div class="products-container">
        <!-- Loading State -->
        <div *ngIf="loading()" class="loading-state">
          <div class="spinner"></div>
          <p>Loading products...</p>
        </div>

        <!-- Empty State -->
        <div *ngIf="!loading() && products().length === 0" class="empty-state">
          <i class="icon-box"></i>
          <h3>No products found</h3>
          <p>Try adjusting your filters or create a new product.</p>
          <button
            class="btn btn-primary"
            (click)="createProduct()"
            *ngIf="canCreateProduct"
          >
            Create Product
          </button>
        </div>

        <!-- Products Grid -->
        <div *ngIf="!loading() && products().length > 0" class="products-grid">
          <product-card
            *ngFor="let product of products(); trackBy: trackByProductId"
            [product]="product"
            [actions]="getProductActions(product)"
            (productClick)="viewProduct($event)"
            (actionClick)="handleProductAction($event)"
            class="product-item"
          >
          </product-card>
        </div>

        <!-- Pagination -->
        <div *ngIf="totalPages() > 1" class="pagination-container">
          <shared-pagination
            [currentPage]="currentPage()"
            [totalPages]="totalPages()"
            [pageSize]="pageSize()"
            [totalItems]="totalItems()"
            (pageChange)="changePage($event)"
          >
          </shared-pagination>
        </div>
      </div>
    </div>
  `,
  styleUrls: ["./product-list.component.scss"],
})
export class ProductListComponent implements OnInit {
  private productService = inject(ProductService);
  private eventBus = inject(EventBusService);
  private loadingService = inject(LoadingService);
  private notificationService = inject(NotificationService);
  private router = inject(Router);

  // Signals for state management
  products = signal<Product[]>([]);
  categories = signal<string[]>([]);
  loading = signal(false);
  currentFilter = signal<ProductFilter>({});
  currentPage = signal(1);
  pageSize = signal(20);
  totalItems = signal(0);
  canCreateProduct = signal(false);

  // Computed values
  totalPages = computed(() => Math.ceil(this.totalItems() / this.pageSize()));

  // Filter subjects
  private filterSubject = new BehaviorSubject<ProductFilter>({});
  private pageSubject = new BehaviorSubject<number>(1);

  ngOnInit(): void {
    this.loadCategories();
    this.setupFilterSubscription();
    this.setupEventBusListeners();
    this.checkPermissions();
  }

  private async loadCategories(): Promise<void> {
    try {
      const categories = await this.productService.getCategories();
      this.categories.set(categories);
    } catch (error) {
      console.error("Failed to load categories:", error);
    }
  }

  private setupFilterSubscription(): void {
    // Combine filter and pagination changes
    combineLatest([
      this.filterSubject.pipe(debounceTime(300), distinctUntilChanged()),
      this.pageSubject,
    ]).subscribe(([filter, page]) => {
      this.currentFilter.set(filter);
      this.currentPage.set(page);
      this.loadProducts(filter, page);
    });
  }

  private setupEventBusListeners(): void {
    // Listen for product updates from other micro-frontends
    this.eventBus.on("product:updated").subscribe((product: Product) => {
      this.updateProductInList(product);
    });

    this.eventBus.on("product:deleted").subscribe((productId: string) => {
      this.removeProductFromList(productId);
    });

    // Listen for cart updates
    this.eventBus.on("cart:item-added").subscribe((data) => {
      this.notificationService.showSuccess(`${data.productName} added to cart`);
    });
  }

  private async checkPermissions(): Promise<void> {
    try {
      const hasCreatePermission =
        await this.productService.hasCreatePermission();
      this.canCreateProduct.set(hasCreatePermission);
    } catch (error) {
      console.error("Failed to check permissions:", error);
    }
  }

  private async loadProducts(
    filter: ProductFilter,
    page: number
  ): Promise<void> {
    this.loading.set(true);

    try {
      const result = await this.productService.getProducts({
        ...filter,
        page,
        pageSize: this.pageSize(),
      });

      this.products.set(result.products);
      this.totalItems.set(result.totalCount);
    } catch (error) {
      console.error("Failed to load products:", error);
      this.notificationService.showError("Failed to load products");
      this.products.set([]);
    } finally {
      this.loading.set(false);
    }
  }

  updateFilter(filter: ProductFilter): void {
    this.filterSubject.next(filter);
    this.pageSubject.next(1); // Reset to first page
  }

  changePage(page: number): void {
    this.pageSubject.next(page);
  }

  viewProduct(product: Product): void {
    this.router.navigate(["/products", product.id]);
  }

  createProduct(): void {
    this.router.navigate(["/products/new"]);
  }

  handleProductAction(event: { action: string; product: Product }): void {
    switch (event.action) {
      case "edit":
        this.router.navigate(["/products", event.product.id, "edit"]);
        break;
      case "delete":
        this.deleteProduct(event.product);
        break;
      case "addToCart":
        this.addToCart(event.product);
        break;
      case "viewDetails":
        this.viewProduct(event.product);
        break;
    }
  }

  private async deleteProduct(product: Product): Promise<void> {
    const confirmed = confirm(
      `Are you sure you want to delete "${product.name}"?`
    );
    if (!confirmed) return;

    try {
      await this.productService.deleteProduct(product.id);
      this.removeProductFromList(product.id);
      this.notificationService.showSuccess("Product deleted successfully");

      // Notify other micro-frontends
      this.eventBus.emit("product:deleted", product.id);
    } catch (error) {
      console.error("Failed to delete product:", error);
      this.notificationService.showError("Failed to delete product");
    }
  }

  private addToCart(product: Product): void {
    // Emit event to cart micro-frontend
    this.eventBus.emit("cart:add-item", {
      productId: product.id,
      productName: product.name,
      price: product.price,
      quantity: 1,
    });
  }

  private updateProductInList(updatedProduct: Product): void {
    const currentProducts = this.products();
    const index = currentProducts.findIndex((p) => p.id === updatedProduct.id);

    if (index >= 0) {
      const newProducts = [...currentProducts];
      newProducts[index] = updatedProduct;
      this.products.set(newProducts);
    }
  }

  private removeProductFromList(productId: string): void {
    const currentProducts = this.products();
    const filteredProducts = currentProducts.filter((p) => p.id !== productId);
    this.products.set(filteredProducts);
  }

  getProductActions(
    product: Product
  ): Array<{ label: string; action: string; icon: string }> {
    const actions = [
      { label: "View Details", action: "viewDetails", icon: "icon-eye" },
      { label: "Add to Cart", action: "addToCart", icon: "icon-cart" },
    ];

    if (this.canCreateProduct()) {
      actions.push(
        { label: "Edit", action: "edit", icon: "icon-edit" },
        { label: "Delete", action: "delete", icon: "icon-trash" }
      );
    }

    return actions;
  }

  trackByProductId(index: number, product: Product): string {
    return product.id;
  }
}
```

## 🔄 **Cross-MFE Communication**

### **1. 📡 Event Bus Implementation**

```typescript
// shared-lib/src/lib/services/event-bus.service.ts
import { Injectable, signal } from "@angular/core";
import { Subject, Observable, filter, share, takeUntil } from "rxjs";

export interface EventBusEvent<T = any> {
  type: string;
  payload: T;
  timestamp: number;
  source?: string;
}

export interface EventSubscription {
  unsubscribe(): void;
}

@Injectable({
  providedIn: "root",
})
export class EventBusService {
  private eventSubject = new Subject<EventBusEvent>();
  private events$ = this.eventSubject.asObservable().pipe(share());

  // Debug mode for development
  private debugMode = signal(false);

  constructor() {
    if (this.debugMode()) {
      this.setupDebugLogging();
    }
  }

  /**
   * Emit an event to the event bus
   */
  emit<T>(type: string, payload: T, source?: string): void {
    const event: EventBusEvent<T> = {
      type,
      payload,
      timestamp: Date.now(),
      source: source || this.getCallerInfo(),
    };

    if (this.debugMode()) {
      console.log("EventBus: Emitting event", event);
    }

    this.eventSubject.next(event);
  }

  /**
   * Listen for events of a specific type
   */
  on<T>(eventType: string): Observable<T> {
    return this.events$.pipe(
      filter((event) => event.type === eventType),
      map((event) => event.payload as T)
    );
  }

  /**
   * Listen for events with pattern matching
   */
  onPattern<T>(pattern: string | RegExp): Observable<EventBusEvent<T>> {
    const matcher =
      typeof pattern === "string"
        ? (type: string) => type.includes(pattern)
        : (type: string) => pattern.test(type);

    return this.events$.pipe(
      filter((event) => matcher(event.type))
    ) as Observable<EventBusEvent<T>>;
  }

  /**
   * Listen for events until a stop condition is met
   */
  onUntil<T>(eventType: string, stopCondition: Observable<any>): Observable<T> {
    return this.on<T>(eventType).pipe(takeUntil(stopCondition));
  }

  /**
   * Once - listen for a single event occurrence
   */
  once<T>(eventType: string): Promise<T> {
    return new Promise((resolve) => {
      const subscription = this.on<T>(eventType).subscribe((payload) => {
        subscription.unsubscribe();
        resolve(payload);
      });
    });
  }

  /**
   * Subscribe with automatic cleanup
   */
  subscribe<T>(
    eventType: string,
    handler: (payload: T) => void,
    errorHandler?: (error: any) => void
  ): EventSubscription {
    const subscription = this.on<T>(eventType).subscribe({
      next: handler,
      error:
        errorHandler || ((error) => console.error("EventBus error:", error)),
    });

    return {
      unsubscribe: () => subscription.unsubscribe(),
    };
  }

  /**
   * Request-response pattern
   */
  async request<TRequest, TResponse>(
    requestType: string,
    payload: TRequest,
    timeout = 5000
  ): Promise<TResponse> {
    const requestId = this.generateRequestId();
    const responseType = `${requestType}:response:${requestId}`;

    // Setup response listener
    const responsePromise = Promise.race([
      this.once<TResponse>(responseType),
      this.createTimeoutPromise<TResponse>(timeout),
    ]);

    // Send request
    this.emit(`${requestType}:request`, {
      ...payload,
      requestId,
      responseType,
    });

    return responsePromise;
  }

  /**
   * Handle request-response pattern
   */
  onRequest<TRequest, TResponse>(
    requestType: string,
    handler: (payload: TRequest) => Promise<TResponse> | TResponse
  ): EventSubscription {
    return this.subscribe(`${requestType}:request`, async (data: any) => {
      try {
        const response = await handler(data);
        this.emit(data.responseType, response);
      } catch (error) {
        this.emit(`${data.responseType}:error`, { error: error.message });
      }
    });
  }

  /**
   * Enable debug mode
   */
  setDebugMode(enabled: boolean): void {
    this.debugMode.set(enabled);
    if (enabled) {
      this.setupDebugLogging();
    }
  }

  /**
   * Get event statistics
   */
  getStats(): { totalEvents: number; eventTypes: Record<string, number> } {
    // Implementation would track statistics
    return { totalEvents: 0, eventTypes: {} };
  }

  private setupDebugLogging(): void {
    this.events$.subscribe((event) => {
      console.log("EventBus: Event received", event);
    });
  }

  private getCallerInfo(): string {
    try {
      const stack = new Error().stack;
      const callerLine = stack?.split("\n")[3];
      return callerLine?.trim() || "unknown";
    } catch {
      return "unknown";
    }
  }

  private generateRequestId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private createTimeoutPromise<T>(timeout: number): Promise<T> {
    return new Promise((_, reject) => {
      setTimeout(() => {
        reject(new Error(`Request timeout after ${timeout}ms`));
      }, timeout);
    });
  }
}

// Event type definitions for type safety
export namespace Events {
  // Product events
  export interface ProductCreated {
    productId: string;
    product: any;
  }
  export interface ProductUpdated {
    productId: string;
    product: any;
  }
  export interface ProductDeleted {
    productId: string;
  }

  // Cart events
  export interface CartItemAdded {
    productId: string;
    quantity: number;
  }
  export interface CartItemRemoved {
    productId: string;
  }
  export interface CartCleared {}

  // User events
  export interface UserLoggedIn {
    userId: string;
    user: any;
  }
  export interface UserLoggedOut {}
  export interface UserUpdated {
    userId: string;
    user: any;
  }

  // Navigation events
  export interface NavigationRequested {
    path: string;
    params?: any;
  }
  export interface BreadcrumbUpdated {
    breadcrumbs: Array<{ label: string; path: string }>;
  }

  // Notification events
  export interface ShowNotification {
    type: "success" | "error" | "warning" | "info";
    message: string;
    duration?: number;
  }
}
```

### **2. 🔄 State Synchronization**

```typescript
// shared-lib/src/lib/services/shared-state.service.ts
import { Injectable, signal, computed, effect } from "@angular/core";
import { BehaviorSubject, Observable, distinctUntilChanged, map } from "rxjs";

import { EventBusService } from "./event-bus.service";

export interface SharedState {
  user: any;
  theme: "light" | "dark";
  cart: {
    items: Array<{ productId: string; quantity: number; price: number }>;
    total: number;
  };
  notifications: Array<{
    id: string;
    type: "success" | "error" | "warning" | "info";
    message: string;
    timestamp: number;
  }>;
}

const initialState: SharedState = {
  user: null,
  theme: "light",
  cart: { items: [], total: 0 },
  notifications: [],
};

@Injectable({
  providedIn: "root",
})
export class SharedStateService {
  private stateSubject = new BehaviorSubject<SharedState>(initialState);

  // Signals for reactive state
  private _state = signal<SharedState>(initialState);

  // Public readonly state
  readonly state = this._state.asReadonly();

  // Computed state slices
  readonly user = computed(() => this.state().user);
  readonly theme = computed(() => this.state().theme);
  readonly cart = computed(() => this.state().cart);
  readonly cartItemCount = computed(() =>
    this.state().cart.items.reduce((sum, item) => sum + item.quantity, 0)
  );
  readonly cartTotal = computed(() => this.state().cart.total);
  readonly notifications = computed(() => this.state().notifications);

  // Observable streams for backward compatibility
  readonly state$ = this.stateSubject.asObservable();
  readonly user$ = this.state$.pipe(
    map((state) => state.user),
    distinctUntilChanged()
  );
  readonly theme$ = this.state$.pipe(
    map((state) => state.theme),
    distinctUntilChanged()
  );
  readonly cart$ = this.state$.pipe(
    map((state) => state.cart),
    distinctUntilChanged()
  );

  constructor(private eventBus: EventBusService) {
    this.setupEventBusListeners();
    this.setupStateSync();
    this.loadPersistedState();
  }

  private setupEventBusListeners(): void {
    // User events
    this.eventBus.on<any>("user:login").subscribe((user) => {
      this.updateUser(user);
    });

    this.eventBus.on("user:logout").subscribe(() => {
      this.updateUser(null);
    });

    // Cart events
    this.eventBus.on<any>("cart:add-item").subscribe((item) => {
      this.addCartItem(item);
    });

    this.eventBus
      .on<{ productId: string }>("cart:remove-item")
      .subscribe(({ productId }) => {
        this.removeCartItem(productId);
      });

    this.eventBus.on("cart:clear").subscribe(() => {
      this.clearCart();
    });

    // Theme events
    this.eventBus.on<"light" | "dark">("theme:change").subscribe((theme) => {
      this.updateTheme(theme);
    });

    // Notification events
    this.eventBus.on<any>("notification:add").subscribe((notification) => {
      this.addNotification(notification);
    });
  }

  private setupStateSync(): void {
    // Sync signal with BehaviorSubject
    effect(() => {
      const currentState = this.state();
      this.stateSubject.next(currentState);
      this.persistState(currentState);
    });
  }

  // State update methods
  updateUser(user: any): void {
    this._state.update((state) => ({ ...state, user }));

    // Emit to other micro-frontends
    this.eventBus.emit("state:user-updated", user);
  }

  updateTheme(theme: "light" | "dark"): void {
    this._state.update((state) => ({ ...state, theme }));

    // Apply theme globally
    document.documentElement.setAttribute("data-theme", theme);

    // Emit to other micro-frontends
    this.eventBus.emit("state:theme-updated", theme);
  }

  addCartItem(item: {
    productId: string;
    quantity: number;
    price: number;
    name: string;
  }): void {
    this._state.update((state) => {
      const existingItem = state.cart.items.find(
        (i) => i.productId === item.productId
      );

      let newItems;
      if (existingItem) {
        newItems = state.cart.items.map((i) =>
          i.productId === item.productId
            ? { ...i, quantity: i.quantity + item.quantity }
            : i
        );
      } else {
        newItems = [...state.cart.items, item];
      }

      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);

      return {
        ...state,
        cart: { items: newItems, total },
      };
    });

    // Emit to other micro-frontends
    this.eventBus.emit("state:cart-updated", this.cart());
  }

  removeCartItem(productId: string): void {
    this._state.update((state) => {
      const newItems = state.cart.items.filter(
        (item) => item.productId !== productId
      );
      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);

      return {
        ...state,
        cart: { items: newItems, total },
      };
    });

    // Emit to other micro-frontends
    this.eventBus.emit("state:cart-updated", this.cart());
  }

  updateCartItemQuantity(productId: string, quantity: number): void {
    this._state.update((state) => {
      const newItems = state.cart.items.map((item) =>
        item.productId === productId ? { ...item, quantity } : item
      );
      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);

      return {
        ...state,
        cart: { items: newItems, total },
      };
    });

    // Emit to other micro-frontends
    this.eventBus.emit("state:cart-updated", this.cart());
  }

  clearCart(): void {
    this._state.update((state) => ({
      ...state,
      cart: { items: [], total: 0 },
    }));

    // Emit to other micro-frontends
    this.eventBus.emit("state:cart-updated", this.cart());
  }

  addNotification(notification: {
    type: "success" | "error" | "warning" | "info";
    message: string;
    duration?: number;
  }): void {
    const newNotification = {
      id: `notif_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
      timestamp: Date.now(),
      duration: 5000,
      ...notification,
    };

    this._state.update((state) => ({
      ...state,
      notifications: [...state.notifications, newNotification],
    }));

    // Auto-remove notification after duration
    if (newNotification.duration > 0) {
      setTimeout(() => {
        this.removeNotification(newNotification.id);
      }, newNotification.duration);
    }
  }

  removeNotification(id: string): void {
    this._state.update((state) => ({
      ...state,
      notifications: state.notifications.filter((n) => n.id !== id),
    }));
  }

  // State persistence
  private persistState(state: SharedState): void {
    try {
      const persistableState = {
        user: state.user,
        theme: state.theme,
        cart: state.cart,
        // Don't persist notifications
      };

      localStorage.setItem("shared-state", JSON.stringify(persistableState));
    } catch (error) {
      console.error("Failed to persist state:", error);
    }
  }

  private loadPersistedState(): void {
    try {
      const persisted = localStorage.getItem("shared-state");
      if (persisted) {
        const state = JSON.parse(persisted);
        this._state.update((current) => ({
          ...current,
          ...state,
          notifications: [], // Don't restore notifications
        }));
      }
    } catch (error) {
      console.error("Failed to load persisted state:", error);
    }
  }

  // Utility methods
  reset(): void {
    this._state.set(initialState);
  }

  getSnapshot(): SharedState {
    return { ...this.state() };
  }

  updatePartial(partialState: Partial<SharedState>): void {
    this._state.update((state) => ({ ...state, ...partialState }));
  }
}
```

## 🚀 **Deployment Strategy**

### **1. 🐳 Docker Configuration**

```dockerfile
# Shell Application Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build:shell

FROM nginx:alpine
COPY --from=builder /app/dist/shell /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```dockerfile
# Products MFE Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build:products

FROM nginx:alpine
COPY --from=builder /app/dist/products /usr/share/nginx/html
COPY nginx-mfe.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# docker-compose.yml - Development Environment
version: "3.8"

services:
  shell:
    build:
      context: ./shell-app
      dockerfile: Dockerfile
    ports:
      - "4200:80"
    environment:
      - NODE_ENV=development
    depends_on:
      - products-mfe
      - orders-mfe
      - users-mfe

  products-mfe:
    build:
      context: ./products-mfe
      dockerfile: Dockerfile
    ports:
      - "4201:80"
    environment:
      - NODE_ENV=development
      - API_BASE_URL=http://api:3000

  orders-mfe:
    build:
      context: ./orders-mfe
      dockerfile: Dockerfile
    ports:
      - "4202:80"
    environment:
      - NODE_ENV=development
      - API_BASE_URL=http://api:3000

  users-mfe:
    build:
      context: ./users-mfe
      dockerfile: Dockerfile
    ports:
      - "4203:80"
    environment:
      - NODE_ENV=development
      - API_BASE_URL=http://api:3000

  shared-mfe:
    build:
      context: ./shared-mfe
      dockerfile: Dockerfile
    ports:
      - "4204:80"
    environment:
      - NODE_ENV=development

  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/ecommerce
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=ecommerce
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### **2. ☸️ Kubernetes Deployment**

```yaml
# k8s/shell-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shell-app
  labels:
    app: shell-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shell-app
  template:
    metadata:
      labels:
        app: shell-app
    spec:
      containers:
        - name: shell-app
          image: my-registry/shell-app:latest
          ports:
            - containerPort: 80
          env:
            - name: NODE_ENV
              value: "production"
            - name: PRODUCTS_MFE_URL
              value: "https://products.example.com"
            - name: ORDERS_MFE_URL
              value: "https://orders.example.com"
            - name: USERS_MFE_URL
              value: "https://users.example.com"
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: shell-service
spec:
  selector:
    app: shell-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shell-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: shell-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: shell-service
                port:
                  number: 80
```

## 📊 **Angular Version Compatibility**

| Feature                   | Angular 15                            | Angular 17             | Angular 19           |
| ------------------------- | ------------------------------------- | ---------------------- | -------------------- |
| **Module Federation**     | @angular-architects/module-federation | Native webpack support | Enhanced performance |
| **Standalone Components** | Basic support                         | Full integration       | Optimized loading    |
| **Signal-based State**    | Not available                         | Experimental           | Production ready     |
| **SSR Compatibility**     | Limited                               | Good                   | Excellent            |
| **Bundle Size**           | Baseline                              | 15% smaller            | 25% smaller          |

## 🧪 **Testing Strategy**

### **1. ✅ Unit Testing**

```typescript
// shell-app/src/app/app.component.spec.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { Router } from "@angular/router";
import { of } from "rxjs";

import { AppComponent } from "./app.component";
import { AuthService } from "./core/services/auth.service";
import { ThemeService } from "./core/services/theme.service";
import { EventBusService } from "@my-org/shared-lib";

describe("AppComponent", () => {
  let component: AppComponent;
  let fixture: ComponentFixture<AppComponent>;
  let mockAuthService: jasmine.SpyObj<AuthService>;
  let mockThemeService: jasmine.SpyObj<ThemeService>;
  let mockEventBus: jasmine.SpyObj<EventBusService>;
  let mockRouter: jasmine.SpyObj<Router>;

  beforeEach(async () => {
    const authSpy = jasmine.createSpyObj(
      "AuthService",
      ["initializeAuth", "logout"],
      {
        currentUser$: of(null),
      }
    );
    const themeSpy = jasmine.createSpyObj("ThemeService", ["setTheme"], {
      isDarkMode$: of(false),
    });
    const eventBusSpy = jasmine.createSpyObj("EventBusService", ["on", "emit"]);
    const routerSpy = jasmine.createSpyObj("Router", ["navigate"], {
      events: of(),
    });

    await TestBed.configureTestingModule({
      declarations: [AppComponent],
      providers: [
        { provide: AuthService, useValue: authSpy },
        { provide: ThemeService, useValue: themeSpy },
        { provide: EventBusService, useValue: eventBusSpy },
        { provide: Router, useValue: routerSpy },
      ],
    }).compileComponents();

    mockAuthService = TestBed.inject(
      AuthService
    ) as jasmine.SpyObj<AuthService>;
    mockThemeService = TestBed.inject(
      ThemeService
    ) as jasmine.SpyObj<ThemeService>;
    mockEventBus = TestBed.inject(
      EventBusService
    ) as jasmine.SpyObj<EventBusService>;
    mockRouter = TestBed.inject(Router) as jasmine.SpyObj<Router>;

    fixture = TestBed.createComponent(AppComponent);
    component = fixture.componentInstance;
  });

  it("should initialize shell services", () => {
    component.ngOnInit();

    expect(mockAuthService.initializeAuth).toHaveBeenCalled();
  });

  it("should handle navigation correctly", () => {
    const testRoute = "/products";

    component.handleNavigation(testRoute);

    expect(mockRouter.navigate).toHaveBeenCalledWith([testRoute]);
    expect(component.sidebarOpen).toBeFalsy();
  });

  it("should handle logout", async () => {
    mockAuthService.logout.and.returnValue(Promise.resolve());

    await component.handleLogout();

    expect(mockAuthService.logout).toHaveBeenCalled();
    expect(mockRouter.navigate).toHaveBeenCalledWith(["/login"]);
  });
});
```

### **2. 🎯 Integration Testing**

```typescript
// e2e/micro-frontend-communication.e2e-spec.ts
import { test, expect } from "@playwright/test";

test.describe("Micro-Frontend Communication", () => {
  test("should communicate between shell and product MFE", async ({
    page,
    context,
  }) => {
    // Start with shell application
    await page.goto("http://localhost:4200");

    // Navigate to products
    await page.click('[data-testid="products-nav"]');

    // Verify products MFE loaded
    await expect(page.locator('[data-testid="products-list"]')).toBeVisible();

    // Add product to cart from products MFE
    await page.click('[data-testid="add-to-cart"]:first-child');

    // Verify cart counter updated in shell
    await expect(page.locator('[data-testid="cart-counter"]')).toHaveText("1");

    // Navigate to cart (different MFE)
    await page.click('[data-testid="cart-nav"]');

    // Verify cart items displayed
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(1);

    // Update quantity in cart MFE
    await page.fill('[data-testid="quantity-input"]', "2");
    await page.blur('[data-testid="quantity-input"]');

    // Verify cart counter updated in shell
    await expect(page.locator('[data-testid="cart-counter"]')).toHaveText("2");

    // Verify total price updated
    await expect(page.locator('[data-testid="cart-total"]')).toContainText("$");
  });

  test("should handle MFE loading errors gracefully", async ({ page }) => {
    // Simulate MFE unavailable
    await page.route("**/remoteEntry.js", (route) => route.abort());

    await page.goto("http://localhost:4200");

    // Try to navigate to unavailable MFE
    await page.click('[data-testid="products-nav"]');

    // Verify error handling
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-message"]')).toContainText(
      "Unable to load"
    );

    // Verify shell remains functional
    await expect(page.locator('[data-testid="shell-header"]')).toBeVisible();
    await expect(page.locator('[data-testid="shell-nav"]')).toBeVisible();
  });

  test("should maintain state across MFE navigation", async ({ page }) => {
    await page.goto("http://localhost:4200");

    // Login in shell
    await page.click('[data-testid="login-button"]');
    await page.fill('[data-testid="email-input"]', "user@example.com");
    await page.fill('[data-testid="password-input"]', "password");
    await page.click('[data-testid="submit-login"]');

    // Verify user logged in
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();

    // Navigate to products MFE
    await page.click('[data-testid="products-nav"]');

    // Verify user state maintained in products MFE
    await expect(page.locator('[data-testid="user-greeting"]')).toContainText(
      "Hello, user@example.com"
    );

    // Navigate to orders MFE
    await page.click('[data-testid="orders-nav"]');

    // Verify user state maintained in orders MFE
    await expect(page.locator('[data-testid="my-orders"]')).toBeVisible();

    // Logout from any MFE
    await page.click('[data-testid="user-menu"]');
    await page.click('[data-testid="logout-button"]');

    // Verify logout state propagated to all MFEs
    await expect(page.locator('[data-testid="login-button"]')).toBeVisible();
  });
});
```

## 🎯 **Key Takeaways**

### **🏗️ Architecture Benefits:**

1. **Independent Development** - Teams can work on separate micro-frontends
2. **Technology Flexibility** - Different MFEs can use different Angular versions
3. **Scalable Deployment** - Deploy and scale micro-frontends independently
4. **Fault Isolation** - One MFE failure doesn't crash entire application
5. **Code Reusability** - Shared libraries and components across MFEs

### **📋 Implementation Checklist:**

1. **Module Federation Setup** - Configure webpack for micro-frontend loading
2. **Shared State Management** - Implement cross-MFE communication
3. **Routing Strategy** - Design URL structure and navigation
4. **Error Handling** - Graceful degradation when MFEs fail
5. **Testing Strategy** - Unit, integration, and E2E testing approach

### **⚠️ Common Challenges:**

1. **Version Management** - Keep Angular versions in sync
2. **Bundle Size** - Optimize shared dependencies
3. **Development Experience** - Setup local development environment
4. **State Synchronization** - Maintain consistent state across MFEs
5. **Performance** - Lazy loading and code splitting optimization

Micro-frontend architecture with Angular provides powerful scalability and team independence - but requires careful planning and robust communication patterns! 🚀
