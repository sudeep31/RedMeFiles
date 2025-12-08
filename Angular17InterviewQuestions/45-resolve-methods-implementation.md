# 🔍 **Angular Resolve Methods: Complete Implementation Guide**

## 🎯 **What You'll Learn**

Master Angular Route Resolvers - the powerful mechanism for pre-loading data before route activation, with advanced patterns for enterprise applications.

---

## 📚 **The Basics: What are Resolvers?**

### **🤔 Understanding Route Resolvers**

**Route Resolvers** are services that fetch data **before** a route is activated. Think of them as **"data doorkeepers"** that ensure your components have everything they need before rendering.

**Without Resolver (Component handles loading):**

```typescript
// ❌ Component must handle loading states
@Component({
  template: `
    <div *ngIf="loading">Loading...</div>
    <div *ngIf="error">Error: {{ error }}</div>
    <div *ngIf="user">{{ user.name }}</div>
  `,
})
export class UserComponent implements OnInit {
  user: User | null = null;
  loading = true;
  error: string | null = null;

  ngOnInit() {
    this.userService.getUser(this.route.snapshot.params["id"]).subscribe({
      next: (user) => {
        this.user = user;
        this.loading = false;
      },
      error: (err) => {
        this.error = err.message;
        this.loading = false;
      },
    });
  }
}
```

**With Resolver (Data pre-loaded):**

```typescript
// ✅ Component gets data immediately
@Component({
  template: `<div>{{ user.name }}</div>`,
})
export class UserComponent implements OnInit {
  user!: User;

  ngOnInit() {
    // Data is already available!
    this.user = this.route.snapshot.data["user"];
  }
}
```

**Benefits of Resolvers:**

- ✅ **Cleaner Components** - No loading states in templates
- ✅ **Better UX** - Route doesn't activate until data is ready
- ✅ **Error Handling** - Centralized data loading errors
- ✅ **Route Guards** - Can prevent navigation if data fails
- ✅ **Caching** - Shared resolver logic across routes

---

## 🛠️ **Basic Resolver Implementation**

### **📝 Function-Based Resolver (Angular 14+)**

```typescript
// src/app/resolvers/user.resolver.ts
import { inject } from "@angular/core";
import { ResolveFn, ActivatedRouteSnapshot } from "@angular/router";
import { Observable, of, EMPTY } from "rxjs";
import { catchError } from "rxjs/operators";

import { UserService } from "../services/user.service";
import { NotificationService } from "../services/notification.service";

export interface User {
  id: string;
  name: string;
  email: string;
  avatar: string;
  department: string;
  role: string;
  lastLogin: Date;
  isActive: boolean;
}

// 👤 Single User Resolver
export const userResolver: ResolveFn<User | null> = (
  route: ActivatedRouteSnapshot
): Observable<User | null> => {
  const userService = inject(UserService);
  const notificationService = inject(NotificationService);

  const userId = route.paramMap.get("id");

  if (!userId) {
    console.warn("⚠️ No user ID provided in route");
    notificationService.showError("Invalid user ID");
    return of(null);
  }

  console.log(`🔍 Resolving user data for ID: ${userId}`);

  return userService.getUserById(userId).pipe(
    catchError((error) => {
      console.error("❌ Failed to resolve user:", error);
      notificationService.showError(`Failed to load user: ${error.message}`);

      // Return null instead of throwing to allow route to continue
      return of(null);
    })
  );
};

// 👥 Users List Resolver
export const usersListResolver: ResolveFn<User[]> = (): Observable<User[]> => {
  const userService = inject(UserService);
  const notificationService = inject(NotificationService);

  console.log("🔍 Resolving users list...");

  return userService.getUsers().pipe(
    catchError((error) => {
      console.error("❌ Failed to resolve users list:", error);
      notificationService.showError("Failed to load users list");

      // Return empty array on error
      return of([]);
    })
  );
};

// 📊 User Statistics Resolver
export const userStatsResolver: ResolveFn<any> = (): Observable<any> => {
  const userService = inject(UserService);

  console.log("📊 Resolving user statistics...");

  return userService.getUserStats().pipe(
    catchError((error) => {
      console.error("❌ Failed to resolve user stats:", error);

      // Return default stats on error
      return of({
        total: 0,
        active: 0,
        inactive: 0,
        departments: {},
      });
    })
  );
};

// Route configuration
const routes: Routes = [
  {
    path: "users",
    component: UsersListComponent,
    resolve: {
      users: usersListResolver,
      stats: userStatsResolver,
    },
  },
  {
    path: "users/:id",
    component: UserDetailComponent,
    resolve: {
      user: userResolver,
    },
  },
];
```

