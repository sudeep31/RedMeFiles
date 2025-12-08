# 🌐 Global State vs Local Component State in Angular

## 🎯 **Question Overview**

_"How do you handle global state & Local component state?"_

## 🔍 **Understanding State Types**

Understanding the distinction between **global** and **local** state is crucial for building maintainable Angular applications. Each serves different purposes and requires different management strategies.

### **🏠 Local Component State**

State that belongs to a **single component** and its children. It's temporary, component-specific, and doesn't need to be shared across unrelated parts of the application.

### **🌐 Global State**

State that needs to be **shared across multiple components**, persists across navigation, or affects the entire application behavior.

## 🏠 **Local Component State Management**

### **1. 🎯 Simple Component Properties**

For basic local state, use simple component properties:

```typescript
@Component({
  selector: "app-todo-form",
  template: `
    <form (ngSubmit)="addTodo()">
      <input
        [(ngModel)]="todoText"
        placeholder="Enter todo..."
        [disabled]="isSubmitting"
      />

      <button type="submit" [disabled]="!todoText.trim() || isSubmitting">
        {{ isSubmitting ? "Adding..." : "Add Todo" }}
      </button>
    </form>

    <div class="validation" *ngIf="showValidation">
      Please enter a valid todo item
    </div>
  `,
  standalone: true,
  imports: [CommonModule, FormsModule],
})
export class TodoFormComponent {
  // Local state properties
  todoText = "";
  isSubmitting = false;
  showValidation = false;

  @Output() todoAdded = new EventEmitter<string>();

  addTodo() {
    if (!this.todoText.trim()) {
      this.showValidation = true;
      setTimeout(() => (this.showValidation = false), 3000);
      return;
    }

    this.isSubmitting = true;

    // Simulate API call
    setTimeout(() => {
      this.todoAdded.emit(this.todoText);
      this.resetForm();
    }, 1000);
  }

  private resetForm() {
    this.todoText = "";
    this.isSubmitting = false;
    this.showValidation = false;
  }
}
```

### **2. 🚀 Local State with Signals (Angular 16+)**

For more reactive local state management:

```typescript
@Component({
  selector: "app-counter",
  template: `
    <div class="counter">
      <h3>Counter: {{ count() }}</h3>
      <p>Status: {{ status() }}</p>
      <p>History: {{ history().join(", ") }}</p>

      <div class="controls">
        <button (click)="increment()" [disabled]="isMax()">+</button>
        <button (click)="decrement()" [disabled]="isMin()">-</button>
        <button (click)="reset()">Reset</button>
      </div>

      @if (showMessage()) {
      <div class="message">{{ message() }}</div>
      }
    </div>
  `,
  standalone: true,
  imports: [CommonModule],
})
export class CounterComponent {
  // Local signals
  count = signal(0);
  history = signal<number[]>([]);
  showMessage = signal(false);

  // Computed signals
  status = computed(() => {
    const value = this.count();
    if (value > 0) return "Positive";
    if (value < 0) return "Negative";
    return "Zero";
  });

  isMax = computed(() => this.count() >= 10);
  isMin = computed(() => this.count() <= -10);

  message = computed(() => {
    const value = this.count();
    if (value === 10) return "🎉 Maximum reached!";
    if (value === -10) return "⚠️ Minimum reached!";
    if (value === 0) return "🔄 Reset to zero";
    return "";
  });

  constructor() {
    // Effect for showing/hiding messages
    effect(() => {
      const msg = this.message();
      if (msg) {
        this.showMessage.set(true);
        setTimeout(() => this.showMessage.set(false), 2000);
      }
    });

    // Effect for history tracking
    effect(() => {
      const currentCount = this.count();
      this.history.update((hist) => [...hist, currentCount].slice(-5));
    });
  }

  increment() {
    if (!this.isMax()) {
      this.count.update((c) => c + 1);
    }
  }

  decrement() {
    if (!this.isMin()) {
      this.count.update((c) => c - 1);
    }
  }

  reset() {
    this.count.set(0);
  }
}
```

### **3. 🔄 Local State with RxJS**

For complex local state with async operations:

