# ⚡ **Angular Signals: Complete Implementation Guide**

## 🎯 **What You'll Learn**

Master Angular Signals - the modern reactive primitive that simplifies state management and improves performance with automatic change detection and declarative reactive programming.

---

## 📚 **The Basics: What are Signals?**

### **🤔 Understanding Signals**

**Signals** are reactive primitives that hold values and notify consumers when the value changes. They provide a simpler, more performant alternative to RxJS for many use cases.

**Traditional Reactive Approach (RxJS):**

```typescript
// ❌ Complex: Multiple observables and subscriptions
export class TraditionalComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  users$ = new BehaviorSubject<User[]>([]);
  loading$ = new BehaviorSubject<boolean>(false);
  error$ = new BehaviorSubject<string | null>(null);

  filteredUsers$ = combineLatest([this.users$, this.searchTerm$]).pipe(
    map(([users, term]) => users.filter((u) => u.name.includes(term))),
    takeUntil(this.destroy$)
  );

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Modern Signals Approach:**

```typescript
// ✅ Simple: Clean, declarative, and automatic cleanup
export class ModernSignalComponent {
  // 📡 Signal state
  users = signal<User[]>([]);
  loading = signal<boolean>(false);
  error = signal<string | null>(null);
  searchTerm = signal<string>("");

  // 🔄 Computed signals (automatically update)
  filteredUsers = computed(() =>
    this.users().filter((user) =>
      user.name.toLowerCase().includes(this.searchTerm().toLowerCase())
    )
  );

  userCount = computed(() => this.filteredUsers().length);