### **🏭 Class-Based Resolver (Traditional)**

```typescript
// src/app/resolvers/user-class.resolver.ts
import { Injectable } from "@angular/core";
import {
  Resolve,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router,
} from "@angular/router";
import { Observable, of, EMPTY, forkJoin } from "rxjs";
import { catchError, map, switchMap } from "rxjs/operators";

import { UserService } from "../services/user.service";
import { PermissionService } from "../services/permission.service";
import { LoadingService } from "../services/loading.service";

export interface UserDetailData {
  user: User;
  permissions: string[];
  recentActivity: any[];
  relatedUsers: User[];
}

@Injectable({ providedIn: "root" })
export class UserDetailResolver implements Resolve<UserDetailData | null> {
  constructor(
    private userService: UserService,
    private permissionService: PermissionService,
    private loadingService: LoadingService,
    private router: Router
  ) {}

  resolve(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<UserDetailData | null> {
    const userId = route.paramMap.get("id");

    if (!userId) {
      console.warn("⚠️ No user ID provided");
      this.router.navigate(["/users"]);
      return of(null);
    }

    console.log(`🔍 Resolving comprehensive user data for: ${userId}`);

    // Show global loading indicator
    this.loadingService.setLoading(true);

    // Load multiple data sources in parallel
    return forkJoin({
      user: this.userService.getUserById(userId),
      permissions: this.permissionService.getUserPermissions(userId),
      recentActivity: this.userService.getUserActivity(userId),
      relatedUsers: this.userService.getRelatedUsers(userId),
    }).pipe(
      map((data) => {
        console.log("✅ All user data resolved successfully");
        return data as UserDetailData;
      }),
      catchError((error) => {
        console.error("❌ Failed to resolve user detail data:", error);

        // Try to load just basic user data
        return this.userService.getUserById(userId).pipe(
          map(
            (user) =>
              ({
                user,
                permissions: [],
                recentActivity: [],
                relatedUsers: [],
              } as UserDetailData)
          ),
          catchError(() => {
            // If even basic data fails, redirect to users list
            this.router.navigate(["/users"]);
            return of(null);
          })
        );
      }),
      // Hide loading indicator when done
      switchMap((result) => {
        this.loadingService.setLoading(false);
        return of(result);
      })
    );
  }
}
```

---

## 🎭 **Advanced Resolver Patterns**

### **🔄 Conditional & Dynamic Resolvers**