```typescript
@Component({
  selector: "app-search",
  template: `
    <div class="search-container">
      <input
        #searchInput
        placeholder="Search users..."
        (input)="onSearchInput($event)"
      />

      @if (loading()) {
      <div class="loading">Searching...</div>
      } @if (error(); as errorMsg) {
      <div class="error">{{ errorMsg }}</div>
      } @if (results().length > 0) {
      <div class="results">
        @for (user of results(); track user.id) {
        <div class="user-item">
          <img [src]="user.avatar" [alt]="user.name" />
          <span>{{ user.name }}</span>
        </div>
        }
      </div>
      } @else if (!loading() && searchTerm() && !error()) {
      <div class="no-results">No users found</div>
      }
    </div>
  `,
  standalone: true,
  imports: [CommonModule],
})
export class SearchComponent implements OnInit, OnDestroy {
  // Local state signals
  searchTerm = signal("");
  results = signal<User[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);

  private searchSubject = new Subject<string>();
  private destroy$ = new Subject<void>();

  constructor(private userService: UserService) {}

  ngOnInit() {
    // Setup search stream
    this.searchSubject
      .pipe(
        debounceTime(300),
        distinctUntilChanged(),
        switchMap((term) => {
          if (!term.trim()) {
            return of([]);
          }

          this.loading.set(true);
          this.error.set(null);

          return this.userService.searchUsers(term).pipe(
            catchError((error) => {
              this.error.set("Search failed: " + error.message);
              return of([]);
            }),
            finalize(() => this.loading.set(false))
          );
        }),
        takeUntil(this.destroy$)
      )
      .subscribe((results) => {
        this.results.set(results);
      });
  }

  onSearchInput(event: Event) {
    const term = (event.target as HTMLInputElement).value;
    this.searchTerm.set(term);
    this.searchSubject.next(term);
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## 🌐 **Global State Management**

### **1. 🔄 Service-Based Global State**

```typescript
// Global User State Service
@Injectable({
  providedIn: "root",
})
export class GlobalUserService {
  private userSubject = new BehaviorSubject<User | null>(null);
  private preferencesSubject = new BehaviorSubject<UserPreferences>(
    defaultPreferences
  );
  private notificationsSubject = new BehaviorSubject<Notification[]>([]);

  // Public observables
  user$ = this.userSubject.asObservable();
  preferences$ = this.preferencesSubject.asObservable();
  notifications$ = this.notificationsSubject.asObservable();

  // Computed observables
  isAuthenticated$ = this.user$.pipe(map((user) => !!user));
  unreadNotifications$ = this.notifications$.pipe(
    map((notifications) => notifications.filter((n) => !n.read).length)
  );

  constructor(private http: HttpClient, private storage: StorageService) {
    this.initializeFromStorage();
    this.setupAutoSync();
  }

  // User management
  async login(credentials: LoginCredentials): Promise<void> {
    try {
      const response = await firstValueFrom(
        this.http.post<LoginResponse>("/api/login", credentials)
      );

      this.userSubject.next(response.user);
      this.storage.setToken(response.token);
      this.loadUserPreferences();
    } catch (error) {
      throw new Error("Login failed");
    }
  }

  logout(): void {
    this.userSubject.next(null);
    this.preferencesSubject.next(defaultPreferences);
    this.notificationsSubject.next([]);
    this.storage.clearToken();
  }

  updateUser(updates: Partial<User>): void {
    const currentUser = this.userSubject.value;
    if (currentUser) {
      const updatedUser = { ...currentUser, ...updates };
      this.userSubject.next(updatedUser);
      this.syncUserToServer(updatedUser);
    }
  }

  // Preferences management
  updatePreferences(updates: Partial<UserPreferences>): void {
    const current = this.preferencesSubject.value;
    const updated = { ...current, ...updates };
    this.preferencesSubject.next(updated);
    this.syncPreferencesToServer(updated);
  }

  // Notifications management
  addNotification(notification: Omit<Notification, "id" | "timestamp">): void {
    const newNotification: Notification = {
      ...notification,
      id: Date.now().toString(),
      timestamp: new Date(),
      read: false,
    };

    this.notificationsSubject.next([
      newNotification,
      ...this.notificationsSubject.value,
    ]);
  }

  markNotificationRead(id: string): void {
    const notifications = this.notificationsSubject.value;
    const updated = notifications.map((n) =>
      n.id === id ? { ...n, read: true } : n
    );
    this.notificationsSubject.next(updated);
  }

  clearAllNotifications(): void {
    this.notificationsSubject.next([]);
  }

  private initializeFromStorage(): void {
    const token = this.storage.getToken();
    if (token) {
      this.validateTokenAndLoadUser(token);
    }
  }

  private setupAutoSync(): void {
    // Auto-sync user data every 5 minutes
    timer(0, 5 * 60 * 1000)
      .pipe(
        filter(() => !!this.userSubject.value),
        switchMap(() => this.refreshUserData()),
        catchError((error) => {
          console.warn("Auto-sync failed:", error);
          return EMPTY;
        })
      )
      .subscribe();
  }