  // ✨ No manual subscription management needed!
}
```

**Benefits of Signals:**

- ✅ **Automatic Change Detection** - Only updates what actually changed
- ✅ **No Memory Leaks** - Automatic cleanup
- ✅ **Type Safety** - Full TypeScript support
- ✅ **Performance** - Fine-grained reactivity
- ✅ **Simplicity** - Less boilerplate code

---

## 🛠️ **Core Signal Types**

### **📡 Signal (Writable)**

```typescript
// src/app/services/user-signals.service.ts
import { Injectable, signal, computed, effect } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class UserSignalsService {
  // 📡 Basic signals (writable)
  users = signal<User[]>([]);
  loading = signal<boolean>(false);
  error = signal<string | null>(null);
  selectedUserId = signal<string | null>(null);

  // 🔄 Update signals
  setUsers(users: User[]): void {
    this.users.set(users);
    console.log("👥 Users updated:", users.length);
  }

  addUser(user: User): void {
    this.users.update((currentUsers) => [...currentUsers, user]);
    console.log("➕ User added:", user.name);
  }

  removeUser(userId: string): void {
    this.users.update((currentUsers) =>
      currentUsers.filter((u) => u.id !== userId)
    );
    console.log("🗑️ User removed:", userId);
  }

  updateUser(userId: string, updates: Partial<User>): void {
    this.users.update((currentUsers) =>
      currentUsers.map((user) =>
        user.id === userId ? { ...user, ...updates } : user
      )
    );
    console.log("🔄 User updated:", userId);
  }

  setLoading(loading: boolean): void {
    this.loading.set(loading);
  }

  setError(error: string | null): void {
    this.error.set(error);
  }

  selectUser(userId: string | null): void {
    this.selectedUserId.set(userId);
  }
}
```

### **🧮 Computed Signals**

```typescript
// src/app/services/user-computed.service.ts
import { Injectable, signal, computed } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class UserComputedService {
  // 📡 Base signals
  users = signal<User[]>([]);
  searchTerm = signal<string>("");
  filterStatus = signal<"all" | "active" | "inactive">("all");
  sortBy = signal<"name" | "email" | "createdAt">("name");
  sortDirection = signal<"asc" | "desc">("asc");

  // 🧮 Computed signals (automatically derived)

  // Filter users by search term
  searchedUsers = computed(() => {
    const users = this.users();
    const term = this.searchTerm().toLowerCase();

    if (!term) return users;

    return users.filter(
      (user) =>
        user.name.toLowerCase().includes(term) ||
        user.email.toLowerCase().includes(term)
    );
  });

  // Filter by status
  filteredUsers = computed(() => {
    const users = this.searchedUsers();
    const status = this.filterStatus();

    if (status === "all") return users;

    return users.filter((user) => user.status === status);
  });

  // Sort users
  sortedUsers = computed(() => {
    const users = [...this.filteredUsers()];
    const sortBy = this.sortBy();
    const direction = this.sortDirection();

    return users.sort((a, b) => {
      let aValue = a[sortBy];
      let bValue = b[sortBy];

      // Handle dates
      if (sortBy === "createdAt") {
        aValue = new Date(aValue).getTime();
        bValue = new Date(bValue).getTime();
      }

      // Handle strings
      if (typeof aValue === "string") {
        aValue = aValue.toLowerCase();
        bValue = bValue.toLowerCase();
      }

      const comparison = aValue < bValue ? -1 : aValue > bValue ? 1 : 0;
      return direction === "asc" ? comparison : -comparison;
    });
  });

  // Statistics (computed from filtered data)
  userStats = computed(() => {
    const users = this.sortedUsers();
    const activeUsers = users.filter((u) => u.status === "active");
    const inactiveUsers = users.filter((u) => u.status === "inactive");

    return {
      total: users.length,
      active: activeUsers.length,
      inactive: inactiveUsers.length,
      activePercentage:
        users.length > 0
          ? Math.round((activeUsers.length / users.length) * 100)
          : 0,
    };
  });

  // User groups by first letter
  userGroups = computed(() => {
    const users = this.sortedUsers();
    const groups = new Map<string, User[]>();

    users.forEach((user) => {
      const firstLetter = user.name.charAt(0).toUpperCase();
      const group = groups.get(firstLetter) || [];
      group.push(user);
      groups.set(firstLetter, group);
    });

    return Array.from(groups.entries())
      .map(([letter, users]) => ({ letter, users }))
      .sort((a, b) => a.letter.localeCompare(b.letter));
  });

  // Selected user details
  selectedUserId = signal<string | null>(null);

  selectedUser = computed(() => {
    const userId = this.selectedUserId();
    const users = this.users();

    return userId ? users.find((u) => u.id === userId) || null : null;
  });

  // Recently active users (last 7 days)
  recentlyActiveUsers = computed(() => {
    const users = this.users();
    const sevenDaysAgo = new Date();
    sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 7);

    return users
      .filter((user) => new Date(user.lastActivity) >= sevenDaysAgo)
      .sort(
        (a, b) =>
          new Date(b.lastActivity).getTime() -
          new Date(a.lastActivity).getTime()
      );
  });

  // 🔄 Methods to update base signals
  updateSearchTerm(term: string): void {
    this.searchTerm.set(term);
  }

  updateFilterStatus(status: "all" | "active" | "inactive"): void {
    this.filterStatus.set(status);
  }

  updateSort(sortBy: "name" | "email" | "createdAt"): void {
    // Toggle direction if same column, otherwise default to asc
    if (this.sortBy() === sortBy) {
      this.sortDirection.update((dir) => (dir === "asc" ? "desc" : "asc"));
    } else {
      this.sortBy.set(sortBy);
      this.sortDirection.set("asc");
    }
  }

  selectUser(userId: string | null): void {
    this.selectedUserId.set(userId);
  }
}
```

### **⚡ Effects (Side Effects)**

```typescript
// src/app/services/user-effects.service.ts
import { Injectable, signal, effect, inject } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class UserEffectsService {
  private http = inject(HttpClient);
  private router = inject(Router);
  private notificationService = inject(NotificationService);

  // 📡 Signals
  users = signal<User[]>([]);
  selectedUserId = signal<string | null>(null);
  searchTerm = signal<string>("");
  autoSaveEnabled = signal<boolean>(true);

  constructor() {
    this.initializeEffects();
  }

  private initializeEffects(): void {
    // 💾 Auto-save search term to localStorage
    effect(() => {
      const searchTerm = this.searchTerm();
      if (searchTerm) {
        localStorage.setItem("userSearchTerm", searchTerm);
        console.log("💾 Search term saved:", searchTerm);
      } else {
        localStorage.removeItem("userSearchTerm");
      }
    });

    // 🔄 Sync selected user with URL
    effect(() => {
      const userId = this.selectedUserId();
      if (userId) {
        this.router.navigate(["/users", userId]);
        console.log("🔄 Navigated to user:", userId);
      }
    });

    // 📊 Analytics tracking
    effect(() => {
      const users = this.users();
      if (users.length > 0) {
        // Track user list view
        this.trackEvent("users_loaded", { count: users.length });
      }
    });

    // 🔔 Show notifications for user changes
    effect(() => {
      const users = this.users();
      const previous = this.previousUserCount();

      if (previous !== null && users.length !== previous) {
        const change = users.length - previous;
        if (change > 0) {
          this.notificationService.showSuccess(`${change} user(s) added`);
        } else {
          this.notificationService.showInfo(
            `${Math.abs(change)} user(s) removed`
          );
        }
      }

      this.previousUserCount.set(users.length);
    });

    // 💽 Auto-save user changes
    effect(() => {
      if (!this.autoSaveEnabled()) return;

      const users = this.users();
      if (users.length > 0) {
        // Debounce auto-save
        setTimeout(() => {
          this.autoSaveUsers(users);
        }, 1000);
      }
    });

    // 🎯 Focus management
    effect(() => {
      const selectedId = this.selectedUserId();
      if (selectedId) {
        // Focus on selected user element
        setTimeout(() => {
          const element = document.getElementById(`user-${selectedId}`);
          element?.scrollIntoView({ behavior: "smooth" });
        }, 100);
      }
    });
  }

  // 📊 Previous state tracking
  private previousUserCount = signal<number | null>(null);

  // 💽 Auto-save implementation
  private autoSaveUsers(users: User[]): void {
    const changes = users.filter((user) => user.isDirty);

    if (changes.length > 0) {
      console.log("💽 Auto-saving user changes:", changes.length);

      // Save to backend
      this.http.put("/api/users/bulk", changes).subscribe({
        next: () => {
          this.markUsersClean(changes.map((u) => u.id));
          console.log("✅ Auto-save completed");
        },
        error: (error) => {
          console.error("❌ Auto-save failed:", error);
          this.notificationService.showError("Failed to auto-save changes");
        },
      });
    }
  }

  private markUsersClean(userIds: string[]): void {
    this.users.update((users) =>
      users.map((user) =>
        userIds.includes(user.id) ? { ...user, isDirty: false } : user
      )
    );
  }

  // 📊 Analytics helper
  private trackEvent(eventName: string, data: any): void {
    // Integrate with your analytics service
    console.log("📊 Analytics:", eventName, data);
  }

  // 🔄 Public methods
  addUser(user: User): void {
    this.users.update((users) => [...users, { ...user, isDirty: true }]);
  }

  updateUser(id: string, updates: Partial<User>): void {
    this.users.update((users) =>
      users.map((user) =>
        user.id === id ? { ...user, ...updates, isDirty: true } : user
      )
    );
  }

  loadInitialData(): void {
    // Restore search term
    const savedSearchTerm = localStorage.getItem("userSearchTerm");
    if (savedSearchTerm) {
      this.searchTerm.set(savedSearchTerm);
    }
  }
}
```

---

## 🎭 **Advanced Signal Patterns**

### **🏪 Signal Store Pattern**

```typescript
// src/app/stores/user.store.ts
import { Injectable, signal, computed, effect } from "@angular/core";

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
  selectedUserId: string | null;
  filters: {
    search: string;
    status: "all" | "active" | "inactive";
    sortBy: "name" | "email" | "createdAt";
    sortDirection: "asc" | "desc";
  };
}