```typescript
// src/app/resolvers/conditional.resolver.ts
import { inject } from "@angular/core";
import { ResolveFn, ActivatedRouteSnapshot, Router } from "@angular/router";
import { Observable, of, EMPTY } from "rxjs";
import { switchMap, catchError } from "rxjs/operators";

import { AuthService } from "../services/auth.service";
import { UserService } from "../services/user.service";
import { AdminService } from "../services/admin.service";

// 🎯 Conditional resolver based on user role
export const conditionalDataResolver: ResolveFn<any> = (
  route: ActivatedRouteSnapshot
): Observable<any> => {
  const authService = inject(AuthService);
  const userService = inject(UserService);
  const adminService = inject(AdminService);
  const router = inject(Router);

  const currentUser = authService.getCurrentUser();

  if (!currentUser) {
    console.warn("⚠️ No authenticated user");
    router.navigate(["/login"]);
    return EMPTY;
  }

  console.log(`🔍 Resolving data based on role: ${currentUser.role}`);

  // Different data based on user role
  switch (currentUser.role) {
    case "admin":
      return adminService
        .getAdminDashboardData()
        .pipe(
          catchError(() =>
            of({
              type: "admin",
              data: null,
              error: "Failed to load admin data",
            })
          )
        );

    case "manager":
      const departmentId = currentUser.departmentId;
      return userService
        .getDepartmentData(departmentId)
        .pipe(
          catchError(() =>
            of({
              type: "manager",
              data: null,
              error: "Failed to load department data",
            })
          )
        );

    case "employee":
      return userService
        .getEmployeeData(currentUser.id)
        .pipe(
          catchError(() =>
            of({
              type: "employee",
              data: null,
              error: "Failed to load employee data",
            })
          )
        );

    default:
      console.warn(`⚠️ Unknown role: ${currentUser.role}`);
      return of({ type: "unknown", data: null, error: "Unknown user role" });
  }
};

// 🔀 Multi-path resolver
export const multiPathResolver: ResolveFn<any> = (
  route: ActivatedRouteSnapshot
): Observable<any> => {
  const userService = inject(UserService);
  const path = route.url.map((segment) => segment.path).join("/");

  console.log(`🔍 Resolving data for path: ${path}`);

  // Different resolving logic based on route path
  if (path.includes("profile")) {
    const userId = route.paramMap.get("id")!;
    return userService.getUserProfile(userId);
  }

  if (path.includes("settings")) {
    const userId = route.paramMap.get("id")!;
    return userService.getUserSettings(userId);
  }

  if (path.includes("activity")) {
    const userId = route.paramMap.get("id")!;
    const days = parseInt(route.queryParamMap.get("days") || "30");
    return userService.getUserActivity(userId, days);
  }

  // Default fallback
  return of({ error: "Unknown path" });
};

// 🎨 Theme-aware resolver
export const themeDataResolver: ResolveFn<any> = (
  route: ActivatedRouteSnapshot
): Observable<any> => {
  const userService = inject(UserService);

  // Get theme from query params or localStorage
  const themeParam = route.queryParamMap.get("theme");
  const savedTheme = localStorage.getItem("user-theme");
  const activeTheme = themeParam || savedTheme || "default";

  console.log(`🎨 Resolving themed data for: ${activeTheme}`);

  return userService.getThemedData(activeTheme).pipe(
    catchError(() => {
      // Fallback to default theme data
      console.warn("⚠️ Failed to load themed data, using default");
      return userService.getThemedData("default");
    })
  );
};
```

### **📊 Cached & Performance-Optimized Resolvers**

```typescript
// src/app/resolvers/cached.resolver.ts
import { inject } from "@angular/core";
import { ResolveFn } from "@angular/router";
import { Observable, of, timer } from "rxjs";
import { switchMap, tap, catchError, shareReplay } from "rxjs/operators";

import { UserService } from "../services/user.service";
import { CacheService } from "../services/cache.service";

interface CacheEntry<T> {
  data: T;
  timestamp: Date;
  expiresAt: Date;
}

// 💾 Cached resolver with TTL
export const cachedUsersResolver: ResolveFn<User[]> = (): Observable<
  User[]
> => {
  const userService = inject(UserService);
  const cacheService = inject(CacheService);

  const cacheKey = "users_list";
  const cacheTTL = 5 * 60 * 1000; // 5 minutes

  console.log("🔍 Checking cache for users list...");

  // Check cache first
  const cachedData = cacheService.get<User[]>(cacheKey);

  if (cachedData) {
    console.log("✅ Serving users from cache");
    return of(cachedData);
  }

  console.log("📡 Fetching fresh users data...");

  // Fetch fresh data and cache it
  return userService.getUsers().pipe(
    tap((users) => {
      cacheService.set(cacheKey, users, cacheTTL);
      console.log(`💾 Cached ${users.length} users for ${cacheTTL}ms`);
    }),
    catchError((error) => {
      console.error("❌ Failed to fetch users:", error);

      // Try to serve stale cache data if available
      const staleData = cacheService.getStale<User[]>(cacheKey);
      if (staleData) {
        console.warn("⚠️ Serving stale cache data due to error");
        return of(staleData);
      }

      return of([]);
    }),
    shareReplay(1) // Share result among concurrent requests
  );
};

// 🔄 Smart refresh resolver
export const smartRefreshResolver: ResolveFn<any> = (): Observable<any> => {
  const userService = inject(UserService);
  const cacheService = inject(CacheService);

  const cacheKey = "dashboard_data";
  const refreshInterval = 30 * 1000; // 30 seconds
  const staleThreshold = 60 * 1000; // 1 minute

  // Check if data is stale
  const lastRefresh = cacheService.getTimestamp(cacheKey);
  const now = Date.now();
  const isStale = !lastRefresh || now - lastRefresh > staleThreshold;

  if (!isStale) {
    // Data is fresh, serve from cache
    const cachedData = cacheService.get(cacheKey);
    if (cachedData) {
      console.log("✅ Serving fresh cached data");
      return of(cachedData);
    }
  }

  console.log("🔄 Refreshing dashboard data...");

  // Fetch fresh data
  return userService.getDashboardData().pipe(
    tap((data) => {
      cacheService.set(cacheKey, data);

      // Schedule next refresh
      timer(refreshInterval).subscribe(() => {
        console.log("⏰ Scheduled refresh triggered");
        userService.getDashboardData().subscribe((freshData) => {
          cacheService.set(cacheKey, freshData);
        });
      });
    }),
    catchError((error) => {
      console.error("❌ Failed to refresh dashboard:", error);

      // Serve stale data if available
      const staleData = cacheService.get(cacheKey);
      return of(staleData || { error: "No data available" });
    })
  );
};

// 🎯 Preemptive resolver (loads related data)
export const preemptiveResolver: ResolveFn<any> = (route): Observable<any> => {
  const userService = inject(UserService);

  const userId = route.paramMap.get("id")!;

  console.log(`🎯 Preemptively loading data for user: ${userId}`);

  // Load current user data
  const userData$ = userService.getUserById(userId);

  // Also preload related data that user might navigate to
  const relatedData$ = userData$.pipe(
    switchMap((user) => {
      // Preload department colleagues
      const colleagues$ = userService.getDepartmentUsers(user.departmentId);

      // Preload user's recent projects
      const projects$ = userService.getUserProjects(userId);

      // Preload notifications
      const notifications$ = userService.getUserNotifications(userId);

      return of({
        user,
        preloaded: {
          colleagues: colleagues$,
          projects: projects$,
          notifications: notifications$,
        },
      });
    })
  );

  return relatedData$.pipe(
    tap((data) => {
      // Start preloading in background (don't block route)
      data.preloaded.colleagues.subscribe((colleagues) => {
        cacheService.set(`colleagues_${userId}`, colleagues);
      });

      data.preloaded.projects.subscribe((projects) => {
        cacheService.set(`projects_${userId}`, projects);
      });

      data.preloaded.notifications.subscribe((notifications) => {
        cacheService.set(`notifications_${userId}`, notifications);
      });

      console.log("💾 Preloaded related data in background");
    }),
    map((data) => data.user), // Return only main user data
    catchError((error) => {
      console.error("❌ Preemptive loading failed:", error);
      return of(null);
    })
  );
};
```