  private async validateTokenAndLoadUser(token: string): Promise<void> {
    try {
      const user = await firstValueFrom(
        this.http.get<User>("/api/user/profile")
      );
      this.userSubject.next(user);
      this.loadUserPreferences();
    } catch (error) {
      this.storage.clearToken();
    }
  }

  private refreshUserData(): Observable<User> {
    return this.http
      .get<User>("/api/user/profile")
      .pipe(tap((user) => this.userSubject.next(user)));
  }

  private async loadUserPreferences(): Promise<void> {
    try {
      const preferences = await firstValueFrom(
        this.http.get<UserPreferences>("/api/user/preferences")
      );
      this.preferencesSubject.next(preferences);
    } catch (error) {
      console.warn("Failed to load preferences:", error);
    }
  }

  private syncUserToServer(user: User): void {
    this.http.put("/api/user/profile", user).subscribe({
      error: (error) => console.error("Failed to sync user:", error),
    });
  }

  private syncPreferencesToServer(preferences: UserPreferences): void {
    this.http.put("/api/user/preferences", preferences).subscribe({
      error: (error) => console.error("Failed to sync preferences:", error),
    });
  }
}
```

### **2. 🚀 Global State with Signals**

```typescript
// Global App State with Signals
@Injectable({
  providedIn: "root",
})
export class GlobalAppStateService {
  // Writable signals for core state
  private _user = signal<User | null>(null);
  private _theme = signal<Theme>("light");
  private _language = signal<Language>("en");
  private _notifications = signal<Notification[]>([]);
  private _connectionStatus = signal<"online" | "offline">("online");

  // Read-only signals
  readonly user = this._user.asReadonly();
  readonly theme = this._theme.asReadonly();
  readonly language = this._language.asReadonly();
  readonly notifications = this._notifications.asReadonly();
  readonly connectionStatus = this._connectionStatus.asReadonly();

  // Computed signals
  readonly isAuthenticated = computed(() => !!this._user());
  readonly unreadCount = computed(
    () => this._notifications().filter((n) => !n.read).length
  );
  readonly userDisplayName = computed(() => {
    const user = this._user();
    return user ? `${user.firstName} ${user.lastName}` : "Guest";
  });
  readonly isDarkMode = computed(() => this._theme() === "dark");

  constructor(private http: HttpClient, private storage: StorageService) {
    this.initializeState();
    this.setupEffects();
    this.setupConnectionMonitoring();
  }

  // User actions
  setUser(user: User | null): void {
    this._user.set(user);
  }

  updateUser(updates: Partial<User>): void {
    this._user.update((user) => (user ? { ...user, ...updates } : null));
  }

  // Theme actions
  setTheme(theme: Theme): void {
    this._theme.set(theme);
  }

  toggleTheme(): void {
    this._theme.update((current) => (current === "light" ? "dark" : "light"));
  }

  // Language actions
  setLanguage(language: Language): void {
    this._language.set(language);
  }

  // Notification actions
  addNotification(notification: Omit<Notification, "id" | "timestamp">): void {
    const newNotification: Notification = {
      ...notification,
      id: crypto.randomUUID(),
      timestamp: new Date(),
      read: false,
    };

    this._notifications.update((notifications) => [
      newNotification,
      ...notifications,
    ]);
  }

  markNotificationRead(id: string): void {
    this._notifications.update((notifications) =>
      notifications.map((n) => (n.id === id ? { ...n, read: true } : n))
    );
  }

  removeNotification(id: string): void {
    this._notifications.update((notifications) =>
      notifications.filter((n) => n.id !== id)
    );
  }

  clearAllNotifications(): void {
    this._notifications.set([]);
  }

  private initializeState(): void {
    // Load saved state from storage
    const savedUser = this.storage.getItem("user");
    if (savedUser) {
      this._user.set(JSON.parse(savedUser));
    }

    const savedTheme = this.storage.getItem("theme") as Theme;
    if (savedTheme) {
      this._theme.set(savedTheme);
    }

    const savedLanguage = this.storage.getItem("language") as Language;
    if (savedLanguage) {
      this._language.set(savedLanguage);
    }
  }