@Injectable({
  providedIn: "root",
})
export class UserStore {
  // 📦 Single state signal
  private state = signal<UserState>({
    users: [],
    loading: false,
    error: null,
    selectedUserId: null,
    filters: {
      search: "",
      status: "all",
      sortBy: "name",
      sortDirection: "asc",
    },
  });

  // 📖 Read-only selectors
  users = computed(() => this.state().users);
  loading = computed(() => this.state().loading);
  error = computed(() => this.state().error);
  selectedUserId = computed(() => this.state().selectedUserId);
  filters = computed(() => this.state().filters);

  // 🔄 Derived computed values
  filteredAndSortedUsers = computed(() => {
    const { users, filters } = this.state();

    let result = [...users];

    // Apply search filter
    if (filters.search) {
      const searchTerm = filters.search.toLowerCase();
      result = result.filter(
        (user) =>
          user.name.toLowerCase().includes(searchTerm) ||
          user.email.toLowerCase().includes(searchTerm)
      );
    }

    // Apply status filter
    if (filters.status !== "all") {
      result = result.filter((user) => user.status === filters.status);
    }

    // Apply sorting
    result.sort((a, b) => {
      let aValue = a[filters.sortBy];
      let bValue = b[filters.sortBy];

      if (filters.sortBy === "createdAt") {
        aValue = new Date(aValue).getTime();
        bValue = new Date(bValue).getTime();
      } else if (typeof aValue === "string") {
        aValue = aValue.toLowerCase();
        bValue = bValue.toLowerCase();
      }

      const comparison = aValue < bValue ? -1 : aValue > bValue ? 1 : 0;
      return filters.sortDirection === "asc" ? comparison : -comparison;
    });

    return result;
  });