### **🔐 Security & Permission-Aware Resolvers**

```typescript
// src/app/resolvers/secure.resolver.ts
import { inject } from '@angular/core';
import { ResolveFn, ActivatedRouteSnapshot, Router } from '@angular/router';
import { Observable, of, EMPTY, throwError } from 'rxjs';
import { switchMap, catchError, map } from 'rxjs/operators';

import { AuthService } from '../services/auth.service';
import { PermissionService } from '../services/permission.service';
import { UserService } from '../services/user.service';
import { AuditService } from '../services/audit.service';

// 🔐 Permission-based resolver
export const secureUserResolver: ResolveFn<User | null> = (
  route: ActivatedRouteSnapshot
): Observable<User | null> => {

  const authService = inject(AuthService);
  const permissionService = inject(PermissionService);
  const userService = inject(UserService);
  const auditService = inject(AuditService);
  const router = inject(Router);

  const targetUserId = route.paramMap.get('id')!;
  const currentUser = authService.getCurrentUser();

  if (!currentUser) {
    console.warn('⚠️ No authenticated user');
    router.navigate(['/login']);
    return EMPTY;
  }

  console.log(`🔐 Checking permissions for user access: ${targetUserId}`);

  // Check if user can access target user's data
  return permissionService.canAccessUser(currentUser.id, targetUserId).pipe(
    switchMap(canAccess => {
      if (!canAccess) {
        console.warn(`🚫 Access denied to user: ${targetUserId}`);

        // Log security violation
        auditService.logSecurityViolation({
          userId: currentUser.id,
          action: 'attempted_user_access',
          targetUserId,
          timestamp: new Date(),
          ip: this.getClientIP(),
          userAgent: navigator.userAgent
        });

        router.navigate(['/access-denied']);
        return of(null);
      }

      // Permission granted, load user data
      return userService.getUserById(targetUserId).pipe(
        tap(user => {
          // Log successful access for audit
          auditService.logDataAccess({
            userId: currentUser.id,
            action: 'user_data_accessed',
            targetUserId,
            timestamp: new Date(),
            dataType: 'user_profile'
          });
        }),
        catchError(error => {
          console.error(`❌ Failed to load user ${targetUserId}:`, error);

          // Log access failure
          auditService.logDataAccessFailure({
            userId: currentUser.id,
            action: 'user_data_access_failed',
            targetUserId,
            error: error.message,
            timestamp: new Date()
          });

          return of(null);
        })
      );
    })
  );

  private getClientIP(): string {
    // Implementation would get real client IP
    return 'unknown';
  }
};

// 👮 Role-based resolver
export const roleBasedResolver: ResolveFn<any> = (
  route: ActivatedRouteSnapshot
): Observable<any> => {

  const authService = inject(AuthService);
  const userService = inject(UserService);
  const router = inject(Router);

  const requiredRole = route.data['requiredRole'] as string;
  const requiredPermissions = route.data['requiredPermissions'] as string[];

  const currentUser = authService.getCurrentUser();

  if (!currentUser) {
    router.navigate(['/login']);
    return EMPTY;
  }

  // Check role requirement
  if (requiredRole && !authService.hasRole(currentUser, requiredRole)) {
    console.warn(`🚫 Insufficient role. Required: ${requiredRole}, Current: ${currentUser.role}`);
    router.navigate(['/access-denied'], {
      queryParams: { reason: 'insufficient_role' }
    });
    return EMPTY;
  }

  // Check permission requirements
  if (requiredPermissions?.length > 0) {
    const hasPermissions = requiredPermissions.every(permission =>
      authService.hasPermission(currentUser, permission)
    );

    if (!hasPermissions) {
      console.warn(`🚫 Insufficient permissions. Required: ${requiredPermissions.join(', ')}`);
      router.navigate(['/access-denied'], {
        queryParams: { reason: 'insufficient_permissions' }
      });
      return EMPTY;
    }
  }

  console.log('✅ Security checks passed, loading data...');

  // Load role-specific data
  const dataLoader = this.getDataLoaderForRole(currentUser.role);
  return dataLoader().pipe(
    catchError(error => {
      console.error('❌ Failed to load role-specific data:', error);
      return of({ error: error.message });
    })
  );

  private getDataLoaderForRole(role: string): () => Observable<any> {
    const userService = inject(UserService);

    switch (role) {
      case 'admin':
        return () => userService.getAdminData();
      case 'manager':
        return () => userService.getManagerData();
      case 'employee':
        return () => userService.getEmployeeData();
      default:
        return () => of({ error: 'Unknown role' });
    }
  }
};

// 🕐 Time-based access resolver
export const timeBasedResolver: ResolveFn<any> = (): Observable<any> => {

  const authService = inject(AuthService);
  const userService = inject(UserService);
  const router = inject(Router);

  const currentUser = authService.getCurrentUser();
  const now = new Date();
  const currentHour = now.getHours();
  const isWeekend = [0, 6].includes(now.getDay()); // Sunday or Saturday

  // Business hours: 9 AM - 6 PM, Monday-Friday
  const isBusinessHours = currentHour >= 9 && currentHour < 18 && !isWeekend;

  console.log(`🕐 Time-based access check: ${now.toLocaleString()}, Business hours: ${isBusinessHours}`);

  // Check if user has after-hours access
  if (!isBusinessHours && !authService.hasPermission(currentUser, 'after_hours_access')) {
    console.warn('🕐 Access denied: Outside business hours');
    router.navigate(['/access-denied'], {
      queryParams: { reason: 'outside_business_hours' }
    });
    return EMPTY;
  }

  // Load appropriate data based on time
  if (isBusinessHours) {
    return userService.getBusinessHoursData();
  } else {
    return userService.getAfterHoursData();
  }
};
```