  private setupEffects(): void {
    // Persist user changes
    effect(() => {
      const user = this._user();
      if (user) {
        this.storage.setItem("user", JSON.stringify(user));
      } else {
        this.storage.removeItem("user");
      }
    });

    // Persist theme changes
    effect(() => {
      const theme = this._theme();
      this.storage.setItem("theme", theme);
      document.documentElement.setAttribute("data-theme", theme);
    });

    // Persist language changes
    effect(() => {
      const language = this._language();
      this.storage.setItem("language", language);
      document.documentElement.setAttribute("lang", language);
    });

    // Auto-remove old notifications
    effect(() => {
      const notifications = this._notifications();
      const oneWeekAgo = new Date();
      oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);

      const filtered = notifications.filter((n) => n.timestamp > oneWeekAgo);

      if (filtered.length !== notifications.length) {
        this._notifications.set(filtered);
      }
    });
  }

  private setupConnectionMonitoring(): void {
    const updateConnectionStatus = () => {
      this._connectionStatus.set(navigator.onLine ? "online" : "offline");
    };

    window.addEventListener("online", updateConnectionStatus);
    window.addEventListener("offline", updateConnectionStatus);
    updateConnectionStatus(); // Initial check
  }
}
```

### **3. 🔗 Parent-Child State Communication**

```typescript
// Parent Component managing both global and local state
@Component({
  selector: "app-dashboard",
  template: `
    <div class="dashboard">
      <app-header
        [user]="globalUser()"
        [notifications]="globalNotifications()"
        (themeToggle)="toggleTheme()"
        (logout)="logout()"
      >
      </app-header>

      <app-sidebar
        [isCollapsed]="sidebarCollapsed()"
        (toggleSidebar)="toggleSidebar()"
      >
      </app-sidebar>

      <main class="main-content" [class.sidebar-collapsed]="sidebarCollapsed()">
        <app-todo-list
          [todos]="localTodos()"
          (addTodo)="addTodo($event)"
          (toggleTodo)="toggleTodo($event)"
          (deleteTodo)="deleteTodo($event)"
        >
        </app-todo-list>

        <app-stats
          [totalTodos]="totalTodos()"
          [completedTodos]="completedTodos()"
          [pendingTodos]="pendingTodos()"
        >
        </app-stats>
      </main>
    </div>
  `,
  standalone: true,
  imports: [
    HeaderComponent,
    SidebarComponent,
    TodoListComponent,
    StatsComponent,
  ],
})
export class DashboardComponent {
  // Global state (from service)
  globalUser = this.appState.user;
  globalNotifications = this.appState.notifications;

  // Local component state (specific to dashboard)
  sidebarCollapsed = signal(false);
  localTodos = signal<Todo[]>([]);

  // Computed local state
  totalTodos = computed(() => this.localTodos().length);
  completedTodos = computed(
    () => this.localTodos().filter((todo) => todo.completed).length
  );
  pendingTodos = computed(
    () => this.localTodos().filter((todo) => !todo.completed).length
  );

  constructor(
    private appState: GlobalAppStateService,
    private todoService: TodoService
  ) {
    this.loadTodos();
  }

  // Global state actions
  toggleTheme(): void {
    this.appState.toggleTheme();
  }

  logout(): void {
    this.appState.setUser(null);
    // Navigate to login page
  }

  // Local state actions
  toggleSidebar(): void {
    this.sidebarCollapsed.update((collapsed) => !collapsed);
  }

  async addTodo(text: string): Promise<void> {
    try {
      const newTodo = await firstValueFrom(
        this.todoService.createTodo({ text, completed: false })
      );
      this.localTodos.update((todos) => [...todos, newTodo]);

      this.appState.addNotification({
        type: "success",
        message: "Todo added successfully!",
      });
    } catch (error) {
      this.appState.addNotification({
        type: "error",
        message: "Failed to add todo",
      });
    }
  }

  async toggleTodo(id: string): Promise<void> {
    const todo = this.localTodos().find((t) => t.id === id);
    if (!todo) return;

    try {
      const updated = await firstValueFrom(
        this.todoService.updateTodo(id, { completed: !todo.completed })
      );

      this.localTodos.update((todos) =>
        todos.map((t) => (t.id === id ? updated : t))
      );
    } catch (error) {
      this.appState.addNotification({
        type: "error",
        message: "Failed to update todo",
      });
    }
  }

  async deleteTodo(id: string): Promise<void> {
    try {
      await firstValueFrom(this.todoService.deleteTodo(id));
      this.localTodos.update((todos) => todos.filter((t) => t.id !== id));

      this.appState.addNotification({
        type: "success",
        message: "Todo deleted successfully!",
      });
    } catch (error) {
      this.appState.addNotification({
        type: "error",
        message: "Failed to delete todo",
      });
    }
  }