  selectedUser = computed(() => {
    const { users, selectedUserId } = this.state();
    return selectedUserId
      ? users.find((u) => u.id === selectedUserId) || null
      : null;
  });

  stats = computed(() => {
    const users = this.filteredAndSortedUsers();
    return {
      total: users.length,
      active: users.filter((u) => u.status === "active").length,
      inactive: users.filter((u) => u.status === "inactive").length,
    };
  });

  // 🔄 Actions (state updates)

  setLoading(loading: boolean): void {
    this.state.update((state) => ({ ...state, loading }));
  }

  setError(error: string | null): void {
    this.state.update((state) => ({ ...state, error }));
  }

  setUsers(users: User[]): void {
    this.state.update((state) => ({
      ...state,
      users,
      loading: false,
      error: null,
    }));
  }

  addUser(user: User): void {
    this.state.update((state) => ({
      ...state,
      users: [...state.users, user],
    }));
  }

  updateUser(userId: string, updates: Partial<User>): void {
    this.state.update((state) => ({
      ...state,
      users: state.users.map((user) =>
        user.id === userId ? { ...user, ...updates } : user
      ),
    }));
  }

  removeUser(userId: string): void {
    this.state.update((state) => ({
      ...state,
      users: state.users.filter((u) => u.id !== userId),
      selectedUserId:
        state.selectedUserId === userId ? null : state.selectedUserId,
    }));
  }

  selectUser(userId: string | null): void {
    this.state.update((state) => ({ ...state, selectedUserId: userId }));
  }

  updateSearch(search: string): void {
    this.state.update((state) => ({
      ...state,
      filters: { ...state.filters, search },
    }));
  }

  updateStatusFilter(status: "all" | "active" | "inactive"): void {
    this.state.update((state) => ({
      ...state,
      filters: { ...state.filters, status },
    }));
  }

  updateSort(sortBy: "name" | "email" | "createdAt"): void {
    this.state.update((state) => ({
      ...state,
      filters: {
        ...state.filters,
        sortBy,
        sortDirection:
          state.filters.sortBy === sortBy &&
          state.filters.sortDirection === "asc"
            ? "desc"
            : "asc",
      },
    }));
  }

  // 🎯 Bulk operations
  bulkUpdateUsers(userIds: string[], updates: Partial<User>): void {
    this.state.update((state) => ({
      ...state,
      users: state.users.map((user) =>
        userIds.includes(user.id) ? { ...user, ...updates } : user
      ),
    }));
  }

  clearFilters(): void {
    this.state.update((state) => ({
      ...state,
      filters: {
        search: "",
        status: "all",
        sortBy: "name",
        sortDirection: "asc",
      },
    }));
  }

  // 💾 Persistence
  saveToLocalStorage(): void {
    const currentState = this.state();
    localStorage.setItem(
      "userStore",
      JSON.stringify({
        users: currentState.users,
        filters: currentState.filters,
        selectedUserId: currentState.selectedUserId,
      })
    );
  }