---

## 🎯 **Component Integration Examples**

### **📱 Component Using Multiple Resolvers**

```typescript
// src/app/components/user-detail/user-detail.component.ts
import { Component, OnInit } from "@angular/core";
import { ActivatedRoute, Router } from "@angular/router";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";

@Component({
  selector: "app-user-detail",
  template: `
    <div class="user-detail-container">
      <div class="user-header">
        <img [src]="user.avatar" [alt]="user.name" class="user-avatar" />
        <div class="user-info">
          <h1>{{ user.name }}</h1>
          <p class="user-email">{{ user.email }}</p>
          <div class="user-meta">
            <span class="department">{{ user.department }}</span>
            <span class="role">{{ user.role }}</span>
            <span
              class="status"
              [class]="user.isActive ? 'active' : 'inactive'"
            >
              {{ user.isActive ? "Active" : "Inactive" }}
            </span>
          </div>
        </div>

        <div class="user-actions">
          <button class="btn btn-primary" (click)="editUser()">✏️ Edit</button>
          <button class="btn btn-secondary" (click)="viewActivity()">
            📊 Activity
          </button>
        </div>
      </div>

      <div class="user-content">
        <div class="section">
          <h3>🔐 Permissions</h3>
          <div class="permissions-list">
            <span *ngFor="let permission of permissions" class="permission-tag">
              {{ permission }}
            </span>
          </div>
        </div>

        <div class="section">
          <h3>📈 Recent Activity</h3>
          <div class="activity-list">
            <div *ngFor="let activity of recentActivity" class="activity-item">
              <span class="activity-time">{{
                activity.timestamp | date : "short"
              }}</span>
              <span class="activity-action">{{ activity.action }}</span>
            </div>
          </div>
        </div>

        <div class="section">
          <h3>👥 Related Users</h3>
          <div class="related-users">
            <div
              *ngFor="let relatedUser of relatedUsers"
              class="related-user"
              (click)="navigateToUser(relatedUser.id)"
            >
              <img
                [src]="relatedUser.avatar"
                [alt]="relatedUser.name"
                class="mini-avatar"
              />
              <span class="related-name">{{ relatedUser.name }}</span>
              <span class="related-role">{{ relatedUser.role }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Debug info -->
      <div class="debug-info" *ngIf="showDebug">
        <h4>🔧 Resolver Debug Info</h4>
        <pre>{{ resolverData | json }}</pre>
      </div>
    </div>
  `,
  styles: [
    `
      .user-detail-container {
        max-width: 1000px;
        margin: 0 auto;
        padding: 20px;
      }

      .user-header {
        display: flex;
        align-items: center;
        gap: 20px;
        background: white;
        padding: 30px;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        margin-bottom: 30px;
      }

      .user-avatar {
        width: 100px;
        height: 100px;
        border-radius: 50%;
        border: 4px solid #e9ecef;
      }

      .user-info {
        flex: 1;
      }

      .user-info h1 {
        margin: 0 0 8px 0;
        color: #333;
      }

      .user-email {
        color: #666;
        margin: 0 0 12px 0;
      }

      .user-meta {
        display: flex;
        gap: 12px;
      }

      .user-meta span {
        padding: 4px 12px;
        border-radius: 16px;
        font-size: 0.875rem;
        font-weight: 500;
      }

      .department {
        background: #e3f2fd;
        color: #1976d2;
      }

      .role {
        background: #f3e5f5;
        color: #7b1fa2;
      }

      .status.active {
        background: #e8f5e8;
        color: #2e7d32;
      }

      .status.inactive {
        background: #ffebee;
        color: #c62828;
      }

      .user-actions {
        display: flex;
        flex-direction: column;
        gap: 10px;
      }

      .user-content {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 24px;
      }

      .section {
        background: white;
        padding: 24px;
        border-radius: 12px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      }

      .section h3 {
        margin: 0 0 16px 0;
        color: #333;
      }

      .permissions-list {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
      }

      .permission-tag {
        background: #f8f9fa;
        color: #495057;
        padding: 6px 12px;
        border-radius: 8px;
        font-size: 0.875rem;
        border: 1px solid #e9ecef;
      }

      .activity-list {
        max-height: 300px;
        overflow-y: auto;
      }

      .activity-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 12px 0;
        border-bottom: 1px solid #f0f0f0;
      }

      .activity-item:last-child {
        border-bottom: none;
      }

      .activity-time {
        color: #666;
        font-size: 0.875rem;
      }

      .related-users {
        display: flex;
        flex-direction: column;
        gap: 12px;
      }

      .related-user {
        display: flex;
        align-items: center;
        gap: 12px;
        padding: 12px;
        border-radius: 8px;
        cursor: pointer;
        transition: background 0.2s ease;
      }

      .related-user:hover {
        background: #f8f9fa;
      }

      .mini-avatar {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        border: 2px solid #e9ecef;
      }

      .related-name {
        font-weight: 600;
        color: #333;
      }

      .related-role {
        color: #666;
        font-size: 0.875rem;
      }

      .btn {
        padding: 10px 20px;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        font-weight: 600;
        transition: all 0.2s ease;
      }

      .btn-primary {
        background: #007bff;
        color: white;
      }

      .btn-secondary {
        background: #6c757d;
        color: white;
      }

      .debug-info {
        margin-top: 30px;
        padding: 20px;
        background: #f8f9fa;
        border-radius: 8px;
      }

      .debug-info pre {
        background: #e9ecef;
        padding: 12px;
        border-radius: 4px;
        font-size: 0.75rem;
        max-height: 300px;
        overflow: auto;
      }
    `,
  ],
})
export class UserDetailComponent implements OnInit {
  // Data from resolvers
  user!: User;
  permissions: string[] = [];
  recentActivity: any[] = [];
  relatedUsers: User[] = [];

  // Debug info
  resolverData: any = {};
  showDebug = false;

  constructor(private route: ActivatedRoute, private router: Router) {}

  ngOnInit(): void {
    // Get resolved data
    this.loadResolvedData();

    // Enable debug mode based on query param
    this.showDebug = this.route.snapshot.queryParamMap.get("debug") === "true";
  }

  private loadResolvedData(): void {
    const data = this.route.snapshot.data;

    // Handle different resolver data structures
    if (data["userDetail"]) {
      // Using UserDetailResolver (returns complete object)
      const userDetail = data["userDetail"] as UserDetailData;
      this.user = userDetail.user;
      this.permissions = userDetail.permissions;
      this.recentActivity = userDetail.recentActivity;
      this.relatedUsers = userDetail.relatedUsers;
    } else {
      // Using individual resolvers
      this.user = data["user"] as User;
      this.permissions = data["permissions"] || [];
      this.recentActivity = data["activity"] || [];
      this.relatedUsers = data["relatedUsers"] || [];
    }

    // Store debug info
    this.resolverData = {
      resolvedAt: new Date(),
      dataKeys: Object.keys(data),
      dataSize: JSON.stringify(data).length,
      user: this.user ? { id: this.user.id, name: this.user.name } : null,
      permissionsCount: this.permissions.length,
      activityCount: this.recentActivity.length,
      relatedUsersCount: this.relatedUsers.length,
    };

    console.log("📊 User detail data loaded:", this.resolverData);
  }

  editUser(): void {
    this.router.navigate(["edit"], { relativeTo: this.route });
  }

  viewActivity(): void {
    this.router.navigate(["activity"], { relativeTo: this.route });
  }

  navigateToUser(userId: string): void {
    this.router.navigate(["/users", userId]);
  }
}

// Route configuration with multiple resolvers
const userRoutes: Routes = [
  {
    path: "users/:id",
    component: UserDetailComponent,
    resolve: {
      userDetail: UserDetailResolver, // Class-based comprehensive resolver
    },
    data: {
      requiredRole: "user",
      requiredPermissions: ["read_user_data"],
    },
  },
  // Alternative with individual resolvers
  {
    path: "users/:id/profile",
    component: UserDetailComponent,
    resolve: {
      user: userResolver,
      permissions: () =>
        inject(PermissionService).getUserPermissions(
          inject(ActivatedRoute).snapshot.params["id"]
        ),
      activity: () =>
        inject(UserService).getUserActivity(
          inject(ActivatedRoute).snapshot.params["id"]
        ),
      relatedUsers: () =>
        inject(UserService).getRelatedUsers(
          inject(ActivatedRoute).snapshot.params["id"]
        ),
    },
  },
];
```

---

## 🎉 **Summary: Resolve Methods Mastery**

### **✅ What We've Covered:**

🔍 **Basic Resolvers** - Function and class-based implementations  
🎭 **Advanced Patterns** - Conditional, cached, and performance-optimized resolvers  
🔐 **Security Resolvers** - Permission-based and role-aware data loading  
📱 **Component Integration** - Using resolved data effectively  
🛠️ **Error Handling** - Graceful failure and fallback strategies

### **🎯 Best Practices:**

- ✅ **Use Function Resolvers** - Simpler and more testable (Angular 14+)
- ✅ **Handle Errors Gracefully** - Return fallback data instead of throwing
- ✅ **Cache When Appropriate** - Avoid redundant API calls
- ✅ **Security First** - Always validate permissions before data access
- ✅ **Performance Monitoring** - Log resolver timing and failures
- ✅ **Preload Related Data** - Improve user experience with background loading

**You're now ready to master data pre-loading in Angular!** 🚀