  private async loadTodos(): Promise<void> {
    try {
      const todos = await firstValueFrom(this.todoService.getTodos());
      this.localTodos.set(todos);
    } catch (error) {
      this.appState.addNotification({
        type: "error",
        message: "Failed to load todos",
      });
    }
  }
}
```

## 🎯 **When to Use Each Approach**

### **🏠 Use Local Component State When:**

- ✅ Data is **component-specific** (form inputs, UI toggles)
- ✅ State doesn't need to **survive component destruction**
- ✅ **No sharing** required with other components
- ✅ **Temporary UI state** (loading, error messages)
- ✅ **Simple data** that doesn't need complex transformations

**Examples:**

- Form validation states
- Modal open/close state
- Local loading indicators
- Component-specific filters
- UI animations state

### **🌐 Use Global State When:**

- ✅ Data needs to be **shared across components**
- ✅ State should **persist across navigation**
- ✅ **User authentication** and permissions
- ✅ **Application-wide settings** (theme, language)
- ✅ **Cached API data** used by multiple components
- ✅ **Real-time updates** from servers

**Examples:**

- User authentication status
- Application theme/settings
- Shopping cart contents
- Notifications
- Real-time chat messages
- Global loading states

## 📊 **State Management Decision Matrix**

| State Type        | Local Component | Global Service | Signals    | NgRx/Akita  |
| ----------------- | --------------- | -------------- | ---------- | ----------- |
| **Form Inputs**   | ✅ Perfect      | ❌ Overkill    | ✅ Good    | ❌ Overkill |
| **User Auth**     | ❌ Wrong        | ✅ Perfect     | ✅ Perfect | ✅ Good     |
| **UI Toggle**     | ✅ Perfect      | ❌ Wrong       | ✅ Good    | ❌ Overkill |
| **Shopping Cart** | ❌ Wrong        | ✅ Good        | ✅ Perfect | ✅ Perfect  |
| **Modal State**   | ✅ Perfect      | ⚠️ Maybe       | ✅ Good    | ❌ Overkill |
| **App Settings**  | ❌ Wrong        | ✅ Good        | ✅ Perfect | ✅ Good     |

## 🚨 **Common Pitfalls & Best Practices**

### **❌ Common Mistakes**

1. **Over-globalizing state**

```typescript
// ❌ Bad - Making everything global
@Injectable()
export class GlobalEverythingService {
  formInput$ = new BehaviorSubject("");
  modalOpen$ = new BehaviorSubject(false);
  buttonClicks$ = new BehaviorSubject(0);
  // This should be local component state!
}
```

2. **Under-sharing necessary state**

```typescript
// ❌ Bad - Duplicating user state in every component
@Component({})
export class HeaderComponent {
  currentUser: User; // Should come from global state
}

@Component({})
export class SidebarComponent {
  currentUser: User; // Duplicate state!
}
```

### **✅ Best Practices**

1. **Clear separation of concerns**

```typescript
// ✅ Good - Clear state boundaries
@Injectable()
export class UserGlobalService {
  // Only global user-related state
  user$ = new BehaviorSubject<User | null>(null);
  permissions$ = new BehaviorSubject<Permission[]>([]);
}

@Component({})
export class TodoFormComponent {
  // Only local form state
  todoText = signal("");
  isSubmitting = signal(false);
}
```

2. **Proper state flow**

```typescript
// ✅ Good - Data flows down, events flow up
@Component({
  template: `
    <app-child
      [globalData]="globalUser()"
      [localData]="componentData()"
      (actionNeeded)="handleChildAction($event)"
    >
    </app-child>
  `,
})
export class ParentComponent {
  globalUser = this.userService.user;
  componentData = signal("local data");

  handleChildAction(action: any) {
    // Handle child events appropriately
    if (action.type === "global") {
      this.userService.updateUser(action.payload);
    } else {
      this.componentData.set(action.payload);
    }
  }
}
```

## 🎯 **Key Takeaways**

### **Design Principles:**

1. **🎯 Keep local what should be local** - Don't over-globalize
2. **🌐 Share what needs to be shared** - Don't duplicate global state
3. **📊 Data down, events up** - Maintain clear data flow
4. **🔄 Single source of truth** - Avoid state duplication
5. **⚡ Use signals for modern reactivity** - Better performance

### **State Management Strategy:**

1. **Start local** - Begin with component state
2. **Promote when necessary** - Move to global when sharing needed
3. **Use appropriate tools** - Match complexity to solution
4. **Test state changes** - Verify state transitions work correctly
5. **Document state flow** - Make state management clear to team

### **Performance Considerations:**

- Use **OnPush change detection** with observables/signals
- **Memoize expensive computations** with computed signals
- **Batch state updates** when possible
- **Clean up subscriptions** properly
- **Use trackBy functions** for lists

The key is finding the right balance between local and global state based on your application's specific needs! 🚀