  loadFromLocalStorage(): void {
    const saved = localStorage.getItem("userStore");
    if (saved) {
      const data = JSON.parse(saved);
      this.state.update((state) => ({
        ...state,
        users: data.users || [],
        filters: { ...state.filters, ...data.filters },
        selectedUserId: data.selectedUserId || null,
      }));
    }
  }
}
```

### **🔄 Signal with HTTP Integration**

```typescript
// src/app/services/user-http-signals.service.ts
import { Injectable, signal, computed, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { catchError, finalize, tap } from "rxjs/operators";
import { of, EMPTY } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class UserHttpSignalsService {
  private http = inject(HttpClient);

  // 📡 Core state signals
  users = signal<User[]>([]);
  loading = signal<boolean>(false);
  error = signal<string | null>(null);

  // 📊 Request tracking
  private requestCount = signal<number>(0);
  private lastRequestTime = signal<Date | null>(null);

  // 🧮 Computed values
  isFirstLoad = computed(
    () => this.users().length === 0 && !this.loading() && !this.error()
  );

  hasData = computed(() => this.users().length > 0);

  requestStats = computed(() => ({
    totalRequests: this.requestCount(),
    lastRequest: this.lastRequestTime(),
    isLoading: this.loading(),
  }));

  // 🔄 HTTP Methods with Signal Integration

  loadUsers(): void {
    this.setLoading(true);
    this.setError(null);

    this.http
      .get<User[]>("/api/users")
      .pipe(
        tap((users) => {
          console.log("👥 Loaded users:", users.length);
          this.users.set(users);
        }),
        catchError((error) => {
          console.error("❌ Failed to load users:", error);
          this.setError("Failed to load users. Please try again.");
          return of([]); // Return empty array on error
        }),
        finalize(() => {
          this.setLoading(false);
          this.incrementRequestCount();
        })
      )
      .subscribe();
  }

  createUser(userData: Partial<User>): void {
    this.setLoading(true);
    this.setError(null);

    this.http
      .post<User>("/api/users", userData)
      .pipe(
        tap((newUser) => {
          console.log("➕ User created:", newUser.id);
          this.users.update((users) => [...users, newUser]);
        }),
        catchError((error) => {
          console.error("❌ Failed to create user:", error);
          this.setError("Failed to create user. Please try again.");
          return EMPTY;
        }),
        finalize(() => {
          this.setLoading(false);
          this.incrementRequestCount();
        })
      )
      .subscribe();
  }

  updateUser(userId: string, updates: Partial<User>): void {
    this.setLoading(true);
    this.setError(null);

    this.http
      .put<User>(`/api/users/${userId}`, updates)
      .pipe(
        tap((updatedUser) => {
          console.log("🔄 User updated:", userId);
          this.users.update((users) =>
            users.map((user) => (user.id === userId ? updatedUser : user))
          );
        }),
        catchError((error) => {
          console.error("❌ Failed to update user:", error);
          this.setError("Failed to update user. Please try again.");
          return EMPTY;
        }),
        finalize(() => {
          this.setLoading(false);
          this.incrementRequestCount();
        })
      )
      .subscribe();
  }

  deleteUser(userId: string): void {
    this.setLoading(true);
    this.setError(null);

    this.http
      .delete(`/api/users/${userId}`)
      .pipe(
        tap(() => {
          console.log("🗑️ User deleted:", userId);
          this.users.update((users) =>
            users.filter((user) => user.id !== userId)
          );
        }),
        catchError((error) => {
          console.error("❌ Failed to delete user:", error);
          this.setError("Failed to delete user. Please try again.");
          return EMPTY;
        }),
        finalize(() => {
          this.setLoading(false);
          this.incrementRequestCount();
        })
      )
      .subscribe();
  }

  refreshUsers(): void {
    console.log("🔄 Refreshing users...");
    this.loadUsers();
  }

  // 🔄 Private helpers
  private setLoading(loading: boolean): void {
    this.loading.set(loading);
  }

  private setError(error: string | null): void {
    this.error.set(error);
  }

  private incrementRequestCount(): void {
    this.requestCount.update((count) => count + 1);
    this.lastRequestTime.set(new Date());
  }

  // 🧹 Utility methods
  clearError(): void {
    this.setError(null);
  }

  reset(): void {
    this.users.set([]);
    this.loading.set(false);
    this.error.set(null);
    this.requestCount.set(0);
    this.lastRequestTime.set(null);
  }
}
```

---

## 🎭 **Component Integration Examples**

### **📋 Signal-Based User Management Component**

```typescript
// src/app/components/user-management/user-management.component.ts
import { Component, inject, computed } from "@angular/core";
import { FormsModule } from "@angular/forms";
import { CommonModule } from "@angular/common";

@Component({
  selector: "app-user-management",
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="user-management">
      <header class="management-header">
        <h2>👥 User Management</h2>

        <!-- Stats Display -->
        <div class="stats" *ngIf="userStats() as stats">
          <div class="stat">
            <span class="label">Total:</span>
            <span class="value">{{ stats.total }}</span>
          </div>
          <div class="stat">
            <span class="label">Active:</span>
            <span class="value">{{ stats.active }}</span>
          </div>
          <div class="stat">
            <span class="label">Inactive:</span>
            <span class="value">{{ stats.inactive }}</span>
          </div>
        </div>
      </header>

      <!-- Controls -->
      <div class="controls">
        <!-- Search -->
        <div class="search-box">
          <input
            type="text"
            placeholder="🔍 Search users..."
            [value]="searchTerm()"
            (input)="updateSearch($event)"
          />
        </div>

        <!-- Status Filter -->
        <select [value]="filterStatus()" (change)="updateStatusFilter($event)">
          <option value="all">All Status</option>
          <option value="active">Active Only</option>
          <option value="inactive">Inactive Only</option>
        </select>

        <!-- Actions -->
        <button
          (click)="loadUsers()"
          [disabled]="loading()"
          class="btn btn-primary"
        >
          {{ loading() ? "⏳ Loading..." : "🔄 Refresh" }}
        </button>

        <button (click)="addSampleUser()" class="btn btn-success">
          ➕ Add Sample User
        </button>
      </div>

      <!-- Error Display -->
      <div class="error" *ngIf="error()">
        ❌ {{ error() }}
        <button (click)="clearError()" class="btn-close">✖</button>
      </div>

      <!-- Loading State -->
      <div class="loading" *ngIf="loading()">⏳ Loading users...</div>

      <!-- Users List -->
      <div class="users-grid" *ngIf="!loading()">
        <div class="grid-header">
          <button
            class="sort-btn"
            (click)="updateSort('name')"
            [class.active]="sortBy() === 'name'"
          >
            Name
            <span *ngIf="sortBy() === 'name'">
              {{ sortDirection() === "asc" ? "↑" : "↓" }}
            </span>
          </button>

          <button
            class="sort-btn"
            (click)="updateSort('email')"
            [class.active]="sortBy() === 'email'"
          >
            Email
            <span *ngIf="sortBy() === 'email'">
              {{ sortDirection() === "asc" ? "↑" : "↓" }}
            </span>
          </button>

          <span>Status</span>
          <span>Actions</span>
        </div>

        <div
          class="user-row"
          *ngFor="let user of sortedUsers(); trackBy: trackByUserId"
          [class.selected]="selectedUserId() === user.id"
          [id]="'user-' + user.id"
        >
          <div class="user-name" (click)="selectUser(user.id)">
            {{ user.name }}
          </div>

          <div class="user-email">
            {{ user.email }}
          </div>

          <div class="user-status">
            <span
              class="status-badge"
              [class.status-active]="user.status === 'active'"
              [class.status-inactive]="user.status === 'inactive'"
            >
              {{ user.status }}
            </span>
          </div>

          <div class="user-actions">
            <button
              (click)="toggleUserStatus(user.id, user.status)"
              class="btn btn-sm"
            >
              {{ user.status === "active" ? "⏸️ Deactivate" : "▶️ Activate" }}
            </button>

            <button (click)="deleteUser(user.id)" class="btn btn-sm btn-danger">
              🗑️ Delete
            </button>
          </div>
        </div>
      </div>

      <!-- Selected User Details -->
      <div class="user-details" *ngIf="selectedUser() as user">
        <h3>👤 User Details</h3>
        <div class="details-content">
          <p><strong>Name:</strong> {{ user.name }}</p>
          <p><strong>Email:</strong> {{ user.email }}</p>
          <p><strong>Status:</strong> {{ user.status }}</p>
          <p><strong>Created:</strong> {{ user.createdAt | date }}</p>
          <p><strong>Last Activity:</strong> {{ user.lastActivity | date }}</p>
        </div>

        <button (click)="selectUser(null)" class="btn">✖ Close Details</button>
      </div>

      <!-- Empty State -->
      <div class="empty-state" *ngIf="!loading() && sortedUsers().length === 0">
        <h3>No users found</h3>
        <p>Try adjusting your search or filter criteria</p>
        <button (click)="clearFilters()" class="btn">Clear Filters</button>
      </div>
    </div>
  `,
  styles: [
    `
      .user-management {
        padding: 20px;
        max-width: 1200px;
        margin: 0 auto;
      }

      .management-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;
        padding: 15px;
        background: #f5f5f5;
        border-radius: 8px;
      }

      .stats {
        display: flex;
        gap: 15px;
      }

      .stat {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 8px 12px;
        background: white;
        border-radius: 4px;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
      }

      .label {
        font-size: 12px;
        color: #666;
      }
      .value {
        font-size: 18px;
        font-weight: bold;
        color: #333;
      }

      .controls {
        display: flex;
        gap: 15px;
        align-items: center;
        margin-bottom: 20px;
        padding: 15px;
        background: white;
        border-radius: 8px;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
      }

      .search-box {
        flex: 1;
      }

      .search-box input {
        width: 100%;
        padding: 8px 12px;
        border: 1px solid #ddd;
        border-radius: 4px;
      }

      .users-grid {
        background: white;
        border-radius: 8px;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
        overflow: hidden;
      }

      .grid-header {
        display: grid;
        grid-template-columns: 1fr 1fr auto auto;
        gap: 15px;
        padding: 15px;
        background: #f8f9fa;
        font-weight: bold;
        border-bottom: 1px solid #eee;
      }

      .sort-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 5px;
        border-radius: 3px;
        transition: background-color 0.2s;
      }

      .sort-btn:hover {
        background: #e9ecef;
      }

      .sort-btn.active {
        background: #007bff;
        color: white;
      }

      .user-row {
        display: grid;
        grid-template-columns: 1fr 1fr auto auto;
        gap: 15px;
        padding: 15px;
        border-bottom: 1px solid #eee;
        transition: background-color 0.2s;
      }

      .user-row:hover {
        background: #f8f9fa;
      }

      .user-row.selected {
        background: #e3f2fd;
        border-left: 4px solid #2196f3;
      }

      .user-name {
        cursor: pointer;
        color: #007bff;
        font-weight: 500;
      }

      .user-name:hover {
        text-decoration: underline;
      }

      .status-badge {
        padding: 4px 8px;
        border-radius: 12px;
        font-size: 12px;
        font-weight: bold;
        text-transform: uppercase;
      }

      .status-active {
        background: #d4edda;
        color: #155724;
      }

      .status-inactive {
        background: #f8d7da;
        color: #721c24;
      }

      .user-actions {
        display: flex;
        gap: 8px;
      }

      .btn {
        padding: 6px 12px;
        border: 1px solid #ddd;
        background: white;
        border-radius: 4px;
        cursor: pointer;
        transition: all 0.2s;
      }

      .btn:hover {
        background: #f8f9fa;
      }

      .btn.btn-primary {
        background: #007bff;
        color: white;
        border-color: #007bff;
      }

      .btn.btn-success {
        background: #28a745;
        color: white;
        border-color: #28a745;
      }

      .btn.btn-danger {
        background: #dc3545;
        color: white;
        border-color: #dc3545;
      }

      .btn:disabled {
        opacity: 0.5;
        cursor: not-allowed;
      }

      .user-details {
        margin-top: 20px;
        padding: 20px;
        background: white;
        border-radius: 8px;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
      }

      .details-content {
        margin: 15px 0;
      }

      .error {
        background: #f8d7da;
        color: #721c24;
        padding: 15px;
        border-radius: 4px;
        margin-bottom: 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
      }

      .btn-close {
        background: none;
        border: none;
        color: #721c24;
        cursor: pointer;
        font-size: 16px;
      }

      .loading {
        text-align: center;
        padding: 40px;
        color: #666;
        font-size: 18px;
      }

      .empty-state {
        text-align: center;
        padding: 40px;
        color: #666;
      }

      .empty-state h3 {
        margin-bottom: 10px;
      }

      .empty-state p {
        margin-bottom: 20px;
      }
    `,
  ],
})
export class UserManagementComponent {
  // 💉 Inject services
  private userService = inject(UserHttpSignalsService);
  private userComputed = inject(UserComputedService);
  private userStore = inject(UserStore);

  // 📡 Expose signals for template
  users = this.userService.users;
  loading = this.userService.loading;
  error = this.userService.error;

  // Computed filters
  searchTerm = this.userComputed.searchTerm;
  filterStatus = this.userComputed.filterStatus;
  sortBy = this.userComputed.sortBy;
  sortDirection = this.userComputed.sortDirection;

  // Computed results
  sortedUsers = this.userComputed.sortedUsers;
  userStats = this.userComputed.userStats;
  selectedUserId = this.userComputed.selectedUserId;
  selectedUser = this.userComputed.selectedUser;

  constructor() {
    // Load initial data
    this.loadUsers();
  }

  // 🔄 Event handlers

  loadUsers(): void {
    this.userService.loadUsers();
  }

  updateSearch(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.userComputed.updateSearchTerm(target.value);
  }

  updateStatusFilter(event: Event): void {
    const target = event.target as HTMLSelectElement;
    const status = target.value as "all" | "active" | "inactive";
    this.userComputed.updateFilterStatus(status);
  }

  updateSort(sortBy: "name" | "email" | "createdAt"): void {
    this.userComputed.updateSort(sortBy);
  }

  selectUser(userId: string | null): void {
    this.userComputed.selectUser(userId);
  }

  toggleUserStatus(userId: string, currentStatus: string): void {
    const newStatus = currentStatus === "active" ? "inactive" : "active";
    this.userService.updateUser(userId, { status: newStatus });
  }

  deleteUser(userId: string): void {
    if (confirm("Are you sure you want to delete this user?")) {
      this.userService.deleteUser(userId);
    }
  }

  addSampleUser(): void {
    const sampleUser = {
      name: `User ${Date.now()}`,
      email: `user${Date.now()}@example.com`,
      status: "active" as const,
      createdAt: new Date().toISOString(),
      lastActivity: new Date().toISOString(),
    };

    this.userService.createUser(sampleUser);
  }

  clearError(): void {
    this.userService.clearError();
  }

  clearFilters(): void {
    this.userComputed.updateSearchTerm("");
    this.userComputed.updateFilterStatus("all");
  }

  // 🎯 Track by function for performance
  trackByUserId(index: number, user: User): string {
    return user.id;
  }
}
```

---

## 🎉 **Summary: Signals Mastery**

### **✅ What We've Covered:**

⚡ **Signal Fundamentals** - Basic reactive primitives  
🧮 **Computed Signals** - Automatic derived values  
⚡ **Effects** - Side effects and reactions  
🏪 **Signal Store** - Centralized state management  
🔄 **HTTP Integration** - Async operations with signals  
🎭 **Component Integration** - Real-world usage patterns

### **🎯 Best Practices:**

- ✅ **Use Computed for Derived State** - Automatic updates
- ✅ **Keep Effects Pure** - Minimal side effects
- ✅ **Centralize State** - Store pattern for complex state
- ✅ **Performance** - Signals only update when changed
- ✅ **Type Safety** - Full TypeScript support
- ✅ **Debugging** - Better DevTools integration

**You're now ready to build reactive Angular apps with Signals!** 🚀
